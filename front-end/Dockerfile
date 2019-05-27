FROM node:10.15.3

RUN apt-get update -qq

# Get the latest stable release of yarn
RUN yarn policies set-version

RUN mkdir /nuxtapp

WORKDIR /nuxtapp

COPY package.json /nuxtapp
COPY yarn.lock /nuxtapp

RUN yarn

COPY . /nuxtapp

EXPOSE 3000
