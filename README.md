<h1 align="center"><strong>How to connect a Prisma docker instance to your AWS RDS MySQL database</strong></h1>

<br />

- Project uses [graphcool/prisma](https://github.com/graphcool/prisma), Docker and AWS RDS MySQL.
- The ultimate goal to is create a graphql server with [`graphql-yoga`](https://github.com/graphcool/graphql-yoga), `prisma` and AWS RDS MySQL.
- In this repo, we are focused on setting up `prisma` so it can talk to our MySQL instance
- Please feel free to make suggested improvements and raise issues.

## Current status

It is working

## Setting up your AWS MySQL RDS database

Use the following [tutorial](https://gist.github.com/marktani/8631cb9c63d0973bcdd8bff19d6162c2) to setup a MySQL database on AWS RDS's free tier

## Check your MySQL instance is available

Using the local mysql client, connect to your AWS RDS database

```sh
mysql -u {username} -p{password} -h {remote server ip} {DB name}
```

## Configure Docker

- Make sure you have Docker installed
- Save the [`docker-compose.yml`](./docker-compose.yml) file
- Create your `.env` file. You can use [`.env.example`](./.env.example) as a template
- `SQL_INTERNAL_DATABASE` needs to be set to `graphcool`. I think it might be an internal db used by prisma and the name of the db is not related to your db name in AWS

**Note** When creating your `.env` file, do not use double quotes (`"`). Especially on the DB_HOST because they cause a `java.net.UnknownHostException` error.

For those interested, I'm using the following docker repo: [`prismagraphql/prisma`](https://hub.docker.com/r/prismagraphql/prisma/)

## Launch Docker

The following command starts the docker instance using the docker-compose.yml file

```sh
docker-compose up 
```

Check your instance is running
```sh
docker ps
```

Get the logs
```sh
docker logs prisma
```

Take down the instance
```sh
docker-compose down
```