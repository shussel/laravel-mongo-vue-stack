# get laravel sail build with options
curl -s https://laravel.build/build-app?with=redis,minio,mailpit,selenium | bash

# test install
cd build-app
./vendor/bin/sail up -d
./vendor/bin/sail test

# test url http://localhost/

# create git archive
git init
git add -A