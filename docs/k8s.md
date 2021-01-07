# K8s load injector

## Overview

You can run the `pivotaleducation/loadtest` container image under K8s.
A scenario might be that you cannot run Docker from development
workstation,
or you want to might want to leverage K8s orchestration
(perhaps through Jobs).

This article describes considerations for running the `loadtest`
container under K8s,
as well as the helper scripts.

## Routing load test injector to target endpoint

There are network scenarios to consider:

-   Whether or not you are running developer grade K8s
    (such as minikube or microk8s),
    or managed/provided K8s clusters with network policies enabled.

-   Whether or not the load test injector is not on the same network as
    the target endpoint.

-   Whether or not the load injector can resolve DNS host names of the
    target endpoint.

The accompanying instructions here assume you are running a vanilla
form of K8s.
If you are not,
consult with your K8s cluster operator to help you out with the
networking aspects outlined in later sections.

### Egress network access

If you are running on vanilla K8s for development use,
such as *minikube* or *microk8s*,
you already have *egress* network traffic enabled.

That means your load injector running as a K8s pod can talk to:

- Other pods running on the K8s cluster
- Host level ip addresses on the same machine where your K8s nodes run
- External IP addresses on your local machine, home or office network

### Host name resolution

Even with egress network access enabled.
you still need to consider how name resolution will work with your
load injector running as a K8s pod.

By default,
containers running in K8s hosted pods will leverage the K8s cluster DNS
server,
which also resolves "public" DNS addresses.

If the url your load injector pod attempts to access is *not* registered
with either K8s DNS or a "public" DNS,
it will fail to resolve the ip address and result in errors in your load
tests.

If you know the ip address of the hosts you wish to target with your
load injector,
you can add a
[host entry to your pod via a host alias](https://kubernetes.io/docs/concepts/services-networking/add-entries-to-pod-etc-hosts-with-host-aliases/).

The accompanying `run-load-test` bash script handles this scenario for
you.

## Prerequisites

The load injector will run in a K8s cluster,
you need the following installed/configured on the machine from which
you will launch and monitor a load test:

- `kubectl` cli
- `kubeconfig` configuration (~/.kube/config)
- `curl` cli

## Setup

Operating and monitoring the `pivotaleducation/loadtest` tool from K8s
requires a bit more effort than from Docker,
given more moving parts,
and running in a managed background process.

The only requirement to run and terminate the load injector are two
bash scripts.

If you are running on your own machine,
you can access the scripts from the
[load test Github repo](https://github.com/platform-acceleration-lab/docker-loadtest/scripts):

- [`run-load-test`](https://raw.githubusercontent.com/platform-acceleration-lab/docker-loadtest/master/run-load-test).
- [`abort-load-test`](https://raw.githubusercontent.com/platform-acceleration-lab/docker-loadtest/master/abort-load-test)

## Run a test

The `run-load-test` script is self documented,
run the help option as follows:

```bash
run-load-test -h
```

Examples of the most common scenarios:

### Publicly accessible url

This scenario is most common if running the load injector and the
target application on separate platforms:

`run-load-test -d 20 -u 10 -w 10 -n development http://yourip.com`

### K8s service

This scenario is most common if you are running your load test
on minikube or microk8s in a dev or testing environment,
and you do not have an ingress
(the load injector will connect directly to the pod(s) of a named
K8s service):

`run-load-test -d 20 -u 10 -w 10 -n development http://tracker`

### Test K8s ingress route without dns resolution

This scenario is most common if you are running your load test on
minikube or microk8s in a dev or testing environment:

`run-load-test -d 20 -u 10 -w 10 -n development -l local http://tracker.k8s.local`

### Running manually

If you cannot or do not want to run via the `run-load-test` script,
you can run it via `kubectl` directly in either an imperative mode,
or the declarative mode,
and manually specifying the `loadtest`
[container required environment variables](./docker.md#configuring-it).

To run imperatively:

`kubectl run loadtest --image=pivotaleducation/loadtest:latest --namespace=development --env="URL=http://yoururl.com" --env="DURATION=20" --env="NUM_USERS=10" --env="REQUESTS_PER_SECOND=10"`

*Note:*
*You cannot add the `hostAlias` records via the imperative method.*

To run declaratively:

1.  Edit the sample `loadtest.yaml` file and fill in the approprate values:

    ```yaml
    ...
            env:
        - name: URL
        value: "${url}"
        - name: DURATION
        value: "${duration}"
        - name: NUM_USERS
        value: "${users}"
        - name: REQUESTS_PER_SECOND
        value: "${workRate}"
    restartPolicy: Never
    hostAliases:
    - ip: "${nodeIpAddress}"
      hostnames:
      - "${hostName}"
    ...
    ```

1.  Remove the `hostAliases` section if you have dns resolution for your
    target URL,
    either publicly or through K8s dns server.

1.  If the target URL is an ingress whose route is local to the K8s
    cluster where your load injector runs (for example, minikube or
    microk8s),
    resolve the K8s node ip adress as your `${nodeIpAddress}`:

    -   Get the node list:

        `kubectl get nodes`

        example output:

        ```nohighlight
        NAME        STATUS   ROLES    AGE   VERSION
        minikube    Ready    <none>   24d   v1.19.5-34+8af48932a5ef06
        ```

    -   Get the node ip address:

        `kubectl describe node/<node name returned from kubectl get nodes>`

        or

        `kubectl describe node/minikube |grep InternalIP`

        example output:

        ```nohighlight
        InternalIP:  10.0.1.150
        ```

1.  Save and apply your load test configuration:

    `kubectl apply -f loadtest.yaml`

## Monitor the test

The supplied `run-load-test` script will not only start the load
injector in the target K8s cluster,
but will also tail the logs so you may monitor status of your test.

Once the test is complete (or you which to stop monitoring the test),
merely issue a SIGQUIT to your terminal (via Ctrl+C).

You can also tail the logs via the following command:

`kubectl logs loadtest -f`

## Abort a test run

If you run a test to completion with the `run-load-test` script will
also terminate the loadtest pod.

But perhaps you set a long running test,
and do not want to monitor it interactively.

You can exit the script during an existing test run by issuing a
SIGQUIT (via Ctrl+C),
but by doing so you are not terminating the load injector pod.
It will continue to run on the Kubernetes cluster until it finishes,
or unless you explicitly terminate it.

You can use the `abort-load-test` script to later abort the test:

```bash
abort-load-test [-n <kubernetes namespace from where to run the load injector]
```

This will terminate and remove the Kubernetes pod where the load
injector runs.

### Automatic clean up

In the scenario you exit the `run-load-test` script before it completes,
and you do not run the `abort-load-test`,
the `loadtest` pod will remain,
but in an `Completed` state.
Any subsequent run of `run-load-test` will delete the completed pod and
start a new one.

### Manual deletion of loadtest pod

You can also delete the pod manually:

`kubectl delete pod/loadtest --namespace=<namespace>`.