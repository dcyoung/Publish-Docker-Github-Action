# Publishes docker containers
[![Actions Status](https://github.com/elgohr/Publish-Docker-Github-Action/workflows/Test/badge.svg)](https://github.com/elgohr/Publish-Docker-Github-Action/actions)
[![Actions Status](https://github.com/elgohr/Publish-Docker-Github-Action/workflows/Integration%20Test/badge.svg)](https://github.com/elgohr/Publish-Docker-Github-Action/actions)
[![Actions Status](https://github.com/elgohr/Publish-Docker-Github-Action/workflows/Integration%20Test%20Github/badge.svg)](https://github.com/elgohr/Publish-Docker-Github-Action/actions)

This Action for [Docker](https://www.docker.com/) uses the Git branch as the [Docker tag](https://docs.docker.com/engine/reference/commandline/tag/) for building and pushing the container.
Hereby the master-branch is published as the latest-tag.

## Usage

## Example pipeline
```yaml
name: Publish Docker
on: [push]
jobs:
  build:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@master
    - name: Publish to Registry
      uses: elgohr/Publish-Docker-Github-Action@master
      with:
        name: myDocker/repository
        username: ${{ secrets.DOCKER_USERNAME }}
        password: ${{ secrets.DOCKER_PASSWORD }}
```

## Mandatory Arguments

`name` is the name of the image you would like to push  
`username` the login username for the registry  
`password` the login password for the registry  

If you would like to publish the image to other registries, these actions might be helpful  

| Registry                                             | Action                                        |
|------------------------------------------------------|-----------------------------------------------|
| Amazon Webservices Elastic Container Registry (ECR)  | https://github.com/elgohr/ecr-login-action    |
| Google Cloud Container Registry                      | https://github.com/elgohr/gcloud-login-action |

## Outputs

`tag` is the tag, which was pushed  
`snapshot-tag` is the tag that is generated by the [snapshot-option](https://github.com/elgohr/Publish-Docker-Github-Action#snapshot) and pushed  
`digest` is the digest of the image, which was pushed  

## Optional Arguments

### registry
Use `registry` for pushing to a custom registry.
> NOTE: GitHub's Docker registry uses a different path format to Docker Hub, as shown below. See [Configuring Docker for use with GitHub Package Registry](https://help.github.com/en/github/managing-packages-with-github-package-registry/configuring-docker-for-use-with-github-package-registry#publishing-a-package) for more information.

```yaml
with:
  name: owner/repository/image
  username: ${{ secrets.DOCKER_USERNAME }}
  password: ${{ secrets.DOCKER_PASSWORD }}
  registry: docker.pkg.github.com
```

### snapshot
Use `snapshot` to push an additional image, which is tagged with  
`{YEAR}{MONTH}{DAY}{HOUR}{MINUTE}{SECOND}{first 6 digits of the git sha}`.  
The date was inserted to prevent new builds with external dependencies override older builds with the same sha.
When you would like to think about versioning images, this might be useful.  

```yaml
with:
  name: myDocker/repository
  username: ${{ secrets.DOCKER_USERNAME }}
  password: ${{ secrets.DOCKER_PASSWORD }}
  snapshot: true
```

### dockerfile
Use `dockerfile` when you would like to explicitly build a Dockerfile.  
This might be useful when you have multiple DockerImages.  

```yaml
with:
  name: myDocker/repository
  username: ${{ secrets.DOCKER_USERNAME }}
  password: ${{ secrets.DOCKER_PASSWORD }}
  dockerfile: MyDockerFileName
```

### shmsize
Use `shmsize` when you would like to explicitly build with a custom shm size.  

```yaml
with:
  name: myDocker/repository
  username: ${{ secrets.DOCKER_USERNAME }}
  password: ${{ secrets.DOCKER_PASSWORD }}
  shmsize: 256m
```

### workdir
Use `workdir` when you would like to change the directory for building.

```yaml
with:
  name: myDocker/repository
  username: ${{ secrets.DOCKER_USERNAME }}
  password: ${{ secrets.DOCKER_PASSWORD }}
  workdir: mySubDirectory
```

### context
Use `context` when you would like to change the Docker build context.

```yaml
with:
  name: myDocker/repository
  username: ${{ secrets.DOCKER_USERNAME }}
  password: ${{ secrets.DOCKER_PASSWORD }}
  context: myContextDirectory
```

### buildargs
Use `buildargs` when you want to pass a list of environment variables as [build-args](https://docs.docker.com/engine/reference/commandline/build/#set-build-time-variables---build-arg). Identifiers are separated by comma.   
All `buildargs` will be masked, so that they don't appear in the logs.  

```yaml
- name: Publish to Registry
  uses: elgohr/Publish-Docker-Github-Action@master
  env:
    MY_FIRST: variableContent
    MY_SECOND: variableContent
  with:
    name: myDocker/repository
    username: ${{ secrets.DOCKER_USERNAME }}
    password: ${{ secrets.DOCKER_PASSWORD }}
    buildargs: MY_FIRST,MY_SECOND
```

### cache
Use `cache` when you have big images, that you would only like to build partially (changed layers).  
> CAUTION: Docker builds will cache non-repoducable commands, such as installing packages. If you use this option, your packages will never update. To avoid this, run this action on a schedule with caching **disabled** to rebuild the cache periodically.

```yaml
name: Publish to Registry
on:
  push:
    branches:
      - master
  schedule:
    - cron: '0 2 * * 0' # Weekly on Sundays at 02:00
jobs:
  update:
    runs-on: ubuntu-latest
    steps:
    - uses: actions/checkout@master
    - name: Publish to Registry
      uses: elgohr/Publish-Docker-Github-Action@master
      with:
        name: myDocker/repository
        username: ${{ secrets.DOCKER_USERNAME }}
        password: ${{ secrets.DOCKER_PASSWORD }}
        cache: ${{ github.event_name != 'schedule' }}
```

### tag_names
Use `tag_names` when you want to push tags/release by their git name (e.g. `refs/tags/MY_TAG_NAME`).  
> CAUTION: Images produced by this feature can be override by branches with the same name - without a way to restore.

```yaml
with:
  name: myDocker/repository
  username: ${{ secrets.DOCKER_USERNAME }}
  password: ${{ secrets.DOCKER_PASSWORD }}
  tag_names: true
```
