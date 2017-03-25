---
layout: post
title: Getting started with Capistrano on Rails
---
Today, I have a task "Change deployment script" and it's about deployment with Capistrano and unicorn.

So I decided that I will research about Capistrano and note something about it.

It will help me to resolve this task tomorrow. And maybe it could help you - 'Capistrano' beginner like me.

# What is Capistrano?

## Capistrano is a remote server automation tool.

It supports the scripting and execution of arbitrary tasks, and includes a set of sane-default deployment workflows.

Capistrano can be used to:

*   Reliably deploy web application to any number of machines simultaneously, in sequence or as a rolling set
*   To automate audits of any number of machines (checking login logs, enumerating uptimes, and/or applying security patches)
*   To script arbitrary workflows over SSH
*   To automate common tasks in software teams.
*   To drive infrastructure provisioning tools such as chef-solo, Ansible or similar.
*   Capistrano is also very scriptable, and can be integrated with any other Ruby software to form part of a larger tool.

# What does it look like?

```
    me@localhost $ cap staging deploy

    DEBUG Uploading /tmp/git-ssh.sh 0%
     INFO Uploading /tmp/git-ssh.sh 100%
     INFO [649ae05d] Running /usr/bin/env chmod +x /tmp/git-ssh.sh on example.com
    DEBUG [649ae05d] Command: /usr/bin/env chmod +x /tmp/git-ssh.sh
     INFO [649ae05d] Finished in 0.048 seconds command successful.
     INFO [6a86a816] Running /usr/bin/env git ls-remote git@github.com:capistrano/rails3-bootstrap-devise-cancan.git on example.com
    DEBUG [6a86a816] Command: ( GIT_ASKPASS=/bin/echo GIT_SSH=/tmp/git-ssh.sh /usr/bin/env git ls-remote git@github.com:capistrano/rails3-bootstrap-devise-cancan.git )
    DEBUG [6a86a816]     3419812c9f146d9a84b44bcc2c3caef94da54758    HEAD
    DEBUG [6a86a816]     3419812c9f146d9a84b44bcc2c3caef94da54758 refs/heads/master
     INFO [6a86a816] Finished in 2.526 seconds command successful.
     INFO [26c22cce] Running /usr/bin/env mkdir -pv /var/www/my-application/shared /var/www/my-application/releases on example.com
    DEBUG [26c22cce] Command: /usr/bin/env mkdir -pv /var/www/my-application/shared /var/www/my-application/releases
     INFO [26c22cce] Finished in 0.439 seconds command successful.
    DEBUG [682cbb14] Running /usr/bin/env [ -f /var/www/my-application/repo/HEAD ] on example.com
    DEBUG [682cbb14] Command: [ -f /var/www/my-application/repo/HEAD ]
    DEBUG [682cbb14] Finished in 0.448 seconds command failed.
    DEBUG [902d6fe6] Running /usr/bin/env if test ! -d /var/www/my-application; then echo "Directory does not exist '/var/www/my-application'" 1>&2; false; fi on example.com
    DEBUG [902d6fe6] Command: if test ! -d /var/www/my-application; then echo "Directory does not exist '/var/www/my-application'" 1>&2; false; fi
    DEBUG [902d6fe6] Finished in 0.047 seconds command successful.
     INFO [70365162] Running /usr/bin/env git clone --mirror git@github.com:capistrano/rails3-bootstrap-devise-cancan.git /var/www/my-application/repo on example.com
    DEBUG [70365162] Command: cd /var/www/my-application && ( GIT_ASKPASS=/bin/echo GIT_SSH=/tmp/git-ssh.sh /usr/bin/env git clone --mirror git@github.com:capistrano/rails3-bootstrap-devise-cancan.git /var/www/my-application/repo )
    DEBUG [70365162]     Cloning into bare repository '/var/www/my-application/repo'...
    DEBUG [70365162]     remote: Counting objects: 598, done.
     INFO [70365162] Finished in 3.053 seconds command successful.
    DEBUG [4d3ef555] Running /usr/bin/env if test ! -d /var/www/my-application/repo; then echo "Directory does not exist '/var/www/my-application/repo'" 1>&2; false; fi on example.com
    DEBUG [4d3ef555] Command: if test ! -d /var/www/my-application/repo; then echo "Directory does not exist '/var/www/my-application/repo'" 1>&2; false; fi
    DEBUG [4d3ef555] Finished in 0.445 seconds command successful.
     INFO [90a42e63] Running /usr/bin/env git remote update on example.com
    DEBUG [90a42e63] Command: cd /var/www/my-application/repo && /usr/bin/env git remote update
    DEBUG [90a42e63]     Fetching origin
     INFO [90a42e63] Finished in 2.078 seconds command successful.
    DEBUG [39a7244f] Running /usr/bin/env if test ! -d /var/www/my-application/repo; then echo "Directory does not exist '/var/www/my-application/repo'" 1>&2; false; fi on example.com
    DEBUG [39a7244f] Command: if test ! -d /var/www/my-application/repo; then echo "Directory does not exist '/var/www/my-application/repo'" 1>&2; false; fi
    DEBUG [39a7244f] Finished in 0.455 seconds command successful.
     INFO [8665f0f1] Running /usr/bin/env git clone --branch master --depth 1 --recursive --no-hardlinks /var/www/my-application/repo /var/www/my-application/releases/20130625064744 on example.com
    DEBUG [8665f0f1] Command: cd /var/www/my-application/repo && ( GIT_ASKPASS=/bin/echo GIT_SSH=/tmp/git-ssh.sh /usr/bin/env git clone --branch master --depth 1 --recursive --no-hardlinks /var/www/my-application/repo /var/www/my-application/releases/20130625064744 )
    DEBUG [8665f0f1]     warning: --depth is ignored in local clones; use file:// instead.
    DEBUG [8665f0f1]     Cloning into '/var/www/my-application/releases/20130625064744'...
    DEBUG [8665f0f1]     done.
     INFO [8665f0f1] Finished in 0.141 seconds command successful.
     INFO [bfd2d6bd] Running /usr/bin/env rm -rf /var/www/my-application/current on example.com
    DEBUG [bfd2d6bd] Command: /usr/bin/env rm -rf /var/www/my-application/current
     INFO [bfd2d6bd] Finished in 0.474 seconds command successful.
     INFO [54ea9e57] Running /usr/bin/env ln -s /var/www/my-application/releases/20130625064744 /var/www/my-application/current on example.com
    DEBUG [54ea9e57] Command: /usr/bin/env ln -s /var/www/my-application/releases/20130625064744 /var/www/my-application/current
     INFO [54ea9e57] Finished in 0.054 seconds command successful.
    DEBUG [b5af33fb] Running /usr/bin/env ls -xt /var/www/my-application/releases on example.com
    DEBUG [b5af33fb] Command: /usr/bin/env ls -xt /var/www/my-application/releases
    DEBUG [b5af33fb]     20130625064744
    DEBUG [b5af33fb] Finished in 0.445 seconds command successful.
    DEBUG [10b6e05d] Running /usr/bin/env if test ! -d /var/www/my-application/releases; then echo "Directory does not exist '/var/www/my-application/releases'" 1>&2; false; fi on example.com
    DEBUG [10b6e05d] Command: if test ! -d /var/www/my-application/releases; then echo "Directory does not exist '/var/www/my-application/releases'" 1>&2; false; fi
    DEBUG [10b6e05d] Finished in 0.452 seconds command successful.
     INFO [dd6ef5b4] Running /usr/bin/env echo "Branch master deployed as release 20130625064744 by leehambley; " >> /var/www/my-application/revisions.log on example.com
    DEBUG [dd6ef5b4] Command: echo "Branch master deployed as release 20130625064744 by leehambley; " >> /var/www/my-application/revisions.log
     INFO [dd6ef5b4] Finished in 0.046 seconds command successful.
```

