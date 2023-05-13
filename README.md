This is the base docker project for Symfony projects. It includes:
* php-fpm
* xdebug
* supervisor
* sqlite
* nginx
* certbot
* alpine as base images

See `Makefile`

### Start a new project

```shell
make init-dev
make start
make sh
symfony new . --version="stable" --no-git
```

### Setup certbot on production

```shell
# issue certbot command to obtains certificate in staging mode 
# --webroot: tells certbot to use the webroot plugin to place files in the webroot folder for authentication.
# --webroot-path: specifies the path of the webroot directory.
# --email: your preferred email for registration and recovery.
# --agree-tos: specifies that you agree to ACME’s Subscriber Agreement.
# --no-eff-email: tells certbot that you do not wish to share your email with the Electronic Frontier Foundation (EFF). feel free to omit this if you would prefer.
# --staging: tells certbot that you would like to use Let’s Encrypt’s staging environment to obtain test certificates. using this option allows you to test your configuration options and avoid possible domain request limits. For more information about these limits, please see Let’s Encrypt’s rate limits documentation.
# -d: specify domain name which obtain the certificate 
certbot certonly --webroot --webroot-path=/var/www/app --email your@email --agree-tos --no-eff-email --staging -d your.host -d www.your.host
```

