# Docker Utils

> A collection of scripts and example configs for easier development using Docker

![License](https://img.shields.io/github/license/41north/docker-utils?style=flat-square)
![Github Stars](https://img.shields.io/github/stars/41north/docker-utils.svg?style=flat-square)
[![GitHub contributors](https://img.shields.io/github/contributors/41north/docker-utils.svg?style=flat-square)](https://github.com/41north/docker-utils/graphs/contributors/)

## Description

This repository is designed to be included as a [git submodule](https://git-scm.com/book/en/v2/Git-Tools-Submodules) as part of 
whatever project you are developing. 

Currently the helper scripts and example configs are geared towards [NodeJS](https://nodejs.org/en/) based development of APIs 
and Frontends. In future we hope to add other platforms.

An example project can be found [here](https://github.com/41north/docker-utils-example) which demonstrates how to use docker-utils
when developing a full-stack node app.

## Prerequisites

Before continuing you will need to ensure you have the following installed:

* [Docker](https://docs.docker.com/install/)
* [Docker Compose](https://docs.docker.com/compose/install/)
* [Direnv](https://direnv.net/)

**NOTE**: this repository has been developed and tested on Linux. It is not designed for use on Windows, and has not been
tested on Mac OS however the scripts *should* work. Any helping verifying on Mac OS would be much appreciated.

## Installation

Run the following command from the root directory of the target repository to which you wish to add `docker-utils`:

```shell
git submodule add git@github.com:41north/docker-utils.git
```

Next copy `.env.example` and `.envrc.example` into the root directory, removing the `.example` suffixes.

Allow the new `.envrc` file to be executed by running `direnv allow`. You should see the following output:

```shell
╭─brian@alpha ~/Development/41north 
╰─$ cd docker-utils-example
direnv: loading .envrc                                                                                                                                                                                                                 
direnv: export +BASE_DOMAIN +ENV_FILE +NODE_VERSION +POSTGRES_DB +POSTGRES_PASSWORD +POSTGRES_USER +POSTGRES_VERSION +REPO_ROOT_DIR ~PATH
╭─brian@alpha ~/Development/41north/docker-utils-example ‹master› 
╰─$
```
 
## Usage

This repository is designed to help in two main areas:

* configuring services via docker compose for frontend or API development
  
* executing every day commands such as `yarn` or `npm` in a fashion consistent with the docker compose setup or in a 
standalone fashion without requiring the installation of the aforementioned tools

### Docker Compose

Included within the `docker/compose` directory are a series of snippets which can be copied into the `docker-compose.yml` file of any 
target repository. Currently there are examples for:

* [Traefik](https://containo.us/traefik/)
* [PostgreSQL](https://www.postgresql.org/)
* [MariaDB](https://mariadb.org/)
* [Mailhog](https://github.com/mailhog/MailHog)
* [Gatsby](https://www.gatsbyjs.org/)
* [Angular](https://angular.io/)
* General Node-based API development
  
When copying the snippets be sure to observe the different sections such as `network`, `volumes` and `services` and be sure
to merge each individually into your `docker-compose.yml` file.

### Bin scripts

There are a series of helper bin scripts which, if you have copied the `.envrc` file correctly and allowed it to execute with `direnv`, will be 
automatically included in your path when you cd into your project directory:

* **sudo** simply ensures that if you require `sudo` to execute docker commands that the user environment is preserved.
* **env** is a helper script which determines the correct uid and gid to ensure that proper file permissions are used when mapping volumes into the docker containers.
* **find_up** is another helper script used to find the nearest parent directory containing a `package.json`. 
* **docker-node** is the main helper script for executing `node` commands. It will be explained in greater detail a bit later on.
* **node**, **npm**, **yarn**, **npx** are alias scripts that use `docker-node` and are designed as replacements for the normal commands you would install.
* **docker-compose** is a simple wrapper for `docker-compose` that ensures the correct file permissions are used by first invoking `env` 
 
### docker-node

It is worth explaining this script in more detail as it forms the underlying basis for all the NodeJS related scripts.

```bash
#!/usr/bin/env bash

SCRIPT_DIR="$( cd "$( dirname "${BASH_SOURCE[0]}" )" && pwd )"

"${SCRIPT_DIR}"/env

VERSION=${NODE_VERSION:-13}

PACKAGE_JSON=$(find_up package.json)

if [ -z "${PACKAGE_JSON}" ]; then
  echo "Could not find a package.json file in the current or parent directories"
  exit 1
fi

MOUNT_DIR="$(dirname "$(realpath "${PACKAGE_JSON}")")"
WORK_DIR="/usr/src/app/$(realpath --relative-to="${MOUNT_DIR}" `pwd`)"

docker run \
  --network host \
  --interactive \
  --tty \
  --rm \
  --user "${USER_ID}":"${GROUP_ID}" \
  --mount type=bind,source="${MOUNT_DIR}",target=/usr/src/app \
  --workdir "${WORK_DIR}" \
  --env-file "${ENV_FILE}" \
  node:"${VERSION}" "$@"
```

To begin it executes the `env` script to determine the correct `USER_ID` and `GROUP_ID` regardless of whether not this script is being executed
with `sudo`.

Next it looks for the nearest parent directory (inclusive of the current one) with a package.json. This directory will form the mount directory of the bind mount specified in `--mount type=bind,source="${MOUNT_DIR}",target=/usr/src/app`

Finally it executes the node command with the following parameters:

* `--network host` to ensure any ports within the docker container are bound to the same ports on the local host.
* `--interactive` to allow for input to the executing command.
* `--tty` hooks up STDIN and STDOUT.
* `--rm` will ensure the container is automatically removed once this command finishes executing.
* `--user "${USER_ID}":"${GROUP_ID}"` ensure the command executing within the container is using the same user and group permissions as the user outside the container to prevent file permissioning issues.
* `--mount type=bind,source="${MOUNT_DIR}",target=/usr/src/app` creates a bind mount between the nearest parent directory with a `package.json` and `/usr/src/app` within the container.
* `--workdir "${WORK_DIR}"` sets the current working directory within the container to a path relative to `/usr/src/app` consistent with the relative path to the nearest parent directory with a `package.json` file outside the container.
* `--env-file "${ENV_FILE}"` pass in the `.env` file from the root of the repository to ensure a consistent environment with any `docker-compose.yml` setup.
* `node:"${VERSION}"` specifies the node image to use. The version is set in the `.env` file, defaulting to `13` if not set.
* `"$@"` is a pass through of the arguments passed to the script.