<h1 align="center"><strong>How to connect a Prisma docker instance to your AWS RDS MySQL database</strong></h1>

<br />

Please feel free to make suggested improvements and raise issues.

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

## Tutorial to follow

