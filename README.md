# Docker Utils

> A collection of scripts and example configs for easier development using Docker

![License](https://img.shields.io/github/license/41north/docker-utils?style=flat-square)
![Github Stats](https://img.shields.io/github/stars/41north/docker-utils.svg?style=flat-square)
[![GitHub contributors](https://img.shields.io/github/contributors/41north/docker-utils.svg?style=flat-square)](https://github.com/41north/docker-utils/graphs/contributors/)

## Description

The basic idea is to include this repository as a [git submodule](https://git-scm.com/book/en/v2/Git-Tools-Submodules) to be used as part of 
whatever project you are developing. 

Currently the helper scripts and example configs are geared towards NodeJS based development of APIs and Frontends.

In future we hope to add other platforms.

## Prerequisites

Before continuing you will need to ensure you have the following installed:

* [Docker](https://docs.docker.com/install/)
* [Docker Compose](https://docs.docker.com/compose/install/)
* [Direnv](https://direnv.net/)

*NOTE*: this repository has been developed and tested on Linux. It is not designed for use on Windows, and has not been
tested on Mac OS however the scripts *should* work. Any helping verifying on Mac OS would be much appreciated.

## Installation

Run the following command from the root directory of the target repository to which you wish to add `docker-utils`:

```bash
git submodule add git@github.com:41north/docker-utils.git
```

Next copy `.env.example` and `.envrc.example` into the root directory, removing the `.example` suffixes.

Allow the new `.envrc` file to be executed by running `direnv allow`.

## Usage


  


