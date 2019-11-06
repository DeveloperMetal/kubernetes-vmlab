# Multi-Node Kubernetes Cluster VMLab In a Single Machine

The majority of the setup comes from this guide:
https://kubernetes.io/blog/2019/03/15/kubernetes-setup-using-ansible-and-vagrant/

Except it is updated to run more recent versions of calico and configurable from .env file

## Cluster configuration
Before you start your cluster you'll need to complete the following steps:

### Step 1
copy your public ssh key to the root of this project and rename it `authorized_key.pub`

### Step 2
Rename the template.env file to .env and edit it accordingly.

### Step 3
Find an interface to expose your master node:

```
VBoxManage list bridgedifs
```

Find the name of the interface to your local network as virtualbox sees it. Ex: "name: eno1"

And update your .env file and set the public interface to use and a static ip in your network:

```bash
NETWORK_PUBLIC_BRIDGE = "eno1"
MASTER_PUBLIC_IP = "192.168.1.100"
```

### Set total number of slave nodes (optional)
In you .env file update the total number of nodes as needed:

```bash
TOTAL_NODES=5
```

This will create 6 nodes, one master. Five slaves.

You can now provision your cluster

## Provisioning
To provision the cluster:

```bash
vagrant up
```

To Access the master:

```bash
vagrant ssh master
```

To Access nodes:

```bash
vagrant ssh node-<node number>
```

Where `<node number>` starts at 1

To destroy the cluster

```bash
vagrant destroy -f
```

# Remote management
This should work if you setup your cluster on a separate server in your network or on the same machine.

You can now install kubectl and copy your master configurations to issue kubectl commands.

To copy your configurations you can rsync the file easily in two ways:

From your vm host:
```
rsync -vPc vagrant@192.168.50.10:/vagrant/.kube/config ~/.kube/
```

From another machine in the network
```
rsync -vPc vagrant@x.x.x.x:/vagrant/.kube/config ~/.kube/
```
Where x.x.x.x is the static public master ip you provided during step 3 of the setup.

Find the version of kubectl that fits your os here: https://kubernetes.io/docs/tasks/tools/install-kubectl/

After installing kubectl you should be able to connect to your cluster:

```bash
kubectl version
```
The result should look similar to the following:
```bash
Client Version: version.Info{Major:"1", Minor:"16", GitVersion:"v1.16.0", GitCommit:"2bd9643cee5b3b3a5ecbd3af49d09018f0773c77", GitTreeState:"clean", BuildDate:"2019-09-18T14:36:53Z", GoVersion:"go1.12.9", Compiler:"gc", Platform:"windows/amd64"}
Server Version: version.Info{Major:"1", Minor:"16", GitVersion:"v1.16.2", GitCommit:"c97fe5036ef3df2967d086711e6c0c405941e14b", GitTreeState:"clean", BuildDate:"2019-10-15T19:09:08Z", GoVersion:"go1.12.10", Compiler:"gc", Platform:"linux/amd64"}
```


# Known issues
This repo has been tested on ubuntu 19.10 only, but should work on recent ubuntu versions.

The fact we are copying the master config file means that there is very little security on the cluster. So, DO NOT USE THIS IN PRODUCTION! This is meant for a developer to test their app fully in a multinode cluster environment and experiment with it.

Also, this project is untested in Windows except for connecting remotely through `kubectl` as of writing this works perfectly fine.

Ansible is not picking up the ansible.cfg file in the repo so errors aren't really human readable, needs more work...