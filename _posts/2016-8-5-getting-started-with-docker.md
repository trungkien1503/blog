---
layout: post
title: Getting started with Rails on Docker
---
I have a short time with Docker, so I want to share some points with you guys.

They may help you to have a good point to start with Docker.

If you like it, please enjoy and apply it for your applications, or apply on some projects for your teams.

## Why Docker?

Docker is a tool which allows developers to define containers for applications; this allows for control over the operating system and software running the application.

If you’re running against an older version of MySQL, a patched version of Ruby, or other dependencies which make setting up a development environment difficult, Docker may simplify development across a team.

Even without complicated dependencies, forced encapsulation within a VM ensures development parity across a team and, if deploying with Docker to staging or production, across environments.

## Getting started on OS X

I’ll be outlining how to install all dependencies on OS X with Homebrew.

I’ll be using VirtualBox to manage my VM; however, Docker Machine supports a number of other drivers in case you’d prefer to use something else.

## Install VirtualBox with Homebrew Cask

Homebrew Cask is a great tool to install applications on OS X. To install VirtualBox: brew install caskroom/cask/brew-cask brew cask install virtualbox

## Install Docker

With VirtualBox installed, we now install Docker and Docker Machine. OS X and Windows don’t support native virtualization, so Docker Machine will use boot2docker under the hood, which is a minimal shim to get around that.

```
    brew install docker docker-machine
```

Once Docker Machine completes installation, create a machine named default using the VirtualBox driver:

```
    docker-machine create --driver virtualbox default
```

Finally, to export the Docker environment variables to your shell, run:

```
    eval "$(docker-machine env default)"
```

You’ll likely want to add the provided exports to your .zshrc, .bashrc, or other appropriate file so they’re available any time you subsequently open a terminal. To do this automatically, add to one of the aforementioned files:

```
    eval `docker-machine env 2>/dev/null`
```

## Install Docker Compose

Once Docker is installed, next up is Docker Compose. Docker Compose provides a configuration syntax (with YAML) and a CLI to manage multiple containers simultaneously.

You can install Docker Compose with Homebrew:

```
    brew install docker-compose
```

With Docker Compose installed, we’re ready to set up our Rails application to run in a Docker container.

Developing with Docker, Docker Compose, and Rails

With Docker and Docker Compose running on OS X, we can move forward by preparing a Rails app to run in a Dockerized environment. You can find an example Rails application running in Docker here.

## Install Latest Docker and Docker-compose on Ubuntu

```
    # Ask for the user password
    # Script only works if sudo caches the password for a few minutes
    sudo true

    # Install kernel extra's to enable docker aufs support
    # sudo apt-get -y install linux-image-extra-$(uname -r)

    # Add Docker PPA and install latest version
    # sudo apt-key adv --keyserver hkp://keyserver.ubuntu.com:80 --recv-keys 36A1D7869245C8950F966E92D8576A8BA88D21E9
    # sudo sh -c "echo deb https://get.docker.io/ubuntu docker main > /etc/apt/sources.list.d/docker.list"
    # sudo apt-get update
    # sudo apt-get install lxc-docker -y

    # Alternatively you can use the official docker install script
    wget -qO- https://get.docker.com/ | sh

    # Install docker-compose
    sudo sh -c "curl -L https://github.com/docker/compose/releases/download/1.6.2/docker-compose-`uname -s`-`uname -m` > /usr/local/bin/docker-compose"
    sudo chmod +x /usr/local/bin/docker-compose
    sudo sh -c "curl -L https://raw.githubusercontent.com/docker/compose/1.6.2/contrib/completion/bash/docker-compose > /etc/bash_completion.d/docker-compose"

    # Install docker-cleanup command
    cd /tmp
    git clone https://gist.github.com/76b450a0c986e576e98b.git
    cd 76b450a0c986e576e98b
    sudo mv docker-cleanup /usr/local/bin/docker-cleanup
    sudo chmod +x /usr/local/bin/docker-cleanup
```

## Write a Dockerfile

Let’s start with a simple Dockerfile, which lives in Rails’ root:

