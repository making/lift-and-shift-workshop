## Step2. Extern the authentioncation functionality


As as OIDC provider, we use Cloud Foundry's [UAA](https://github.com/cloudfoundry/uaa).

### Deploy Cloud Foundry's UAA as a stand-alone application

Download a pre-packaged war file:

```bash
wget https://github.com/starkandwayne/uaa-war-releases/releases/download/v4.19.2/cloudfoundry-identity-uaa-4.19.2.war -O ROOT.war
```

Create a UAA's manifest file as a [bosh](https://bosh.io/docs/cli-int/) template file:

```yaml
cat <<EOF > uaa.yml
issuer:
  uri: https://((route))

encryption:
  encryption_keys:
  - label: uaa-encryption-key-1
    passphrase: ((uaa_encryption_key_1))
  active_key_label: uaa-encryption-key-1

scim:
  users:
  - admin|((admin_user_password))|admin||||uaa
  userids_enabled: true
  user:
    override: true

require_https: true

oauth:
  authorize:
    ssl: true
  clients:
    uaa_admin:
      override: true
      authorized-grant-types: client_credentials
      scope: ""
      authorities: clients.read,clients.write,clients.secret,uaa.admin,scim.read,scim.write,password.write
      secret: ((admin_client_secret))
  user:
    authorities:
    - openid
    - scim.me
    - password.write
    - uaa.user
    - uaa.offline_token

jwt:
  token:
    queryString:
      enabled: true
    revocable: true
    policy:
      accessTokenValiditySeconds: 43200
      refreshTokenValiditySeconds: 2592000
      global:
        accessTokenValiditySeconds: 43200
        refreshTokenValiditySeconds: 2592000
      activeKeyId: uaa-jwt-key-1
      keys:
        uaa-jwt-key-1:
          verification-key: ((uaa_jwt_signing_key.public_key))
          signingKey: ((uaa_jwt_signing_key.private_key))
    refresh:
      restrict_grant: false
      unique: false
      format: jwt

login:
  selfServiceLinksEnabled: true
  serviceProviderKey: ((uaa_service_provider_ssl.private_key))
  serviceProviderKeyPassword: "" # TODO: Remove this when UAA defaults this value
  serviceProviderCertificate: ((uaa_service_provider_ssl.certificate))

zones:
  internal:
    hostnames:
      - ((route))

variables:
- name: admin_user_password
  type: password

- name: admin_client_secret
  type: password

- name: uaa_jwt_signing_key
  type: rsa

- name: uaa_encryption_key_1
  type: password

- name: default_ca
  type: certificate
  options:
    is_ca: true
    common_name: ca

- name: uaa_service_provider_ssl
  type: certificate
  options:
    ca: default_ca
    common_name: ((route))
    alternative_names: [((route))]
EOF
```

Add bosh [ops-files](https://bosh.io/docs/cli-ops-files/) to customize `uaa.yml`

* Adding an OAuth2 client for a gateway (Zuul)

```bash
mkdir -p ops-files

cat <<EOF > ops-files/add-zuul.yml
- type: replace
  path: /oauth/clients/zuul?
  value:
    name: Zuul
    authorities: uaa.none
    authorized-grant-types: authorization_code,refresh_token,password
    override: true
    redirect-uri: http://localhost:8080/login
    scope: openid,role
    secret: ((zuul_client_secret))
    app-launch-url: http://localhost:8080
- type: replace
  path: /variables/name=zuul_client_secret?
  value:
    name: zuul_client_secret
    type: password
EOF
```

* Adding the smtp notification configuration:

```bash
cat <<EOF > ops-files/smtp.yml
- type: replace
  path: /smtp?
  value:
    host: ((smtp_host))
    port: ((smtp_port))
    user: ((smtp_user))
    password: ((smtp_password))
    starttls: true
EOF
```

Create a script to compose `uaa.yml` and embbed it in the war file:

```bash
cat <<EOF > embbed-manifest.sh 
#!/bin/bash
set -e

bosh int uaa.yml \
  -o ops-files/smtp.yml \
  -o ops-files/add-zuul.yml \
  -v route=tour-uaa.cfapps.io \
  -v smtp_host=smtp.gmail.com \
  -v smtp_port=587 \
  --vars-store=credentials.yml \
  > WEB-INF/classes/uaa.yml

jar -uvf ROOT.war .profile WEB-INF
EOF
chmod +x embbed-manifest.sh 
```

Create a pre-runtime hook script to retrieve credentials to access a database (MySQL):

```bash
cat <<'EOF' > .profile
CREDS=$(echo $VCAP_SERVICES | jq -r ".[] | map(select(.name == \"tour-uaa-db\"))[0].credentials")

export DATABASE_HOSTNAME=$(echo $CREDS | jq -r .hostname)
export DATABASE_PORT=$(echo $CREDS | jq -r .port)
export DATABASE_USERNAME=$(echo $CREDS | jq -r .username)
export DATABASE_PASSWORD=$(echo $CREDS | jq -r .password)
export DATABASE_NAME=$(echo $CREDS | jq -r .name)
export DATABASE_URL=jdbc:mysql://${DATABASE_HOSTNAME}:${DATABASE_PORT}/${DATABASE_NAME}
EOF
```

Run the script:

```bash
mkdir -p WEB-INF/classes
./embbed-manifest.sh 
```

Create a manifest file for deploying UAA to Cloud Foundry:

```bash
cat <<EOF > manifest.yml
applications:
- name: tour-uaa
  memory: 1g
  instances: 1
  path: ROOT.war
  health-check-type: http
  health-check-http-endpoint: /healthz
  services:
  - tour-uaa-db
  env:
    SPRING_PROFILES: mysql
    DATABASE_DRIVERCLASSNAME: org.mariadb.jdbc.Driver
    DATABASE_MAXACTIVE: 4
    DATABASE_MAXIDLE: 3
    DATABASE_MINIDLE: 1
EOF
```

CF Push!

```bash
cf create-service cleardb spark tour-uaa-db
cf push
```
