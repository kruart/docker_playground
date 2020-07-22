FROM node:13.12.0-alpine

# create app folder inside container
WORKDIR /usr/src/app

# copy package* files to our /usr/src/app folder
COPY package*.json ./

# run command inside container
RUN npm install

COPY . .

