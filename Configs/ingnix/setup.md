* Set up a NFS share drive. Use OMV or something like that set that up
* Install nfs-common library on the nodes
  ```
  sudo apt install nfs-common
  ```

# Dynamic Provisioning

Dynamic provisioning works the following way.
When a need for persistant storage arises, the pods will request it using a persistant volume claim.  

The persistant volume claim is linked with a storage class. The storage class has provisioners to provision the required space to the claim and its mounted as a volume in the pod which container can map to a mount point.

The first step is to find what kind of provisioner you need.

The available provisioners is listed [here](https://kubernetes.io/docs/concepts/storage/storage-classes/#nfs)

Here I am using a NFS provisioner because I have an NFS drive available by OMV. I am using [NFS subdir external provisioner](https://github.com/kubernetes-sigs/nfs-subdir-external-provisioner)

I used helm to install the provisioner.

```
helm install nfs-subdir-external-provisioner nfs-subdir-external-provisioner/nfs-subdir-external-provisioner \
    --set nfs.server=server ip \
    --set nfs.path=/nfs/folder/path \
--set storageClass.onDelete=retain \
--set storageClass.accessModes=ReadWriteMany 
```

Once this is run, a storage class is created.
You can verify it by running the below query

```
kubectl get sc --all-namespaces
```

Now you can use this to create PVCs and use the PVC in volume mount in deployment. The sample yamls do these.

# Static Provisioning

In static provisioning, admin have to create a Persistant Volume, use that in a Persistant Volume claim and use the claim in a deployment. The space is not dynamically provisioned. 

* Mount the share on all the nodes
* Edit /etc/fstab to add your detail. This will make sure that the drive will be mount even in the event of a reboot
* Add the below entry
  ```
  <externalip:/sharedfolder>  <mount point>   <driver>  <access>        ?       ?
  ip:/foldername              /mnt/folder     nfs       defaults        0       0

  ```
  
* Mount the drive. This only needs to be done once
```
# can use wither
mount ip:/foldername
or
mount /mnt/folder
```

* Verify by ls

```
ls /mnt/folder
```


The order in which its created

* PersistantVolume
* NameSpace
* PersistantVolumeClaim (from this point we can club everything into one namespace)
* nginx
* Service

Now you can copy the required html files to the directory and you can browse it on the nodeport exposed
