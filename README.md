# Git Job Docker

[![Build Status][build-status-image]][build-status]
[![Docker Stars][docker-star-image]][docker-hub-url]
[![Docker Pulls][docker-pull-image]][docker-hub-url]
[![GitHub release (latest by date)][latest-release]][repository-url]
[![GitHub][license-image]][repository-url]

Trigger a source code pull with a push event from the git webhook. And then execute the commands.

Download size of this image is:

[![Image Size][docker-image-size]][docker-hub-url]

[Docker hub image: funnyzak/git-job][docker-hub-url]

**Docker Pull Command**: `docker pull funnyzak/git-job:latest`

The nginx service is enabled by default, and the proxy ports are 80 and 9000, default webhook event is `push`. And the webhook url path is `/hooks/git-webhook`, url parameter is `token`. For example:

- `http://hostname:80/hooks/git-webhook?token=HOOK_TOKEN`
- `http://hostname:9000/hooks/git-webhook?token=HOOK_TOKEN`

**Attention:** Current version is not compatible old version, please use tag `1.1.0` if you want to use old version.

## Environment

The following environment variables are used to configure the container:

### Required

The following flags are a list of all the currently supported options that can be changed by passing in the variables to docker with the -e flag.

- `GIT_USER_EMAIL` - The email of the git committer. Required.
- `GIT_REPO_URL` - The remote url of the git repository. Required. Example: `git@github.com:funnyzak/vp-starter.git`.
- `HOOK_TOKEN` - The token of the webhook. Required.

### Optional

The following environment variables are optional:

- `USE_HOOK` - Set to `true` to enable the webhook. Default is `true`.
- `GIT_USER_NAME` - The username of the git. Optional.
- `GIT_BRANCH` - The branch of the git repository to pull. Optional. Default is the repo main branch.
- `STARTUP_COMMANDS` - Optional. A shell command that will be run at the end of the start shell script. left blank, will not execute.
- `BEFORE_PULL_COMMANDS` - Optional. A shell command that will be run before pull code. left blank, will not execute.
- `AFTER_PULL_COMMANDS` - Optional. A shell command that will be run after pull code. left blank, will not execute.
- `CODE_DIR` - The code dir of the git repository. Optional. Default is `/app/code`.
- `TARGET_DIR` - If after pull code, you want to execute build action, you can set the target dir. Optional. Default is `/app/target`.
- `INSTALL_DEPS_COMMAND` - If after pull code, you want to execute install dependencies action, you can set the install command. Optional. For example: `npm install`.
- `BUILD_COMMAND` - If after install deps, you want to execute build action, you can set the build command. Optional. For example: `npm run build`.
- `BUILD_OUTPUT_DIR` - Set the build output dir. Optional. It is relative to `CODE_DIR`. For example: `dist`. Build output dir will rsync to `TARGET_DIR`.
- `AFTER_BUILD_COMMANDS` - If after build, you want to execute other action, you can set the commands. Optional. For example: `npm run deploy`.

### Pushoo

If you want to receive message with pushoo, you need to set `PUSHOO_PUSH_PLATFORMS` and `PUSHOO_PUSH_TOKENS`.

- `SERVER_NAME` - The server name, used for pushoo message. Optional.
- `PUSHOO_PUSH_PLATFORMS` - The push platforms, separated by commas. Optional.
- `PUSHOO_PUSH_TOKENS` - The push tokens, separated by commas. Optional.

