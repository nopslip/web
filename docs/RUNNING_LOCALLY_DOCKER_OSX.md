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

#### Running Gitcoin Web Manually 

At this point you should have the Gitcoin web app up and running locally which you can see by navigating to http://localhost:8000 - It's worth note, that if you're testing on a local machine and doing a lot of browsing to other sites, those sites could access your dev app which often times opens you up to a host of security issues. If you're not actively testing and/or you're visiting un-trusted sites, make sure to stop your local app instance or better yet, do your dev work on a separate machine from where you browse. 

After your initial launch of the Gitcoin web app using `docker-compose up --build` you then can use the following command to launch the app and it will launch much more quickly: 

```bash
docker-compose up
```

In a separate terminal lets a take a look at what's going on in the background when we run the above command. 

```bash
docker ps
```

This command will show all the containers running. Uou will notice that the `names` column match the different containers in the docker-compose.yaml file in the root of application.  

While manually running `docker-compose up` works fine, if we want to run the app in the background we can also use this command:

```
docker-compose up -d
```

As that's running in the background, you will need to use this command to stop the app:

```
docker-compose down
```
#### Running Gitcoin Web from Shell Alias (Recommend)
However, for what we're going to be hacking on, there is a short cut that Owocki gave me that we can use to quickly run a slimmed down version of the app.  Using the following steps we can get that setup. 

As of OSX 10.6 the ZSH shell is used, so we'll update that file to include a couple nice aliases. 

```
nano ~/.zshrc 
```
Then add the following lines to the file:

```
# added by <me> for for Gitcoin Web shortcut  
alias gtc_web='docker-compose down; docker-compose up -d; slimweb'
alias slimweb='docker stop web_worker_1; docker stop web_testrpc_1; docker stop web_ipfs_1; docker stop web_chat_1;docker stop web_elasticsearch_1'
```

Then we should reload our ZSH profile and launch the Gitcoin Web app as a deamon in the background (minus some unneeded services) like so:
```
source ~/.zshrc
```

And now we can launch a slimmed down version of Gitcoin Web easily with this command: 

```
gtc_web
```

#### First Third Party Integration Setup - Github 
The next thing you'll want to do is get your Github integration setup and working. This is pretty quick and easy and can be setup by following the [Github Integration guide here](https://docs.gitcoin.co/mk_third_party_integrations/). 



### Work in progress

TODO - add docker install guide including rootless 

https://docs.docker.com/engine/security/rootless/