# DigitalOcean Ubuntu 14.04 x64 + Rails 4 + Nginx + Unicorn + PostgreSQL + Capistrano 3

### SSH into Root

```
    $ ssh root@123.123.123.123
```

### Change Root Password

```
    $ passwd
```

### Add Deploy User

```
    $ adduser deployer
```

### Update Sudo Privileges

```
    $ visudo
    username ALL=(ALL:ALL) ALL
```

### Configure SSH

```
    $ vi /etc/ssh/sshd_config
    Port 22 # Change (1025..65536)
    Protocol 2 # Change
    PermitRootLogin no  # Change
    UseDNS no # Add
    AllowUsers deployer # Add
```

### Reload SSH

```
    $ reload ssh
```

### SSH with Deploy User (Don't close root)

```
    $ ssh -p 1026 deployer@123.123.123.123
```

### Install Curl

```
    $ sudo apt-get update
    $ sudo apt-get install curl
```

### Install RVM

```
    $ curl -L get.rvm.io | bash -s stable
    $ source ~/.rvm/scripts/rvm
    $ rvm requirements
    $ rvm install 2.2.0
    $ rvm use 2.2.0 --default
    $ rvm rubygems current
```

### Install PostgreSQL

```
    $ sudo apt-get install postgresql postgresql-server-dev-9.3
    $ gem install pg -- --with-pg-config=/usr/bin/pg_config
```