```
    FROM ruby:2.2.0

    RUN apt-get update -qq && apt-get install -y build-essential

    # for postgres
    RUN apt-get install -y libpq-dev

    # for nokogiri
    RUN apt-get install -y libxml2-dev libxslt1-dev

    # for capybara-webkit
    RUN apt-get install -y libqt4-webkit libqt4-dev xvfb

    # for a JS runtime
    RUN apt-get install -y nodejs

    ENV APP_HOME /myapp
    RUN mkdir $APP_HOME
    WORKDIR $APP_HOME

    ADD Gemfile* $APP_HOME/
    RUN bundle install

    ADD . $APP_HOME
```

There’s only a handful of commands that we’re using with Docker, but let’s go through them:

```
FROM: This describes the image we’ll be basing our container off of; in this case, we’re using the Ruby 2.2 image.

RUN: This is how we run commands; in this example, RUN is used primarily to install various pieces of software with Apt.

WORKDIR: This defines the base directory from which all our commands are executed.

ADD: This copies files from the host machine (in our case, relative to Dockerfile on OS X) to the container.
```

Each command generates a new cached image, which can be built upon later; files with higher churn, or packages that may change, should be placed further down in the Dockerfile to leverage the cache as much as possible.

To build this Docker image, from Rails root run:

```
    docker build .
```

This will build an image with the appropriately installed software - but we’re not done.

## Configure Docker

Even though we’re able to successfully build a Docker image to house our Rails app, we have other dependencies. In this case, we’ll also need to configure a database like Postgres, and ensure that the database container and the application container can talk to one another.

This is where Docker Compose really shines.

Here’s an example docker-compose.yml file:

```
    version: '2'
    services:
      db:
        image: postgres:9.4.1
        ports:
          - "5432:5432"

      web:
        build: .
        command: bin/rails server --port 3000 --binding 0.0.0.0
        ports:
          - "3000:3000"
        links:
          - db
        volumes:
          - .:/myapp
```

In docker-compose.yml, we’re describing two containers. The first is db, which is based on another image (postgres:9.4.1) and exposes port 5432 on port 5432 to the outside world.

The second is web, which uses the Dockerfile (build: .), spins up a Rails server when docker-compose up is run, exposes port 3000 (the Rails app) on port 3000, links the database container, and bases the directory /myapp (WORKDIR from the Dockerfile) off of the Rails app on the host machine.

With docker-compose.yml complete, run:

```
    docker-compose build
```

This will build all the necessary containers.

## Configure the database

Because we’re connecting to a container running Postgres for our database, we’ll need to update our config/database.yml, leveraging the appropriate host based on the name in the docker-compose.yml.

```
    development: &default
      adapter: postgresql
      database: backbone_data_bootstrap_development
      min_messages: WARNING
      pool: 5
      username: postgres
      host: db

    test:
      <<: *default
      database: backbone_data_bootstrap_test
```

Be sure to use the correct environment variables and host names available to your Rails app.

With these settings in place, create and set up the database:

```
    docker-compose run web rake db:create db:setup
```

## Serve the app

To bring the Rails app up, it’s as simple as docker-compose up. You should be able to access the application via a web browser by visiting the IP address of Docker Machine (you can find this out with docker-machine ip default) on port 3000.

### Interact via a Rails console

Running a Rails console is as simple as:

```
    docker-compose run web rails console
```

The structure of the commands here is docker-compose run {CONTAINER_NAME} {COMMAND}. Note that running commands with docker-compose run are in new containers and not a running container (e.g. if you’re already running docker-compose up).

## Docker with Heroku

The first thing you’ll need to do is install the heroku-docker plugin for the Heroku toolbelt:

```
    heroku plugins:install heroku-docker
```

Once you’ve set up a normal Rails app (either new or existing), you’ll need to add an additional file to the root called app.json.

```
    {
      "name": "My App Name",
      "description": "An example app.json for heroku-docker",
      "image": "heroku/ruby",
      "addons": [
        "heroku-postgresql"
      ]
    }
```

Heroku has a number of different images you can use for each of the different languages they support. Next we’ll create a file called Procfile which Heroku uses to help start our app.

```
    web: bundle exec puma -C config/puma.rb
```

The next step is to generate a new Dockerfile. We can use Heroku’s toolbelt for this and by running the command heroku docker:init we’ll get both a Dockerfile file and a docker-compose.yml file.

