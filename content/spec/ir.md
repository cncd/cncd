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
      "https://github.com/cncd/cncd/tree/1.0.0",
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

# Format

The intermediate representation is a JSON document that defines the pipeline execution environment and execution steps. The intermediate representation consists of a top-level object that contains both required and optional members. Each of the members are defined below, as well as how their values are processed.

```typescript
interface Spec {
  pipeline: Stage[]
  networks: Network[]
  volumes: Volume[]
}
```

The `pipeline` attribute must contain a collection of one or more `stage` objects. The stages defined in the pipeline must be executed sequentially in the oder in which they are defined.

The `networks` attribute should contain a collection of one or more `network` objects. Networks may be referenced by individual steps and must be created prior to executing steps where a reference exists.

The `volumes` attribute should contain a collection of one or more `volume` objects. Volumes may be referenced by individual steps and must be created prior to executing steps where a reference exists.

## The `Stage` object

The `stage` object represents a set of processes that are grouped together and executed in parallel. The stage does not complete until all processes have exited.

```typescript
interface Stage {
  name: string
  steps: Step[]
}
```

The `name` attribute is required and must match `[a-zA-Z0-9_-]`

The `steps` attribute is required and must contain at least one `step`. If the stage contains multiple steps, each step is executed in parallel. The runtime agent should wait until all steps complete prior before it continues to the next stage in the pipeline.

## The `Step` object

The `step` object defines a container process in the pipeline. The step attributes define how a container is created and started. Each of these attributes are defined below, as well as how their values are processed.

```typescript
class Step {
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
  networks: Network[]
  resources: Resources
  auth_config: AuthConfig
  on_failure: boolean
  on_success: boolean
}
```

### The `name` attribute

The name of the container. This attribute is of type `string` and is required. This container name should be globally unique and must match `[a-zA-Z0-9_-]`.

### The `alias` attribute

The name of the container as defined by the user (i.e. user-friendly name). This attribute is of type `string` and is required. The container alias must match `[a-zA-Z0-9_-]`.

Note that the alias is used to create network links. You can communicate with containers in your pipeline over the network using the alias as the hostname.

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

todo

### The `command` attribute

todo

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

todo

```typescript
interface Network {
  name: string
  aliases: string[]
}
```

### The `resources` attribute

The `resources` attribute can be used to reserve resources and apply resource limits to the container.

```typescript
interface Resources {
  limits: Limits
  reservations: Limits
}

interface Limits {
  cpus: string
  memory: number
}
```

### The `auth_config` section

todo

```typescript
interface AuthConfig {
  username: string
  password: string
  email: string
}
```

### The `on_success` attribute

Execute the step when the pipeline state is passing. This attribute is of type `boolean` and is required. If this value is missing the step may be ignored and may not execute.

### The `on_failure` attribute

Execute the step even when the pipeline state is failing. This attribute is of type `boolean` and is optional. If this value is missing the the default value of false is assumed.

This attribute can be used in conjunction with `on_success`. If both attributes are true, the step will always execute regardless of the pipeline state. This may be useful for configuring a notification step that executes both on success and on failure (non-normative).

## The `Networks` section

todo: add description

```typescript
interface Network {
  name: string
  driver: string
  driver_opts: [string, string]
}
```

Example default network:

```json
{
  "networks": [
    {
      "name": "default",
      "driver": "local"
    }
  ]
}
```

### The `name` attribute

The name of the network. This attribute is of type `string` and is required. The name should be globally unique and must match `[a-zA-Z0-9_-]`.

### The `driver` attribute

The name of the network driver. This attribute is of type `string` and is required.

### The `driver_opts` attribute

Additional network driver options in key value format.

## The `Volumes` section

todo: add description

```typescript
interface Volume {
  name: string
  driver: string
  driver_opts: [string, string]
}
```

Example default volume:

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

### The `name` attribute

The name of the volume. This attribute is of type `string` and is required. The name should be globally unique and must match `[a-zA-Z0-9_-]`.

### The `driver` attribute

The name of the volume driver. This attribute is of type `string` and is required.

### The `driver_opts` attribute

Additional volume driver options in key value format.

# Security

_This section is non-normative._

Backends should audit the use of privileged features and capabilities because hostile authors could otherwise use these settings to compromise the host machine.

# Disk Space

_This section is non-normative._

Backends should limit the total amount of space allowed for volume storage, because hostile authors could otherwise use this feature to exhaust the system's available disk space.

Backends should also limit the total amount of space allowed for caching images (e.g. Docker images), and should regularly purge the cache to remove unused or stale images.

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