### Create Postgres User

```
    $ sudo -u postgres psql
    create user deployer with password 'password';
    alter role deployer superuser createrole createdb replication;
    create database MYAPP_production owner deployer;
```

### Install GIT

```
    $ sudo apt-get install git-core
```

### Install Bundler

```
    $ gem install bundler
```

### Setup Nginx

```
    $ sudo apt-get install nginx
    $ nginx -h
    $ cat /etc/init.d/nginx
    $ /etc/init.d/nginx -h
    $ sudo service nginx start
    $ sudo vim /etc/nginx/sites-enabled/default

    upstream unicorn {
      server unix:/tmp/unicorn.MYAPP.sock fail_timeout=0;
    }

    server {
      listen 80 default deferred;
      # server_name example.com;
      root /home/deployer/apps/MYAPP/current/public;

      location ^~ /assets/ {
        gzip_static on;
        expires max;
        add_header Cache-Control public;
      }

      location ~ ^/(robots.txt|sitemap.xml.gz)/ {
        root /home/deployer/apps/MYAPP/current/public;
      }

      try_files $uri/index.html $uri @unicorn;
      location @unicorn {
        proxy_set_header X-Forwarded-For $proxy_add_x_forwarded_for;
        proxy_set_header Host $http_host;
        proxy_redirect off;
        proxy_pass http://unicorn;
      }

      error_page 500 502 503 504 /500.html;
      client_max_body_size 4G;
      keepalive_timeout 10;
    }
```

### Add Unicorn

```
    $ vim Gemfile
    gem 'unicorn'
```

### Update Unicorn Config

```
    $ vim config/unicorn/production.rb

    root = "/home/deployer/apps/MYAPP/current"
    working_directory root

    pid "#{root}/tmp/pids/unicorn.pid"

    stderr_path "#{root}/log/unicorn.log"
    stdout_path "#{root}/log/unicorn.log"

    worker_processes Integer(ENV['WEB_CONCURRENCY'])
    timeout 30
    preload_app true

    listen '/tmp/unicorn.spui.sock', backlog: 64

    before_fork do |server, worker|
      Signal.trap 'TERM' do
        puts 'Unicorn master intercepting TERM and sending myself QUIT instead'
        Process.kill 'QUIT', Process.pid
      end

      defined?(ActiveRecord::Base) and
        ActiveRecord::Base.connection.disconnect!
    end

    after_fork do |server, worker|
      Signal.trap 'TERM' do
        puts 'Unicorn worker intercepting TERM and doing nothing. Wait for master to send QUIT'
      end

      defined?(ActiveRecord::Base) and
        ActiveRecord::Base.establish_connection
    end

    # Force the bundler gemfile environment variable to
    # reference the capistrano "current" symlink
    before_exec do |_|
      ENV['BUNDLE_GEMFILE'] = File.join(root, 'Gemfile')
    end
```

### Add Capistrano

```
    $ vim Gemfile

    group :development do
      gem 'capistrano-rails'
      gem 'capistrano-rvm'
      gem 'capistrano3-unicorn'
    end
```

