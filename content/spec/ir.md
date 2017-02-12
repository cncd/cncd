+++
date = "2017-02-06T22:57:35+07:00"
title = "Intermediate Representation Specification (Draft)"

[params]
  [[editors]]
    name = "bradrydzewski"
    link = "https://github.com/bradrydzewski"

  [versions]
    current = "https://github.com/cncd/cncd"
    previous = [
      "https://github.com/cncd/cncd/tree/master",
    ]

  [participate]
    issues = "https://github.com/cncd/cncd/issues"
+++

# Abstract

This specification introduces an intermediate representation (IR) for defining continuous delivery pipelines and their container execution environments. A continuous delivery pipeline is an automated manifestation of your process for getting software from version control to release.

# Introduction

_This section is non-normative._

This specification introduces an intermediate representation (IR) for defining continuous delivery pipelines and their container execution environments. The intermediate representation should be machine writable, machine executable, and platform agnostic.

## Frontends

_This section is non-normative._

The intermediate representation is not intended to be written by humans. Instead higher-level file formats are compiled to the intermediate representation. These compilers are known as frontends. Example frontend compilers may include:

* Travis Yaml
* GitLab Yaml
* Bitbucket Pipeline Yaml

The Cloud Native Continuous Delivery working group is also authoring a [specification]({{< relref "yaml.md" >}}) for a YAML representation of a continuous delivery pipeline. This will include a reference implementation for compiling to the intermediate representation.

## Backends

_This section is non-normative._

The intermediate representation should be platform agnostic. This means it can be consumed by different container engines and orchestration platforms. These engines and platforms are known as backends. Example backends may include:

* Docker
* Docker Swarm
* Kubernetes
* Rocket

# Conformance

