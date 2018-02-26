<h1 align="center"><strong>How to connect a Prisma docker instance to your AWS RDS MySQL database</strong></h1>

<br />

- Project uses [graphcool/prisma](https://github.com/graphcool/prisma), Docker and AWS RDS MySQL.
- The ultimate goal to is create a graphql server with [`graphql-yoga`](https://github.com/graphcool/graphql-yoga), `prisma` and AWS RDS MySQL.
- In this repo, we are focused on setting up `prisma` so it can talk to our MySQL instance
- Please feel free to make suggested improvements and raise issues.

## Current status

I had to hardcode `CLUSTER_PUBLIC_KEY` into docker-compose.yml for Prisma but it **works!**

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

## Configure Prisma

Open `~/.prisma/config.yml` and add a new entry (here called example-cluster) to the clusters map:

```
clusters:
  example-cluster:
    host: 'http://localhost:4466'
```

Before applying that definition, we have to generate a public/private-keypair so that the CLI is able to communicate with this Prisma server. Head over to https://api.cloud.prisma.sh/ and execute the following query:

```
{
  generateKeypair {
    public
    private
  }
}
```

Make sure to store those values in a safe place. Normally I would copy the public key and paste it into your `.env` file under `CLUSTER_PUBLIC_KEY=your-key`. However I've had issues here and have been hardcoding my `CLUSTER_PUBLIC_KEY` into my `docker-compose.yml` file.

Copy and paste the private key into `~/.prisma/config.yml` 

```
clusters:
  example-cluster:
    host: 'http://localhost:4466'
    clusterSecret: my-private-key
```

Alternatively you can use the `prisma cluster add` command to manage your `~/.prisma/config.yml` file

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

Your instance playground should now be available at http://localhost:4466/cluster

If you need it you can use the following to take down the instance
```sh
docker-compose down
```

## Configure and deploy your prisma service

Initialize your project using

```
prisma init
```

In your `.env` file, update `PRISMA_SECRET` (and if you selected the graphql-server option your `APP_SECRET`) with a secret. You could use the `openssl rand -hex 20`.


Deploy your project

```
prisma deploy
```

Select your new private cluster (in this case `example-cluster`)

```
? Please choose the cluster you want to deploy "project-name@dev" to 

  workspace/prisma-eu1            Free development cluster (hosted on Prisma Cloud) 
  workspace/prisma-us1            Free development cluster (hosted on Prisma Cloud) 
❯ example-cluster        Local cluster (requires Docker) 
                       
  You can learn more about deployment in the docs: http://bit.ly/prisma-graphql-deployment
```

**Alternatively you can edit your `.env` file on an exiting project**
<br>
In order to deploy to your local prisma cluster from an existing project, you can edit your `.env` file. 

```
PRISMA_ENDPOINT=http://localhost:4466/<prisma-service>/<stage>
PRISMA_CLUSTER=example-cluster
```

## Open playground

To query with your new service on your cluster. Open your browser and go to: `http://localhost:4466/<name-of-service>/<stage>`. `name-of-service` and `stage` can be found in your `prisma.yml` file which you used to deploy your prisma service. 

At this stage you haven't been authenticated so in the CLI go to your `prisma.yml` directory and run the `prisma token` command.

In your browser, copy and paste your token into your playground in the `HTTP HEADERS` section like so:

`{ "Authorization": "Bearer <token>"}`

## Deploying your cluster to AWS EC2 or another cloud provider

I'm not doing to tell you how to configure your Linux server in this section. That will up to you. You need to install [Docker](https://www.docker.com/get-docker) and [Caddy](https://caddyserver.com/download) (our webserver).

I've been using Ubuntu on AWS EC2 so open up ports 80 and 443 on `ufw` and in the security group for HTTP and HTTPS respectively. Don't worry Caddy automatically forwards all HTTP traffic to HTTPS for you.

I then save the `docker-compose.yml` file and `Caddyfile` from this repo on to my server.

Launch your `prisma` app with:

```
docker-compose up -d
```

Edit your `Caddyfile` so it looks like the example below. `Caddy` will automatically generate and renew your SSL certificates with Let's Encrypt which is amazing. The first line of the config file is your domain at which you want to host your prisma cluster. **Note** Let's Encrypt does not like AWS domains so you can't use your EC2 URL. I created a subdomain and pointed it to my EC2 instance.

```
prisma.my-domain.com

proxy / localhost:4466/ {
 transparent
}

errors proxieserrors.log
```

In the same directory as your `Caddyfile`, launch `caddy` using:

```
caddy&
```

Then your done and your private prisma cluster should be up and running with an encrypted endpoint.