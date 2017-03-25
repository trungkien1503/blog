---
layout: post
title: Docker - It's not too hard to use - Enjoy it (Part 1)
---

Hi everyone, I see some devs who still feel uncomfortable or hard to use Docker. So today, I will collect some commands and information to help you that feel more familiar with Docker.

# Using docker WITHOUT SUDO

It's strange if we need to type `sudo` every command. Right? Use docker without `sudo`, maybe you need to log out to apply this

```
  sudo groupadd docker
  sudo gpasswd -a ${USER} docker
  sudo service docker restart
```

# Some useful commands

## docker-compose up (you might be use -d option)

The `docker-compose up` command aggregates the output of each container. When the command exits, all containers are stopped. Running `docker-compose up -d` starts the containers in the background and leaves them running.

If there are existing containers for a service, and the service’s configuration or image was changed after the container’s creation, docker-compose up picks up the changes by stopping and recreating the containers (preserving mounted volumes). To prevent Compose from picking up changes, use the `--no-recreate` flag.

If you want to force Compose to stop and recreate all containers, use the `--force-recreate` flag.

Please check this link for more detail: https://docs.docker.com/compose/reference/up/

## docker-compose down

Stops containers and removes containers, networks, volumes, and images created by up.

By default, the only things removed are:

Containers for services defined in the Compose file Networks defined in the networks section of the Compose file The default network, if one is used Networks and volumes defined as external are never removed.

Please check this link for more detail: https://docs.docker.com/compose/reference/down/

## docker-compose stop

Stops running containers without removing them. They can be started again with `docker-compose start`.

## docker ps

The docker ps command only shows running containers by default. To see all containers, use the -a (or --all) flag:

```
  $ docker ps

  CONTAINER ID        IMAGE                        COMMAND                CREATED              STATUS              PORTS               NAMES
  4c01db0b339c        ubuntu:12.04                 bash                   17 seconds ago       Up 16 seconds       3300-3310/tcp       webapp
  d7886598dbe2        crosbymichael/redis:latest   /redis-server --dir    33 minutes ago       Up 33 minutes       6379/tcp            redis,webapp/db
```

Please check this link for more detail: https://docs.docker.com/engine/reference/commandline/ps/

## docker images

The default docker images will show all top level images, their repository and tags, and their size.

```
  kienpt@kienpt-K53E:~$ docker images
  REPOSITORY                                  TAG                 IMAGE ID            CREATED             SIZE
  yourdocker.example.com/hackathon_team_1              1.1                 fe3490e45cdf        7 weeks ago         1.037 GB
  update_gems                                 latest              fe3490e45cdf        7 weeks ago         1.037 GB
  update_gem                                  latest              ec3281a87ba7        7 weeks ago         1.022 GB
  add_rails_12factor                          latest              64c6db5f2d08        7 weeks ago         963.3 MB
  add_rails_erd                               latest              98843513d573        7 weeks ago         945.8 MB
  add_faker_gem                               latest              6864636190c0        7 weeks ago         930.4 MB
  add_whenever                                latest              82c08af729c2        7 weeks ago         914.9 MB
  add_cancancan                               latest              a57f9c42b97f        7 weeks ago         898.4 MB
  add_gems                                    latest              3546d07d17dc        7 weeks ago         883.6 MB
  hackathonteam1_web                          latest              106c45630c1b        7 weeks ago         841.5 MB
  yourdocker.example.com/hackathon_team_1              1.0                 106c45630c1b        7 weeks ago         841.5 MB
  nppgbookingdev_delayed_job                  latest              e1836549b217        10 weeks ago        1.157 GB
  nppgbookingdev_web                          latest              e1836549b217        10 weeks ago        1.157 GB
  yourdocker.example.com/nppg-booking_newsprintgroup   1.0                 0d9780fb9dcf        12 weeks ago        1.208 GB
  yourdocker.example.com/mailcatcher                   latest              bc1f03ae36b7        12 weeks ago        350.2 MB
  yourdocker.example.com/docker-rvm                    1.0                 a1164134e118        12 weeks ago        692.1 MB
  yourdocker.example.com/postgresql                    9.4                 57882b6193a5        12 weeks ago        231.4 MB
```

For more detail: https://docs.docker.com/engine/reference/commandline/images/

## docker stop

```
  Usage:  docker stop [OPTIONS] CONTAINER [CONTAINER...]

  Stop one or more running containers

  Options:
        --help       Print usage
    -t, --time int   Seconds to wait for stop before killing it (default 10)
```

## One liner to stop / remove all of Docker containers:

```
  docker stop $(docker ps -a -q)
  docker rm $(docker ps -a -q)
```

## To run a command ( rails console, rake task, rspec ...)

```
  docker-compose run web /bin/bash -l -c "rails c"
  docker-compose run web /bin/bash -l -c "rake db:migrate"
```

It's like a command with Heroku. Right? You feel it's too long. OK, we will improve it now.

# Command maps into short command

Custom with alias command:

## Change bashrc file

```
  $ sudo vi ~/.bashrc
```

## Add below content:

```
  customdocker_command() {
    docker-compose run web /bin/bash -l -c "$1"
  }
  alias customdocker=customdocker_command
```

## Apply your changes

```
  source ~/.bashrc
```

## Now you could test it.

```
  kienpt@kienpt-K53E:~/workspace/app$ customdocker 'rake db:migrate'
  kienpt@kienpt-K53E:~/workspace/app$ customdocker 'rails c'
  Loading development environment (Rails 4.1.6)
  [1] pry(main)>
```

# Conclusion

I hope you feel better with Docker. It's not too hard. Is this correct?

It's really long for this article, so I will give you more commands on the next part.

See you later and enjoy Docker. Have fun!