For more details, please refer to [pushoo-cli](https://github.com/funnyzak/pushoo-cli).

## Volume

- `/app/code` - Git code folder. Must same as `CODE_DIR`. For example: `./code:/app/code`.
- `/root/.ssh` - Git ssh key folder. For example: `./ssh:/root/.ssh`.
- `/app/target` - The target of the code build. Must same as `TARGET_DIR`. For example: `./target:/app/target`.
- `/app/scripts` - The main scripts folder. contains `hook.sh`, `utils.sh`, `entrypoint.sh`.
- `/custom_scripts/on_startup` - which the scripts are executed at startup. For example: `./scripts/on_startup:/custom_scripts/on_startup`.
- `/custom_scripts/before_pull` - which the scripts are executed at before pull. Same as `/custom_scripts/on_startup`.
- `/custom_scripts/after_pull` - which the scripts are executed at after pull. Same as `/custom_scripts/on_startup`.

## Usage

### Command

Follow the example below to use docker to start the container, you should acdjust the environment variables according to your needs.

```bash
docker run -d -t -i --name git-job --restart on-failure:5 --privileged=true \
-e TZ=Asia/Shanghai \
-e LANG=C.UTF-8 \
-e USE_HOOK=true \
-e GIT_USER_NAME=Leon \
-e GIT_USER_EMAIL=silenceace@gmail.com \
-e GIT_REPO_URL=git@github.com:funnyzak/git-job.git \
-p 81:80 funnyzak/git-job
```

### Compose

#### Full configuration

Follow the example below to use docker-compose to start the container, and the environment variables are fully configured.

 ```yaml
version: '3'
services:
  app:
    image: funnyzak/git-job
    privileged: false
    container_name: gitjob
    working_dir: /app/code
    tty: true
    environment:
      - TZ=Asia/Shanghai
      - LANG=C.UTF-8
      # repo config
      - USE_HOOK=true
      - GIT_USER_NAME=Leon
      - GIT_USER_EMAIL=silenceace@gmail.com
      - HOOK_TOKEN=XqMWRndVuxXQDNzbE9Z
      - GIT_REPO_URL=git@github.com:funnyzak/git-job.git
      - GIT_BRANCH=main
      # commands
      - STARTUP_COMMANDS=echo start time:$$(date)
      - BEFORE_PULL_COMMANDS=echo before pull time:$$(date)
      - AFTER_PULL_COMMANDS=echo after pull time:$$(date)
      # pushoo 
      - SERVER_NAME=demo server
      - PUSHOO_PUSH_PLATFORMS=dingtalk,bark
      - PUSHOO_PUSH_TOKENS=dingtalktoken:barktoken
      - PUSHOO_PUSH_OPTIONS={"dingtalk":{"msgtype":"markdown"}}
      # custom environment for build
      - INSTALL_DEPS_COMMAND=echo install deps time:$$(date)
      - BUILD_COMMAND=mkdir target && zip -r ./target/release.zip ./*
      - BUILD_OUTPUT_DIR=./dist
      - AFTER_BUILD_COMMANDS=echo after build time:$$(date)
      # custom environment for aliyun oss
      - ALIYUN_OSS_ENDPOINT=oss-cn-beijing-internal.aliyuncs.com
      - ALIYUN_OSS_AK_ID=123456789
      - ALIYUN_OSS_AK_SID=sxgh645etrdgfjh4635wer
      # optional
      - CODE_DIR=/app/code
      - TARGET_DIR=/app/target
    restart: on-failure
    ports:
      - 1038:80
    volumes:
      - ./target:/app/target
      - ./ssh:/root/.ssh
      - ./scripts/after_pull/after_pull_build_app.sh:/custom_scripts/after_pull/3.sh
 ```

#### Simple configuration

Follow the example below to use docker-compose to start the container, and the environment variables are not fully configured.

 ```yaml
version: '3'
services:
  app:
    image: funnyzak/git-job
    privileged: false
    container_name: gitjob
    tty: true
    environment:
      - GIT_USER_NAME=Leon
      - GIT_USER_EMAIL=silenceace@gmail.com
      - HOOK_TOKEN=XqMWRndVuxXQDNzbE9Z
      - GIT_REPO_URL=git@github.com:funnyzak/git-job.git
      - GIT_BRANCH=main
      # pushoo 
      - SERVER_NAME=demo server
      - PUSHOO_PUSH_PLATFORMS=dingtalk,bark
      - PUSHOO_PUSH_TOKENS=dingtalk:xxxx,bark:xxxx
      # custom environment for build
      - INSTALL_DEPS_COMMAND=echo install deps time:$$(date)
      - BUILD_COMMAND=mkdir target && zip -r ./target/release.zip ./*
      - BUILD_OUTPUT_DIR=./dist
    restart: on-failure
    ports:
      - 1038:80
    volumes:
      - ./target:/app/target
      - ./ssh:/root/.ssh
      - ./scripts/after_pull/after_pull_build_app.sh:/custom_scripts/after_pull/3.sh
 ```

## Other

### SSH Key

If you want to use ssh-key, you need to mount the ssh-key folder to `/root/.ssh`. Generally, you need to mount the `id_rsa` and `id_rsa.pub` files. For example:

 ```yaml
volumes:
  - ./ssh:/root/.ssh
 ```

 Your can use `ssh-keygen` to generate the ssh-key.For example:

 ```bash
ssh-keygen -t rsa -b 4096 -C "youremail@gmail.com" -N "" -f ./id_rsa
```

## Modules

The following modules are installed in the image.

### Base Module

- **nginx** 1.22
- **git** 2.30.2
- **curl** 7.74.0
- **wget** 1.21
- **nrm** 1.2.5
- **ossutil64** 1.7.14
- **ttf-mscorefonts**
- **go** 1.20
- **java** 1.8.0_292
- **mvn** 3.3.9
- **python** 3.9.2
- **node** 16.19.0
- **npm** 8.19.3
- **yarn** 1.22.19
- **certbot**
- **n** 8.2.0
- **tar** 1.34
- **zip** 10.2.1
- **bash** 5.1.4
- **rsync** 3.2.3
- **gzip** 1.10
- **bzip2** 1.0.8
- **openssl** 1.1.1n
- **tree** 1.8.0
- **crontab** 1.5.2
- **rclone** 1.53.3
- **mysql-client** 10.19
- **[webhook 2.8.0](https://github.com/adnanh/webhook)**

### Other

- **tzdata**
- **gcc**
- **g++**
- **[pushoo-cli](https://github.com/funnyzak/pushoo-cli)**

More details, please refer to [funnyzak/java-nodejs-python-go-etc-docker](https://github.com/funnyzak/java-nodejs-python-go-etc-docker).

## Contribution

If you have any questions or suggestions, please feel free to submit an issue or pull request.

<a href="https://github.com/funnyzak/git-job-docker/graphs/contributors">
  <img src="https://contrib.rocks/image?repo=funnyzak/git-job-docker" />
</a>

## License

MIT License © 2022 [funnyzak](https://github.com/funnyzak)

[build-status-image]: https://github.com/funnyzak/git-job-docker/actions/workflows/build.yml/badge.svg
[build-status]: https://github.com/funnyzak/git-job-docker/actions
[repository-url]: https://github.com/funnyzak/git-job-docker
[license-image]: https://img.shields.io/github/license/funnyzak/git-job-docker?style=flat-square&logo=github&logoColor=white&label=license
[latest-release]: https://img.shields.io/github/v/release/funnyzak/git-job-docker
[docker-star-image]: https://img.shields.io/docker/stars/funnyzak/git-job.svg?style=flat-square
[docker-pull-image]: https://img.shields.io/docker/pulls/funnyzak/git-job.svg?style=flat-square
[docker-image-size]: https://img.shields.io/docker/image-size/funnyzak/git-job
[docker-hub-url]: https://hub.docker.com/r/funnyzak/git-job
