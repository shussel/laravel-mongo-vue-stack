# Build a Laravel-Vue-Mongo App with Laravel Sail

This stack includes an up-to-date Laravel sail installation with all latest versions as of December 2023. Ubuntu 22.04, Laravel 10, PHP 8.2, Mongodb 7, Vue 3.

To complete this build you need docker running on your system and some basic docker knowledge. On Windows you need WSL2 up and running, ubuntu dist recommended.

## Get Laravel sail installation
Run this command where you want your project created. Customize with= to include different services.
```
curl -s https://laravel.build/<app name>?with=redis,minio,mailpit,selenium | bash
```
Test your installation
```
cd <app path>
./vendor/bin/sail up -d
./vendor/bin/sail test
```
Browse http://localhost/ for your laravel app home.

## Add shell alias
Add this alias to your shell to shorten the sail command
```
alias sail='[ -f sail ] && sh sail || sh vendor/bin/sail'`
```

## Publish sail docker files
Copy your docker files to [docker/](docker/) for customization.
```
sail artisan sail:publish
```
Comment out services you don't need to speed up build and testing. If you make changes, rebuild and test your new stack.
```
sail build
sail up
```
Browse http://localhost/ for laravel app home.

## Add php mongodb extension
Add php mongodb pecl install to the run command in [docker/8.2/Dockerfile](docker/8.2/Dockerfile) just after the php install.
```
&& echo '' | pecl install mongodb \
```
Add the extension to [docker/8.2/php.ini](docker/8.2/php.ini)
```
extension = mongodb.so`
```
Rebuild the stack to register Dockerfile run changes.
```
sail build --no-cache
```
Check that mongodb extension is active before proceeding.
```
php -m
```

## Add mongodb container
There is no preset mongodb setup for laravel sail, so you will have to build your own container. Add the following service to [docker-compose.yml](docker-compose.yml)
```
services:
    mongo:
        image: 'mongo'
        build:
            context: ./docker/mongodb
            dockerfile: Dockerfile
            args:
                WWWGROUP: '${WWWGROUP}'
        environment:
            MONGO_INITDB_ROOT_USERNAME: '${DB_USERNAME}'
            MONGO_INITDB_ROOT_PASSWORD: '${DB_PASSWORD}'
            MONGO_INITDB_DATABASE: '${DB_DATABASE}'
        ports:
            - 27017:27017
        extra_hosts:
            - "host.docker.internal:host-gateway"
        command: 'mongod --config /etc/mongod.conf'
        healthcheck:
            test: |
                test $$(mongosh --quiet -u '${DB_USERNAME}' -p '${DB_PASSWORD}' --eval "try { rs.initiate({ _id: 'rs0', members: [{ _id: 0, host: 'mongo' }] }).ok } catch (_) { rs.status().ok }") -eq 1
            interval: 10s
            start_period: 30s
        volumes:
            - 'sail-mongo:/data/db'
            - "sail-mongo-config:/data/configdb"
        networks:
            - sail
volumes:
    sail-mongo:
        driver: local
    sail-mongo-config:
        driver: local
```
The most interesting part of this code is the use of healthcheck to trigger rs.initiate, after the server is ready. This was tricky, but this method is the best of many variations I tried.

Next replace the default database settings in [.env](.env.example) with these. This default root user will be used by both your docker setup and laravel.
```
DB_CONNECTION=mongodb
DB_HOST=mongo
DB_PORT=27017
DB_DATABASE=<app name>
DB_USERNAME=root
DB_PASSWORD=root
```
Create [docker/mongodb/mongod.conf](docker/mongodb/mongod.conf) These settings create a one member mongodb replica set which allows you to use advanced functions in mongo like transactions.
```
security:
  authorization: enabled
  keyFile:  keyfile.txt
net:
  bindIpAll: true
  port: 27017
replication:
  replSetName: "rs0"
```
Create [docker/mongodb/Dockerfile](docker/mongodb/Dockerfile) extending the base mongo build to include [docker/mongodb/mongod.conf](docker/mongodb/mongod.conf) and generate a replication key.
```
FROM mongo:latest

# get project config
COPY  ./mongod.conf /etc/mongod.conf

# generate security keyfile
RUN   openssl rand -base64 755 > keyfile.txt
RUN   chown 999:999 keyfile.txt
RUN   chmod 400 keyfile.txt
```
Rebuild and restart
```
sail build
sail up -d
```
Connect to mongo shell on container. Check login and status.
```
docker-compose exec mongo mongosh
> use admin
> db.auth('root','root')
> rs.status()
```

