# Alpine Rails API

<!-- START doctoc generated TOC please keep comment here to allow auto update -->
<!-- DON'T EDIT THIS SECTION, INSTEAD RE-RUN doctoc TO UPDATE -->


- [Introduction](#introduction)
- [Included packages](#included-packages)
- [Included gems](#included-gems)
  - [Updating versions](#updating-versions)
- [Using the Docker image](#using-the-docker-image)
  - [Ruby Server](#ruby-server)
  - [NGINX](#nginx)
- [Rebuilding the Docker image](#rebuilding-the-docker-image)

<!-- END doctoc generated TOC please keep comment here to allow auto update -->

## Introduction

Opinionated Docker image based on Alpine Linux 3.6 that contains the bare minimum to run a GraphQL API server.

**Note:** You must have [Packer](https://www.packer.io) installed if you want to rebuild the Docker image!

## Included packages

- [supervisor](https://pkgs.alpinelinux.org/package/v3.6/main/s390x/supervisor) (3.2.4-r0)
- [nginx](https://pkgs.alpinelinux.org/package/v3.6/main/s390x/nginx) (1.12.2-r0)
- [ruby](https://pkgs.alpinelinux.org/package/v3.6/main/s390x/ruby) (2.4.2-r0)
- [ruby-rake](https://pkgs.alpinelinux.org/package/v3.6/main/s390x/ruby-rake) (2.4.2-r0)
- [ruby-bigdecimal](https://pkgs.alpinelinux.org/package/v3.6/main/s390x/ruby-bigdecimal) (2.4.2-r0)
- [ruby-bundler](https://pkgs.alpinelinux.org/package/edge/main/ppc64le/ruby-bundler) (1.15.4-r0)
- [ruby-json](https://pkgs.alpinelinux.org/package/v3.6/main/s390x/ruby-json) (2.4.2-r0)

## Included gems

- [graphql-libgraphqlparser](https://rubygems.org/gems/graphql-libgraphqlparser)
- [nokogiri](https://rubygems.org/gems/nokogiri)
- [cityhash](https://rubygems.org/gems/cityhash)
- [bcrypt](https://rubygems.org/gems/bcrypt)
- [rails](https://rubygems.org/gems/rails)
- [pg](https://rubygems.org/gems/pg)
- [oj](https://rubygems.org/gems/oj)

### Updating versions

At the top of the `packer.yml` file you will see the version numbers specified like so:

```yaml
variables:
  rails_version: '5.1.4'
  nokogiri_version: '1.8.1'
  cityhash_version: '0.9.0'
  bcrypt_version: '3.1.11'
  graphql_version: '1.1.0'
  oj_version: '3.3.9'
  pg_version: '0.21'
```

Just fill out the verson numbers that you want installed, and run `./build.rb` to rebuild the image. _Almost works like magic!_

## Using the Docker image

1) Add this to the top of your `Dockerfile`:

```
FROM nater540/alpine-rails-api:latest
```

2) Copy your application code into `/home/app/project_name_goes_here`

3) End your `Dockerfile` with this:

```
CMD ["/usr/bin/supervisord", "-c", "/etc/supervisor/supervisor.conf"]
```

### Ruby Server

This container does **NOT** include a Ruby server because everyone has their own preference; having said that, I generally perfer to use [Puma](https://rubygems.org/gems/puma) as my server of choice.

Here is the supervisor config that I use for Puma:

```
[program:puma]
directory=/home/app/project_name_goes_here
command=/usr/bin/puma -C config/puma.rb
process_name=%(program_name)s
stdout_events_enabled=true
stderr_events_enabled=true
stdout_logfile=/dev/stdout
stdout_logfile_maxbytes=0
stderr_logfile=/dev/stderr
stderr_logfile_maxbytes=0
stopasgroup=false
killasgroup=false

autorestart=true
autostart=true
startretries=3
startsecs=10

stopsignal=TERM
stopwaitsecs=10
exitcodes=0,2

username=app
priority=999
numprocs=1
umask=022
```

This file should be copied into `/etc/supervisor/conf.d` to work properly.

### NGINX

Nginx is included and set to startup when the container is brought up.

The default site file is located at `/etc/nginx/conf.d/default.conf` and is configured to return a `404 Not Found` error.

## Rebuilding the Docker image

**Heads Up**

It is strongly recommended that you read [this guide](https://docs.docker.com/engine/userguide/eng-image/baseimages/ "Create a base image") first so that you are familiar with creating base images.

Without further ado, here is how you rebuild the base image and publish it!

1) Install any required gems:
```shell
bundle install
```

2) Run the following command:

```shell
./build.rb
```
