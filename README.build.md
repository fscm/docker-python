# Python for Docker

Docker image with Python.

## Synopsis

This script will create a Docker image with Python installed.

The Docker image resulting from this script should be the one used to run any
of the Python tools.

## Getting Started

There are a couple of things needed for the script to work.

### Prerequisites

Docker, either the Community Edition (CE) or Enterprise Edition (EE), needs to
be installed on your local computer.

#### Docker

Docker installation instructions can be found
[here](https://docs.docker.com/install/).

### Usage

In order to create a Docker image using this Dockerfiles you need to run the
`docker` command with a few options.

```shell
docker image build --force-rm --no-cache --quiet --file <VARIANT>/Dockerfile --tag <USER>/<IMAGE>:<TAG> <PATH>
```

* `<USER>` - *[required]* The user that will own the container image (e.g.: "johndoe").
* `<IMAGE>` - *[required]* The container name (e.g.: "unbound").
* `<TAG>` - *[required]* The container tag (e.g.: "latest").
* `<PATH>` - *[required]* The location of the Dockerfile folder.
* `<VARIANT>` - *[required]* The variant that is being build (`centos` or `debian`).

A build example:

```shell
docker image build --force-rm --no-cache --quiet --file debian/Dockerfile --tag johndoe/my_python:debian .
```

To clean any _`none`_ image(s) left by the build process the following
command can be used:

```shell
docker image rm `docker image ls --filter "dangling=true" --quiet`
```

You can also use the following command to achieve the same result:

```shell
docker image prune -f
```

### Add Tags to the Docker Image

Additional tags can be added to the image using the following command:

```shell
docker image tag <image_id> <user>/<image>:<extra_tag>
```

### Push the image to Docker Hub

After adding an image to Docker, that image can be pushed to a Docker registry... Like Docker Hub.

Make sure that you are logged in to the service.

```shell
docker login
```

When logged in, an image can be pushed using the following command:

```shell
docker image push <user>/<image>:<tag>
```

Extra tags can also be pushed.

```shell
docker image push <user>/<image>:<extra_tag>
```

## Contributing

1. Fork it!
2. Create your feature branch: `git checkout -b my-new-feature`
3. Commit your changes: `git commit -am 'Add some feature'`
4. Push to the branch: `git push origin my-new-feature`
5. Submit a pull request

Please read the [CONTRIBUTING.md](CONTRIBUTING.md) file for more details on how
to contribute to this project.

## Versioning

This project uses [SemVer](http://semver.org/) for versioning. For the versions
available, see the [tags on this repository](https://github.com/fscm/docker-python/tags).

## Authors

* **Frederico Martins** - [fscm](https://github.com/fscm)

See also the list of [contributors](https://github.com/fscm/docker-python/contributors)
who participated in this project.

## License

This project is licensed under the MIT License - see the [LICENSE](LICENSE)
file for details
