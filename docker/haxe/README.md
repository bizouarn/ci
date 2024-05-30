# Custom Haxe build

A custom Docker image for Haxe 4.2.5, based on the official image, with the integration of NodeJs 20.11.1.  
This image is optimized for Haxe JavaScript compilation in CI/CD pipelines and is lighter than the official one (57.34 MB).  

## Image Details

- Haxe Version: 4.2.5
- NodeJs Version: 20.11.1
- Size: 57.34 MB

## Purpose

This custom Docker image is designed specifically for compiling Haxe to JavaScript within CI/CD pipelines, providing a streamlined and efficient build process.

## Rebuild image

```
docker build -t registry.gitlab.com/bizouarn/ci/haxe:4.2.5 .
docker push registry.gitlab.com/bizouarn/ci/haxe:4.2.5
```
