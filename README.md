# README

## Overview

The scope of this project is to provide a simple load injector that
facilitates demonstration *availability* and *horizontal scaling*
characteristics of PaaS or container orchestrated platforms in
scaled-down development environments where simplicity is paramount.

Authoring of complex load scenario scripts is not desired.
It is not intended for use for load testing of applications with a
*workload profile* where the test requires a specific set of operations
stitched together and correlated in a concurrent test flow.

There are better tools for that,
such as [Gatling](https://gatling.io) or
[Apache Jmeter](https://jmeter.apache.org).

The scope explicitly excludes performance or capacity assessments of an
application.

This
[pivotaleducation/loadtest docker image](https://hub.docker.com/repository/docker/pivotaleducation/loadtest)
runs the
[loadtest](https://www.npmjs.com/package/loadtest)
command as a single containerized load injector.

It is inspired by the now archived
[bunchjesse/docker-loadtest Github project](https://github.com/bunchjesse/docker-loadtest)
and associated
[docker image hosted at Dockerhub](https://hub.docker.com/r/bunchjesse/loadtest/).

The advantages of using this project are that:

-   You do not explicitly need to install node or npm dependencies to
    run install or run `loadtest`.

-   You can run `loadtest` from a workstation via docker,
    or you may deploy it to run in a K8s cluster.

-   Enhanced instructions on assessing `loadtest` log output and
    summary reporting.

## Running a load test scenario

There are 4 minimum steps to run a load test.
You may also need to abort a load test early if you determine it is
not meeting your goals during its runtime.

1.  Identify goals of the test
1.  Configure the application environment
1.  Configure the load test parameters
1.  Start the load test
1.  Monitor the running test environment
1.  Abort the test if monitoring activity demonstrates test is not
    meeting the goals
1.  Assess the results
1.  Adjust the configuration as necessary, and repeat from step 3,
    until assessment yields the desired goal.

### Identify Goals

Identify what you want to demonstrate or verify in the test.

Some examples:

-   Autoscaling behaviors under load -
    verify platform autoscaling configures correct number of instances
    according to its rules against its environment metrics.

-   Availability behaviors during upgrades, or when inducing failures -
    verify platform minimizes failures

### Configure application environment

### Configure load injector

Identify the parameters to set for the load test:

-   Duration needed to run the test -
    should be as short as practical to verify desired behaviors

-   Number of concurrent users -
    should be multi-user to simulate a production concurrency or
    threading model

-   Work-rate -
    simulate a fixed throughput, blocking web requests per second via
    http or https protocols.
    The value should contribute to the goals of the test,
    especially for scaling test relying on http throughput.

-   Target endpoint of the test -
    this is the single URL endpoint you wish the load injector to
    execute.

### Start it

Start the load test according whether or not you are executing on
[Docker](./docs/docker.md#start-it)
or
[K8s](./docs/k8s.md#start-it).

### Monitor it

Monitoring the load test relies on observing the following:

- Load injector metrics/telemetry
- Application and Platform under test

[Assess](#assess) the behaviors *during* the test,
as well as *after* the test.

For this load injector,
monitoring is done via the logs.

See the following how to tail the logs based on your run method:

- [Tail Docker load injector logs](./docs/docker.md#monitor-it)
- [Tail K8s load injector logs](./docs/k8s.md#monitor-it)

If you are monitoring Kubernetes, consider the following tools:

- [Octant](https://octant.dev/)
- `watch kubectl get all --namespace=<target namespace>`
- `kubectl logs <workload type>/<workload resource name>`

If you are monitoring applications running on
[Tanzu Application Services](https://docs.pivotal.io/application-service/2-10/overview/release-notes/runtime-rn.html),
consider the following tools:

- [App Manager](https://docs.pivotal.io/application-service/2-10/console/dev-console.html)
- `watch cf apps`
- `watch cf app <app name>`
- `watch cf events <app name>`
- `cf logs <app name> -f`

### Abort it

Aborting the load injector is simply terminating a single deployment
on your choice of deployment:

- [Aborting a load injector running on Docker](./docs/docker.md#abort-it)
- [Aborting a load injector running on K8s](./docs/k8s.md#abort-it)

### Assess

After either aborting or completion of a load test,
assess the telemetry collected:

- [Load injector telemetry assessment](./docs/monitoring.md#assess-log)
- Application and platform behaviors

Can you verify the behaviors according to the goals of your test?

If not, [abort the test](#abort-it) if still running.
Assess what changes you need to make to the
[test goals](#identify-goals),
[app configuration](#configure-application-environment) or
[load injector configuration](#configure-load-injector), and
[run the test again](#start-it) as necessary.

## Background

### Selection of *loadtest* tool

Tools such as
[*Apache Jmeter*](https://jmeter.apache.org) or
[*Gatling*](https://gatling.io) are great choices for implementing
production-like simulations for load tests.
They allow authoring complex correlated workflows.

The scope of this project is not to test application workflows,
but rather to demonstrate platform behaviors,
without the overhead of authoring workflows.

Tools such as
[Apache HTTP server benchmarking (ab)](https://httpd.apache.org/docs/2.4/programs/ab.html)
tool, or
[Siege](https://www.joedog.org/siege-manual/)
are much simpler and popular tools for unstructured stress testing of
simple web apps.
They have limited-to-no capability to test workflows,
and are not suitable tools for demonstrating scaling, availability
or stability characteristics of applications for a simple reason:

*They do not provide a simple way to regulate the precise throughput or*
*work-rate the load injector applies to an application.*

The [npm loadtest](https://www.npmjs.com/package/loadtest#usage) tool
is selected because it is based on the simplicity of the
*ab* (https://httpd.apache.org/docs/2.4/programs/ab.html)
tool,
but while giving the capability to specify a precise work-rate that is
necessary to demonstrate scaling and availability characteristics of
blocking web applications.

Note that [loadtest](https://www.npmjs.com/package/loadtest)
is not designed for large scale production use.
This project runs `loadtest` as a single injector process,
and it does not support building correlated user workflow test scenarios.

See the associated
["Usage Don'ts" in the loadtest reference](https://www.npmjs.com/package/loadtest#usage-donts).

### Building it

Pehaps you need to tune or tweak the tool if as-is it does not fit your
needs.

Feel free to fork or clone the project,
make the associated change to the `Dockerfile` and rebuild it:

`docker build -t <your customer container repository:tag> .`

## References

View this image on
[Docker Hub](https://hub.docker.com/r/pivotaleducation/loadtest/).

View the project at [Github](https://github.com/platform-acceleration-lab/docker-loadtest).
