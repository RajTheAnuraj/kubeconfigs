# Resetting stuff
```
sudo systemctl enable docker
sudo systemctl enable kubelet
systemctl daemon-reload
systemctl restart docker
systemctl restart kubelet

kubeadm reset
```

To see the logs for errors while joining or starting cluster
```
journalctl -u kubelet
```

* **if you want to add a new worker node afterwards and wants the join command** Run following
```
kubeadm token create --print-join-command
```

* install ubuntu on the machine
* Enable ssh by creating a file ssh on boot partition
* boot the system and watch for the ip that dhcp assigns for the system. It also make sense to configure DHCP reservations on your router for these devices so the device gets the same ip everytime it starts. Make sure you give an IP towards the upper limit just to make sure other devices wont steal the IP during reboot
* log into the system using ssh , default user and password and make sure everything works
* change the hostname appropriately by modifying /etc/hostname file and /etc/hosts file. hosts file might not have any reference to the hostname in which case you can ignore that file. The host names should be unique for each node otherwise you get unnecessary issues while joining the nodes to the cluster


**Create New User**
* create a new user
```
sudo adduser newuser
```

* add the user to sudo group
```
sudo usermod -aG sudo newuser
```

* login as the new user
```
su newuser
```

* verify privileges
```
sudo ls -la /root
```

**Create and use RSA keys for ssh login**
* create an rsa pair using ssh-keygen
* Copy the public key to the remote server using ssh-copy-id
```
ssh-copy-id -i /path/to/public/key newuser@hostip
```

* test logging in with the private key
```
ssh -i /path/to/private/key user@hostip
```

**Install Docker on all nodes**
```
sudo apt install -y docker.io
```

**Change the cgroup driver to systemd**
```
sudo cat > /etc/docker/daemon.json <<EOF
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "storage-driver": "overlay2"
}
EOF
```


**Enable limit support for docker***
```
sudo sed -i '$ s/$/ cgroup_enable=cpuset cgroup_enable=memory cgroup_memory=1 swapaccount=1/' /boot/firmware/cmdline.txt
```
https://opensource.com/article/20/6/kubernetes-raspberry-pi

**Allow IPTables to see bridged traffic**
```
cat <<EOF | sudo tee /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
$ sudo sysctl --system
```

**Add kubernetes repo to ubuntus sources**
```
# Add the packages.cloud.google.com atp key
$ curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -

# Add the Kubernetes repo
cat <<EOF | sudo tee /etc/apt/sources.list.d/kubernetes.list
deb https://apt.kubernetes.io/ kubernetes-xenial main
EOF
```

**Install the three packages needed for k8**
```
sudo apt update && sudo apt install -y kubelet kubeadm kubectl
```

**Disable updates for these packages.**
These should be manual updates. Automatic updates might mess something up in unexpected times
```
sudo apt-mark hold kubelet kubeadm kubectl
```

**Initializing control plane**

* Decide on a CIDR to be used for the cluster. 10.244.0.0/16 is a common one
* Use the kubeadm to generate a token. This token is used to init the cluster
```
TOKEN=$(sudo kubeadm token generate)
```

* find the version of k8
```
kubectl version
```

* initialize the control plane
```
sudo kubeadm init --token=${TOKEN} --kubernetes-version=v1.18.2 --pod-network-cidr=10.244.0.0/16
```

Pay close attention to the output of the above command
A config file is created . You can save it and use it to run kubectl from local
And there is also a command it generates so that you can run that on other nodes to join the cluster

if you run `kubectl get nodes` at this point you can see your master node is already there. But it will be in a status of `NotReady`. This is because we did not install CNI . For this we will use flannel as CNI

* install flannel CNI 
Use the link from flannel documentation and download the yml and use kubectl to apply it
```
curl -sSL https://raw.githubusercontent.com/coreos/flannel/v0.12.0/Documentation/kube-flannel.yml | kubectl apply -f -
```

Now monitor the pods and see if all are running. Sometimes the flannel pods or coredns pods get stuck in ContainerCreating or crashloop. In this case its best to reset the node and try again . use *kubeadm reset* to reset the node. Once all pods are running and happy we can move to next step

* join the nodes to cluster.
This is done using the the command that we got in the output of kubeadm init 
If you are rejoining the current node, there might already be a ca certificate in there and it will prvent you from installing the new certificate for joiining. You can manually delete everything. But easy way is to run kubeadm reset before running the join command. This will reset the node , but retain earlier configuration that you want to delete manually if needed. /etc/kubernetes

```
sudo kubeadm join 192.168.2.114:6443 --token zqqoy7.9oi8dpkfmqkop2p5 \
    --discovery-token-ca-cert-hash sha256:71270ea137214422221319c1bdb9ba6d4b76abfa2506753703ed654a90c4982b
```
 verify by runnning kubectl get nodes
 
 If you want the pods to be scheduled on all nodes including master, we have to untain the master by running following command
 Substitute the [nodename] with the name of your master node
 ```
 kubectl taint [nodename] yasin node-role.kubernetes.io/master-
 ```

* Accessing kubectl from local*
  Install kubectl on local.
  For mac run
  ```
  brew install kubernetes-cli
  ```
  
  Once its installed copy the admin.conf file from the server to local so that the kubectl knows how to connect to the server
  The admin.conf file path is provided while you ran kubeadm init.
  For me it was on `/etc/kubernetes/admin.conf`
  Copy it to `~/.kube/config`
  
  Now you can connect to the server
  
