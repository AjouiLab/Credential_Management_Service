[![img](https://img.shields.io/badge/Lifecycle-Maturing-007EC6)](https://github.com/bcgov/repomountie/blob/master/doc/lifecycle-badges.md)
[![License](https://img.shields.io/badge/License-Apache%202.0-blue.svg)](LICENSE)

# Issuer Web
https://github.com/bcgov/issuer-kit 참조

## 사전 설치

- [Docker](https://www.docker.com/products/docker-desktop)

- [s2i](https://github.com/openshift/source-to-image/releases)

- [jq](https://stedolan.github.io/jq)

- [ngrok](https://ngrok.com)

- [von-network](https://github.com/bcgov/von-network) - 로컬모드로 실행

## Issuer-Web 실행방법

- A local maildev server will be running at http://localhost:8050: you can use it to monitor the outgoing emails sent from the admin app when creating a new invite.

- ID / Password
  - 관리자 : `issuer-admin`/`issuer-admin`
  - 사용자: `issuer-user`/`issuer-user`

### Demo 모드

 [BCovrin Test](http://test.bcovrin.vonx.io) ledger를 사용하며
 and 로컬 개발 환경에서 인터넷을 통해 웹 애플리케이션에 안전하게 접근할 수 있도록 `ngrok` 사용

1. 터미널 1: [ngrok] 폴더 진입
    
    - `./start-ngrok.sh' 실행

2. 터미널 2: [docker] 폴더 진입
    - run `./manage build` 실행
    - 빌드 완료 시  `./manage start-demo` 실앵

컨테이너 단말의 주소:

- `api`: http://localhost:5000

- `issuer-admin` Administrator app: http://localhost:8081

- `issuer-web` Issuer app: http://localhost:8082

- `db`: http://localhost:27017

- `keycloak`: http://localhost:8180

- `agent`: http://localhost:8024

- `maildev`: http://localhost:8050

재시작 방법:
  - 터미널 2에서 Ctrl-C 누름
  - `./manage stop` 실행 후 다시 `./manage start-demo` 실앵

- 종료 방법:
  - 터미널 2에서 Ctrl-C 누르고 `./manage down`
  - In the first terminal, hit Ctrl-C to stop `ngrok`

### Local Mode

In Local Mode the entire application runs locally and uses a locally deployed Indy ledger. In Local Mode you cannot use the Streetcred agent because it is unable to access the ledger being used.

To run in local mode, open two shell/terminal sessions:

1. Follow the [instructions](https://github.com/bcgov/von-network#running-the-network-locally) to start `von-network` running locally
2. Change to the Issuer Kit [docker](./docker) folder:
   - run `./manage build` to assemble the runtime images for the services.
   - run `./manage start` to start the containers

Once started, the services will be exposed on localhost at the following endpoints:

- `api`: http://localhost:5000

- `issuer-admin` Administrator app: http://localhost:8081

- `issuer-web` Issuer app: http://localhost:8082

- `db`: http://localhost:27017

- `keycloak`: http://localhost:8180

- `agent`: http://localhost:8024

- `maildev`: http://localhost:8050

:information_source: When running in local OR demo mode, changes to the code will not be reflected in the containers unless a rebuild using `./manage build` and restart using `./manage start` is performed.

To restart the applications:

- In the second terminal, hit Ctrl-C and then:
  - run `./manage stop` to stop the apps so you can update the code and restart by rerunning the `./manage` commands above. 

- To stop and delete the storage for the apps:
  - In the second terminal, hit Ctrl-C and run `./manage down`
  - In the first terminal, hit Ctrl-C to stop `ngrok`

## Credential Schema

Each api/controller can issue several credentials matching different schemas: the schema definitions that can be processed by the api/controller are described in [this file](api/config/schemas.json). There are two ways of defining a schema: using the `id` of the schema on the target ledger or, alternatively, defining the `schema_name`, `schema_version` and `attributes` for the schema. Additionally, ***one schema in the provided list must be marked with the `default: true` property***: this describes which schema will be used if no explicit request is forwarded to the api/controller.

When using Issuer Kit in demo mode the api/controller will use the schema marked as default, which corresponds to the schema definition that was published to the BCovrin Test Ledger by the BCGov issuer, and issue credentials that match that definition. In most cases updating the schema definition should not be necessary, however if this was the case the following steps will be required to instruct the controller/agent to publish a new schema definition on the target ledger, and use it:

- update the schema definition(s) in [schemas.json](api/config/schemas.json) using the desired fields.

- update the configuration of the public-facing web application to support the new fields and request the new schema. The public web application is contained in the [issuer-web](./issuer-web) folder and the files to update are `claim-config.json` for the form definition and `config.json` to add (or update) the following section:
```
"credentials": {
    "schema_id": "85459GxjNySJ8HwTTQ4vq7:2:verified_person:1.4.0"
  }
```

:information_source: If you are planning on using Issuer Kit in your own production-like environment - regardless of wether you will be re-using the BCGov schema or creating your own - you may want to update the `AGENT_WALLET_SEED` environment variable with a unique value used only by your agent/organization rather than using the default value used for demo purposes.

## App Configuration

The api, administrator and issuer applications can be configured by using a number of environment variables and settings stored in configuration files. The application is shipped with default configurations that work well when running in the provided dockerized environment, if settings need to be updated look for:

  - `api`: all the settings are defined in the [config/default.json](api/config/default.json) file. The API is a NodeJS application built on [FeathersJS](https://docs.feathersjs.com/api/configuration.html#configuration-directory). Rather than defining multiple files for each environment, the default configuration has been extended to use environment variables that can be inected at runtime. Take a look at the relevant sections in [docker/manage](./docker/manage) and [docker/docker-compose.yml](docker/docker-compose.yml) to learn what is being injected into the api container. Additionally, the body of the email sent out as invite can be customized by updating the [api/config/invite-email.html](invite-email.html) file.

  - `issuer-admin` and `issuer-web` are configured using the configuration files in the respective `public/config` folders. Overriding the file shipped with the application with your custom settings file at deployment time will cause the web application to pick up the settings.

### SMTP Settings

The api/controller uses [nodemailer](https://nodemailer.com) to send email invitations. When running on localhost a `maildev` service is used to intercept outgoing email messages.

If you are running a deployment and require emails to be sent, set the following environment variables appropriately:

  - *SMTP_HOST*: your SMTP server host.

  - *SMTP_PORT*: the port used by your SMTP server.

  - *SMTP_USERNAME*: the username to authenticate with SMTP server.

  - *SMTP_PASS*: the password to authenticate with the SMTP server.
  
  - *ADMIN_EMAIL*: the email address that will be used as sender of your emails.

### API Docs

The `api` service exposes a Swagger documentation page at `http://localhost:5000/swagger/docs`.
Publicly available APIs are documented there.

:information_source: at the time of writing, the response structure of the `/connection` POST method to request a new connection invitation is not accurate. The API docs represent the response the same as for the GET method, however in this case the response represent a new Aries connection invitation that matches the AriesConnection data model in [this file](api/src/models/connection.ts).

### Customizing the services

The configuration settings for the services in Issuer Kit can be found [here](docs/ConfiguringIssuerKit.md).
