# My Laravel/Mongo/Vue dev stack

![dev screen](https://raw.githubusercontent.com/shussel/laravel-mongo-vue-stack
/main/devscreen.png)

When I'm starting a new project, I take the opportunity to update my development stack. Laravel has long been my goto for PHP work. MongoDB is great for quick development with minimal setup. Vue is most flexible and versatile on the front end. Together they make one of the best current stacks available, and what I use today.

I also try to keep up with the latest versions of the parts involved. This might cause setup headaches, but I'd rather take the pain at the start to build an app that's not already old at its launch. Right now this means using PHP 8.2, Laravel 10, MongoDB 7, and Vue 3.

## Docker with Laravel Sail
Laravel has a great toolkit, but setting up for local development used to be a pain. Docker changed everything. No more XAMPP or other clunky Windows setups. With Docker, you have efficient custom environments that spin up easily as self-contained images you can share or deploy. No more "it doesn't work on my machine" between developers.

[Laravel Sail](https://laravel.com/docs/10.x/sail) makes Laravel development with Docker even better. Sail sets up a Docker network with multiple containers for Laravel/php, databases, and other dev tools. It extends the Laravel command line to work in a Docker environment.

## MongoDB
MongoDB is great for developers. Its document centered model makes sense for web apps. Schemas that change on the fly save time and invite experimentation. Its latest integration with Laravel Eloquent makes building and querying models like using any other database. It speaks javascript.

Since there is no standard MongoDB option in Sail, I replaced the default MySQL container with MongoDB. I set up MongoDB latest in [docker-compose.yml](docker-compose.yml) with a custom [Dockerfile](docker/mongodb/Dockerfile). It's configured as a single member replica set in order to use all the latest features like transactions. I can add additional replica set servers to test advanced setups, or change the whole configuration in [mongod.conf](docker/mongodb/mongod.conf)

## Laravel Breeze
Breeze is a Laravel component that builds an app skeleton with login and registration functions every app needs, and it configures one of the many front-end implementations available with Laravel. My choice is the Vue stack, using Inertia as the connection between Laravel and Vue. 

## Vue 3 with Inertia
Vue is the perfect way to build a front end, the evolutionary leader of reactive javascript. I now use it on every project. Using Vue with Laravel raises some questions. Pure Vue apps handle their own routing and connect to an API. Instead of using Laravel to make a simple API, I would probably use Express JS instead. 

If you prefer to work in Laravel like me, the best way to use Vue is with [Inertia](https://inertiajs.com/) a javascript library that acts as a mediator between your Laravel back-end and Vue front. Laravel handles routing and the page generation process normally, while Inertia serves the result in such a way that provides a true SPA experience. This stack seems very valid to me. Vue is best reserved for a rich user interface.

## Other Tools
Sail includes many useful testing and build tools which I use in my stack at various points in development. [Tailwind CSS](https://tailwindcss.com/) is the logical succesoor to Bootstrap for CSS. [Mailpit](https://mailpit.axllent.org/) solves the problem of local email testing and keeps test emails from escaping to the real world. [MinIO](https://min.io/) is great for working with Amazon S3. [Selenium](https://www.selenium.dev/) is available for advanced testing. My IDE for PHP is [PHPStorm](https://www.jetbrains.com/phpstorm/), I use [VSCode](https://code.visualstudio.com/) for pure Javascript projects.

## Sail like a pirate
Laravel Sail commands can be long. It's good to create aliases to cut down on typing. All this sailing put me in the mind of pirates, so I created this set of themed aliases to smooth the Laravel seas.

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

**>fleet** `sail ps` Look at all those containers sailing on the Laravel seas.

[.bash_aliases](.bash_aliases)

Maybe I went a little overboard?

## Build & Use
This repo is a fully working installation of this stack. Clone, initialize, customize, fire up Docker Desktop then sail up, or 'ahoy".

Build it yourself step by step here [BUILD.md](BUILD.md)
