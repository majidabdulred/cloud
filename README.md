# Setting up cloud server

### Setting up ssh

1. `ssh-keygen`
2. `ssh root@ip`

### Creating new Global User

1. `adduser majid`
2. `usermod -aG sudo majid`
3. `su majid`
4. `mkdir .ssh`

### Copying ssh key to server

1. `nano .ssh/authorized_keys`
2. Copy the public key from your local machine

### Setting up firewall

1. `sudo ufw allow OpenSSH`
2. `sudo ufw enable`
3. `sudo ufw status`

### Change user password

1. `passwd majid`
2. `sudo passwd root`

### Setting up python

1. `sudo apt-get update`
2. `sudo apt-get install python3`
3. `sudo apt-get install virtualenv`
4. `virtualenv venv_backend`

### Setting up git

1. `sudo apt-get install git`
2. `git init`
3. `git remote add origin git@username:username/repo.git`
4. `cd ~/.ssh`
5. `ssh-keygen -t rsa -b 4096 -C "name@email.com"`
6. Copy the public key to GitHub deploy keys
7. `cd /home/majid/apps/backend`
8. `nano update.sh`
```shell
@echo off
echo updating repo
eval $(ssh-agent)
ssh-add ~/.ssh/github_backend
git pull origin main
```
9. `chmod +x update.sh`
10. `./update.sh`

### Setting up pm2

1. `npm install pm2 -g`
2. `nano config.json`
```json
{
    "apps": [{
        "name": "backend",
        "script": "main.py",
        "instances": "1",
        "wait_ready": true,
        "autorestart": true,
        "max_restarts": 10,
        "interpreter" : "/home/ubuntu/apps/backend/venv_backend/bin/python",
    }]
}
```
3. `pm2 start config.json`
4. `pm2 monit`

### Setting up nginx

1. `sudo apt-get install nginx`
2. `sudo nano /etc/nginx/sites-enabled/backend`
```nginx
server {
    listen 80;
    server_name 52.15.210.61;
    location / {
        proxy_pass http://127.0.0.1:8000;
    }
    client_max_body_size 100M;
}
```
3. `systemmctl restart nginx`

### Setting up papertrail

1. `dpkg --print-architecture`
2. `wget https://github.com/papertrail/remote_syslog2/releases/download/v0.21/remote-syslog2_0.21_amd64.deb`
3. `sudo dpkg -i remote-syslog2_0.21_amd64.deb`
4. `sudo apt-get install -f`
5. `sudo nano /etc/log_files.yml`
```yml
files:
  - /home/ubuntu/.pm2/logs/app-out-0.log
  - /home/ubuntu/.pm2/logs/backend-error-0.log
  - /home/ubuntu/.pm2/logs/backend-out-0.log
destination:
  host: logs3.papertrailapp.com # NOTE: change this to YOUR papertrail host!
  port: 31572   # NOTE: change to your Papertrail port
  protocol: tls
exclude_patterns:
  - don't log on me
```
6.`ps auxww | grep [r]emote_syslog`
7. `kill -91233`
8. `sudo remote_syslog -D`
9. `sudo remote_syslog`
