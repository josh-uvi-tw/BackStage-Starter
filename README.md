# [Backstage](https://backstage.io)

This guide will teach you how to install and configure a standalone OSS version of the backStage application.
We assume you have basic understanding of working on a Linux based operating system, using tools like npm, yarn, curl.

## Prerequisites

Some development tools are required to be installed on your computer to get started with Backstage. These tools include;

- [Node](https://nodejs.org/en) Version 16 or higher (We recommend using nvm for this)
  - Install and change Node version with [nvm](https://nodejs.org/en/download/package-manager#nvm)
- [Yarn](https://classic.yarnpkg.com/en/docs/install/#mac-stable) for dependency management
- [Docker](https://docs.docker.com/engine/install/) used for features such as Software Templates and Tech Docs
- [Git](https://github.com/git-guides/install-git) for source control

## Standing Up Backstage

To create a backstage app, run the command below to start the process. The wizard will create a subdirectory inside your
current working directory.

```sh
npx @backstage/create-app demo
```

To start the app, follow the `@backstage/create-app` cli post-installation steps and run the below command from the
newly created app folder.

```sh
yarn dev
```

The Backstage app is ready once you see this line in the terminal.

```sh
[0] Webpack compiled successfully
```

From a web browser, go to http://localhost:3000/, and you should see the Backstage software catalog running.
This is your newly scaffolded Backstage App, Good Luck!

## Configuring Backstage

Backstage is up and running! As you might have seen, it's not yet tailored to your infrastructure. We are going to make
some changes to customise the Backstage app; See the below list

- ### Setting up PostgresSQL

  Backstage offers SQLite, an in-memory database, for easy local setup with no external dependencies. It is recommended
  to
  set up PostgresSQL for a production-ready deployment. Backstage uses the Knex
  database library under the covers, which supports many popular databases. To configure backstage with postgresSQL, see
  the below steps;

  - Follow the
    instructions [here](https://backstage.spotify.com/learn/standing-up-backstage/configuring-backstage/5-config-2/)
    to
    install postgresSQL desktop application
  - Alternatively, you can use docker to run postgresSQL locally
    - Install [docker](https://formulae.brew.sh/formula/docker)
      and [docker-compose](https://formulae.brew.sh/formula/docker-compose) cli's on your local machine
    - Create a docker compose file ( e.g. `touch docker-compose-pg-local.yml`)
    - Copy and paste below example code to the newly created docker compose file
      ```yml
      version: '3.8'
      services:
        db:
          image: postgres:latest
          restart: always
          environment:
            POSTGRES_USER: ${POSTGRES_USER}
            POSTGRES_PASSWORD: ${POSTGRES_PASSWORD}
          ports:
            - '5432:5432'
          volumes:
            - db:/var/lib/postgresql/data
          healthcheck:
            test: ['CMD-SHELL', 'pg_isready -U spotify']
            interval: 5s
            timeout: 5s
            retries: 5
      volumes:
        db:
          driver: local
      ```
    - To pull and start the postgresSQL container using docker-compose, run
      ```sh
      docker-compose -f docker-compose-pg-local.yml up -d
      ```
      > option "-d" means running in detach mode (i.e. background)
    - To stop the postgresSQL container using docker-compose, run
      ```sh
      docker-compose -f docker-compose-pg-local.yml stop
      ```
    - To start the postgresSQL container using docker-compose, run
      ```sh
      docker-compose -f docker-compose-pg-local.yml start
      ```
    - To remove the postgresSQL container and volumes using docker-compose, run
      ```sh
      docker-compose -f docker-compose-pg-local.yml down -v
      ```

- ### Setup Enviroment Variables

  You need to create and set enviroment variables to connect to the Postgres database with backstage. To do that, create a new file `touch .env` in the root directory, copy and paste the below;

  ```sh
  export POSTGRES_HOST=127.0.0.1
  export POSTGRES_PORT=5432
  export POSTGRES_USER=your-user
  export POSTGRES_PASSWORD=your-password
  ```

  Alternatively, run the below command to copy the contents of example.env file to the new .env file.

  ```sh
  cp example.env .env
  ```

  Then, load the .env file to the shelll

  ```sh
  source .env
  ```

- ### Configuring PostgresSQL

  Now that PostgresSQL is installed on your machine, we can tell Backstage to use it.

  The main Backstage configuration file, `app-config.yaml` in the root directory of your Backstage app can be configured
  to
  connect to the local PostgresSQL DB.
  Backstage also supports environment-specific configuration overrides, by way of an `app-config.<environment>.yaml`
  file.
  This is where you'll add the database configuration, since you’re setting up a database on your local machine right
  now.

  - Copy and paste the below example code to `app-config.local.yaml` file in the root directory of your Backstage
    app (
    create
    if it doesn't exist `touch app-config.local.yaml`)

  ```yml
  backend:
  database:
    client: pg
    connection:
      host: ${POSTGRES_HOST}
      port: ${POSTGRES_PORT}
      user: ${POSTGRES_USER}
      password: ${POSTGRES_PASSWORD}
  ```

  - Start Backstage from the terminal, by typing `yarn dev` and pressing enter. As soon as you see:

  ```sh
  [0] webpack compiled successfully
  ```

  You're good to go and explore Backstage again. Note that if you've made any changes before, they might be gone. The
  default database is not persistent.

- ### Setting the application name

  In this step, you'll be updating the application and company name shown in the Backstage UI.

  In the root directory of the Backstage app, open `app-config.yaml` or `app-config.local.yml` and update the values of
  title and name:

  ```yml
  app:
    title: <Any name>
  organization:
    name: <Any name>
  ```
