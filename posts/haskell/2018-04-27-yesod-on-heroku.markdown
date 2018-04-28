---
title: Deploy Yesod Applications to Heroku
author: cohomology
tags: haskell, yesod, heroku, postgresql
---

In this post I will show you a step by step guide on how to deploy an yesod application 
to heroku, with optional support of postgresql. If you follow this guide along,
you should have a demo application running on heroku inthe end.

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

4. Now `cd` into the `config` directory in you project folder and open up the
   `settings.yml` file and find the sectin called `database` and replace the
   credentials with what you find in previous step.

5. Create `Procfile` in the root of your project directory and fill it with
   the following lines:
```
web: myproj
```
   where myproj is the name you pick for your project(replace it with your real
   project name).

### Deploy it
Now your project is ready to be deployed! Just do `git push heroku`, go grab a
drink and come back in 20 minutes or so and you should see the demo application
running on heroku! Congratulations!
