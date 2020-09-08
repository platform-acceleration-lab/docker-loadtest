# README

This project builds a docker image to run the
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
targeting `localhost` url for 5 minutes
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

### Running it

-   To run via docker:

    `docker run -t -i -e URL=http://yoururl.com pivotaleducation/loadtest`

    Note the `-e` option for environment variables.
    If you need to override the other parameters,
    merely add another `-e` option and the associated values for each.

-   To run via K8s *declaratively* as a K8s Job,
    make the appropriate configuration changes to the name and
    `env` configuration of the accompanying `loadtest-job.yaml`,
    then run the following:

    `kubectl apply -f ./loadtest-job.yaml`

    Note the `env` object in the `load-test-pod.yaml` file to
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

    `kubectl delete -f ./load-test-pod.yaml`

1.  To terminate while running from K8s imperatively
    (before the 5 minute duration completes):

    `kubectl delete pod/<podname>`

## Limitations and Caveats

This project is not intended for small scale,
educational use.

It is not designed for large scale production use.

This project runs `loadtest` as a single injector process.

See the associated
["Usage Don'ts" in the loadtest reference](https://www.npmjs.com/package/loadtest#usage-donts)

## References

View this image on
[Docker Hub](https://hub.docker.com/r/pivotaleducation/loadtest/).

View the project at [Github](https://github.com/platform-acceleration-lab/docker-loadtest).