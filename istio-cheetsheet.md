
# ISTIO

## Notes

### Mixer

Istio Mixer is responsible for providing policy and telemetry configuration data.  This is handled via the following objects.

| Object |  Description |
|---|---|
| Adapters | Used to abstract telemetry and policy backends to istio. |
| Attributes | Individual KV pair data defined in specific adapters. |
| Handlers | Determine what adapters will be used and how they operate. |
| Instances | Instances represent a chunk of data that one or more adapters will operate on, and describe how to map a requests attributes to an adapter input. |
| Roles | Describe when a particular adapter is called and which instances it will be given. |

Envoy Side-Cars cache policy data from Mixer that is used for each request, and buffer telemetry data to send to Mixer in bulk.

## Configuration
### *Enable log level setting on istio-proxy*
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

### *Add labels to a namespace*
```bash
kubectl label namespace solarmori istio-injection=enabled
```

### *Remove labels from a namespace*
```bash
kubectl label namespace solarmori istio-injection-
```

### *Disable automatic side-car injection for a pod*

* To disable automatic side-car injection for a specific pod, you should iinclude the following in its `spec`

```yaml
spec:
  template:
    metadata:
      annotations:
        sidecar.istio.io/inject: "false"
```

### *Whitelisting IP Addresses*

#### Mixer Settings:

##### Handler
```yaml
apiVersion: config.istio.io/v1alpha2
kind: handler
metadata:
  name: whitelist
spec:
  compiledAdapter: listchecker
  params:
    # providerUrl: ordinarily black and white lists are maintained
    # externally and fetched asynchronously using the providerUrl.
    overrides: ["v1", "v2"]  # overrides provide a static list
    blacklist: false
```

##### Instance 
```yaml
apiVersion: config.istio.io/v1alpha2
kind: instance
metadata:
  name: appversion
spec:
  compiledTemplate: listentry
  params:
    value: source.labels["version"]
---
apiVersion: config.istio.io/v1alpha2
kind: rule
metadata:
  name: checkversion
spec:
  match: destination.labels["app"] == "ratings"
  actions:
  - handler: whitelist
    instances: [ appversion ]
---
```
## Troubleshooting

### *Reasons for 503 Errors*

* Backend pod is not running successfully
* Connecting to resources outside of local namespace, it is required to use the full k8s DNS name.
    * i.e. ~~core-api-svc~~ -> core-api-svc.core-namespace.svc.cluster.local

<!--stackedit_data:
eyJoaXN0b3J5IjpbMTIxOTcwMzEwNyw5Nzc0NzY3NDgsMTQ2Nz
MyODg2MywtMTM1NDQ4NTQ5LDQwNTgzMjAwLDEzMzkwMzkzMzcs
LTEyNTQ2NzUwOTcsLTE4Mzg2ODM0NDQsNjk4MDYyMjM0LC01Nj
k3Nzk1N119
-->