# boarddirector.co
Boarddirector started as a normal Django app.   
it integrates with wordpress and pspdfkit containers (each having their own dockerized
DB).  
A few apps are build as Vue apps.   
We have also created a  "Quasar shell" application.  
Quasar is a Vue derivative that loads legacy Django applications in iframes 
alongside native Quasar applications.


# Getting started

## Folder and File Layout

Folders:

*  backend/ - django app (REST backend and legacy apps)
*  frontend/ - Vue/Quasar app (new frontend)
*  scripts/ - small utility programs for building and running the service
*  conf/ - production configuration files and information

Files:

*  bitbucket-pipelines.yml - bitbucket automated testing configuration
*  docker-compose.yml - configuration for our docker swarm
*  Dockerfile - defines a container to run our Django backend
*  requirements.txt - production package list for Django app
*  requirements-dev.txt - dev requirements for Django app
*  dev-pspdfkit-ssh-key/.pub - public and private keys used for PSPDFKit in dev
*  backend/settings/local.py.example - sample configuration for Django backend - DEPRECATED
*  dotenv.sample - sample environment variable definitions DEPRECATED
*  fabfile.py - outdated deployment scripts (may be updated soon) DEPRECATED
*  runtime.txt - heroku config (used for QA) DEPRECATED
*  Procfile - heroku config (used for QA) DEPRECATED
*  env_files - this is new replacement for custom local.py files
### setup env_files

from the folder, copy these files to other locations:

- `.env` to the top level folder

contents of the .env file sets the ENVIRONMENT variable to the foldername where docker-compose will look for the module 
specific environment variables.

e.g. 

  ENVIRONMENT=local

docker will look in `env_files/local` for the env files it needs.

valid environment values are:

- local - this is local container development
- runserver - this is django runserver, no django container needed, but good idea to start the others, esp DB.
- dev
- test
- prod


    DJANGO_SETTINGS_MODULE=settings.base

and usage of env variable files under a folder named `env_files`.

#### there a utility script to parse a yaml file and create these files.
it's in the scripts folder. but there is a function `backup` that's available. 
see other section in this file about `.aliases`.  

## get the pspdfkit container 

    docker pull pspdfkit/pspdfkit

## troubles with Apple silicon processors 
you might find following error when you try to build services

   `image: mysql:5.7 no matching manifest for linux/arm64/v8 in the manifest list entries`
 
just simply add `platform: linux/x86_64` to this service


## Frontend

Our backend and database will run in Docker containers. The frontend
tools are expected to run on your development workstation outside of
containers.

Gulp is used to compile SASS files into CSS. Is it still used though?
You'll need to install Docker for your platform and at least version 8
of node.js.

