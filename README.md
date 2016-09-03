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

For more information, you can visit http://capistranorb.com/documentation/getting-started/authentication-and-authorisation/