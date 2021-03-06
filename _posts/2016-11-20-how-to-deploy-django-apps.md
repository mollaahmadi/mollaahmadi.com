---
layout: post
title: How to deploy django apps?
description: deployment of django applications
tags: djnago nginx uwsgi pyenv
comments: false
permalink: How-to-deploy-django-applications
sitemap:
  lastmod: 2016-11-20
---

Create a lxc container
=================

```bash
lxc-create -n jessie -t debian
```

Configure network
========
```bash
lxc.include = /usr/share/lxc/config/debian.common.conf
lxc.tty = 4
lxc.arch = amd64
lxc.network.type = veth
lxc.network.link = lxcbr0
lxc.network.flags = up
lxc.network.hwaddr = 00:16:3e:59:4b:71
lxc.network.ipv4 = 10.0.3.101/24
lxc.network.ipv4.gateway = 10.0.3.1
lxc.rootfs = /var/lib/lxc/jessie/rootfs
lxc.rootfs.backend = dir
lxc.utsname = jessie
```

```bash
lxc-start -n jessie
lxc-attach -n jessie
```

```bash
adduser user
```


How to install pyenv
=================
Read instruction from [here](https://github.com/yyuu/pyenv-installer)

```bash
apt-get install curl
apt-get install git
apt-get install -y make build-essential libssl-dev zlib1g-dev libbz2-dev libreadline-dev libsqlite3-dev wget curl llvm libncurses5-dev libncursesw5-dev xz-utils
```
* login as user and continue instructions:

```bash
su user
```

* download installer and run it using bash:

```bash
curl -L https://raw.githubusercontent.com/yyuu/pyenv-installer/master/bin/pyenv-installer | bash
```
after install pyenv you must add paths to .bash_profile 

```bash
vim ~/.bash_profile
```

```bash
export PATH="/home/user/.pyenv/bin:$PATH"
eval "$(pyenv init -)"
eval "$(pyenv virtualenv-init -)"
```

* install python 3.5.2 using 

```bash
pyenv update
pyenv doctor
env PYTHON_CONFIGURE_OPTS="--enable-shared" pyenv install -fkv 3.5.2
```

create a virtual env

```bash
pyenv virtualenv 3.5.2 webapp
```

Install app's requirements
=============

```bash
pyenv shell webapp
pip install --upgrade pip
pip freeze
pip install -r requirements.txt
pip install uwsgi
pip install django
```

How to deploy application?
==============
```bash
pyenv shell webapp
python manage.py collectstatic
```

How to create uwsgi service?
==============

```bash
vim /etc/systemd/system/myapp.service
```

```bash
[Unit]
Description=uWSGI Emperor service

[Service]
User = user
ExecStart=/home/user/.pyenv/versions/webapp/bin/uwsgi --http 127.0.0.1:8080 --wsgi-file /home/user/webapp/webapp/wsgi.py --chdir /home/user/webapp/
Restart=always
KillSignal=SIGQUIT
Type=notify
NotifyAccess=all

[Install]
WantedBy=multi-user.target
```

```bash
systemctl start myapp
systemctl enable myapp
```

Install nginx and configure it
=========
aptitude install nginx

```bash
vim /etc/nginx/site-enabled/default
server {
	listen 80 default_server;
	location /static/ {
		expires 30d;
		root /home/user/webapp/;
	}
	server_name _;
	location /{
		proxy_pass http://127.0.0.1:8080;
	}
}
```

notes:
=======
* your STATIC_ROOT in setting.py is:

```python
STATIC_ROOT = os.path.join(BASE_DIR, "static/")
```

Some usefull links
==========
[Digital Ocean](https://www.digitalocean.com/community/tutorials/how-to-serve-django-applications-with-uwsgi-and-nginx-on-ubuntu-14-04)