### Setup:
consider using Node Version Manager (nvm)[
https://github.com/nvm-sh/nvm/blob/master/README.md]

Install latest Node.js locally

    npm install -g \
      gulp         \
      yarn         \
      quasar-cli
    cd frontend
    yarn install
    cd ../backend/apps/common/static
    npm install

we also have some steps saying something similar but from a different folder:

    cd frontend
    yarn install
    # yarn build is new. it basically runs 'quasar build'
    yarn build

### Use:


- Static assets (such as font files and images) should be placed in Quasars `statics` folder.   
  These files should NOT be placed in `assets`.
From host machine (i.e. not Docker container):
from the repo root


in root (create & test similar for Linux environment)

    start-gulp.cmd 

or simply  
  
    cd apps/common/static; gulp
  
It'll start BrowserSync on :3000 port with proxying to Django.  
This way styles are refreshed without reload, also Django templates are watched and page is refreshed upon change.

### Caveats


### reference for versions
#### dev output of `quasar info` from frontend folder
    quasar i
    
    Operating System         	Linux(5.4.0-1041-aws) - linux/x64
    NodeJs                   	15.12.0
    
    Global packages
      NPM                    	7.6.3
      yarn                   	1.22.10
      quasar-cli             	0.17.26
      vue-cli                	Not installed
      cordova                	Not installed
    
    Important local packages
      quasar-cli             	Not installed
      quasar-framework       	Not installed
      quasar-extras          	Not installed
      vue                    	2.6.11	(Reactive, component-oriented view layer for modern web interfaces.)
      vue-router             	3.2.0	(Official router for Vue.js 2)
      vuex                   	3.4.0	(state management for Vue.js)
      electron               	Not installed
      electron-packager      	Not installed
      electron-builder       	Not installed
      @babel/core            	7.13.14	(Babel compiler core.)
      webpack                	4.43.0	(Packs CommonJs/AMD modules for the browser. Allows to split your codebase into multiple bundles, which can be loaded on demand. Support loaders to preprocess files, i.e. json, jsx, es7, css, less, ... and your custom stuff.)
      webpack-dev-server     	3.11.0	(Serves a webpack app. Updates the browser on changes.)
      workbox-webpack-plugin 	4.3.1	(A plugin for your Webpack build process, helping you generate a manifest of local files that workbox-sw should precache.)
      register-service-worker	1.7.1	(Script for registering service worker, with hooks)

## Daily development

In one shell run this:

    docker-compose up db boarddirector

In another run:

    cd frontend
    yarn dev

To run build command:

    cd frontend
    yarn build

You can login at http://localhost:8090/, but keep in mind that the landing page
may not be the development server.  
After logging in, manually navigate back to http://localhost:8090/ 
to ensure you're looking at the right version of the code.

# bash aliases and functions

## two alias files 
these can be found in top level folder in file `.aliases` and are for host level usage.
Instead of reviewing them here, please review the actual file.
they contain aliases that will help deploy code, build code, manage containers, get inside container, etc.
If you want aliases to use inside the container, there is also another file `.docker_bash_aliases` 

## .aliases file

bash function `lenv`

    function lenv () { export $(grep -v ^# "$*" | xargs -0) }

bash alias

    alias dc='docker-compose'

in case you need to pip install on a mac, use `CLFLAGS` below to allow reportlab to compile  

    CFLAGS="-Wno-error=implicit-function-declaration" pip install -r requirements.txt 

# install pyenv
docker containers don't use virtualenvs, but local developers might want this for their IDE
or to run django inside local command line. 

install both of these tools to create the python virtual environment
- (pyenv)[https://github.com/pyenv/pyenv#homebrew-in-macos]
- (pyenv-virtualenv)[https://github.com/pyenv/pyenv-virtualenv#installation]

to see versions of python available

    pyenv install -l

likely best to install same version i have (or whatever are in the contents of `.python-version` file)

    pyenv install 3.7.16

this command would then be used to create the virtual env named `bd37` for the above python version

    pyenv  virtualenv   3.7.16  bd37

you would then activate it like this:

    pyenv activate bd37

once it’s “activated”, it changes you prompt , adding `(bd37)` to it from command line.
next step is to install the python libraries that the virtualenv will use.

    pip install -r requirements-dev.txt

after python libraries are installed, it's possible to run django commands, like runserver. 
if you would simply prefer to use django runserver during development or want to run management commands 
or start django `shell`, then use this script

    scripts/django.runserver.sh runserver 8080

but it's just a wrapper around `manage.py` command. 
you can pass in other things, like  any of these

  scripts/django.runserver.sh shell
  scripts/django.runserver.sh makemigrations
  scripts/django.runserver.sh showmigratoins
  scripts/django.runserver.sh migrate
  scripts/django.runserver.sh --help  

if you don't want to run the script every time, you can 'source' it instead.

    source scripts/django.runserver.sh

run this after the last command to confirm it DJANGO_SETTINGS_MODULE variable is set:
     
    env|grep DJ

above command should show this on screen

    DJANGO_SETTINGS_MODULE=settings.base

the script will change your current working directory to "backend"
you can then run `./manage.py` commands directly.
e.g.

    ./manage.py runserver 8080
    ./manage.py shell

assuming you already ran this  `source .aliases` , you need to ensure that the DB is up.
see next section on .aliases

start the DB container

    dc up db -d

## Django migrations

get inside container first

    docker-compose exec boarddirector bash

once inside, you can create an alias for the `manage` command

  alias manage='python manage.py'

this is the manage command for running inside container:

    python manage.py

migrations should be run from inside the container.

to migrate schema forward (this applies new changes) 

    python manage.py migrate

other things you can run:

show if migrations are applied

    python manage.py showmigrations

create new migrations

    ../scripts/manage.py makemigrations

to migrate schema backward (this applies reverting changes) 
    replace `$` variables below

    # show us which migrations exist 
    python manage.py showmigrations $app_name
  
    # use output from last command below
    python manage.py migrate $app_name $version_number


# Sign up for a test account 
(this is no longer necessary)

Open a browser at http://localhost:8090/ and click the signup link.
You can use any email address, just make sure that your boarddirector
url is "test-company".

After signing up...

with your docker running, in another shell run this command:

    docker exec -it boarddirector_boarddirector_1 scripts/create_superuser.sh

that will create a superuser with the username admin@boarddirector.co and the password of password
then you will go to the admin which should be running at: 
http://localhost:8080/login/account/login/?next=/app-admin/
login using those credentials… from there you have access to your local admin console which we can activate the user/membership
so you would go here: http://localhost:8080/app-admin/profiles/user/
click on the user
then tick the “active” checkbox and save

# DB Backup and Restore
Backup Wordpress

    mysqldump --add-drop-table -h localhost -u wordpressuser -p wordpress
    mysqldump --no-tablespaces --add-drop-table -h localhost -u wordpressuser -p wordpress

Restore mysql wordpress

    mysql -h localhost -u wordpressuser -p wordpress < backup.2020.20.23.sql
    

once restored, if DNS name is different (for example restoring prod database to uat database), to get it working without
being redirected back to prod, you need to update some tables.

change this

     select * from wp_options where option_name like '%siteurl%';
    +-----------+-------------+------------------------------+----------+
    | option_id | option_name | option_value                 | autoload |
    +-----------+-------------+------------------------------+----------+
    |         1 | siteurl     | https://www.boarddirector.co | yes      |
    +-----------+-------------+------------------------------+----------+

    update wp_options set option_value='https://aws-uat.boarddirector.co' where option_name like '%siteurl%';

this updates the www.boarddirector.co values inside "option_value" field with uat values 

    update wp_options set option_value=replace(option_value, 'www.boarddirector.co', 'aws-uat.boarddirector.co');

this replace similarly but in different table
   
    update wp_yoast_seo_links set url=replace(url, 'www.boarddirector.co', 'aws-uat.boarddirector.co');  

mysql update wordpress users table example

    update wp_users set display_name='David', user_email='pyguy411@gmail.com', 
      user_nicename='David',user_login='pyguy411', 
      user_pass=MD5("secret123") where id=9; 

Postgres

To connect

    psql -d$POSTGRES_DB -U$POSTGRES_USER

use these 2 commands to setup the environment variable you will need both inside and outside of the container 
for several of the examples below using `${backup_filename}`

note to update these two variables appropriately

    export name=app
    export name=pdfkit
    export environment=prod

These are common ones again.

    export timestamp=`date  +%F`
    export backup_filename=${environment}.${name}.${timestamp}.sql


Backup of postgres (for django and pdfkit) 
    
    pg_dump -d$POSTGRES_DB -U$POSTGRES_USER -f ${backup_filename} 

don't forget to copy the new file from container to host

    docker cp boarddirector_db_1:/${backup_filename} .
    docker cp boarddirector_pdf_db_1:/${backup_filename} .
  
from outside container, run this to copy backup file to db where restoration will occur
set the ${backup_filename} variable using steps just above.

    docker cp ${backup_filename} boarddirector_db_1:/

before restoring, it's sometimes important to follow these steps

steps to connect to DB and then drop (from inside psql shell)
1. disconnect all sql clients (stop the app)
  

    docker-compose stop boarddirector


- delete/recreate the database


    psql -d$POSTGRES_DB -U$POSTGRES_USER
    \c postgres
    drop database boarddocuments_db;
    create database boarddocuments_db;
    \q
  
to restore data from backup file 
(example using variable `${backup_filename}`, change aptly or run the same commands used to set the var name)

    psql -d$POSTGRES_DB -U$POSTGRES_USER -f ${backup_filename} 

  

## some notes about restore
when restoring pdfkit to uat, i had to stop the containers, remove the mount volume, start containers, restore data
it's actually much easier to drop the database, create it, and then restore the data
you must stop the containers related to it first though

    dc stop boarddirector
    dc stop pspdfkit


## scripted backup via script
1st arg helps name the file
2nd arg identifies which `.env` file to use in the env_files
script looks up it's own ENVIRONMENT variable in top level `.env` file

    /data/boarddirector/scripts/backup.sh  app db
    /data/boarddirector/scripts/backup.sh  pdf pdf_db


## cronjobs for automatic backup (every 12 hours, gzipped into /data/boarddirector/backups)

    0 */12  * * * /data/boarddirector/scripts/backup.sh  app db >>/data/boarddirector/backup_app.log 2>&1    
    0 */12  * * * /data/boarddirector/scripts/backup.sh  pdf pdf_db >>/data/boarddirector/backup_pdf.log 2>&1                                                                     


### upgrading postgres 

* backup data
* stop containers
* delete the correct mount volume
* start db container only
* get inside it
* connect to postgres db using `psql`
  - drop the original database
  - create database
  - restore from the backup file
start the rest of the containers
    
### resetting your DB steps
If you are just resetting your DB, restoring from dev locally etc.
you can just do these steps (similar to above, but starting after step 3 where mount volume is delete):

### Frontend and Backend

The frontend is a Quasar app and lives in the ./frontend directory.  
You should cd into this directory and run `quasar dev` to do development. 
`quasar build` should be run before each commit.


The backend is a Django app and lives in the ./backend directory.

# Goals

- Move all apps into Quasar/Vue with materialize styles
- Increase test coverage

# Development setup

### Frontend and Backend

## Installation
## web certificates

### Creating a public/private key for PSPDFKit

When prompted for a passphrase, leave it blank just press ENTER.

    ssh-keygen -t rsa -b 4096 -m PEM -f pspdfkit-ssh-key
    openssl rsa -in pspdfkit-ssh-key -pubout -outform PEM -out pspdfkit-ssh-key.pub

### When importing cert for GO DADDY
(should no longer be needed b/c we now use let's encrypt in all environments)

the key file remains the same (if you chose file for apache)
the only thing to replace is the certificate file www.boarddirector.co itself
go daddy renews the certificate annually and the original CSR used that key.
to avoid having to update the config, you can do this (assuming you have the files already in /etc/ssl)

rename original crt that is being replaced

      mv www.boarddirector.co.crt www.boarddirector.co.expired.crt

rename the new new file to match what's in the config

      mv 3021a66a4502339a.crt www.boarddirector.co.crt

restart nginx service

      systemctl restart nginx


### let's encrypt is used to manage the PDF certificates automatically.
to get it working, port 80 (AWS firewall port) needs to be open if you run this command 


    do certbot certonly --nginx

### Board director
   
Note that the `DOCKER_AWS_STORAGE_BUCKET_NAME` and other S3 related variables are
optional and, if they are not defined, then boarddirector will store its data in a 
docker volume.

dev-server will use docker-compose to launch several servers.   
Use `docker container ls` to find the one with 'boarddirector' as its base name.
Then run this command:

    docker container exec name_of_container /app/scripts/initial-setup.sh

### Unit Testing
before we upgrade beyond django 1.11 (currently at 1.11) we should fix any broken tests
and look for the deprecation warnings `python -Wd`
 
(1.11 release notes)[https://docs.djangoproject.com/en/1.11/releases/1.11/]

### Swagger
There is swagger endpoint , with `url` being specific to `Account.url` of `request.user` Membership
For example `url` is `local-dev-mill`

replace `URL` appropriately 

    /{URL}/api/v1/swagger/#/api

e.g.

    /local-dev-mill/api/v1/swagger/#/api


### JSON API docs

In development there are two URLs where you can view the API docs.  
You must login with an account first then you may use these URLs 
(replacing some-company-url with your company URL in your dev account):

    /some-company-url/api/v1/
    /api/v1/


### Deployment:

- No special steps required: generated css files are stored in the repo
- No special steps required: generated css files are stored in css.generated and committed into git.
- no minification yet in gulp (there is some in compress, I think).
- .map files are currently not committed, use locally for development.

### Notes on Bootstrap:

Project mostly uses outdated hand-made css layouts, with a lot of things messing up in all.css + custom.css.

To switch page to newer styles replace base styles:
```
{% block base_resources %}
  <link rel="stylesheet" href="{% static 'css.generated/base.css' %}"/>
  <script src="{% static 'js/libs/jquery-1.12.4/jquery.min.js' %}"></script>
{% endblock %}
```

This way you'll have:
- Bootstrap (so default `.row`/`.col.*.*`/`btn`, etc)
- Use single per-page stylesheet (possibly split into partials via SASS) so that it's under control
- Use common variables via SASS for colors, margins, etc


### REST Development:

Some highlights:

- `api_urls` is place to link Routers for each module, reason: to load it separately from `urls` modules and their dependencies.
  Not strictly required, might be changed if needed
- `api_views` modules used to store viewsets and other things
- `/{account_url}/api/v1/` is binding point for all apis
- `/api/v1/` is binding point for "global" apis like `accounts/*`
- `common.api_urls` single router is constructed here, uniting routers from different apps
- currently mapping is done without any app prefixes, there was no other good way to have single entrypoint
  to list all available APIs from different apps (Yuri), if you know better way - you're welcome to fix it.
- there is a couple of useful mixins in `common.mixin` like
  - `GetMembershipWithURLFallbackMixin` (used to get_membership and account without session requirement)
  - `PerActionSerializerModelViewSetMixin` (to have different serializers for different verbs, for less detailed list for example)
- and also `permissions.rest_permissions` with some common permission classes

### REST Usage:

Auth:

* Token Auth:
    * `POST /api/v1/token-auth/` body: `{"username": "user@email.com", "password": "password-here"}`
    * Store `token` field from response
    * Then add `Authorization: Token $token$` header to any subsequent request, this will authorize as user
    * Token may expire some time in future (currently no) in this case extra login is required

# Rest Implementation

The check_permission method of REST framework has been overridden.

We invoke the and_permission method as a pre-flight check. If this if false
no other checks will be done.

We then try has_role_permission and has_object_permission. If both of those
are false then or_permission is invoked as a last chance to allow the API call.


# setting up new servers - aws ec2 notes
## Google Social Auth  
- uses [OAuth 2.0 Client IDs](https://console.cloud.google.com/apis/credentials?organizationId=137224225219&project=sapient-mariner-163308&authuser=1)
- [link to new keys](https://console.cloud.google.com/apis/credentials/oauthclient/112922430236-3g34lkfcl49tffb61422u4pah9sgq4vc.apps.googleusercontent.com?authuser=1&organizationId=137224225219&project=sapient-mariner-163308)
- which are only in dev so far, but will be deployed to prod soon.

## Data volume
### If you create an volume from a snapshot, the UUID will need to be changed.

to see the uuid and your volumes, run 

    blkid

to generate a new one, specify correct path from above output 

    xfs_admin -U generate /dev/xvdg

it might have an error, i believe it will guide you ("xfs_repair -L <path>")
    
### to mount the data volume:

    mount -t auto /dev/xvdf1 /data
    
setup sym link to main nginx.conf file in the repo (replacing the system original)
    
in folder /etc/nginx$ 

    sudo ln -s /data/boarddirector/scripts/nginx.conf
    
    
in folder /etc/nginx/sites-enabled, create symlink to the application specific nginx to be included.
    
    sudo ln -s /data/boarddirector/scripts/app_nginx.conf

After running `collectstatic` we need to make a copy for spa assets to be downloadable.

this should do it
    
    cd /data/boarddirector/backend/
    cp -R public/static/nav.spa/css public/static/nav/

might need to fix permissions afterwards

## wordpress can be difficult to migrate to new server. 
this will allow you to copy the mounted docker folder from digital ocean to aws
`./aws-uat/env_files/wordpress_mount_docker/_data` is taken directly from prod digital ocean

    mv ./aws-uat/env_files/wordpress_mount_docker/_data ./aws-uat/env_files/wordpress_mount_docker/html 
    docker cp ./aws-uat/env_files/wordpress_mount_docker/html boarddirector_wordpress_1:/var/www

#PWA

## Icons

icongenie generate -i src/statics/logo.png --skip-trim --splashscreen-color 474747

## Dev mode
yarn dev --mode pwa
