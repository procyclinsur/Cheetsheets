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
