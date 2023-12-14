# Setup

To start a new project with this package
```
git clone https://github.com/shussel/laravel-mongo-vue-stack <app folder>

cd <app folder>

# copy environment settings
cp .env.example .env

# get composer dependencies
docker run --rm \
    -u "$(id -u):$(id -g)" \
    -v "$(pwd):/var/www/html" \
    -w /var/www/html \
    laravelsail/php82-composer:latest \
    composer install --ignore-platform-reqs
    
# rebuild to reset bind-links
./vendor/bin/sail/sail build --no-cache

# Start sail containers
./vendor/bin/sail up -d

# Generate laravel key
./vendor/bin/sail artisan key:generate

# set up database
./vendor/bin/sail artisan migrate

# get node modules
./vendor/bin/sail npm install

# build vite resources
./vendor/bin/sail npm run build

./vendor/bin/sail test
```
Set up your [.bash aliases](.bash aliases) so you can just type 'sail'.

If something is not working at any step, rebuild your containers from the ground up with this. It will take a while but can clear up issues.
```
sail build --no-cache
```
