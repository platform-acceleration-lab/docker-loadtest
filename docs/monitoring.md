# Monitoring a load test

## Overview

## Assess log

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
