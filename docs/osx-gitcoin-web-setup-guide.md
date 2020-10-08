## OXS Gitcoin Web Setup Guide For Beginners  

### Prerequisites 

While you don't have to run Gitcoin Web from Docker it makes things much easier. For that reason, this guide will stick to a Docker based setup. 

Install Docker - https://hub.docker.com/editions/community/docker-ce-desktop-mac/

Confirm docker & docker compose are running on the latest version 

From the command line:

```bash
docker --version
```

Docker version 19.03.13, build 4484c46d9d

```bash
docker-compose --version
```

docker-compose version 1.27.4, build 40524192

### Install & Setup 

Clone the repository, change into web dir, copy a over a template for your environmental variables.  

```
git clone https://github.com/gitcoinco/web.git
cd web
cp app/app/local.env app/app/.env
```

#### Environmental Variable Setup - Postgress 

Open your .env file up and look it over. I'll use `code` command to open with vscode but you can use whatever editor you like. 

```
code app/app/.env
```

The Gitcoin Web documentation has a nice section that explains all of the environmental variables [here](https://docs.gitcoin.co/mk_envvars/). It's worth taking a look at that to get familiar with all the different envars. 

Setting up postgres connection string 

For reference, in the docker-compose.yaml file we can see that when the web app is started it spins up a local postgres db container for us. 

```yaml
services:
  db:
    image: postgres:10.1-alpine
    restart: unless-stopped
    ports:
      - "5432:5432"
    volumes:
      - pgdata:/var/lib/postgresql/data
    deploy:
      replicas: 1
      resources:
        limits:
          memory: 1024M
        reservations:
          memory: 128M
```

In order to get the web app to successful connect to this DB I found that I needed to specify my local IP address in the connection string. This is because the app is running inside a docker container and it doesn't see 'localhost' from the same perspective. By specifying your local IP, docker will see and connect to the DB. So, get your local IP and save your database connection string accordingly in your .env file. 


Update your .env file as such:

```
DATABASE_URL=psql://postgres:@<your-local-IP-goes-here>:5432/postgres
```

For example, my config looks like this:

```
DATABASE_URL=psql://postgres:@10.1.10.42:5432/postgres
```

Now, to test that it's working lets go ahead and start the web application up with: 

```
docker-compose up --build
```

The `--build` flag is passed to the `up` command to "Build images before starting containers." If it's the first time running, this while probably take a little while (~30 mins for me behind a VPN). Once the docker images have been downloaded and the app is started, I took note of the following: 

```
db_1             | WARNING: No password has been set for the database.
db_1             |          This will allow anyone with access to the
db_1             |          Postgres port to access your database. In
db_1             |          Docker's default configuration, this is
db_1             |          effectively any other container on the same
db_1             |          system.
db_1             | 
db_1             |          Use "-e POSTGRES_PASSWORD=password" to set
db_1             |          it in "docker run".
```

TODO - set a local pg password. 

#### Running Gitcoin Web 

At this point you should have the Gitcoin web app up and running locally which you can see by navigating to http://localhost:8000 - It's worth note, that if you're testing on a local machine and doing a lot of browsing to other sites, those sites could access your dev app which often times opens you up to a host of security issues. If you're not actively testing and/or you're visiting untrusted sites, make sure to stop your local app instance or better yet, do your dev work on a seperate machine from where you browse. 

After your initial launch of the Gitcoin web app using `docker-compose up --build` you then can use the following command to launch the app and it will launch much more quickly: 

```bash
docker-compose up
```

In a separate terminal lets a take a look at what's going on in the background when we run the above command. 

```bash
docker ps
```

This command will show all the containers running. Uou will notice that the `names` column match the different containers in the docker-compose.yaml file in the root of application.  

Back in my terminal that I ran my docker-compose up command in, I see a lot of errors related to chat_1.  As i'm not going to be doing any chat testing, I can remove that app from the docker-compose.yaml file and restart the app. Make sure to back up your docker-compose.yaml file first just in case.  Then stop and start the app again with docker-compose. If you run `docker ps` in your other terminal again you can see that the chat_1 container is no longer running :) 

It's also worth note, I did just notice [this guide to setting up chat to run locally](https://github.com/gitcoinco/web/blob/master/docs/RUNNING_CHAT_LOCALLY_DOCKER.md) I suspect would help with the chat errors (like no db found) so if you don't want to kill chat all together you can explore that guide. 

The next thing you'll want to do is get your Github integration setup and working. This is pretty quick and easy and following the [Github Integration guide here](https://docs.gitcoin.co/mk_third_party_integrations/) will walk you through those steps. 


### To be continued ... 


