---
title: Deploy Yesod Applications to Heroku with Docker
author: cohomology
tags: haskell, yesod, heroku, postgresql, docker
---
This post is based on a previous post but we will be pushing a docker image to
heroku instead. If you follow this guide  step by step, 
you should have a demo application running on heroku(in a docker container)! 

<!--more-->

If you have not installed the heroku cli tools on your local computer, please do
so now. I recommend that you follow the heroku documentation to install the cli
tools and add your personal ssh key. You will also need to install the stack
tool for building haskell applications on your local computer. Then come back to
this guide.

### Create a new project and set up git

1. `stack new myproj yesod-postgres` to have a new project created by `stack`
   using the `yesod-postgres` template. I assume you want to call your project
   `myproj`, for instance.   

2. Go to your project folder and do
```shell
git init
git add .
git commit -m "initial commit"
```

### Create a project on heroku

1. Create a project on heroku server: `heroku apps:create appname -b
   https://github.com/mfine/heroku-buildpack-stack.git`. Note we are using a
   special buildpack for deloying haskell application to heroku.
2. Add the free "Hobby Dev Plan" posgresql add-on: `heroku addon:add
   heroku-postgresql:hobby-dev`. More detalis
   [here](https://devcenter.heroku.com/articles/heroku-postgresql#promote-your-database-and-begin-using-it).

3. Now you need to get the connection info for the pg database by `heroku
   pg:credentials:url DATABASE` and the output will be sth. in the line of
```
Connection information for default credential.
Connection info string:
"dbname=DBNAME host=HOSTNAME
port=5432 user=USERNAME
password=PASSWORD
sslmode=require"
Connection URL:
   postgres://USERNAME:PASSWORD@HOSTNAME:5432/DBNAME
```



4. Now go to the heroku web console, click on the project that you just created,
   then click on `Setting` and then `Config vars` and add these env vars to you
   project: `YESOD_PGHOST`,`YESOD_PGPORT`, `YESOD_PGUSER`, `YESOD_PGPASS` and
   `YESOD_PGDATABASE`, the values are those you got from the last step.  
   N.B. the name of these variables may change from one build plan to another,
   you would need to look at `config/settings.yaml` file to see what the names need
   to be. 

5. Now go visit `config/settings.yaml` and find the line `port
   "_env:PORT:3000"` and make sure you have `PORT` and nothing else after the
   `_env:`. This is very important because `PORT` is an env var set by heroku
   and your app is only able to bind to this port. If you have anything else
   there your app will fail to start.

6. Now create the `Dockerfile` and fill it with the following content

```
FROM heroku/heroku:16

ENV LANG C.UTF-8

# Install required packages.
RUN apt-get update
RUN apt-get upgrade -y --assume-yes
# Install packages for stack and ghc.
RUN apt-get install -y --assume-yes xz-utils gcc libgmp-dev zlib1g-dev
# Install packages needed for libraries used by our app.
RUN apt-get install -y --assume-yes libpq-dev
# Install convenience utilities, like tree, ping, and vim.
RUN apt-get install -y --assume-yes tree iputils-ping vim-nox

# Remove apt caches to reduce the size of our container.
RUN rm -rf /var/lib/apt/lists/*

# Install stack to /opt/stack/bin.
RUN mkdir -p /opt/stack/bin
RUN curl -L https://www.stackage.org/stack/linux-x86_64 | tar xz --wildcards
--strip-components=1 -C /opt/stack/bin '*/stack'

# Create /opt/yesod-pg-docker/bin and /opt/yesod-pg-docker/src.  Set
# /opt/yesod-pg-docker/src as the working directory.
RUN mkdir -p /opt/yesod-pg-docker/src
RUN mkdir -p /opt/yesod-pg-docker/bin
WORKDIR /opt/yesod-pg-docker/src

# Set the PATH for the root user so they can use stack.
ENV PATH "$PATH:/opt/stack/bin:/opt/yesod-pg-docker/bin"

# Install GHC using stack, based on your app's stack.yaml file.
COPY ./stack.yaml /opt/yesod-pg-docker/src/stack.yaml
RUN stack --no-terminal setup

# Install all dependencies in app's .cabal file.
COPY ./yesod-pg-docker.cabal /opt/yesod-pg-docker/src/yesod-pg-docker.cabal
RUN stack --no-terminal test --only-dependencies

# Build application.
COPY . /opt/yesod-pg-docker
RUN stack --no-terminal build

# Install application binaries to /opt/yesod-pg-docker/bin.
RUN stack --no-terminal --local-bin-path /opt/yesod-pg-docker/bin install

# Remove source code.
#RUN rm -rf /opt/yesod-pg-docker/src

# Add the apiuser and setup their PATH.
RUN useradd -ms /bin/bash apiuser
RUN chown -R apiuser:apiuser /opt/yesod-pg-docker
USER apiuser
ENV PATH "$PATH:/opt/stack/bin:/opt/yesod-pg-docker/bin"

# Set the working directory as /opt/yesod-pg-docker/.
WORKDIR /opt/yesod-pg-docker

CMD /opt/yesod-pg-docker/bin/yesod-pg-docker-api
```

You may want to search and replace `yesod-pg-docker` by the name of your own
  app in this `Dockefile`. 



### Deploy it
Now your project is ready to be deployed! Just do `heroku container:push web;
heroku container:release web`, go grab a
drink and come back in 20 minutes or so and you should see the demo application
running on heroku! Congratulations!
