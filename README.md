# Deploying on Ruby on Rails using Unicorn, PostgreSQL and Capistrano 3 for Ubuntu on DigitalOcean

## DigitalOcean

Create a droplet and install the Ubuntu 16.04 system. Following that, login into your droplet using ssh:

```
$ ssh root@xxx.xxx.xxx.xxx
```

## Fixing the irritating locale error.

Run the following commands:

```
$ nano /etc/environment
```

Add the lines below and save:

```
LANG=en_US.UTF-8
LC_ALL=en_US.UTF-8

```
## Setting up user

$ sudo adduser deploy
$ password deployer
$ adduser deploy sudo

```

## Enabling SSH Authentication

```
$ ssh-add ~/.ssh/id_rsa

```
$ ssh-copy-id -i ~/.ssh/id_rsa.pub deployer@xxx.xxx.xxx.xxx

```
Now you can access to server:

```
$ ssh deploy@xxx.xxx.xxx.xxx
```

## Installing Ruby


The first step is to install some dependencies for Ruby.

```
sudo apt-get update
sudo apt-get install git-core curl zlib1g-dev build-essential libssl-dev libreadline-dev libyaml-dev libsqlite3-dev sqlite3 libxml2-dev libxslt1-dev libcurl4-openssl-dev python-software-properties libffi-dev
```

Installing with rbenv is a simple two step process. First you install rbenv, and then ruby-build:

```
cd
git clone https://github.com/rbenv/rbenv.git ~/.rbenv
echo 'export PATH="$HOME/.rbenv/bin:$PATH"' >> ~/.bashrc
echo 'eval "$(rbenv init -)"' >> ~/.bashrc
exec $SHELL

git clone https://github.com/rbenv/ruby-build.git ~/.rbenv/plugins/ruby-build
echo 'export PATH="$HOME/.rbenv/plugins/ruby-build/bin:$PATH"' >> ~/.bashrc
exec $SHELL

rbenv install 2.3.1
rbenv global 2.3.1
ruby -v
```

Tell Rubygems not to install the documentation locally, and then install bundler

```
echo "gem: --no-ri --no-rdoc" > ~/.gemrc
```

```
gem install bundler
```
and run

```
rbenv rehash
```

after installing bundler.

## Installing Nginx

```
sudo apt-get install nginx
```

So now we have Nginx installed, and we can start the Nginx webserver by using the service command:

```
sudo service nginx start
```

The service command also provides some other methods such as restart and stop that allow you to easily restart and stop your webserver.


## PostgreSQL Database Setup

All you need to do in order to install PostgreSQL is to run the following command:

```
sudo apt-get install postgresql postgresql-contrib libpq-dev
```

Next we need to setup our postgres user:

```
sudo su - postgres
createuser --your_user
exit
```

The password you type in here will be the one to put in your my_app/current/config/database.yml later when you deploy your app for the first time.

## Capistrano Setup

The first step is to add Capistrano to your development group Gemfile:

```
group :development do
  gem 'capistrano', '~> 3.6', '>= 3.6.1'
  gem 'capistrano-rails', '~> 1.1', '>= 1.1.7'
  gem 'capistrano-rbenv', '~> 2.0', '>= 2.0.4'
end
```

After that, run bundle --binstubs and then cap install STAGES=production to generate your capistrano configuration.

Next we need to make some additions to our Capfile to include bundler, rails, and rbenv. Edit your Capfile and add these lines:

```
require "capistrano/setup"
require "capistrano/deploy"
require "capistrano/rbenv"
require 'capistrano/bundler'
require 'capistrano/rails'
```

Now we can configure the config/deploy.rb to setup our general configuration for our app. Edit that file and make it like the following replacing "myapp" with the name of your application and git repository:

```
set :application, 'myapp'

set :repo_url, "git@github.com:myapp.git"

set :deploy_to, '/var/www/myapp'

set :unicorn_pid, "#{deploy_to}/shared/pids/unicorn.pid"

set :unicorn_config, "#{current_path}/config/unicorn.rb"

set :unicorn_binary, "unicorn"

append :linked_files, 'config/database.yml', 'config/secrets.yml'

append :linked_dirs, 'log', 'tmp/pids', 'tmp/cache', 'tmp/sockets', 'public/system'

namespace :unicorn do
  desc 'Stop Unicorn'
  task :stop do
    on roles(:app) do
      if test("[ -f #{fetch(:unicorn_pid)} ]")
        execute :kill, capture(:cat, fetch(:unicorn_pid))
      end
    end
  end

  desc 'Start Unicorn'
  task :start do
    on roles(:app) do
      within current_path do
        with rails_env: fetch(:rails_env) do
          execute :bundle, "exec unicorn -c #{fetch(:unicorn_config)} -D"
        end
      end
    end
  end

  desc 'Reload Unicorn without killing master process'
  task :reload do
    on roles(:app) do
      if test("[ -f #{fetch(:unicorn_pid)} ]")
        execute :kill, '-s USR2', capture(:cat, fetch(:unicorn_pid))
      else
        error 'Unicorn process not running'
      end
    end
  end

  desc 'Restart Unicorn'
  task :restart
  before :restart, :stop
  before :restart, :start
end

after "deploy:cleanup", "unicorn:restart"
```

Unicorn namespace need to do in order for restart unicorn server after deploy automatically, or we can do this with a special command:

```
$ cap production unicorn:restart - restart unicorn
$ cap production unicorn:start
$ cap production unicorn:stop
```

Now we need to edit our config/deploy/production.rb

```
set stage: :production

server 'xxx.xxx.xxx.xx', user: 'deploy', roles: %w{web app db}

set :rbenv_ruby, '2.3.1'

set :rbenv_type, :user
```

Replace xxx.xxx.xxx.xx with your server's IP address and set your actual ruby version for rbenv_ruby option.








