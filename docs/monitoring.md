# Monitoring a load test

## Overview

Developers should take the time to explore what happens to their
applications when placed under load.

The scope of this project is to provide a simple tool to exercise an
endpoint of your application.
This is useful for demonstration, education, or light verification
purposes.

It is not intended for use for load testing of applications with a
*workload profile* where the test requires a specific set of operations
stitched together in a test flow.
There are better tools for that,
such as [Gatling](https://gatling.io) or
[Apache Jmeter](jmeter.apache.org).

See the [Limitations and Caveats](#limitations-and-caveats) section for
more details when the tool presented here is not a good fit.

### Monitoring the test

An example test run configured for 20 seconds and the
`http://yourip.com` URL might look like this:

```no-highlight
╰─$ run-load-test -d 20 -u 10 -w 10 -n development http://yourip.com
[Wed Jan  6 19:03:16 UTC 2021]run load test in development namespace with parameters:
  duration: 20
  number of users: 10
  work rate (requests per second: 10
  url: http://yourip.com
Connecting to kubernetes to get node ip address
Starting to serve on 127.0.0.1:9090
node ip address is 10.0.1.150, used for local name resolution
pod/loadtest created
waiting for pod/loadtest to start.
..

[Wed Jan 06 2021 19:03:20 GMT+0000 (Coordinated Universal Time)] INFO Requests: 0, requests per second: 0, mean latency: 0 ms
[Wed Jan 06 2021 19:03:25 GMT+0000 (Coordinated Universal Time)] INFO Requests: 50, requests per second: 10, mean latency: 34.4 ms
[Wed Jan 06 2021 19:03:30 GMT+0000 (Coordinated Universal Time)] INFO Requests: 100, requests per second: 10, mean latency: 32.8 ms
[Wed Jan 06 2021 19:03:35 GMT+0000 (Coordinated Universal Time)] INFO Requests: 150, requests per second: 10, mean latency: 32.3 ms
[Wed Jan 06 2021 19:03:40 GMT+0000 (Coordinated Universal Time)] INFO
[Wed Jan 06 2021 19:03:40 GMT+0000 (Coordinated Universal Time)] INFO Target URL:          http://yourip.com
[Wed Jan 06 2021 19:03:40 GMT+0000 (Coordinated Universal Time)] INFO Max time (s):        20
[Wed Jan 06 2021 19:03:40 GMT+0000 (Coordinated Universal Time)] INFO Concurrency level:   10
[Wed Jan 06 2021 19:03:40 GMT+0000 (Coordinated Universal Time)] INFO Agent:               none
[Wed Jan 06 2021 19:03:40 GMT+0000 (Coordinated Universal Time)] INFO Requests per second: 10
[Wed Jan 06 2021 19:03:40 GMT+0000 (Coordinated Universal Time)] INFO
[Wed Jan 06 2021 19:03:40 GMT+0000 (Coordinated Universal Time)] INFO Completed requests:  200
[Wed Jan 06 2021 19:03:40 GMT+0000 (Coordinated Universal Time)] INFO Total errors:        0
[Wed Jan 06 2021 19:03:40 GMT+0000 (Coordinated Universal Time)] INFO Total time:          20.001398063 s
[Wed Jan 06 2021 19:03:40 GMT+0000 (Coordinated Universal Time)] INFO Requests per second: 10
[Wed Jan 06 2021 19:03:40 GMT+0000 (Coordinated Universal Time)] INFO Mean latency:        33.2 ms
[Wed Jan 06 2021 19:03:40 GMT+0000 (Coordinated Universal Time)] INFO
[Wed Jan 06 2021 19:03:40 GMT+0000 (Coordinated Universal Time)] INFO Percentage of the requests served within a certain time
[Wed Jan 06 2021 19:03:40 GMT+0000 (Coordinated Universal Time)] INFO   50%      32 ms
[Wed Jan 06 2021 19:03:40 GMT+0000 (Coordinated Universal Time)] INFO   90%      35 ms
[Wed Jan 06 2021 19:03:40 GMT+0000 (Coordinated Universal Time)] INFO   95%      36 ms
[Wed Jan 06 2021 19:03:40 GMT+0000 (Coordinated Universal Time)] INFO   99%      48 ms
[Wed Jan 06 2021 19:03:40 GMT+0000 (Coordinated Universal Time)] INFO  100%      97 ms (longest request)
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

### Errors

Monitoring for errors will be critical for monitoring and assessing your
application during the load test

You will get valuable information from the load test logs to show *when*
errors occur *during* the test run,
as well as the impact of errors across the entire load test.

Run the following load test scenario with an invalid URL to induce
failures:

1.  `run-load-test -d 20 -u 10 -w 10 -n development http://dude`

1.  Review the output *during* the test,
    you will see the following pattern:

    ```nohighlight
    [Wed Jan 06 2021 08:25:55 GMT+0000 (Coordinated Universal Time)] INFO Requests: 50, requests per second: 10, mean latency: 3.2 ms
    [Wed Jan 06 2021 08:25:55 GMT+0000 (Coordinated Universal Time)] INFO Errors: 50, accumulated errors: 50, 100% of total requests
    [Wed Jan 06 2021 08:26:00 GMT+0000 (Coordinated Universal Time)] INFO Requests: 100, requests per second: 10, mean latency: 1.5 ms
    [Wed Jan 06 2021 08:26:00 GMT+0000 (Coordinated Universal Time)] INFO Errors: 50, accumulated errors: 100, 100% of total requests
    [Wed Jan 06 2021 08:26:05 GMT+0000 (Coordinated Universal Time)] INFO Requests: 150, requests per second: 10, mean latency: 2.5 ms
    [Wed Jan 06 2021 08:26:05 GMT+0000 (Coordinated Universal Time)] INFO Errors: 50, accumulated errors: 150, 100% of total requests
    ```

1.  Notice that once errors occur in the load test,
    there are *two* lines logged for each monitor sample interval.
    The second line gives you both the number of errors that occurred
    within the interval,
    as well as the accumulated errors.
    and the percentage of errors across the entire load test:

    ```nohighlight
    [Wed Jan 06 2021 08:25:55 GMT+0000 (Coordinated Universal Time)] INFO Errors: 50, accumulated errors: 50, 100% of total requests
    ```

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

This project runs `loadtest` as a single injector process,
and it does not support building a user workflow.

See the associated
["Usage Don'ts" in the loadtest reference](https://www.npmjs.com/package/loadtest#usage-donts).

Consider other tools like
[Gatling](https://gatling.io) or
[Apache Jmeter](https://jmeter.apache.org) for use in real projects.

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
