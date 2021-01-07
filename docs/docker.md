# Docker load injector

## Overview

This
[pivotaleducation/loadtest](https://hub.docker.com/repository/docker/pivotaleducation/loadtest)
docker image runs the
[`loadtest` command](https://www.npmjs.com/package/loadtest)
as a single containerized load injector.

It is inspired by the now archived
[bunchjesse/docker-loadtest Github project](https://github.com/bunchjesse/docker-loadtest)
and associated docker image hosted at
[Dockerhub](https://hub.docker.com/r/bunchjesse/loadtest).

The advantages of using the image are that:

1.  You do not explicitly need to install `node` or `npm` dependencies
    to run install or run `loadtest`.

1.  You can run `loadtest` from a workstation via docker,
    or you may deploy it to run in a K8s cluster.

## Prerequisites

You need `docker` installed/configured on the machine from which you
will launch and monitor a load test.

## Configuring it

The image is set to run 10 concurrent users at 10 requests per second
targeting `http://localhost:8080` url for 5 minutes
(after which the load test gracefully terminates).

If you need to override the default configuration
(you will for at least the url),
configure through environment variables as follows:

-   `URL`:
    full qualified URL (including protocol, host and path) for the
    target of your load injector

-   `DURATION`:
    duration of the test in seconds
    (default of 300 seconds, or 5 minutes).
    This provides input parameter for the `loadtest -d` option.

-   `NUM_USERS`:
    number of concurrent users issuing requests.
    This provides input parameter for the `loadtest -c` option.

-   `REQUESTS_PER_SECOND`:
    throughput - target number of requests per second workrate for
    all the concurrent users.
    This provides input parameter for the `loadtest --rps` option.

## Routing load test injector to target endpoint

There are network scenarios to consider:

-   Whether or not the load test injector is not on the same network as
    the target endpoint.

-   Whether or not the load injector can resolve DNS host names of the
    target  endpoint.

### Egress network access

If you are running your injector from Docker,
you should already have *egress access*.

That means your load injector running as a container to talk to:

- Other Docker containers running on same Docker network
- Host level ip addresses on the same machine where your K8s nodes run
- External IP addresses on your local machine, home or office network

### Host name resolution

Even with egress network access enabled.
you still need to consider how name resolution will work with your
load injector running as a container.

Docker will work with public DNS host resolution.

But if the url your load injector container attempts to access is *not*
registered with a "public" DNS,
it will fail to resolve the ip address and result in errors in your load
tests.

In this case you can use the
[--addhost](https://docs.docker.com/engine/reference/commandline/run/)
option to add manual host entries to your load injector container.

## Running it

-   To run interactively:

    `docker run -t -i -e URL=http://yoururl.com pivotaleducation/loadtest`

    Note the `-e` option for environment variables.
    If you need to override the other parameters,
    merely add another `-e` option and the associated values for each.

    If you are running with locally resolved addresses using `/etc/hosts`
    or `dnsmasq`,
    you will need to add host records to your docker run command as
    follows:

    `docker run --add-host <host>:<ip> -t -i -e URL=http://yoururl.com pivotaleducation/loadtest`

-   To run in the background:

    `docker run --name loadtest -d -e URL=http://yoururl.com pivotaleducation/loadtest`

    Follow the same practices as the interactive option if you need to
    add host records.

## Terminating it

1.  To terminate while running interactively.
    (before the 5 minute duration completes),
    just issue a sigquit via `Ctrl+C` command.
    Note that you must have started docker with the `-i` and `-t`
    options and not in the background (`-d` option).

1.  To terminate while running in the background
    (with the `-d` option):

    `docker rm loadtest -f`

## Monitoring

The load test logs all direct to standard out.

If you are running a test interactively,
you will see the logs as the output of the running docker command.

If you are running a test in the background,
run the following:

`docker logs -f loadtest`
