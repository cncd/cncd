+++
date = "2017-02-06T22:57:35+07:00"
title = "Pipeline Yaml Specification (Draft)"


[params]
  [[editors]]
    name = "bradrydzewski"
    link = "https://github.com/bradrydzewski"

  [versions]
    current = "https://github.com/cncd/runtime-spec"
    previous = [
      "https://github.com/cncd/runtime-spec/tree/1.0.0",
    ]

  [participate]
    issues = "https://github.com/cncd/runtime-spec/issues"
+++

# Abstract

This specification introduces a YAML format for defining continuous delivery pipelines and their container execution environments. A continuous delivery pipeline is an automated manifestation of your process for getting software from version control to release.

# Introduction

_This section is non-normative._

This specification introduces a YAML format for defining continuous delivery pipelines and their container execution environments. The YAML format described in this specification is a superset of the [docker-compose](https://docs.docker.com/compose/compose-file/) file. It is meant to be human readable and human writable, and should compile to the [intermediate representation]({{< relref "ir.md" >}}) per the specification.

This specification therefore targets both end-users writing pipeline configuration files, and compiler tools that convert the YAML representation to the intermediate representation.

# Conformance

As well as sections marked as non-normative, all authoring guidelines, diagrams, examples, and notes in this specification are non-normative. Everything else in this specification is normative.

The key words "MUST", "MUST NOT", "REQUIRED", "SHALL", "SHALL NOT", "SHOULD", "SHOULD NOT", "RECOMMENDED", "MAY", and "OPTIONAL" in this document are to be interpreted as described in [RFC2119](https://tools.ietf.org/html/rfc2119).

# Types

This section defines custom types and structures used to represent the pipeline configuration.

## The `StringOrSlice` type

The `StringOrSlice` is type alias for a `string[]` that can be represented as a string or a string array.

Example string representation:

```yaml
commands: echo hello
```

Example string array representation:

```yaml
commands:
  - echo hello
  - echo world
```

## The `MapEqualSlice` type

The `MapEqualSlice` is type alias for a map `[string, string]` that can be represented as a map or a string array of items in `KEY=VALUE` format.

Example map representation:

```yaml
environment:
  foo: bar
  baz: qux
```

Example string array representation:

```yaml
environment:
  - foo=bar
  - baz=qux
```

## The `Container` object

The `Container` object defines a container process in the pipeline. The step attributes define how a container is created and started. Each of these attributes are defined below, as well as how their values are processed.

```typescript
interface Container {
  image: string
  pull: boolean
  detached: boolean
  privileged: boolean
  labels: MapEqualSlice
  environment: MapEqualSlice
  entrypoint: StringOrSlice
  command: StringOrSlice
  commands: StringOrSlice
  devices: StringOrSlice
  extra_hosts: StringOrSlice
  dns: StringOrSlice
  dns_search: StringOrSlice
  tmpfs: StringOrSlice
  volumes: StringOrSlice
  networks: Networks
  shm_size: number
  resources: Resources
  auth_config: AuthConfig
  when: ConstraintGroup
}
```

### The `image` attribute

The name of the image to start the container from. This attribute is of type `string` and is required. This can be any valid image name or location.

```yaml
image: redis:latest
image: library/redis:latest
image: index.docker.io/library/redis:latest
```

### The `pull` attribute

Pull the latest version of the image. This attribute is of type `boolean` and is optional.

```
image: golang
pull: true
```

### The `detached` attribute

Start the container and run in the background. This attribute is of type `boolean` and is optional. Note that service containers run in detached mode by default, and will ignore this attribute.

```yaml
pipeline:
  redis:
    image: redis
    detached: true
```

### The `privileged` attribute

Start the container with extended privileges. This attribute is of type `boolean` and is optional.

```yaml
image: golang
privileged: true
```

### The `labels` attribute

todo

### The `environment` attribute

Set environment variables in the container. The attribute is of type `MapEqualSlice` and is optional.

Example map representation:

```yaml
image: golang
environment:
  GOPATH: /go
```

Example slice representation:

```yaml
image: golang
environment:
  - GOPATH=/go
```

### The `entrypoint` attribute

todo

### The `command` attribute

todo

### The `commands` attribute

Execute the commands in the container. This attribute is of type `StringOrSlice` and is optional. This section is generally where you define your build commands (non-normative). Note that specifying a list of commands may override the default `entrypoint` and `command` attributes.

```yaml
commands:
  - go get
  - go install
  - go test -v
```

### The `devices` attribute

Expose devices to a container. For example, a specific block storage device or loop device or audio device can be added to an otherwise unprivileged container. This attribute is of type `StringOrSlice` and is optional.

```yaml
devices:
  - /dev/sdc:/dev/xvdc
```

### The `dns` attribute

Sets the IP addresses as server lines to the container's `/etc/resolv.conf` file. This attribute is of type `StringOrSlice` and is optional.

```yaml
dns:
  - 8.8.8.8
  - 9.9.9.9
```

### The `dns_search` attribute

Sets the domain names that are searched when a bare unqualified hostname is used. This attribute is of type `StringOrSlice` and is optional.

```yaml
dns_search:
  - dc1.example.com
  - dc2.example.com
```

### The `extra_hosts` attribute

Add additional lines to the container's `/etc/hosts` file. This attribute is of type `StringOrSlice` and is optional.

```yaml
extra_hosts:
  - somehost:162.242.195.82
  - otherhost:50.31.209.229
```

### The `shm_size` attribute

Sets the size of `/dev/shm` in bytes. This attribute is of type `int64` and is optional.

```json
shm_size": 64000000
```

### The `tmpfs` attribute

Mounts an empty temporary file system inside of the container. This attribute is of type `StringOrSlice` and is optional.

```yaml
tmpfs:
  - /run
  - /tmp
```

### The `volumes` attribute

Mount paths or named volumes, optionally specifying a path on the host machine. This attribute is of type `StringOrSlice` and is optional.

```yaml
volumes:
  - /tmp/root:/root
```

### The `networks` section

todo

### The `resources` section

todo

### The `auth_config` section

todo

### The `when` section

todo

## The `Constraint` object

todo

```typescript
interface Constraint {
  includes: [string, string]
  excludes: [string, string]
}
```

todo: add description

```yaml
branches: master
```

todo: add description

```yaml
branches: [ master, feature/* ]
```

todo: add description

```yaml
branches:
  includes: master
```

todo: add description

```yaml
branches:
  includes: [ master, feature/* ]
```

todo: add description

```yaml
branches:
  includes: [ master, feature/* ]
  excludes: feature/foo
```

todo: add description

```yaml
branches:
  includes: [ master, feature/* ]
  excludes: [ feature/foo, feature/*/bar ]
```

## The `ConstraintMap` object

todo

```typescript
interface ConstraintMap {
  includes: [string, string]
  excludes: [string, string]
}
```

todo: add description

```
when:
  matrix:
    foo: bar
    baz: qux
```

todo: add description

```
when:
  matrix:
    includes:
      foo: bar
      baz: qux
    excludes:
      qoo: qux
```

## The `ConstraintGroup` object

todo

```typescript
interface ConstraintGroup {
  repo: Constraint
  instance: Constraint
  platform: Constraint
  environment: Constraint
  event: Constraint
  branch: Constraint
  status: Constraint
  matrix: ConstraintMap
}
```

## The `Workspace` object

todo: add description

```typescript
interface Workspace {
  base: string
  path: string
}
```

Example workspace:

```
workspace:
  base: /go
  path: src/github.com/octocat/hello-world
```

### The `base` attribute

todo

### The `path` attribute

todo

## The `Volume` object

todo: add description

```typescript
interface Volume {
  name: string
  driver: string
  driver_opts: [string, string]
}
```

### The `name` attribute

The name of the volume. This value is required and must match `[a-zA-Z0-9_-]`.

### The `driver` attribute

The name of the volume driver. This value is required.

### The `driver_opts` attribute

Additional volume driver options in key value format.

## The `Network` object

todo: add description

```typescript
interface Volume {
  name: string
  driver: string
  driver_opts: [string, string]
}
```

### The `name` attribute

The name of the network. This value is required and must match `[a-zA-Z0-9_-]`.

### The `driver` attribute

The name of the network driver. This value is required.

### The `driver_opts` attribute

Additional network driver options in key value format.

## The `Networks` object

todo: add description

```typescript
interface Network {
  name: string
  aliases: string[]
}
```

## The `Resources` object

The `resources` object is used to reserve resources and apply resource limits to the container.

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

Example resource reservations and limits:

```yaml
resources:
  limits:
    cpus: 1,2
    memory: 2048
  reservations:
    cpus: 1
    memory: 1024
```

## The `AuthConfig` object

todo: add description

```
interface AuthConfig {
  username: string
  password: string
}
```

### The `username` attribute

The username used to authenticate to the remote registry.

### The `password` attribute

The password used to authenticate to the remote registry.

## The `Event` enum

todo: add description

```
pull_request
push
tag
deployment
```

## The `Status` enum

todo: add description

```
success
failure
changed
```

# Format

The pipeline configuration is a YAML document that defines the pipeline execution environment and execution steps. The document is unmarshaled to the following structure:

```typescript
interface Configuration {
  version: string
  platform: string
  branches: Constraint
  workspace: Workspace
  clone: [string, Container]
  pipeline: [string, Container]
  services: [string, Container]
  networks: Network[]
  volumes: Volume[]
  labels: string[]
}
```

## The `version` attribute

The `version` attribute defines the version of the specification used to define the pipeline and to unmarshal the document. This attribute is of type `string` and is optional.

```
version: 1
```

## The `platform` section

The `platform` attribute specifies the target operating system and architecture on which the pipeline must execute. This attribute is of type `string` and is optional. The platform must be one of the following values:

```
linux/amd64
linux/arm
linux/arm64
windows/amd64
windows/arm
windows/arm64
freebsd/amd64
```

## The `clone` section

The `clone` section defines a list of containers executed prior to the pipeline, intended for fetching source from the version control system. This section is of type `[string, Container]` and is optional. The order of the set must be retained, and the keys must be used as the container aliases.

Example clone section overrides default configuration:

```
clone:
  git:
    image: plugins/git
    depth: 1
```

## The `workspace` section

The `workspace` section defines a shared volume and working directory for your pipeline. This section is of type `Workspace` and is optional.

Example workspace configuration:

```
workspace:
  base: /go
  path: src/github.com/octocat/hello-world
```

The `base` attribute defines a volume that is mounted to all steps in your pipeline. This shared volume is what allows your source code and artifacts to persist between steps and containers.

The `path` attribute is relative to the `base` and represents the working directory for all steps in your pipeline. It is also the root of your git repository where your code is cloned.

## The `pipeline` section

todo

## The `services` section

todo

## The `networks` section

todo

## The `volumes` section

todo

## The `branches` section

The `branches` section defines matching criteria for the target branch that must evaluate to true in order for the pipeline to execute. This section is of type `Constraint` and is optional.

Example limits pipeline execution to target branch `master`

```
branches: master
```

Example limits pipeline execution to target branch `master` or `feature/*`

```
branches: [ master, feature/* ]
```

Example limits pipeline execution to target branch `master` using alternate syntax.

```
branches:
  includes: master
```

Example limits pipeline execution to values matching `master` or `feature/*`

```
branches:
  includes: [ master, feature/* ]
```

Example limits pipeline execution to values matching `master` or `feature/*` that do not match the values in the `excludes` section.

```
branches:
  includes: [ master, feature/* ]
  excludes: [ feature/foo, feature/*/bar ]
```

## The `labels` section

The `labels` section provides a list of user-defined key value pairs used to label the pipeline. These labels may be used by the runtime to control how or where containers are executed (non-normative). The section is of type `MapEqualSlice` and is optional.

Example labels defined in map format:

```
labels:
  foo: bar
  baz: qux
```

Example labels defined as a list of items in `KEY=VALUE` format:

```
labels:
  - foo=bar
  - baz=qux
```

# Version

_This section is non-normative_

todo

# Plugins

_This section is non-normative_

todo

# Commands

_This section is non-normative_

The section demonstrates how commands may be executed at runtime in their container environments. These implementation guidelines are optional and are for reference purposes only.

## Linux

_This section is non-normative_

This section demonstrates how the commands section may be converted to a shell script for runtime execution. The following is an example configuration:

```yaml
image: golang
commands:
  - go build
  - go test
```

The commands section may be converted to a Linux shell script:

```text
#!/bin/sh

set -e
go build
go test
```

The shell script can be executed as the container entrypoint and command:

```json
{
  "image": "golang:latest",
  "entrypoint": [ "/bin/bash", "-c" ],
  "command": [ "set -e; go build; go test" ]
}
```

## Windows

_This section is non-normative_

This section demonstrates how the commands section may be converted to a powershell script for runtime execution. The following is an example configuration:

```yaml
image: golang
commands:
  - go build
  - go test
```

The commands section may be converted to a Windows powershell script:

```text
#!/bin/sh

set -e
go build
go test
```

The powershell script can be executed as the container entrypoint and command:

```json
{
  "image": "golang:latest",
  "entrypoint": [ "/bin/bash", "-c" ],
  "command": [ "set -e; go build; go test" ]
}
```

# Substitution

_This section is non-normative_

todo: describe substitution and possible use cases.

```
${VALUE}
```

todo: describe escaping to prevent substitution

```
$${VALUE}
```

# Security

_This section is non-normative._

Compilers should audit the use of privileged features and capabilities because hostile authors could otherwise use these settings to compromise the host machine.

# Examples

_This section is non-normative._

This section shows example features of this specification.

## Example Pipeline

_This section is non-normative._

Example pipeline configured to execute two steps. The first step builds and tests the frontend code, and the second step builds and tests the backend code.

```yaml
pipeline:
  frontend:
    image: node
    commands:
      - npm install
      - npm run tests

  backend:
    image: golang:1.7
    commands:
      - go get
      - go test
```

## Example Pipeline with Services

_This section is non-normative._

Example defines a service container (redis) that is started prior to pipeline execution. Steps in the pipeline are able to access the service container using its alias hostname (redis).

```yaml
pipeline:
  test:
    image: golang:1.7
    commands:
      - go get
      - go test

services:
  redis:
    image: redis:latest
```

## Example Pipeline with Workspace

_This section is non-normative._

Example demonstrates customizing the workspace location. The `/go` directory represents the volume mounted into all pipeline containers, and `src/github.com/foo/bar` is the working directory in which pipeline containers are started.

```yaml
workspace:
  base: /go
  path: src/github.com/foo/bar

pipeline:
  test:
    image: golang:1.7
    commands:
      - go get
      - go test
```

## Example Pipeline with Clone

_This section is non-normative._

Example demonstrates customizing the clone stage. The image is set to `plugins/git` and includes the depth parameter which is passed to the container as environment variable `PLUGINS_DEPTH=50`.

```yaml
clone:
  git:
    image: plugins/git
    depth: 50

pipeline:
  test:
    image: golang:1.7
    commands:
      - go get
      - go test
```

## Example Pipeline limits Branches

_This section is non-normative._

Example limits the target branches for which the pipeline is executed.

```yaml
pipeline:
  test:
    image: golang:1.7
    commands:
      - go get
      - go test

branches:
  include:
    - master
    - feature/*
  exclude:
    - feature/experiment/*
```

# References

## Normative References

YAML SPECIFICATION
: Oren Ben-Kiki, Clark Evans, Ingy döt Net. September 2009. http://yaml.org/spec

## Informative References

DOCKER RUN
: [Docker Run Reference](https://docs.docker.com/engine/reference/run/), Docker Inc.

DOCKER COMPOSE
: [Compose File Reference](https://docs.docker.com/compose/compose-file/), Docker Inc.
