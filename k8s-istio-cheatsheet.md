# K8S

# ISTIO

### Enable log level setting on istio-proxy
```bash
POD_NAME=<APPLICATION_LABEL>
NAMESPACE=<NAMESPACE>
LEVEL=<LOG_LEVEL>
kubectl exec $(kubectl get pods -n $NAMESPACE \
                                -l app=$POD_NAME \
                                -o jsonpath='{.items[0].metadata.name}') \
             -n $NAMESPACE -c istio-proxy \
             -- curl -X POST localhost:15000/logging?filter=$LEVEL -s
```
