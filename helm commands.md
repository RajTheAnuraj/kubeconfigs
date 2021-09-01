## installing node exporter 
This is for the prometheus to scrap system load stuff 
Node exporter is like an agent wchich wll scrape stuff and send to prometheus.  

```
docker run -d -p 9100:9100 --user 995:995 -v "/:/hostfs" --net="host" prom/node-exporter --path.rootfs=/hostfs
```

Once started check
```
curl http://localhost:9100/metrics
```

If thats working, add the ip to prometheus.yml and restart prometheus server
\\Codecyan\Config\prometheus\prometheus.yml


## nfs-subdir-external-provisioner 

```
helm install nfs-subdir-external-provisioner nfs-subdir-external-provisioner/nfs-subdir-external-provisioner \
    --set nfs.server=192.168.86.250 \
    --set nfs.path=/pihole/dynamicprovision \
--set storageClass.onDelete=retain \
--set storageClass.accessModes=ReadWriteMany 
```

## Ghost

```
helm install ghost-release bitnami/ghost \
--set replicaCount=3 \
--set service.type=NodePort \
--set service.nodePorts.http=30034 \
--set persistence.storageClass="nfs-client" \
--set mariadb.primary.persistence.storageClass="nfs-client" 
```