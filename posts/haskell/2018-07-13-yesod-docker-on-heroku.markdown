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

6. 6. 6. 6. 6. 6. 


### Deploy it
Now your project is ready to be deployed! Just do `git push heroku`, go grab a
drink and come back in 20 minutes or so and you should see the demo application
running on heroku! Congratulations!
