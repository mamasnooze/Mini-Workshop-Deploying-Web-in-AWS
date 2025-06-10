# Mini-Workshop-Deploying-Web-in-AWS

## EC2 (Elastic Compute Cloud)
- Open EC2 --> Launch Instance
- AMI: Amazon Linux
- Type: t2.micro
- KeyPair: Create New or use existing keypair (.pem)
- Security Group: create new abd allow in bound rules for port SSH, HTTP, HTTPS, MYSQL, port:3000

## SSH to the Instance IP Public (Public IPv4)
```
ec2-user
sudo -i
```

### Update and Install NodeJS
```
sudo yum update
sudo yum install nodejs npm -y
```

### Install PM2
```
sudo npm install -g pm2
```

### Install Git
```
sudo yum update -y
sudo yum install -y git
git --version
```

> redirect to /home/ec2-user/

### Upload The Web App
```
git clone https://github.com/mamasnooze/Mini-Workshop-Deploying-Web-in-AWS.git
```

> redict to Mini-Workshop-Deploying-Web-in-AWS/

```
npm install
pm2 start
```

## Create Database Instance
- RDS --> Create DB --> mySQL
- Easy Create
- Free Tier
- Set the username and password however you like
- Make sure it's connected to the EC2 we've made earlier

## Declaring the Database
```
rpm --import https://repo.mysql.com/RPM-GPG-KEY-mysql-2023
sudo dnf clean all
sudo yum install -y mysql-community-client
mysql -h <RDS_ENDPOINT> -u <username> -p
```
- Enter the password you've made

### Create New Database
```
CREATE DATABASE database_name;
```
- Then just press ctrl+z

### Open File config/db.js
```
const { Sequelize } = require('sequelize');

const sequelize = new Sequelize('DATABASE_NAME, USERNAME, PASSWORD', {
  host: '<rds endpoint>',
  dialect: 'mysql'
});

module.exports = sequelize;
```

## Install NginX
> Make sure you are already have a domain or a subdomain
```
sudo yum install nginx -y
sudo systemctl start nginx
sudo systemctl enable nginx
sudo systemctl status nginx
```
> Check on your browser, make sure that the page show " welcome to nginx " page, either with your IPv4 and your domain

## Reverse Proxy to NodeJS
- Go to file configuration
```
sudo nano /etc/nginx/nginx.conf
```
- Put this in the http tag {} :
```
server {
    listen 80;
    server_name <subdomain>.<domain>;

    location / {
        proxy_pass http://localhost:3000;
        proxy_http_version 1.1;
        proxy_set_header Upgrade $http_upgrade;
        proxy_set_header Connection 'upgrade';
        proxy_set_header Host $host;
        proxy_cache_bypass $http_upgrade;
    }
}
```

- Test it using :
```
sudo nginx -t
```
> Make sure the configuration file /etc/nginx/nginx.conf test is successful

- Restart NginX
```
sudo systemctl restart nginx
```

> Make sure you are in the app folder (that contains app.js)
- Run using PM2
```
pm2 start app.js
pm2 startup
pm2 save
```

> To see services in pm2 use :
``` pm2 list ```
> To stop service in pm2 use :
``` pm2 stop (app name) ```
> To delete service in pm2 use :
``` pm2 delete (app name) ```

## SSL / HTTPS
> There is a service in AWS that can provide / generate certificate called ACM (AWS Certificate Manager), but in this workshop we will use an easier one, which is using certbot

- Install Certbot
```
sudo yum update
sudo yum install certbot python3-certbot-nginx -y
sudo certbot --nginx -d <subdomain>.<domain>
```
- Enter your email and just answer each question with Y
- There should be this message if it has been successful :
> Deploying certificate
Successfully deployed certificate for <subdomain>.<domain> to /etc/nginx/sites-enabled/<subdomain>.<domain>
Congratulations! You have successfully enabled HTTPS on https://<subdomain>.<domain>

- Restart NginX
``` sudo systemctl restart nginx ```

- Then check your browser