The Dockerfile is extremely small, only pulling from the heroku/ruby image:

```
    FROM heroku/ruby
```

The docker-compose.yml file is a little more involved and sets up two linked images (one for Ruby/Rails and one for Postgres):

```
    web:
        build: .
        command: 'bash -c ''bundle exec puma -C config/puma.rb'''
        working_dir: /app/user
        environment:
          PORT: 8080
          DATABASE_URL: 'postgres://postgres:@herokuPostgresql:5432/postgres'
        ports:
          - '8080:8080'
        links:
          - herokuPostgresql
      shell:
        build: .
        command: bash
        working_dir: /app/user
        environment:
          PORT: 8080
          DATABASE_URL: 'postgres://postgres:@herokuPostgresql:5432/postgres'
        ports:
          - '8080:8080'
        links:
          - herokuPostgresql
        volumes:
          - '.:/app/user'
      herokuPostgresql:
        image: postgres
```

You’ll notice that there is an environment variable called DATABASE_URL in this file. We can modify our database.yml file to use this ENV variable to connect to the database.

```
    default: &default
      adapter: postgresql
      encoding: unicode
      pool: 5
      timeout: 5000
      username: postgres
      host: <%= ENV['DATABASE_URL'] %>
```

We’ll be using the Puma web server for this Rails application, and we need to set up a config file located in config/puma.rb. Please make sure that you have puma in your Gemfile as well!

```
    port ENV['PORT'] || 3000
    environment ENV['RACK_ENV'] || 'development'
```

## Deploying to Heroku

To deploy to Heroku we’ll first need to make sure we have created an app on Heroku. Take note of the name they gave to your app in the output.

```
    heroku create
```

After that we can deploy it using the following command. Keep in mind that you should use your app name on Heroku, not mine:

```
    heroku docker:release --app YOUR_APP
```

Once it is done deploying (which will take a few minutes the first time as it uploads the somewhat large image slug to Heroku), you can open it in the browser using the command:

```
    heroku open
```

## Useful links

*   https://robots.thoughtbot.com/rails-on-docker
*   http://jonathan.bergknoff.com/journal/building-good-docker-images
*   https://docs.docker.com/compose/compose-file/
*   https://docs.docker.com/mac/step_three/
*   https://semaphoreci.com/community/tutorials/dockerizing-a-ruby-on-rails-application
*   https://blog.abevoelker.com/rails-development-using-docker-and-vagrant/
*   http://robin.wenglewski.de/posts/2014/08/07/docker/
*   https://www.udemy.com/the-docker-for-devops-course-from-development-to-production/?couponCode=SEMAPHORE_CI
*   https://github.com/vallard/docker/blob/master/rails/Dockerfile
*   https://github.com/neckhair/rails-on-docker/blob/master/docker-compose.yml
*   http://stackoverflow.com/questions/33621640/how-to-connect-mysql-in-a-rails-appliation-with-docker
*   http://codetunes.com/2015/docker-compose/
*   http://qiita.com/oshin/items/0d011610238eb4df83fd
*   https://hub.docker.com/r/dockerbase/devbase-rvm/~/dockerfile/
*   https://blog.codeship.com/deploying-docker-rails-app/
*   https://github.com/seapy/dockerfiles/tree/master/rails-nginx-unicorn-pro
*   http://labs.septeni-technology.jp/technote/gioi-thieu-docker/
*   http://blog.codeship.com/testing-rails-application-docker/
*   http://devblog.dwarvesf.com/post/docker-introduction/
*   https://www.digitalocean.com/community/tutorials/how-to-set-up-a-private-docker-registry-on-ubuntu-14-04
*   https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-getting-started
*   https://github.com/wsargent/docker-cheat-sheet
*   https://docs.docker.com/compose/rails/
*   https://gist.github.com/wdullaer/f1af16bd7e970389bad3

## Conclusion

These are some things to share with you guys.

Next time I give you an article how to build a base image which we will use to develop our projects.

Or I apply this base image into our project. So I could come back with Windows 10 and Docker and work normally.
Within a short time, I could get some mistakes.

All comments are welcome. We will have a cup of coffee, then discuss about Docker.

Thank for your reading.