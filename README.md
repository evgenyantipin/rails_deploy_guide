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






