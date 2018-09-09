##  Step1. Make your app 5 factors enabled


Follow at least following 5 factors in [12 factor app](https://12factor.net) to make your app cloud ready:

* [III. Config](https://12factor.net/config)
* [VI. Processes](https://12factor.net/processes)
* [VII. Port binding](https://12factor.net/port-binding)
* [VIII. Concurrency](https://12factor.net/concurrency)
* [XI. Logs](https://12factor.net/logs)


Let's take [an example](https://ja.osdn.net/projects/terasoluna/downloads/66350/terasoluna-server4jweb-toursample_2.0.6.2.zip).

The app uses a legacy Java stack such as:

* Apache Struts 1.2.x
* iBatis 2.3.x
* Spring Framework 3.2.x
* Apache Ant
* Apache Tomcat 6.0
* PostgreSQL

```bash
wget https://ja.osdn.net/projects/terasoluna/downloads/66350/terasoluna-server4jweb-toursample_2.0.6.2.zip
unzip terasoluna-server4jweb-toursample_2.0.6.2.zip
cd terasoluna-server4jweb-toursample_2.0.6.2
unzip toursample-javaweb.zip
cd toursample-javaweb
```

### Install software

* [Apache Tomcat (latest will work)](https://tomcat.apache.org/download-90.cgi)
* [Apache Ant](https://ant.apache.org/bindownload.cgi)

For `brew` users:

```bash
brew install ant
brew install tomcat
```

### Modify `sources/log4j.properties`

Logging to a file does not meet [XI. Logs](https://12factor.net/logs).

Remove configurations about `org.apache.log4j.RollingFileAppender` in `sources/log4j.properties`:

```properties
cat <<EOF > sources/log4j.properties
# Keep
log4j.rootCategory=INFO, consoleLog, logfile
log4j.category.jp.terasoluna=DEBUG
log4j.category.org.springframework=INFO
log4j.category.org.apache.struts=INFO

log4j.appender.consoleLog=org.apache.log4j.ConsoleAppender
log4j.appender.consoleLog.Target = System.out
log4j.appender.consoleLog.layout = org.apache.log4j.PatternLayout
log4j.appender.consoleLog.layout.ConversionPattern=[%d{yyyy/MM/dd HH:mm:ss}][%p][%C{1}] %m%n

# Remove
# log4j.appender.logfile=...
EOF
```

### Modify `ant/build.properties`


```properties
cat <<EOF > ant/build.properties
# Keep
source.dir=./sources
web.inf.dir=./webapps/WEB-INF
lib.dir=./webapps/WEB-INF/lib
zip.dir=./terasoluna/src

## Change according to your tomcat env
webapsvr.home=/tmp/apache-tomcat-9.0.11
# webapsvr.home=/usr/local/Cellar/tomcat/9.0.11/libexec # for brew user
webapsvr.lib.dir=${webapsvr.home}/lib
deploy.dir=${webapsvr.home}/webapps

## Change
context.name=ROOT
EOF
```

In order to comply [VII. Port binding](https://12factor.net/port-binding), I strongly recommend to use `ROOT` as a context name so that users can visit your app via URL like `http://localhost:8080`.


> If you follow [II. Dependencies](https://12factor.net/dependencies), I'd recommend to use [Apache Maven](https://maven.apache.org/) or [Gradle](https://gradle.org/).

### Build the app


```bash
ant -f ant/build.xml
```

The output will look like:

```bash
Buildfile: /private/tmp/terasoluna-server4jweb-toursample_2.0.6.2/toursample-javaweb/ant/build.xml

clean:

compile:
    [javac] /private/tmp/terasoluna-server4jweb-toursample_2.0.6.2/toursample-javaweb/ant/build.xml:69: warning: 'includeantruntime' was not set, defaulting to build.sysclasspath=last; set to false for repeatable builds
    [javac] Compiling 97 source files to /private/tmp/terasoluna-server4jweb-toursample_2.0.6.2/toursample-javaweb/webapps/WEB-INF/classes
     [copy] Copying 10 files to /private/tmp/terasoluna-server4jweb-toursample_2.0.6.2/toursample-javaweb/webapps/WEB-INF/classes

native2ascii:
[native2ascii] Converting 16 files from /private/tmp/terasoluna-server4jweb-toursample_2.0.6.2/toursample-javaweb/sources to /private/tmp/terasoluna-server4jweb-toursample_2.0.6.2/toursample-javaweb/webapps/WEB-INF/classes

build:

deploy:
      [jar] Building jar: /private/tmp/terasoluna-server4jweb-toursample_2.0.6.2/toursample-javaweb/ROOT.war
     [copy] Copying 1 file to /tmp/apache-tomcat-9.0.11/webapps
   [delete] Deleting: /private/tmp/terasoluna-server4jweb-toursample_2.0.6.2/toursample-javaweb/ROOT.war

BUILD SUCCESSFUL
Total time: 2 seconds
```

### Download flyway for DB migration

```bash
wget https://repo1.maven.org/maven2/org/flywaydb/flyway-commandline/5.1.4/flyway-commandline-5.1.4-linux-x64.tar.gz
tar xzvf flyway-commandline-5.1.4-linux-x64.tar.gz
mv flyway-5.1.4/ flyway
```

### Convert the character encoding of SQL files to UTF-8

Don't use SHIFT_JISX0213 any longer

```bash
iconv -f SHIFT_JISX0213 -t UTF-8 sqls/postgres/create_all_sequences.sql > flyway/sql/V1__create_all_sequences.sql
iconv -f SHIFT_JISX0213 -t UTF-8 sqls/postgres/create_all_tables.sql 	> flyway/sql/V2__create_all_tables.sql
iconv -f SHIFT_JISX0213 -t UTF-8 sqls/postgres/create_all_index.sql 	> flyway/sql/V3__create_all_index.sql
iconv -f SHIFT_JISX0213 -t UTF-8 sqls/postgres/insert_departure.sql 	> flyway/sql/V4__insert_departure.sql
iconv -f SHIFT_JISX0213 -t UTF-8 sqls/postgres/insert_arrival.sql 	> flyway/sql/V5__insert_arrival.sql
iconv -f SHIFT_JISX0213 -t UTF-8 sqls/postgres/insert_accommodation.sql > flyway/sql/V6__insert_accommodation.sql
iconv -f SHIFT_JISX0213 -t UTF-8 sqls/postgres/insert_age.sql 		> flyway/sql/V7__insert_age.sql
iconv -f SHIFT_JISX0213 -t UTF-8 sqls/postgres/insert_employee.sql 	> flyway/sql/V8__insert_employee.sql
iconv -f SHIFT_JISX0213 -t UTF-8 sqls/postgres/insert_customer.sql 	> flyway/sql/V9__insert_customer.sql
iconv -f SHIFT_JISX0213 -t UTF-8 sqls/postgres/insert_tourinfo.sql 	> flyway/sql/V10__insert_tourinfo.sql
iconv -f SHIFT_JISX0213 -t UTF-8 sqls/postgres/insert_tourcon.sql 	> flyway/sql/V11__insert_tourcon.sql
iconv -f SHIFT_JISX0213 -t UTF-8 sqls/postgres/insert_reserve.sql 	> flyway/sql/V12__insert_reserve.sql
```

### CF Push!

Let's deploy to Cloud Foundry.

Cloud Foundry's buildpack will make your app [VII. Port binding](https://12factor.net/port-binding), [VIII. Concurrency](https://12factor.net/concurrency) and [XI. Logs](https://12factor.net/logs) enabled.

To inject configurations via environment vairables according to [III. Config](https://12factor.net/config), 
it is a convenient way to use [Pre-Runtime Hooks](https://docs.cloudfoundry.org/devguide/deploy-apps/deploy-app.html#profile).

Create a `.profile` script for the hook. This script supposes a service instance of PostgreSQL to be bound that will be provisioned by a service broker later by `cf create-service <service name> <plan name> tour-db`.
In this script, we will extract username, password and url of the database from `VCAP_SERVICES` environment variable and override JNDI configuration with these information.

At the same time, we will run the flyway migration command to the database configured in it.

```bash
cat <<'PROF' > .profile
#!/bin/bash

SERVICE_INSTANCE_NAME=tour-db
CREDS=$(echo $VCAP_SERVICES | jq -r ".[] | map(select(.name == \"${SERVICE_INSTANCE_NAME}\"))[0].credentials")

export URL=$(echo $CREDS | jq -r .uri)
export DATABASE_USERNAME=$(echo $URL | awk -F '//' '{print $2}' | awk -F ':' '{print $1}')
export DATABASE_PASSWORD=$(echo $URL | awk -F '//' '{print $2}' | awk -F ':' '{print $2}' | awk -F '@' '{print $1}')
export DATABASE_URL=$(echo $URL | sed 's/postgres/jdbc:postgresql/g' | sed "s/$DATABASE_USERNAME:$DATABASE_PASSWORD@//g")


cat << EOF > ./META-INF/context.xml
<Context>
  <Resource
     name="jdbc/terasolunaTourDataSource"
     type="javax.sql.DataSource"
     driverClassName="org.postgresql.Driver"
     password="${DATABASE_PASSWORD}"
     maxIdle="2"
     maxWait="5000"
     username="${DATABASE_USERNAME}"
     url="${DATABASE_URL}"
     maxActive="4"/>
</Context>
EOF

if [ "${INSTANCE_INDEX}" == "0" ];then
  # This could be "XII. Admin processes"
  
  ./.java-buildpack/open_jdk_jre/bin/java \
    -cp `ls ./.java-buildpack/postgresql_jdbc/*.jar`:./flyway/lib/* \
    org.flywaydb.commandline.Main migrate \
    -url=${DATABASE_URL} \
    -user=${DATABASE_USERNAME} \
    -password=${DATABASE_PASSWORD}
fi
PROF
```

> If you use a modern framework like Spring Boot, you don't need to prepare such a script as the framework should support [externalized configuration](https://docs.spring.io/spring-boot/docs/current/reference/html/boot-features-external-config.html). 

Embed `.profle` and external dependencies on flyway 

```bash
TOMCAT_HOME=/tmp/apache-tomcat-9.0.11

cp ${TOMCAT_HOME}/webapps/ROOT.war ./
jar -uvf ROOT.war .profile flyway/lib flyway/jars flyway/sql
```


```bash
cf create-service elephantsql turtle tour-db
cf push toursample -p ROOT.war --no-start --random-route
cf bind-service toursample tour-db
cf start toursample
```

Conguraturations!

Your legacy app is now running on Cloud Foundry :)

https://toursample.cfapps.io

and scale out easily by `cf scale` command.

```bash
cf scale -i 2 toursample
```

### (Option) Run locally

Prepare PostgreSQL:

```bash
createuser -P -e sample
# > sample
psql -d postgres -c 'create database teradb'
```

Run Tomcat:

```bash
TOMCAT_HOME=/tmp/apache-tomcat-9.0.11

wget https://java-buildpack.cloudfoundry.org/postgresql-jdbc/postgresql-jdbc-9.4.1212.jar -P ${TOMCAT_HOME}/lib/
rm -rf ${TOMCAT_HOME}/ROOT*

${TOMCAT_HOME}/bin/startup.sh
```

Access [http://localhost:8080](http://localhost:8080)

### (Option) Prefer Kubernetes?

[`packs`](https://github.com/buildpack/packs) is a standalone builder with buildpacks that builds a droplet without a CF platform and exports it as a docker image.

Build a droplet:

```bash
docker run --rm \
  -v "$(pwd):/workspace" \
  -v "$(pwd)/out:/out" \
  -e PACK_APP_ZIP=/workspace/ROOT.war \
  -e VCAP_SERVICES='{"postgresql":[{"name":"tour-db","credentials":{"uri":"postgres://user:password@localhost:5432/tour"},"tags":["postgresql"]}]}' \
  packs/cf:build
```

Run a droplet:


```bash
docker run --rm \
  -p 8080:8080 \
  -v "$(pwd)/out:/workspace" \
  -e VCAP_SERVICES='{"postgresql":[{"name":"tour-db","credentials":{"uri":"postgres://sample:sample@docker.for.mac.host.internal:5432/teradb"},"tags":["postgresql"]}]}' \
  packs/cf:run \
  -droplet droplet.tgz \
  -metadata result.json
```

Make a `config.json`:

```bash
DOCKER_USERNAME=yourname
DOCKER_PASSWORD=password

cat <<EOF > config.json
{"auths": {"https://index.docker.io/v1/": {"auth": "$(echo -n ${DOCKER_USERNAME}:${DOCKER_PASSWORD} | base64)"}}}
EOF
```

Export to Docker registry:

```
docker run --rm \
    -v "$(pwd)/out:/workspace" \
    -v "$(pwd)/config.json:/root/.docker/config.json" \
    packs/cf:export -droplet droplet.tgz -metadata result.json making/legacy-app
```

or

Export to Docker daemon:

```
docker run --rm \
    -v "$(pwd)/out:/workspace" \
    -v "$(pwd)/config.json:/root/.docker/config.json" \
    -v "/var/run/docker.sock:/var/run/docker.sock" \
    packs/cf:export -daemon -droplet droplet.tgz -metadata result.json making/legacy-app
```


```
docker run --rm \
  -p 8080:8080 \
  -e VCAP_SERVICES='{"postgresql":[{"name":"tour-db","credentials":{"uri":"postgres://sample:sample@docker.for.mac.host.internal:5432/teradb"},"tags":["postgresql"]}]}' \
  making/legacy-app
```

Make a k8s manifest (Note that this manifest is not production ready!):

```yaml
cat <<EOF > legacy-app.yml
kind: Service
apiVersion: v1
metadata:
  name: legacy-app-service
spec:
  type: LoadBalancer
  selector:
    app: legacy-app
  ports:
  - protocol: TCP
    port: 8080
---
apiVersion: apps/v1
kind: Deployment
metadata:
  name: legacy-app
spec:
  replicas: 1
  selector:
    matchLabels:
      app: legacy-app
  template:
    metadata:
      labels:
        app: legacy-app
    spec:
      containers:
      - image: making/legacy-app:latest
        imagePullPolicy: Always
        name: legacy-app
        env:
        - name: VCAP_SERVICES
          value: '{"postgresql":[{"name":"tour-db","credentials":{"uri":"postgres://username:password@postgres.example.com:5432/tour"},"tags":["postgresql"]}]}'
        ports:
        - containerPort: 8080
        readinessProbe:
          httpGet:
            path: /SC_A99_01_01SCR.do
            port: 8080
          initialDelaySeconds: 5
          timeoutSeconds: 3
          failureThreshold: 3
          periodSeconds: 5
EOF
```

Deploy it:

```
kubectl apply -f legacy-app.yml
```

> See also https://buildpacks.io/
