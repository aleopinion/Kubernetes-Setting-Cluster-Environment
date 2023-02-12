**========>>> Setting up a Kubernetes Cluster with 1 Master and 2 Worker Nodes <<<=======**

**Create a VM called master-node,**
**Machine Type => based on k8s documentation, you need @ least**
**2 VCPU and 2 GB RAM  as the requirement for the VM to take part** 
**in a k8s set up. Let the Image be CentOS7. Youc could use Ubuntu** 
**or REDHat Ent8, it works exactly the same way.**

**1. => To update the master server so it can have all the latest libraries**
```
sudo yum update -y
```

**2. => Set up Firewall Rule and grant access to all TCP protocols**
```
0-65535
```

**3. => Switch to root while still working in your user home directory**
```
sudo su 
```

**4. => To keep you in your user home directory**
```
cd 
```

**5. To install particular utilities of yum that are needed**
```
yum install -y yum-utils 
```

**6. => To install LVM2 (Logical Volume management) and device mapping for persistent disk.**
**This is because we are using the persistent disk to store data. In this command, we are**
**installing 2 things at the same time - device map and also LVM2.**
```
**yum install -y device-mapper-persistent-data lvm2 
```

**7. => To download or import the Docker registry. The file saved to /etc/yum.repos.d/docker-ce.repo**
```
yum-config-manager --add-repo https://download.docker.com/linux/centos/docker-ce.repo 
```

**8. => To install the community edition of Docker**
```
yum install -y docker-ce 
```

**9. => To start Docker**
```
systemctl start docker 
```

**10. => To refresh docker**
```
systemctl daemon-reload 
```
 
**11. => To make Docker persistent**
```
systemctl enable docker 
```

**12. => To checker the status of Docker => active (running)**
```
systemctl status docker 
```

**13. => To setup the IP Tables => For the CLuster IP Service**
``` 
cat <<EOF >  /etc/sysctl.d/k8s.conf
net.bridge.bridge-nf-call-ip6tables = 1
net.bridge.bridge-nf-call-iptables = 1
EOF
```

**14. => To refresh and make sure the system captures the changes made by the IP Tables**
```
sysctl --system 
```
 
**15. => To cat the file where Selinux is declared before going ahead to disable it**
```
cat /etc/sysconfig/selinux 
```

**16. => To begin the process of disabling Selinux, you run this command**
```
setenforce 0 
```
 
**17. => To use the sed command to update Selinux without actually going into the file**
```
sed -i --follow-symlinks 's/SELINUX=enforcing/SELINUX=disabled/g' /etc/sysconfig/selinux  
```

**18. => To cat the Selinux file and make sure that Selinux is actually disabled**
```
cat /etc/sysconfig/selinux 
```

**19. => To turn off SWAP => Swapoff for performance sake**
```
swapoff -a 
```

**20. => After turning off swap, you want to persist the data by writting a path to** 
**that file in the fstable in Linux, and by default, whenever the system reboots itself,** 
**it will make sure it gets that configuration back to the state that it was before.**
```
sed -i '/ swap / s/^/#/' /etc/fstab
```

**21. => Using cat to establish/create a particular repository within your 
**yum repo environment for k8s which is needed to configure k8s**
```
cat <<EOF > /etc/yum.repos.d/kubernetes.repo
[kubernetes]
name=Kubernetes
baseurl=http://mirrors.aliyun.com/kubernetes/yum/repos/kubernetes-el7-x86_64
enabled=1
gpgcheck=0
repo_gpgcheck=0
gpgkey=http://mirrors.aliyun.com/kubernetes/yum/doc/yum-key.gpg http://mirrors.aliyun.com/kubernetes/yum/doc/rpm-package-key.gpg
exclude=kubelet kubeadm kubectl
EOF
```

**22. => To reboot the vm. You can either reboot from your cloud console or you go this route.**
```
reboot
```

```
sudo su 
```
```
cd 
```

**25. => You are telling the k8s installation process to continue or pass should** 
**it experience anything or an error that might disrupt the process.**
```
yum install -y kubelet kubeadm kubectl --disableexcludes=kubernetes 
```
 
**26. => To start kubelet => but will fail or break because it needs kubeadm to be running before it can work**
```
systemctl start kubelet 
```

**27. => To enable kubelet for persistency**
```
systemctl enable kubelet 
```

**28. => Status = Failed => Beacause kubeadm has not started running. kubelet starts when initializing kubeadm.**
```
systemctl status kubelet
```

**29. => To check the version of the kublet**
```
sudo kubelet --version 
```

**-----------------------------------------------------------------------------------------**
**Time to create a machine Image out of the Master and used to create the worker nodes**
**You may stop the VM or not before taking the Machine Image of the VM.**
**CREATE MACHINE IMAGE OUT OF THE VM**
**THEN CREATE 2 WORKER NODES USING THE MACHINE IMAGE.**
**-------------------------------------------------------------------------------------------**

**=====> CONTINUE TO CONFIGURE THE MASTER NODE <======**

**Log unto the Master Node**

```
sudo su
```
```
cd 
```

**32. => To check the status of firewalld => Active (running)**
```
systemctl status firewalld 
```

**33. => To stop firewalld**
```
systemctl stop firewalld 
```

**34. => To disable firewalld and make it inactive**
```
systemctl disable firewalld 
```

**35. => To check that firewalld is actually disabled => inactive (dead)**
```
systemctl status firewalld 
```

**36. => To initialize the kubeadm => Error => complaining that container runtime = Docker not running.**
```
kubeadm init --pod-network-cidr=192.168.0.0/16 --ignore-preflight-errors=NumCPU 
```
  
**37. => To show the status of Docker => Active (running)**
```
systemctl status docker 
```

