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

```bash
function helm-tmpl-install() {
    ###########
    # Variables
    ###########
    CHRT=$1
    NMSPC=$2
    OPTS=${@:3}  # All options after 2
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
    FILE=$(mktemp)
    helm template --namespace $NMSPC $OPTS $CHRT_DIR | tee $FILE
    read -p "Should we deploy this configuration? yes/no [no]: " DEPLOY
    DEPLOY=${DEPLOY:-no}
    if [ $DEPLOY == yes ]; then
        kubectl apply -n $NMSPC -f "$FILE"
    fi
    rm -rf /tmp/helm
    cd $BASE
}
```
> Written with [StackEdit](https://stackedit.io/).
<!--stackedit_data:
eyJoaXN0b3J5IjpbLTIxMjQ4MDU1MTNdfQ==
-->