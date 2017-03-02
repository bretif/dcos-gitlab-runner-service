# dcos-gitlab-runner-service

A customized Docker image for running scalable GitLab CI runners on DC/OS via Marathon.

This is a fork of orginal  [mesosphere/dcos-gitlab-runner-service](https://github.com/mesosphere/dcos-gitlab-runner-service)
In this image we do specify directly **CI_SERVER_URL** to register to gitlab

## Configuration

The GitLab runner can be configured by environment variables. For a complete overview, have a look at the [docs/gitlab_runner_register_arguments.md](docs/gitlab_runner_register_arguments.md) file.

The most important ones are:

* `CI_SERVER_URL`: URL of your gitlab CI server For instance https://gitlab.domain.com/ci. **(mandatory)**
* `REGISTRATION_TOKEN`: The registration token to use with the GitLab instance. See the [docs](https://docs.gitlab.com/ce/ci/runners/README.html) for details. **(mandatory)**
* `RUNNER_EXECUTOR`: The type of the executor to use, e.g. `shell` or `docker`. See the [executor docs](https://github.com/ayufan/gitlab-ci-multi-runner/blob/master/docs/executors/README.md) for more details. **(mandatory)**
* `RUNNER_CONCURRENT_BUILDS`: The number of concurrent builds this runner should be able to handel. Default is `1`.
* `RUNNER_TAG_LIST`: If you want to use tags in you `.gitlab-ci.yml`, then you need to specify the comma-separated list of tags. This is useful to distinguish the runner types.

## Run on DC/OS

This version of the GitLab CI runner for Marathon project uses Docker-in-Docker techniques, with all of its pros and cons. See also [jpetazzo's article](http://jpetazzo.github.io/2015/09/03/do-not-use-docker-in-docker-for-ci/) on this topic.

### Shell runner

An example for a shell runner. This enables the build of Docker images.

```javascript
{
  "id": "gitlab-runner-shell",
  "container": {
    "type": "DOCKER",
    "docker": {
      "image": "mesosphere/dcos-gitlab-runner-service:v1.10",
      "network": "HOST",
      "forcePullImage": true,
      "privileged": true
    }
  },
  "instances": 1,
  "cpus": 1,
  "mem": 2048,
  "env": {
    "CI_SERVER_URL": "https://gitlab.marathon.mesos/ci",
    "REGISTRATION_TOKEN": "zzNWmRE--SBfeMfiKCMh",
    "RUNNER_EXECUTOR": "shell",
    "RUNNER_TAG_LIST": "shell,build-as-docker",
    "RUNNER_CONCURRENT_BUILDS": "4"
  }
}
``` 

### Docker runner

Here's an example for a Docker runner, which enables builds *inside* Docker containers:

```javascript
{
  "id": "gitlab-runner-docker",
  "container": {
    "type": "DOCKER",
    "docker": {
      "image": "mesosphere/dcos-gitlab-runner-service:v1.10",
      "network": "HOST",
      "forcePullImage": true,
      "privileged": true
    }
  },
  "instances": 1,
  "cpus": 1,
  "mem": 2048,
  "env": {
    "CI_SERVER_URL": "https://gitlab.marathon.mesos/ci",
    "REGISTRATION_TOKEN": "zzNWmRE--SBfeMfiKCMh",
    "RUNNER_EXECUTOR": "docker",
    "RUNNER_TAG_LIST": "docker,build-in-docker",
    "RUNNER_CONCURRENT_BUILDS": "4",
    "DOCKER_IMAGE": "node:6-wheezy"
  }
}
```

Make sure you choose a useful default Docker image via `DOCKER_IMAGE`, for example if you want to build Node.js projects, the `node:6-wheezy` image. This can be overwritten with the `image` property in the `.gitlab-ci.yml` file (see the [GitLab CI docs](https://docs.gitlab.com/ce/ci/yaml/README.html).

## Usage in GitLab CI

### Builds as Docker

An `.gitlab-ci.yml` example of using the `build-as-docker` tag to trigger a build on the runner(s) with shell executors:

```yaml
stages:
  - ci

build-job:
  stage: ci
  tags:
    - build-as-docker
  script:
    - docker build -t tobilg/test .
```

This assumes your project has a `Dockerfile`, for example

```
FROM nginx
```

### Builds in Docker

An `.gitlab-ci.yml` example of using the `build-in-docker` tag to trigger a build on the runner(s) with Docker executors:

```yaml
image: node:6-wheezy

stages:
  - ci

test-job:
  stage: ci
  tags:
    - build-in-docker
  script:
    - node --version
```
