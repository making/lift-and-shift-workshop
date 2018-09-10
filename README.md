# Lift and Shift Workshop

This workshop will show you how to lift and shift your app.

Main flow is following:

1. Make your app 5 factors enabled
2. Extern the authentioncation functionality (ex. OpenID Connect provider)
3. Put a gateway in front of your app
4. Develop a new API behind the gateway as a greenfield app

Rewriting everything would be the best way to modernize the application if the situaiton allows.
But it could be quite risky when you have to migrate legacy code and add new feature at the same time.
In this workshop, we will take a strategy to keep the legacy app as it is and add new functionality as a new app then migrate gradually.

The idea is putting a gateway that receives all requests to the system, checks the user session and redirects to login form of a OIDC provider.
Netflix's Zuul or Spring Cloud Gateway (or other API Gateway products) can take this role. In this workshop, we will use Zuul because SCG does not suppport OIDC integration out of the box as of this writing.   

The gateway passes the JWT issued by the OIDC provider through all requests. We have to add a new filter that verifies and decodes the JWT into the legacy app so that it can garantee the token is issued properly and retieve the user information from the token.
All new APIs will behave as a resoruce server.

![image](https://user-images.githubusercontent.com/106908/45280073-fb4b0480-b50d-11e8-9b96-5491f95901ea.png)


Don't forget what you have to do is "Lift and **Shift**".

DON'T LIFT AND **STAY**.

## [Step1. Make your app 5 factors enabled](step-01.md)

## [Step2. Extern the authentioncation functionality](step-02.md)

## Step3. Put the gateway in front of the app

Under construction...

## Step4. Develop new API as a green field app

Under construction...
