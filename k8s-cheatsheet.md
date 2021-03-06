# K8S

## K8S Internals

- https://jvns.ca/blog/2017/07/27/how-does-the-kubernetes-scheduler-work/
- Deployments manage ReplicaSets
	- Deployment automagically creates a replicaset to manage pods.
- ReplicaSet manages Pods

## TIPS

- Use a mix of pre-emptible (spot) nodes and reserved nodes to save costs.
	- deploy nodes that can support failure on pre-emptible nodes.
- Use liveness and readiness probes to prevent traffic from being directed to your application until it is ready ( preventing loss of data )
- Define resource limits and requests for your pods/containers ( prevents resource hogging )
- Operations best practice should be to run a `kubectl diff -f file.yaml` before deploying a manifest to ensure, the changes you made are what will happen on the cluster.
- When troubleshooting containers on EKS if a pod is stuck in `Creating` state, then it may be due to an over allocation of IP'S for the node's CNI Plugin.

## Tools

### Descheduler

Can be run every so often to rebalance the cluster preventing mistakes made by the scheduler.
- https://github.com/kubernetes-incubator/descheduler

### Kubespy

Watch changes to a resource over time.
- https://github.com/pulumi/kubespy

### Kubesquash

Attach gdb or dlv (golang debugger) to any container process.
- https://github.com/solo-io/kubesquash

### Stern

View logs from groups of pods, or containers at one time, each pod/container log will be prefixed with their name.
- https://github.com/wercker/stern

## Functions

### Get YAML deployment manifests for running resource

```bash
kubectl get <resource-type> \
            -n <namespace> \
            <resource-name> \
            -o yaml --export > my-manifest.yaml
```

### Check what would change before deploying

```bash
kubectl diff -f my-manifest.yaml
```

### Refresh All Pods ([ORIGINAL](https://gist.github.com/jmound/ff6fa539385d1a057c82fa9fa739492e))
```bash
function refresh-all-pods() {
  echo
  DEPLOYMENT_LIST=$(kubectl -n $1 get deployment -o jsonpath='{.items[*].metadata.name}')
  echo "Refreshing pods in all Deployments"
  for deployment_name in $DEPLOYMENT_LIST ; do
    TERMINATION_GRACE_PERIOD_SECONDS=$(kubectl -n $1 get deployment "$deployment_name" -o jsonpath='{.spec.template.spec.terminationGracePeriodSeconds}')
    if [ "$TERMINATION_GRACE_PERIOD_SECONDS" -eq 30 ]; then
      TERMINATION_GRACE_PERIOD_SECONDS='31'
    else
      TERMINATION_GRACE_PERIOD_SECONDS='30'
    fi
    patch_string="{\"spec\":{\"template\":{\"spec\":{\"terminationGracePeriodSeconds\":$TERMINATION_GRACE_PERIOD_SECONDS}}}}"
    kubectl -n $1 patch deployment $deployment_name -p $patch_string
  done
  echo
}
```

### Switch K8S Contexts (for EKS: may work with other setups)
```bash
function swikube() {
    for context in $(kubectl config get-contexts --output=name); do 
        if [ ${context##*/} == $1 ]; then
            kubectl config use-context $context
            status=0
            break
        else
            status=1
        fi
    done
    if [ $status == 1 ]; then
        echo ERROR:  Enter a valid cluster name
    fi
    status=0
}
```
### Manually deploy container for troubleshooting, w/autoremoval and AWS Role

This assumes that you have a working installation of kube2iam.

```bash
override='{
  "spec": {
    "template": {
      "metadata": {
        "annotations": {
          "iam.amazonaws.com/role": "kafka-s3-rules"
        }
      }
    }
  }
}'

kubectl run --generator=deployment/apps.v1 test-logstash \
            -n atp-system --stdin --tty --rm \
            --overrides "$override" \
            --image docker.elastic.co/logstash/logstash-oss:7.1.1 bash
```

### Manually deploy container for troubleshooting cockroachdb

```bash
kubectl run temp-db-client -n solarmori --rm -it \
            --image=cockroachdb/cockroach \
	    --restart=Never -- sql --insecure \
	    --host solar-rdbm-cockroachdb-public:26257
```

<!--stackedit_data:
eyJoaXN0b3J5IjpbLTEyOTAwMDU2NTYsOTYyOTY0MjAyLDE4OT
I0MzMxMzAsLTExODIzNzk3ODJdfQ==
-->

### Change k8s contexts with ease

```bash
# func: cm           - Context Manager for k8s
cm () {
    err() {
        echo -e "\e[01;31m$@\e[0m" >&2
    }
    helpme () {
        err "  CONTEXT MANAGER______________________"
        err "    USAGE: cm [[l], [c <context-name>]]"
        err "      l - Lists all available contexts"
        err "      c - Changes to specified context"
    }
    ctxs=$(kubectl config get-contexts --output=name)
    cctx=$(kubectl config current-context)
    case $1 in
    l)
        for ctx in $ctxs; do
        [ $cctx == $ctx ] && echo "* ${ctx##*/}"
        ! [ $cctx == $ctx ] && echo "  ${ctx##*/}"
        done
        ;;
    c)
        [ $2 ] || helpme
        for ctx in $ctxs
        do
    if [ $2 == ${ctx##*/} ]; then
        kubectl config use-context $ctx
        stat=0
        break
            else
        stat=1
    fi
        done
        [ $stat == 0 ] || helpme
        stat=0
        ;;
    *)
        helpme
    esac
}
```