### Install Capistrano

```
    $ bundle exec cap install
```

### Update Capistrano Capfile

```
    $ vim Capfile

    require 'capistrano/setup'
    require 'capistrano/deploy'
    require 'capistrano/rvm'
    require 'capistrano/bundler'
    require 'capistrano/rails/assets'
    require 'capistrano/rails/migrations'
    require 'capistrano3/unicorn'
```

#### Load custom tasks from `lib/capistrano/tasks' if you have any defined

```
    Dir.glob('lib/capistrano/tasks/*.rake').each { |r| import r }
```

### Update Capistrano Deploy Config

```
    $ vim config/deploy.rb

    lock '3.3.5'

    set :application, 'spui'
    set :repo_url, 'git@github.com:MYGITHUB/MYAPP.git'

    ask :branch, proc { `git rev-parse --abbrev-ref HEAD`.chomp }.call

    set :use_sudo, false
    set :bundle_binstubs, nil
    set :linked_files, fetch(:linked_files, []).push('config/database.yml')
    set :linked_dirs, fetch(:linked_dirs, []).push('log', 'tmp/pids', 'tmp/cache', 'tmp/sockets', 'vendor/bundle', 'public/system')

    after 'deploy:publishing', 'deploy:restart'

    namespace :deploy do
      task :restart do
        invoke 'unicorn:reload'
      end
    end
    Update Production Deploy Config

    $ vim config/deploy/production.rb

    set :port, 22
    set :user, 'deployer'
    set :deploy_via, :remote_cache
    set :use_sudo, false

    server '123.333.333.333',
      roles: [:web, :app, :db],
      port: fetch(:port),
      user: fetch(:user),
      primary: true

    set :deploy_to, "/home/#{fetch(:user)}/apps/#{fetch(:application)}"

    set :ssh_options, {
      forward_agent: true,
      auth_methods: %w(publickey),
      user: 'deployer',
    }

    set :rails_env, :production
    set :conditionally_migrate, true
```

### Add SSH Key to DigitalOcean

```
    $ cat ~/.ssh/id_rsa.pub | ssh -p 22 username@123.123.123.123 'cat >> ~/.ssh/authorized_keys'
```

### Say Hi to Github

Follow the steps in this guide if receive permission denied(public key) <https://help.github.com/articles/error-permission-denied-publickey>

```
    $ ssh github@github.com
```

### Check Deployment (Commit and Push)

```
    $ cap production deploy:check
```

### Deploy

```
    $ cap production deploy
```

# Puma + Rbenv + Capistrano

If you want to find a guide for your app with another combo (Puma + Rbenv + Capistrano) Please check out this link: [Deploying a Rails App on Ubuntu 14.04 with Capistrano, Nginx, and Puma][1]

### Here is another example for Capistrano + Unicorn:

*   <https://coderwall.com/p/yz8cha/deploying-rails-app-using-nginx-unicorn-postgres-and-capistrano-to-digital-ocean>
*   [A perfect minimal Capistrano deploy.rb for Rails and Unicorn with rolling restarts][2]

And if you feel familiar with JAV, please check out this version:

*   [Capistrano3 を使って Rails4 + unicorn + nginx + rbenv にデプロイする][3]

### [Plugin] Capistrano tasks for automatic and sensible unicorn + nginx configuration

*   <https://github.com/capistrano-plugins/capistrano-unicorn-nginx>

### Check out more information

*   [Capistrano][4]
*   <https://github.com/capistrano-plugins>

### Special thank to ChuckJHardy with his [article][5]

 [1]: https://www.digitalocean.com/community/tutorials/deploying-a-rails-app-on-ubuntu-14-04-with-capistrano-nginx-and-puma
 [2]: https://intercityup.com/blog/a-perfect-minimal-capistrano-deploy-rb-for-rails-and-unicorn-with-rolling-restarts.html
 [3]: http://qiita.com/katsuhiko/items/90cb84033c305df2238b
 [4]: http://capistranorb.com/
 [5]: https://gist.github.com/ChuckJHardy/f44dda5f94c6bbdba9a4