**38. => To remove the file distubbing Docker from containerd**
```
rm /etc/containerd/config.toml ==> y 
```

**. 39 => To restart containerd. Failure to do this can throw you into errors.**
```
systemctl restart containerd 
```

**40. => To check Docker status ==> Docker running**
```
systemctl status docker 
```

**41. => Successful run for me. The "--ignore-preflight-errors=NumCPU" in the command it to tell**
**k8s to ignore any error that could arise from cpu not enough for the process**
```
kubeadm init --pod-network-cidr=192.168.0.0/16 --ignore-preflight-errors=NumCPU 
```




**====>>> Commands displayed after kubeadm initialisation which you need to run either a regular user or as root <<<====**   
**To start using your cluster, you need to run the following as a regular user:** 

  **mkdir -p $HOME/.kube**
  **sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config**
  **sudo chown $(id -u):$(id -g) $HOME/.kube/config**

**Alternatively, if you are the root user, you can run:**

  **export KUBECONFIG=/etc/kubernetes/admin.conf**


**42. => To identify any node running within the cluster. Command failed because we have not** 
**set majority of the global variables and k8s configurations.**
```
kubectl get nodes 
```

**43. => 1st command to be set up as a regular user**
```
mkdir -p $HOME/.kube
```

**44. => 2st command to be set up as a regular user. We did not use the sudo since we are working as root**
```
sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config 
```

**45. => 3st command to be set up as a regular user. We did not use the sudo since we are working as root**
```
sudo chown $(id -u):$(id -g) $HOME/.kube/config
```

**46. => To identify the nodes running within the cluster => STATUS = "NotReady" because 
**the cluster network has not being established.**
```
kubectl get nodes 
```

**----------------------------------------------------------------------------------------**

**https://github.com/flannel-io/flannel =======> GitHub link for kubernetes networking provider = Flannel**

**https://raw.githubusercontent.com/flannel-io/flannel/master/Documentation/kube-flannel.yml ==> Http path to the file to configure kubernetes networking from Flannel documentation**
**-----------------------------------------------------------------------------------------**

**47. => To configure the Flannel k8s networking**
```
kubectl apply -f https://raw.githubusercontent.com/flannel-io/flannel/master/Documentation/kube-flannel.yml
```

**Same as command #47.
``` 
kubectl create -f https://raw.githubusercontent.com/flannel-io/flannel/master/Documentation/kube-flannel.yml
```

**48. => To identify the nodes running within the cluster, and to confirm that the network 
**just set up is good. STATUS = "Ready"**
```
kubectl get nodes 
```
 
**49. => To check the stauts of kubelet after kubeadm is initialized**
```
systemctl status kubelet 
```

**Identifies the Master Node (control Plane)**
```
kubectl get nodes 
```



**====> TO CONFIGURE THE WORKER NODES <=======**

**50. => You run this command at the Masteer Node and use the Token generated at the Workers**
```
kubeadm token create --print-join-command 
```

**51. => the JOIN Command => to join the workers to the cluster. This will be different for every set up.**
```
kubeadm join 10.182.0.10:6443 --token o5mz47.b0wovaxcrr2fmbxx --discovery-token-ca-cert-hash sha256:c0d9ecb0800c85a58607751ddd1e3c7955bf2a11e4d29c4e592d17948cc32872
```
```
kubeadm join 10.182.0.10:6443 --token o5mz47.b0wovaxcrr2fmbxx \
  --discovery-token-ca-cert-hash sha256:c0d9ecb0800c85a58607751ddd1e3c7955bf2a11e4d29c4e592d17948cc32872
```

**Error when we ran the join command at the level of the worker to join the cluster.**
**Same error when setting up the master. So went to the workers and ran the following commands,** 
**as earlier done with the Master node so as to get the workers working properly**
```
sudo su 
```
```
cd 
```

**54. => Remove the config.toml file from containerd**
```
rm /etc/containerd/config.toml      y 
```
```
y
```

**55. => Restart containerd => very important step**
```
systemctl restart containerd 
```

**56. => Restart Docker and run the join command again as shown below*
```
systemctl restart docker 
```

**57. => the JOIN Command => to join the workers to the cluster. This will be different for every set up.**
```
kubeadm join 10.182.0.10:6443 --token o5mz47.b0wovaxcrr2fmbxx --discovery-token-ca-cert-hash sha256:c0d9ecb0800c85a58607751ddd1e3c7955bf2a11e4d29c4e592d17948cc32872
```
```
kubeadm join 10.182.0.10:6443 --token o5mz47.b0wovaxcrr2fmbxx \
  --discovery-token-ca-cert-hash sha256:c0d9ecb0800c85a58607751ddd1e3c7955bf2a11e4d29c4e592d17948cc32872
```

**58. => Run from the Master-Node to identify the nodes that are in the cluster**
```
kubectl get nodes
```

**59. => To set up a global-wide variable for the k8s administrator so he can administrate over the k8s cluster from anywhere**
```
export KUBECONFIG=/etc/kubernetes/admin.conf 
```

**=======>>>>> CONGRATULATIONS => THE SET UP IS COMPLETE <<<<<========**

**--------------------------------------------------------------------------------**

**60. => Gives you the services that are currently running. But we only have the "Cluster IP" service running for now.**
```
kubectl get services 
``` 

**61. => Gives you every pod running within the different namspaces in the cluster environment.**
```
kubectl get all --all-namespaces 
```

**62. => Shows you all the pods that are running across all the namespaces in the cluster.**
```
kubectl get pods --all-namespaces 
```

**63. => To make docker persistent in both the master and the workers**
```
systemctl enable docker 
```

**64. => To enable kubelet and make it persistent across board => If kubelet is down, you cannot communicate**
```
systemctl enable kubelet 
``` 

