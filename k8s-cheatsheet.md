# K8S



## Tools



## Functions


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



<!--stackedit_data:
eyJoaXN0b3J5IjpbMTAzNjI4MTg3MV19
-->