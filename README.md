# docker-php73-fpm

A PHP 7.3 based Docker base image.

## Pulling the image

```
docker pull ilyes512/php73-fpm:latest
```

## Building the docker image(s)

There are 2 targets at the moment:
  - **runtime**: this is for *production*. It does not contain any development tools like composer and xdebug.
  - **builder**: this is for *development*. This is based on the runtime-target and it adds xdebug, composer etc.

Building runtime-target:

```
docker build --tag ilyes512/php73-fpm:fromsource --target runtime .
```

Building builder-target:

```
docker build --tag ilyes512/php73-fpm/builder:fromsource --target builder .
```
