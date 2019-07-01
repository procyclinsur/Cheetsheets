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
