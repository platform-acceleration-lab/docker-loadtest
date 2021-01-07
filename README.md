# README

## Overview

Developers should take the time to explore what happens to their
applications and/or the platforms where the applications run when placed
under load:

-   Does it *scale linearly* under load?
-   How does it maintain availability of the overall application when
    the comprised application components fail?
-   What are the performance bottle necks?
-   How much resources does it take when exercised like in production?
-   Under what conditions does the application fail?

The scope of this project is to provide a simple tool that satisfies the
following:

-   Demonstrate *availability* and *horizontal scaling* characteristics
    of PaaS or container orchestrated platforms in scaled-down
    development environments.

-   Authoring of complex load scenario scripts is not necessary.
    It is not intended for use for load testing of applications with a
    *workload profile* where the test requires a specific set of
    operations stitched together and correlated in a concurrent test
    flow.
    There are better tools for that,
    such as [Gatling](https://gatling.io) or
    [Apache Jmeter](jmeter.apache.org).

The scope explicitly excludes performance or capacity assessments of an
application.

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

- [Load injector telemetry assessment](./docs/monitoring.md)
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

Authoring load test scripts is a tedious task.

Tools such as *Apache Jmeter* or *Gatling* require authoring a test
script.

Tools such as
[Apache HTTP server benchmarking (ab)](https://httpd.apache.org/docs/2.4/programs/ab.html)
tool, or
[Siege](https://www.joedog.org/siege-manual/)
are much simpler and popular tools for unstructured stress testing of
simple web apps.
But they are terrible tools for demonstrating scaling, availability or
stability characteristics of applications for a simple reason:

*They do not provide a simple way to regulate the precise throughput or*
*work-rate of an application.*

The [npm loadtest](https://www.npmjs.com/package/loadtest#usage) tool
is selected because it is based on the simplicity of the *ab* tool,
but while giving the capability to specify a precise work-rate that is
necessary to demonstrate scaling and availability characteristics of
blocking web applications.

Note that it is not designed for large scale production use.
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
