# Docker Webstack

[![Docker Build Status](https://img.shields.io/travis/eXistenZNL/Docker-Webstack.svg?style=flat-square)](https://travis-ci.org/eXistenZNL/Docker-Webstack) [![Docker Pulls](https://img.shields.io/docker/pulls/existenz/webstack.svg?style=flat-square)](https://hub.docker.com/r/existenz/webstack/) [![License](https://img.shields.io/github/license/existenznl/docker-webstack.svg?style=flat-square)](https://github.com/eXistenZNL/Docker-Webstack/blob/master/LICENSE)

## About
This container is a fairly simple Nginx / PHP-FPM container that can be used as a base for your own web containers. It makes use of [s6-overlay](https://github.com/just-containers/s6-overlay) as it's init daemon / process supervisor, and comes in three flavours: PHP 5.6, PHP 7.0 and PHP 7.1. It is rebuilt and tested every day on Travis-CI, so you will always have the latest security patches of Nginx and PHP on hand.

## Why?
I can hear you thinking "aren't there already plenty good Nginx / PHP containers out there?".
To me, there weren't, as I found that all existing containers either run some kind of bash script to start both Nginx and PHP, or use [supervisord](http://supervisord.org/) to start one or more processes in the background.
The former felt really hacky to me, and the latter is not meant to be used as an init daemon as [it does not handle the different signals for process 1 properly](https://blog.phusion.nl/2015/01/20/docker-and-the-pid-1-zombie-reaping-problem/) and makes your container possibly end up with zombie processes.

So I started looking for proper init daemons that can take care of this situation and I found [s6-overlay](https://github.com/just-containers/s6-overlay) which explains in great detail how they overcame the aforementioned problems.

## The goals of this container

- Be always up to date with the latest [packages from Alpine Linux](https://pkgs.alpinelinux.org/packages)
- Minimize the lines of code needed in your own Dockerfile and optimize readibility.
- Have sane defaults for Nginx, PHP, and FPM that can be easily overwritten if needed.

## How can I use it?

You can create your own containers based upon this container with a simple FROM in your Dockerfile.

### Before you start

Before start hacking away, you should know this:
- Nginx runs under the system's nginx user, and PHP-FPM runs under the system's php user.
- The code should be copied into /www, as this is the default directory Nginx and PHP work with in this container.
- Any PHP modules needed in your project should be installed by using apk, Alpine Linux's package manager and the package names for installing can be looked up [here](https://pkgs.alpinelinux.org).

Then there are some tips or rather guidelines that I adhere to personally, but ultimately this is just a matter of taste:
- [S6-overlay can set permissions when the container starts up](https://github.com/just-containers/s6-overlay#fixing-ownership--permissions), but this can be slow if a lot of permissions need to be set, so just do this when building the container.

### Basic example
Now that we know all that, we can do something like this:
```
FROM existenz/webstack:7.0

COPY src/ /www

RUN chown -R php:nginx /www \
    && find /www -type d -exec chmod -R 555 {} \; \
    && find /www -type f -exec chmod -R 444 {} \; \
    && find /www/var -type d -exec chmod -R 755 {} \; \
    && find /www/var -type f -exec chmod -R 644 {} \; \
    && apk -U --no-cache add \
    php7-ctype \
    php7-json \
    php7-mbstring
```
And you should now have a working container that runs your PHP project!

### Overriding or extending the configuration

If you want to augment of replace the configuration of Nginx, PHP or FPM, there are multiple options:
- Place one or more configuration files in specific directories to augment the configuration
- If that does not suit your needs, you can also simply overwrite the configuration files altogether

These are the files to add or overwrite in order to configure the different parts of the webstack:

| Application               | Copy files into this directory | Overwrite this file if needed |
|---------------------------|--------------------------------|-------------------------------|
| PHP core directives (5.6) | /etc/php5/conf.d/              | /etc/php5/php.ini             |
| PHP-FPM (5.6)             | /etc/php5/php-fpm.d/           | /etc/php5/php-fpm.conf        |
| PHP core directives (7.x) | /etc/php7/conf.d/              | /etc/php7/php.ini             |
| PHP-FPM (7.x)             | /etc/php7/php-fpm.d/           | /etc/php7/php-fpm.conf        |
| Nginx                     | /etc/nginx/conf.d/             | /etc/nginx/nginx.conf         |

## Bugs, questions, and improvements

If you found a bug or have a question, please open an issue on the GitHub Issue tracker.
Improvements can be sent by a Pull Request against the master branch and are greatly appreciated!
