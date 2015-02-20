# docker-hook

> Automatic Docker Deployment via Hooks

## Features

* No dependencies
* Super lightweight
* Dead **simple setup process**
* Authentification support

## Setup

#### On Docker Hub

...

#### On Your Server

No worries - it just downloads a bash script. There won't be anything installed or written elsewhere.

```sh
$ curl https://raw.github.com/schickling/docker-hook/master/docker-hook > /usr/local/bin/docker-hook; chmod +x /usr/local/bin/docker-hook
```

## Usage

```sh
$ docker-hook <auth-token> <command>
```

#### Command

#### Authentification

## Example

This example will stop the current running `yourname/app` container, pull the newest version and start a new container.

```sh
$ docker-hook my-super-safe-token ./deploy.sh
```

#### `deploy.sh`

```sh
#! /bin/bash

IMAGE="yourname/app"
docker ps | grep $IMAGE | awk '{print $1}' | xargs docker stop
docker pull $IMAGE
docker run -d $IMAGE
```

## How it works

`docker-hook` is written in plain Bash and does have **no further dependencies**. It uses `nc` to listen for incoming HTTP requests from Docker Hub and then executes the provided [command](#command) if the [authentification](#authentification) was successful.

## License

[MIT License](http://opensource.org/licenses/MIT)
