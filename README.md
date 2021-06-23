# Python for Docker

A Docker image with Python.

## Supported tags

* `3-centos`, `centos`
* `3-debian`, `3`, `debian`, `latest`

## What is Python?

> Python is an interpreted, interactive, object-oriented programming language. It incorporates modules, exceptions, dynamic typing, very high level dynamic data types, and classes. It supports multiple programming paradigms beyond object-oriented programming, such as procedural and functional programming.

*from* [python.org](https://docs.python.org/3/faq/general.html#what-is-python)

## Getting Started

There are a couple of things needed for the script to work.

### Prerequisites

Docker, either the Community Edition (CE) or Enterprise Edition (EE), needs to
be installed on your local computer.

#### Docker

Docker installation instructions can be found
[here](https://docs.docker.com/install/).

### Usage

To start a container with this image - and have a shell - use the following
command (the container will be deleted after exiting the shell):

```shell
docker container run --rm --interactive --tty fscm/python
```

This will allow you to run any of the Python tools inside this image. To take
the most out of this method you can add your project code to the running
container by defining your project folder as a working folder inside the
container.

To start a container with this image and your project folder available inside
use the following command:

```shell
docker container run --volume LOCAL_PROJECT_PATH:/work:rw --rm --interactive --tty fscm/python
```

## Build

Build instructions can be found
[here](https://github.com/fscm/docker-python/blob/master/README.build.md).

## Versioning

This project uses [SemVer](http://semver.org/) for versioning. For the versions
available, see the [tags on this repository](https://github.com/fscm/docker-python/tags).

## Authors

* **Frederico Martins** - [fscm](https://github.com/fscm)

See also the list of [contributors](https://github.com/fscm/docker-python/contributors)
who participated in this project.
