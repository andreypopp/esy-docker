# esy-docker

**WARNING:** This is a proof of concept!

A set of make rules to produce docker images for esy projects.

## Usage

Drop `esy-docker.mk` in your project and create a `Makefile` which includes it:

    include esy-docker.mk

    build: esy-docker-build
    shell: esy-docker-shell

Then you can run

    % make build

to build the app within the container and

    % make shell

to run an interactive bash session inside the container.

Note the container produced by `esy-docker.mk` isn't suitable for deployment as
it contains all the build time dependencies (including an entire OCaml toolchain
which is big).

You can refer to `.docker/image.app` file which contains image id of the app
builder container and then copy over needed build artifacts from there:

    FROM <image id from .docker/image.app> as builder

    FROM alpine:3.8

    MKDIR /app
    WORKDIR /app
    COPY --from=builder /app/_esy/default/build/install /app
    RUN /app/bin/myapp.exe

## Further Work

- Configure `esy-docker.mk` so that an app builder image can be tagged:
  ```
  % make TAG=app build
  ```

- Parallelize building dependencies.

  Currently build is generated in such a way that it builds each dependency
  separately using separate `RUN` statement in a Dockerfile. This is done so
  each dependency build is cached in a separate docker layer.

  Instead we can perform an enitire build in parallel in a single container and
  then export builds artifacts from there as `*.tgz`. Then such generated
  Dockerfile can import `*.tgz` instead whcih is much faster.

- Drop NodeJS dependency.

  Currently we use NodeJS script to read a project lockfile and generate a
  Dockerfile. We can use something more widely available.

## Related work

This work was done based on:

- ["Example Dockerfile for static compilation of ReasonML / OCaml projects using esy"][ref1]
  by [@mscharley][]
- ["Example project using esy and reason"][ref2]
  by [@joprice][]

[ref1]: https://gist.github.com/mscharley/b9e7e9d4038938a54278a73ea929f5fc
[ref2]: https://github.com/joprice/reason-esy-example
[@joprice]: https://github.com/joprice
[@mscharley]: https://github.com/mscharley

