# Docker

### Clean Docker Images and Containers
```bash
clean_docker() {
    for i in `sudo docker container list --all | grep -v COMMAND | awk '{print $1}'`
        do sudo docker container rm $i
    done
    for i in `sudo docker image list | grep -v CREATED | awk '{print $3}'`
        do sudo docker rmi -f $i
    done
}
```

### Edit `docker ps` output format

Example of psformat addition to ~/.docker/config

```json
{
  "psFormat": "table {{.Names}}\\t{{.Networks}}\\t{{.RunningFor}} ago\\t{{.Status}}"
}
```

Available options can be found using the command:

```bash
docker ps --format '{{ json .}}' | jq
```

### appbld - Build an application and push to ECR with todays tag

```bash
appbld() {
    err() {
        echo -e "\e[01;31m$@\e[0m" >&2
    }
    appbld_usage() {
        err "  APP BUILDER"
        err "    Build Docker Container and send to AWS ECR\n"
        err "      appbld [<app_name> <gitlab_token> <docker_context>] \n"
        err "    Alternatively you may set the following environmental variables"
        err "      APP to <app_name>"
        err "      GLT to <gitlab_token>"
        err "      CTX to <context>"
    }

    APP=${1:-$APP}
    GLT=${2:-$GLT}
    CTX=${3:-$CTX}
    TAG=$(date -Idate)
    if [ -z $APP ] || [ -z $CTX ] || [ -z $GLT ]; then
        appbld_usage
        return
    fi

    docker build -f Dockerfile -t $APP --build-arg GITLAB_TOKEN=$GLT $CTX
    $(aws ecr get-login --no-include-email)
    aws ecr create-repository --repository-name $APP &> /dev/null
    sleep 1
    REPO_URI=$(aws ecr describe-repositories --repository-names $APP --output text --query 'repositories[*].repositoryUri')
    docker tag $APP:latest $REPO_URI:$TAG
    docker push $REPO_URI:$TAG
}
```
