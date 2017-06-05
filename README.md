# Lugdunum's website

This repository contains the jekyll powered website and blog.

# Build

## Local build

```
bundle exec jekyll serve
```

## EIP build

The following comment will create an `eip` directory with the data to send to the svn.
```
bundle exec jekyll build --config _config_eip.yml
```
