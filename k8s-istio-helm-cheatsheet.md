# K8S

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


# ISTIO

### Enable log level setting on istio-proxy
```bash
function log-level-change-k8s() {
    POD_NAME=$1
    NAMESPACE=$2
    LEVEL=$3
    L_TYPE=$4
    kubectl exec $(kubectl get pods -n $NAMESPACE \
                                    -l app=$POD_NAME \
                                    -o jsonpath='{.items[0].metadata.name}') \
                 -n $NAMESPACE -c istio-proxy \
                 -- curl -X POST localhost:15000/logging?$L_TYPE=$LEVEL -s
}
```

### Add labels to a namespace
```bash
kubectl label namespace solarmori istio-injection=enabled
```

### Remove labels from a namespace
```bash
kubectl label namespace solarmori istio-injection-
```

### Reasons for 503 Errors

* Backend pod is not running successfully
* Connecting to resources outside of local namespace, it is required to use the full k8s DNS name.
    * i.e. ~~core-api-svc~~ -> core-api-svc.core-namespace.svc.cluster.local

### Disable automatic side-car injection for a pod

* To disable automatic side-car injection for a specific pod, you should iinclude the following in its `spec`

```yaml
spec:
  template:
    metadata:
      annotations:
        sidecar.istio.io/inject: "false"
```

# Helm

### Update a release

```bash
helm upgrade --install \
             --set cp-schema-registry.enabled=false \
             --set cp-kafka-rest.enabled=false \
             --set cp-ksql-server.enabled=false \
             --set cp-kafka.prometheus.jmx.enabled=false \
             --set cp-zookeeper.prometheus.jmx.enabled=false \
             --set cp-kafka.imageTag=$KAFKA_IMG_VER \
             --set cp-zookeeper.imageTag=$KAFKA_ZK_IMG_VER \
             --namespace atp-system \
             atp-cmb \ # RELEASE NAME
             confluentinc/cp-helm-charts # CHART PATH
```

### Test templating before deploy

assumes your template `override.yml` file is in `$PWD/helm`.

```bash
function helm-tmpl-oride() {
    ###########
    # Variables
    ###########
    RLSE=$1  # Helm release name
    CHRT=$2  # Helm chart name
    FILE=$3  # File path/name
    BASE=$PWD
    
    #############
    # Fetch Chart
    #############
    mkdir -p /tmp/helm
    cd /tmp/helm
    helm fetch --untar --untardir . $CHRT
    CHRT_DIR=$(ls -td /tmp/helm/*/ | head -n 1)
    
    ################
    # Check template
    ################
    helm template --name $RLSE $CHRT_DIR -f $BASE/$FILE > manifest.yml
    vim manifest.yml
    cd $BASE
}
```
