#!/bin/bash

usage() { echo "Usage: $0 [-n <kubernetes namespace from where to run the load injector]" 1>&2; exit 1; }

while getopts ":n:" o; do
    case "${o}" in
        n)
            namespace=${OPTARG}
            ;;
        *)
            usage
            ;;
    esac
done

if [ -z "$namespace" ] ; then
    usage
    exit 1
fi

if [ ! -z `kubectl get pod|grep loadtest|cut -f 1 -d " "` ]; then
    echo "aborting existing load test, or cleaning up previous load test..."
    kubectl delete pod/loadtest -n $namespace
else
    echo "load test no longer running."
fi