## Add laravel-mongodb
There is a new official package [mongodb/laravel-mongodb](https://github.com/mongodb/laravel-mongodb) extending laravel eloquent models and queries for mongodb.
```
sail composer require mongodb/laravel-mongodb
```
Add the service provider to [config/app.php](config/app.php)
```
MongoDB\Laravel\MongoDBServiceProvider::class,
```
Make these changes in [config/database.php](config/database.php)`
```
'default' => env('DB_CONNECTION', 'mongodb'),

mongodb' => [
    'driver' => 'mongodb',
    'host' => env('DB_HOST'),
    'port' => env('DB_PORT'),
    'database' => env('DB_DATABASE'),
    'username' => env('DB_USERNAME'),
    'password' => env('DB_PASSWORD'),
    'options' => [
        'database' => env('DB_AUTHENTICATION_DATABASE', 'admin'),
    ],
],
```
Restart and test with database migration
```
sail up -d
sail artisan migrate
```
If this creates the base database tables, you have clear sailing ahead.

## Install laravel breeze
Laravel breeze provides a complete app starter kit uniting laravel with many front end implementations. You get fully functioning login and registration pages, ready to build upon.
```
sail composer require laravel/breeze --dev
```
We are choosing Vue front end with ssr support, out favorite stack.
```
sail artisan breeze:install vue --ssr
```
Update [app/Models/user.php](app/Models/user.php) to replace Authenticatable with MongoDB version
```
use MongoDB\Laravel\Auth\User as Authenticatable;
```
Test migration (probably none needed) then run laravel sail default tests. (all but one should pass)
```
sail artisan migrate
sail test
```
If the tests (mostly) pass, your breeze app skeleton is up and running. You now have working http://localhost/login and http://localhost/register pages along with related services, and a solid foundation to build your app upon. Build something great.

## Sail like a pirate
Sail commands can be long. It's good to create aliases to cut down on typing. All this sail put me in the mind of pirates, so I created this set of themed aliases to smooth the Laravel seas.

**>ship** `cd to your project app folder` You have to be in your ship to sail. Make different ship aliases for different projects.

**>seas** `sail up` Start containers in console debug view, useful when you suspect troubled waters ahead.

**>ahoy** `sail up -d` The skies are clear and you are ready to dev. Cast off with your containers running and your console ready.

**>sink** `sail down` Stop the ship, swim back to shore.

**>dock** `sail build` Return to the dock and take on new code.

**>board** `docker-compose exec <service> <cmd>` Run commands on your containers

**>chest** `board mongo bash` Open bash shell on your running database container.

**>booty** `board mongo mongosh <with authentication>` Open the mongodb shell to manage your data treasure.

**>loot** `sail composer require <package>` Snag your favorite composer package and get away quick.

**>heave** `sail artisan <command>` Put your shoulder into it as you push code with artisan.

**>plank** `sail test` When you think you have it all figured out, walk the plank and find out.

**>fleet** `sail ps` Look at all those containers sailing on the Laravel sea.

[.bash_aliases](.bash_aliases)
```
alias sail='[ -f sail ] && sh sail || sh vendor/bin/sail'
alias ship='cd <app folder>'
alias seas='sail up'
alias ahoy='sail up -d'
alias sink='sail down'
alias dock='sail build'
alias board='docker-compose exec'
alias chest='board mongo bash'
alias booty='board mongo mongosh -u root -p root --authenticationDatabase admin --quiet'
alias loot='sail composer require'
alias heave='sail artisan'
alias plank='sail test'
alias fleet='sail ps'
```

Maybe I went a little overboard? Happy sailing!

