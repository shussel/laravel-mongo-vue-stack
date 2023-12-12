FROM mongo:latest

# get project config
COPY  ./mongod.conf /etc/mongod.conf

# generate security keyfile
RUN   openssl rand -base64 755 > keyfile.txt
RUN   chown 999:999 keyfile.txt
RUN   chmod 400 keyfile.txt