As well as sections marked as non-normative, all authoring guidelines, diagrams, examples, and notes in this specification are non-normative. Everything else in this specification is normative.

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC2119](https://tools.ietf.org/html/rfc2119).

# Structure

This section defines the structure of the intermediate representation and custom types used to define the pipeline configuration and execution environment.

## The `Config` object

The `Config` is the top-level object used to represent the pipeline.

```typescript
interface Config {
  version: string
  pipeline: Stage[]
  networks: Network[]
  volumes: Volume[]
}
```

## The `Stage` object

The `Stage` object defines a group of steps.

```typescript
interface Stage {
  name: string
  steps: Step[]
}
```

### The `name` attribute

The name of the stage. This attribute is of type string and is required. The name should be globally unique and must match `[a-zA-Z0-9_-]`.

### The `steps` attribute

The `steps` attribute is required and must contain at least one `step`. If the stage contains multiple steps, each step is executed in parallel. The runtime agent should wait until all steps complete prior before it continues to the next stage in the pipeline.

## The `Step` object

The `Step` object defines an individual container process in the pipeline.

```typescript
interface Step {
  name: string
  alias: string
  image: string
  pull: boolean
  detached: boolean
  privileged: boolean
  working_dir: string
  environment: [string, string]
  entrypoint: string[]
  command: string[]
  devices: string[]
  extra_hosts: string[]
  dns: string[]
  dns_search: string[]
  shm_size: number
  tmpfs: string[]
  volumes: string[]
  networks: Conn[]
  auth_config: Auth
  on_failure: boolean
  on_success: boolean
}
```

### The `name` attribute

The name of the container. This attribute is of type `string` and is required. This container name should be globally unique and must match `[a-zA-Z0-9_-]`.

### The `alias` attribute

The name of the container as defined by the user (i.e. user-friendly name). This attribute is of type `string` and is required. The container alias must match `[a-zA-Z0-9_-]`. The alias should be used to create network links, allowing inter-container communication using the alias as the hostname.

### The `image` attribute

The fully qualified image to start the container from. This attribute is of type string and is required. The image name must also include the tag, where applicable.

```
image: redis:latest
image: library/redis:latest
image: index.docker.io/library/redis:latest
```

### The `pull` attribute

Pull the latest version of the image from the remote image registry. This attribute is of type `boolean` and is optional. The default value is false.

### The `detached` attribute

Start the container and send to the background, and proceed to the next step in the pipeline. This attribute is of type `boolean` and is optional. The default value is false.

### The `privileged` attribute

Start the container with extended privileges. This attribute is of type `boolean` and is optional. The default value is false.

### The `working_dir` attribute

Start the container in the specified working directory. This attribute is of type `string` and is optional. Note that the value must be the absolute path of the directory inside the container.

### The `environment` attribute

Set environment variables in the container. This value is of type `[string, string]` and is optional.

```json
{
  "GOPATH": "/go",
  "DOCKER_HOST": "unix:///var/run/docker.sock"
}
```

### The `entrypoint` attribute

Override the default image entrypoint. __TODO__ describe what this means in non-docker terms.

### The `command` attribute

Override the default image command. __TODO__ describe what this means in non-docker terms.

### The `devices` attribute

Expose devices to a container. This value is of type `string[]` and is optional.

```json
[
  "/dev/sdc:/dev/xvdc"
]
```

### The `dns` attribute

Sets the IP addresses added as server lines to the container's `/etc/resolv.conf` file. This value is of type `string[]` and is optional.

```json
[
  "8.8.8.8",
  "9.9.9.9"
]
```

### The `dns_search` attribute

Sets the domain names that are searched when a bare unqualified hostname is used by writing search lines to the container's `/etc/resolv.conf` file. This value is of type `string[]` and is optional.

```json
[
  "dc1.example.com",
  "dc2.example.com"
]
```

### The `extra_hosts` attribute

Adds additional lines to the container's `/etc/hosts` file. This value is of type `string[]` and is optional.

```json
[
  "somehost:162.242.195.82",
  "otherhost:50.31.209.229"
]
```

### The `shm_size` attribute

Sets the size of `/dev/shm` in bytes. This value is of type `int64` and is optional.

```json
{
  "shm_size": 64000000
}
```

### The `tmpfs` attribute

Mount an empty temporary file system inside of the container. This value is of type `string[]` and is optional.

```json
[
  "/run",
  "/tmp"
]
```

### The `volumes` attribute

Mount paths or named volumes, optionally specifying a path on the host machine. This value is of type `string[]` and is optional.

```json
[
  "default:/root",
  "/var/run/docker.sock:/var/run/docker.sock"
]
```

### The `networks` attribute

Connects the container to zero or many networks. This attribute is of type `Conn[]` and is optional.

### The `auth_config` section

Authentication credentials used to download a container image. This attribute is of type `Auth` and is optional.

### The `on_success` attribute

Execute the step when the pipeline state is passing. This attribute is of type `boolean` and is required. If this value is missing the step may be ignored and may not execute.

### The `on_failure` attribute

Execute the step even when the pipeline state is failing. This attribute is of type `boolean` and is optional. If this value is missing the the default value of false is assumed.

This attribute can be used in conjunction with `on_success`. If both attributes are true, the step will always execute regardless of the pipeline state. This may be useful for configuring a notification step that executes both on success and on failure (non-normative).

## The `Auth` object

The `Auth` object defines authentication credentials used to download container images.

```typescript
interface Auth {
  username: string
  password: string
}
```

## The `Conn` object

The `Conn` object defines a container network connection. This information is used to connect a container to a network with optional support for inter-container communication using hostname aliases.

```typescript
interface Conn {
  name: string
  aliases: string[]
}
```

<!-- ### The `name` attribute

The name of the `Network`. This attribute is of type `string` and is required.

### The `alias` attribute

The optional list of hostnames for a container on the network. Containers on the same network can use an alias to connect to the container. -->

## The `Network` object

The `Network` object defines a network interface. Each pipeline can have zero or many networks and use these network to facilitate inter-container communication. Example use cases may include linking a service container (e.g. redis) to the build container for integration testing.

```typescript
interface Network {
  name: string
  driver: string
  driver_opts: [string, string]
}
```

### The `name` attribute

The name of the network. This attribute is of type `string` and is required. The name should be globally unique and must match `[a-zA-Z0-9_-]`.

### The `driver` attribute

The name of the network driver. This attribute is of type `string` and is required.

### The `driver_opts` attribute

Additional network driver options in key value format.

## The `Volume` object

The `Volume` object defines a container volume. Each pipeline can have zero or many volumes and use these volumes to to persist data and share state. Example use cases may include cloning a git repository to a volume so that subsequent containers can access the source.

```typescript
interface Volume {
  name: string
  driver: string
  driver_opts: [string, string]
}
```

### The `name` attribute

The name of the volume. This attribute is of type `string` and is required. The name should be globally unique and must match `[a-zA-Z0-9_-]`.

### The `driver` attribute

The name of the volume driver. This attribute is of type `string` and is required.

### The `driver_opts` attribute

Additional volume driver options in key value format.

# Definition

The intermediate representation is a JSON document that defines the pipeline execution environment and execution steps. The intermediate representation consists of a top-level object of type `Config`.

```json
{
  "version": "1",
  "pipeline": [],
  "networks": [],
  "volumes": []
}
```

## The `version` attribute

The `version` attribute specifies the version of the intermediate representation. This attribute is of type `string` and is optional. When empty the runtime should assume to the latest supported version of the specification.

## The `volumes` section

The `volumes` section defines a list of volumes created at runtime for the pipeline. Individual steps are optionally configured to mount these volumes. Note that volumes are scoped to a single running pipeline, and may not be shared with other running pipelines.

Example volume configuration:

```json
{
  "volumes": [
    {
      "name": "default",
      "driver": "local"
    }
  ]
}
```

Example step configured to mount the default volume:

```json
{
  "stages": [
    {
      "name": "stage_1",
      "steps": [
        {
          "name": "step_1",
          "image": "golang:latest",
          "volumes": [
            "default:/go"
          ]
        }
      ]
    }
  ]
}
```

## The `networks` section

The `networks` section defines a list of networks created at runtime for the pipeline. Individual steps are optionally configured to join these networks. Note that networks are scoped to a single running pipeline, and may not be shared with other running pipelines.

Example bridge network configuration:

```json
{
  "networks": [
    {
      "name": "default",
      "driver": "bridge"
    }
  ]
}
```

Example step configured to connect to the default network:

```json
{
  "stages": [
    {
      "name": "stage_0",
      "steps": [
        {
          "name": "step_0",
          "image": "redis:latest",
          "networks": [
            {
              "name": "default",
              "aliases": [ "redis "]
            }
          ]
        }
      ]
    }
  ]
}
```

Note the above example defines a network alias for the container. Containers on the same network can use the alias as a hostname to connect to the container.

## The `pipeline` section

The `pipeline` section defines a list of stages that are executed sequentially. Each stage contains of a list of one or more steps.

Example pipeline with multiple stages:

```json
{
  "pipeline": [
    {
      "name": "stage_1",
      "steps": []
    }
    {
      "name": "stage_2",
      "steps": []
    }
  ]
}
```

### The `steps` section

The `steps` section defines a list of steps executed in parallel. The runtime must wait until all steps in the current stage are finished before moving to the next stage in the pipeline.

Example stage with multiple steps:

```json
{
  "pipeline": [
    {
      "name": "stage_1",
      "steps": [
        {
          "name": "step_01",
          "image": "golang:latest",
          "entrypoint": [ "/bin/sh" ],
          "command": [ "-c", "set -e; go build; go test"],
          "on_success": true,
          "on_failure": false
        },
        {
          "name": "step_02",
          "image": "node:latest",
          "entrypoint": [ "/bin/sh" ],
          "command": [ "-c", "set -e; npm install; npm run test"],
          "on_success": true,
          "on_failure": false
        }
      ]
    }
  ]
}
```

### The `step` attributes

The `step` object defines an individual container process. The runtime starts the container process and waits for the container to exit. If the container exit code `!= 0` the pipeline is set to a failed state.

Example step:

```json
{
  "pipeline": [
    {
      "name": "stage_1",
      "steps": [
        {
          "name": "step_01",
          "image": "golang:latest",
          "entrypoint": [ "/bin/sh" ],
          "command": [ "-c", "go test"],
          "on_success": true,
          "on_failure": false
        }
      ]
    }
  ]
}
```

Example docker command used to run the step:

```sh
docker run --name "step_01" --entrypoint "/bin/sh" golang:latest "-c" "go test"
```

# Services

__TODO__ describe how to configure services using detached mode and their impact on pipeline state.

# States

__TODO__ describe the pipeline states and their relationship to `on_success` and `on_failure`

## The `on_success` attribute

__TODO__

## The `on_failure` attribute

__TODO__

# Security

_This section is non-normative._

Backends should audit the use of privileged features and capabilities because hostile authors could otherwise use these settings to compromise the host machine.

# Disk Space

_This section is non-normative._

Backends should limit the total amount of space allowed for volume storage, because hostile authors could otherwise use this feature to exhaust the system's available disk space.

Backends should also limit the total amount of space allowed for caching images (e.g. Docker images), and should regularly flush the cache to remove unused or stale images.

# Examples

_This section is non-normative._

This section provides samples of the intermediate representation to highlight various features of this specification.

## Example Pipeline

_This section is non-normative._

In the following example the intermediate representation defines two stages. The first stage clones the github project to a shared volume. The second stage executes the test suite for the project.

```json
{
  "pipeline": [
    {
      "name": "clone_stage",
      "steps": [
        {
          "name": "git_clone_step",
          "image": "git:latest",
          "entrypoint": [
            "/bin/sh",
            "-c"
          ],
          "command": [
            "git clone git://github.com/foo/bar.git /go/src/github.com/foo/bar"
          ],
          "volumes": [
            "default:/go"
          ],
          "on_success": true,
          "on_failure": false
        }
      ],
    },
    {
      "name": "test_stage",
      "steps": [
        {
          "name": "go_test_step",
          "image": "golang:latest",
          "entrypoint": [
            "/bin/sh",
            "-c"
          ],
          "command": [
            "go test -v github.com/foo/bar"
          ],
          "volumes": [
            "default:/go"
          ],
          "on_success": true,
          "on_failure": false
        }
      ],
    }
  ],

  "volumes": [
    {
      "name": "default",
      "driver": "local"
    }
  ]
}
```

## Example Pipeline with Services

_This section is non-normative._

In the following example the intermediate representation defines a service container (redis) that runs in detached mode, which is non-blocking. Subsequent steps in the pipeline are able to access the service container using its alias hostname.

```json
{
  "pipeline": [
    {
      "name": "clone_stage",
      "steps": [
        {
          "name": "clone",
          "image": "git:latest",
          "entrypoint": [
            "/bin/sh",
            "-c"
          ],
          "command": [
            "git clone git://github.com/foo/bar.git /go/src/github.com/foo/bar"
          ],
          "volumes": [
            "default:/go"
          ],
          "on_success": true,
          "on_failure": false
        }
      ],
    },
    {
      "name": "service_stage",
      "steps": [
        {
          "name": "redis_step",
          "alias": "redis",
          "image": "redis:latest",
          "detach": true,
          "networks": [
            {
              "name": "default",
              "aliases": [ "redis" ]
            }
          ],
          "volumes": [
            "default:/go"
          ],
          "on_success": true,
          "on_failure": false
        }
      ],
    },
    {
      "name": "test_stage",
      "steps": [
        {
          "name": "test_step",
          "image": "golang:latest",
          "entrypoint": [
            "/bin/sh",
            "-c"
          ],
          "command": [
            "go test -v github.com/foo/bar"
          ],
          "networks": [
            {
              "name": "default",
              "aliases": [ "redis" ]
            }
          ],
          "volumes": [
            "default:/go"
          ],
          "on_success": true,
          "on_failure": false
        }
      ],
    }
  ],

  "networks": [
    {
      "name": "default",
      "driver": "bridge"
    }
  ],

  "volumes": [
    {
      "name": "default",
      "driver": "local"
    }
  ]
}
```

## Example Pipeline with Parallelism

_This section is non-normative._

In the following example the intermediate representation defines two stages. The first stage clones the github project to a shared volume. The second stage builds and tests the frontend and backend in parallel.

```json
{
  "pipeline": [
    {
      "name": "clone_stage",
      "steps": [
        {
          "name": "git_clone_step",
          "image": "git:latest",
          "working_dir": "/go/src/github.com/foo/bar",
          "entrypoint": [
            "/bin/sh",
            "-c"
          ],
          "command": [
            "git clone git://github.com/foo/bar.git /go/src/github.com/foo/bar"
          ],
          "volumes": [
            "default:/go"
          ],
          "on_success": true,
          "on_failure": false
        }
      ],
    },
    {
      "name": "test_stage",
      "steps": [
        {
          "name": "go_test_step",
          "image": "golang:latest",
          "working_dir": "/go/src/github.com/foo/bar",
          "entrypoint": [
            "/bin/sh",
            "-c"
          ],
          "command": [
            "go test -v; go build"
          ],
          "volumes": [
            "default:/go"
          ],
          "on_success": true,
          "on_failure": false
        },
        {
          "name": "go_test_step",
          "image": "golang:latest",
          "working_dir": "/go/src/github.com/foo/bar",
          "entrypoint": [
            "/bin/sh",
            "-c"
          ],
          "command": [
            "npm install; npm run tests; npm run bundle"
          ],
          "volumes": [
            "default:/go"
          ],
          "on_success": true,
          "on_failure": false
        }
      ],
    }
  ],

  "volumes": [
    {
      "name": "default",
      "driver": "local"
    }
  ]
}
```

# References

## Normative References

JSON SPECIFICATION
: A. Barth. HTTP State Management Mechanism. April 2011. https://tools.ietf.org/html/rfc6265

## Informative References

DOCKER RUN
: [Docker Run Reference](https://docs.docker.com/engine/reference/run/), Docker Inc.

DOCKER COMPOSE
: [Compose File Reference](https://docs.docker.com/compose/compose-file/), Docker Inc.
