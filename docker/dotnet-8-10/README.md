Provide both haxe nightly and dotnet commands. Will need to be updated with
proper haxe 4.2 when released.

## Resources

* Base dotnet image: https://hub.docker.com/_/microsoft-dotnet-runtime/
* Base haxe image: https://github.com/HaxeFoundation/docker-library-haxe/blob/ca98e17b6eb6a696fc31788f43b81dba0effa417/4.1/alpine3.12/Dockerfile

## Rebuild image

```
docker buildx build --platform=linux/amd64 -t registry.gitlab.com/bizouarn/ci/dotnet:8.0-10.0 .
docker push registry.gitlab.com/bizouarn/ci/dotnet:8.0-10.0
```