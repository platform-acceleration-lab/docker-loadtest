# README

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

## How to use it

The image is set to run 10 concurrent users at 10 requests per second
targeting `http://localhost:8080` url for 5 minutes
(after which the load test gracefully terminates).

### Configuring it

If you need to override the defaults
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

### Routing load test injector to target endpoint

There are network scenarios to consider:

- Whether or not the load test injector is not on the same network as the target endpoint.
- Whether or not the load injector can resolve DNS of the target endpoint.

If you are running your injector from Docker,
you should have egress access to external IP addresses.
If the endpoint DNS name is configured through `/etc/hosts`
or `dnsmasq`, you will need to manually add the appropriate host record(s)
to the `docker run` command, via the `--addhost` option.
If the endpoint is resolved through DNS server configured on your
host machines,
docker should resolve it automatically.

If you are running your load injector on K8s and your workloads
under test are co-located in the same K8s cluster,
you can configure your load injector target urls with the service
dns addresses
(either in shortform if colocated in the same namespace,
or long form if across namespaces).

If you are running your load injector on K8s,
and your workloads are external,
verify with your K8s cluster operators that you have egress
access and DNS resolution for the endpoint you wish to test.

If you are pointing your load injector to an internal or private address
that whose IP is accessible,
but not resolvable via DNS,
you can add a
[host entry to your pod via a host alias](https://kubernetes.io/docs/concepts/services-networking/add-entries-to-pod-etc-hosts-with-host-aliases/).

### Running it

-   To run via docker:

    `docker run -t -i -e URL=http://yoururl.com pivotaleducation/loadtest`

    Note the `-e` option for environment variables.
    If you need to override the other parameters,
    merely add another `-e` option and the associated values for each.

    If you are running with locally resolved addresses using `/etc/hosts`
    or `dnsmasq`,
    you will need to add host records to your docker run command as
    follows:

    `docker run --add-host <host>:<ip> -t -i -e URL=http://yoururl.com pivotaleducation/loadtest`

-   To run via K8s *declaratively* as a K8s Job,
    download the
    [`loadtest-job.yaml`](https://raw.githubusercontent.com/platform-acceleration-lab/docker-loadtest/master/loadtest-job.yaml)
    descriptor,
    make the appropriate configuration changes to the name and
    `env` configuration of the accompanying `loadtest-job.yaml`,
    then run the following:

    `kubectl apply -f ./loadtest-job.yaml`

    Note the `env` object in the `loadtest-job.yaml` file to
    list environment variables.
    If you need to add the other parameters,
    merely add another map entry for each.

-   To run via K8s imperatively as a pod from the command line:

    `kubectl run loadtest --image=pivotaledu/loadtest --env="URL=http://yoururl.com"`

    Note the `--env` option for environment variables.
    If you need to override the other parameters,
    merely add via additional `--env` options for each.

-   To see the status of the K8s objects:

    `kubectl get all`

-   Given running under K8s will be not be run in the foreground,
    if you want to tail the logs:

    If you run as a job:

    `kubectl logs job/loadtest -f`

    If you run as a pod:

    `kubectl logs pod/loadtest -f`

### Terminating it

1.  To terminate while running from docker
    (before the 5 minute duration completes),
    just issue a sigquit via `Ctrl+C` command.
    Note that you must have started docker with the `-i` and `-t`
    options and not in the background (`-d` option).

1.  To terminate while running from K8s declaratively
    (before the 5 minute duration completes):

    `kubectl delete -f ./loadtest-job.yaml`

1.  To terminate while running from K8s imperatively
    (before the 5 minute duration completes):

    `kubectl delete pod/<podname>`

## Running with the wrapper scripts in K8s

Operating and monitoring the `pivotaleducation/loadtest` tool from K8s
requires a bit more effort than from Docker,
given more moving parts,
and running in a managed background process.

The following wrapper scripts are provided for you if you elect to run
the load injector as a pod in a Kubernetes cluster:

- `run-load-test`
- `abort-load-test`

The rest of this section documents how to use them.

### Prerequisites

The load injector will run in a K8s cluster,
all you need is `kubectl` and access to the K8s cluster from which the
load injector will run.

### Setup

The only requirement to run and terminiate the load injector are two
bash scripts.

If you are running on an instructor provided remote desktop,
check the accompanying remote desktop guide for the script location.

If you are running on your own machine,
you can access the scripts from the
[load test Github repo](https://github.com/platform-acceleration-lab/docker-loadtest):

- [`run-load-test`](https://raw.githubusercontent.com/platform-acceleration-lab/docker-loadtest/master/run-load-test).
- [`abort-load-test`](https://raw.githubusercontent.com/platform-acceleration-lab/docker-loadtest/master/abort-load-test)

### Running it

There is no explicit setup for the load test tool.
You merely need to run the following script supplied

```bash
run-load-test [-d <duration in seconds>] [-u <number of concurrent users>] [-w <work rate - requests/second] [-n <kubernetes namespace from where to run the load injector] [-h <host name for /etc/hosts> ] [-i <ip address for /etc/hosts> ] [<target url> ]
```

Where:

-   `-d` option specifies the duration of the test specified *in seconds*.
    The default value is 300 seconds, or 5 minutes.
-   `-u` option specifies the number of simulated concurrent users
    issuing requests against your application.
    The default value is 10.
-   `-w` option specifies the number of requests-per-second issued to the
    specified `url` *across all concurrent users*.
    The default is 10 requests/second.
-   `-n` option specifies the target namespace with the load injector will run.
    The default namespace configured is `development`.
-   `h` option is the host name part of the /etc/hosts record to add for the test.
    This is applicable for when the target of your load test is not resolvable via
    DNS.
-   `i` option is the ip address part of the /etc/hosts record to add for the test.
    This is applicable for when the target of your load test is not resolvable via
    DNS.
-   `target url` is the fully qualified http url (including protocol specifier,
    host/port and path)
    against which the load injector will issue requests.
    The default value is `http://localhost:8080`,
    useful for testing the load injector access to an external endpoint.

The defaults should suffice for most of the labs,
and for the labs that need specific tuning,
will call out specific values.

### Monitoring the test

The supplied `run-load-test` script will not only start the load
injector in the target K8s cluster,
but will also tail the logs so you may monitor status of your test.

Once the test is complete (or you which to stop monitoring the test),
merely issue a SIGQUIT to your terminal (via Ctrl+C).

An example test run configured for 20 seconds and the
`http://yourip.com` URL might look like this:

```no-highlight
run load test in development namespace with parameters:
  duration: 20
  number of users: 10
  work rate (requests per second: 10
  url: http://yourip.com
pod/loadtest created
waiting for pod/loadtest to start.
..

[Fri Sep 11 2020 06:17:03 GMT+0000 (Coordinated Universal Time)] INFO Requests: 0, requests per second: 0, mean latency: 0 ms
[Fri Sep 11 2020 06:17:08 GMT+0000 (Coordinated Universal Time)] INFO Requests: 49, requests per second: 10, mean latency: 41.5 ms
[Fri Sep 11 2020 06:17:13 GMT+0000 (Coordinated Universal Time)] INFO Requests: 99, requests per second: 10, mean latency: 37.5 ms
[Fri Sep 11 2020 06:17:18 GMT+0000 (Coordinated Universal Time)] INFO Requests: 149, requests per second: 10, mean latency: 33.7 ms
[Fri Sep 11 2020 06:17:23 GMT+0000 (Coordinated Universal Time)] INFO
[Fri Sep 11 2020 06:17:23 GMT+0000 (Coordinated Universal Time)] INFO Target URL:          http://yourip.com
[Fri Sep 11 2020 06:17:23 GMT+0000 (Coordinated Universal Time)] INFO Max time (s):        20
[Fri Sep 11 2020 06:17:23 GMT+0000 (Coordinated Universal Time)] INFO Concurrency level:   10
[Fri Sep 11 2020 06:17:23 GMT+0000 (Coordinated Universal Time)] INFO Agent:               none
[Fri Sep 11 2020 06:17:23 GMT+0000 (Coordinated Universal Time)] INFO Requests per second: 10
[Fri Sep 11 2020 06:17:23 GMT+0000 (Coordinated Universal Time)] INFO
[Fri Sep 11 2020 06:17:23 GMT+0000 (Coordinated Universal Time)] INFO Completed requests:  198
[Fri Sep 11 2020 06:17:23 GMT+0000 (Coordinated Universal Time)] INFO Total errors:        0
[Fri Sep 11 2020 06:17:23 GMT+0000 (Coordinated Universal Time)] INFO Total time:          20.001319573 s
[Fri Sep 11 2020 06:17:23 GMT+0000 (Coordinated Universal Time)] INFO Requests per second: 10
[Fri Sep 11 2020 06:17:23 GMT+0000 (Coordinated Universal Time)] INFO Mean latency:        37.6 ms
[Fri Sep 11 2020 06:17:23 GMT+0000 (Coordinated Universal Time)] INFO
[Fri Sep 11 2020 06:17:23 GMT+0000 (Coordinated Universal Time)] INFO Percentage of the requests served within a certain time
[Fri Sep 11 2020 06:17:23 GMT+0000 (Coordinated Universal Time)] INFO   50%      30 ms
[Fri Sep 11 2020 06:17:23 GMT+0000 (Coordinated Universal Time)] INFO   90%      55 ms
[Fri Sep 11 2020 06:17:23 GMT+0000 (Coordinated Universal Time)] INFO   95%      84 ms
[Fri Sep 11 2020 06:17:23 GMT+0000 (Coordinated Universal Time)] INFO   99%      127 ms
[Fri Sep 11 2020 06:17:23 GMT+0000 (Coordinated Universal Time)] INFO  100%      139 ms (longest request)
```

Notice the startup phase - this might take up to a minute the first time
you execute it,
as Kubernetes will need to download the accompanying loadtest image
the first time.

Notice the monitoring status during the test,
logging requests errors and latencies on roughly 5 second intervals.

Notice when the test finishes,
it will show you summary stats.

Monitoring will be a useful tool for you when you run the accompanying
labs.

### Aborting a test run

You can exit the script during an existing test run by issuing a
SIGQUIT (via Ctrl+C),
but by doing so you are not terminating the load injector.
It will continue to run on the Kubernetes cluster until it finishes,
or unless you explicitly terminate it.

You can use the `abort-load-test` script to abort the test:

```bash
abort-load-test [-n <kubernetes namespace from where to run the load injector]
```

This will terminate and remove the Kubernetes pod where the load
injector runs.

## Limitations and Caveats

This project is intended for small scale,
educational use.

It is not designed for large scale production use.

This project runs `loadtest` as a single injector process.

See the associated
["Usage Don'ts" in the loadtest reference](https://www.npmjs.com/package/loadtest#usage-donts)

## Building it

Pehaps you need to tune or tweak the tool if as-is it does not fit your
needs.

This is a simple Docker build,
with no other dependencies thay what is in the `Dockerfile`.
Feel free to fork or clone the project,
make the associated change to the `Dockerfile` and rebuild it:

`docker build -t <your customer container repository:tag> .`

## References

View this image on
[Docker Hub](https://hub.docker.com/r/pivotaleducation/loadtest/).

View the project at [Github](https://github.com/platform-acceleration-lab/docker-loadtest).
