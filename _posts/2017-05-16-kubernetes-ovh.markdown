---
layout: post
title:  "Setup kubernetes with flannel on OVH"
date:   2017-05-16 23:55:00
Tags: [kubernetes, flannel, orchestration, OVH, kubectl, kubeadm]
Categories: [devops]
---


Hello everyone,

This is a the first article of a series about deploying a webapp on a Kubernetes cluster running on OVH infrastructure.

<br>In the following article I will demonstrate how to :

- Deploy Kubernetes with 2 [nodes](https://kubernetes.io/docs/concepts/architecture/nodes/) (the master an a minion)
- Setup flannel as the network plugin

# Infrastructure description

## Servers

- 2x Cloud VPS-SSD running Ubuntu 16.10

![vps-ssd](/img/vps-ssd.png)

# Install Kubernetes

## Log as root
Some commands need to be run as root. To log as root simply type.
{% highlight bash %}
sudo su -
{% endhighlight %}

More info on : [https://www.ovh.co.uk/g1786.become_root_and_select_a_password](https://www.ovh.co.uk/g1786.become_root_and_select_a_password)

## Install Docker

We have followed the [Get Docker for Ubuntu guide](https://docs.docker.com/engine/installation/linux/ubuntu/). Here is the summury of commands :

Remove previous version of docker
{% highlight bash %}
sudo apt-get remove docker docker-engine
{% endhighlight %}

Install community edition
{% highlight bash %}
sudo apt-get install apt-transport-https ca-certificates curl software-properties-common
curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo apt-key add -
sudo apt-key fingerprint 0EBFCD88
sudo add-apt-repository "deb [arch=amd64] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable"
sudo apt-get update
sudo apt-get install docker-ce
{% endhighlight %}

## Install Kubernetes

We are going to use [the getting started kubeadm guide](https://kubernetes.io/docs/getting-started-guides/kubeadm/). We are going to sum-up the various command we have to run.

### On all nodes
We followed the Ubuntu install part.

{% highlight bash %}
apt-get update && apt-get install -y apt-transport-https
curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | apt-key add -
cat <<EOF >/etc/apt/sources.list.d/kubernetes.list
deb http://apt.kubernetes.io/ kubernetes-xenial main
EOF
apt-get update
apt-get install -y kubelet kubeadm kubectl kubernetes-cni
{% endhighlight %}

### On the Master
Before starting you need to know that you will need to pass the --pod-network-cidr flag for the init command when using flannel (see [https://kubernetes.io/docs/admin/kubeadm#kubeadm-init](https://kubernetes.io/docs/admin/kubeadm#kubeadm-init) and [https://github.com/kubernetes/kubernetes/issues/36575#issuecomment-264622923](https://github.com/kubernetes/kubernetes/issues/36575#issuecomment-264622923))

{% highlight bash %}
kubeadm init --pod-network-cidr=10.244.0.0/16
{% endhighlight %}

Save the kubeadm join command that kubeadm init will output at the end. You will need your token for your node.
`kubeadm join --token <token> <master-ip>:<master-port>`

Logout as a normal user and start the cluster

{% highlight bash %}
logout
sudo cp /etc/kubernetes/admin.conf $HOME/
sudo chown $(id -u):$(id -g) $HOME/admin.conf
export KUBECONFIG=$HOME/admin.conf
{% endhighlight %}

At this stage the master node should <b>NOT</b> be ready because it is missing a network plugin to start.

{% highlight shell %}
kubectl get nodes

NAME       STATUS     AGE       VERSION
server-1   NotReady   1m        v1.6.3
{% endhighlight %}

You can run
{% highlight shell %}
kubectl describe nodes
{% endhighlight %}

You should find in the `conditions` section
`KubeletNotReady runtime network not ready: NetworkReady=false reason:NetworkPluginN`

### Installing a pod network
We are going to install flannel as a pod network.
{% highlight bash %}
kubectl apply -f https://github.com/coreos/flannel/raw/master/Documentation/kube-flannel.yml

serviceaccount "flannel" created
configmap "kube-flannel-cfg" created
daemonset "kube-flannel-ds" created
{% endhighlight %}

Check installation worked
{% highlight bash %}
kubectl get pods --all-namespaces

NAMESPACE     NAME                              READY    STATUS              RESTARTS  AGE
kube-system   etcd-server-1                      1/1       Running             0       3m
kube-system   kube-apiserver-server-1            1/1       Running             0       3m
kube-system   kube-controller-manager-server-1   1/1       Running             0       4m
kube-system   kube-dns-3913472980-2cknz          0/3       ContainerCreating   0       4m
kube-system   kube-flannel-ds-xcd15              1/2       CrashLoopBackOff    5       3m
kube-system   kube-proxy-j7tb0                   1/1       Running             0       4m
kube-system   kube-scheduler-server-1            1/1       Running             0       3m
{% endhighlight %}

Oups ! <b>Something is wrong</b> and flannel did not start nor the dns pod. You can have a look at the logs of the flannel pod.

{% highlight bash %}
kubectl logs -n kube-system kube-flannel-ds-xcd15 -c kube-flannel

E0523 11:39:28.562300       1 main.go:127] Failed to create SubnetManager: error retrieving pod spec for 'kube-system/kube-flannel-ds-xcd15': the server does not allow access to the requested resource (get pods kube-flannel-ds-xcd15)
{% endhighlight %}

You need to setup some RBAC permissions !

{% highlight bash %}
kubectl create -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel-rbac.yml

clusterrole "flannel" created
clusterrolebinding "flannel" created
{% endhighlight %}

Then wait for some time (time for pods to restart automatically) you should get everything runing !

{% highlight bash %}
kubectl get pods --all-namespaces

NAMESPACE     NAME                               READY     STATUS    RESTARTS   AGE
kube-system   etcd-server-1                      1/1       Running   0          1h
kube-system   kube-apiserver-server-1            1/1       Running   0          1h
kube-system   kube-controller-manager-server-1   1/1       Running   0          1h
kube-system   kube-dns-3913472980-2cknz          3/3       Running   33         1h
kube-system   kube-flannel-ds-xcd15              2/2       Running   23         1h
kube-system   kube-proxy-j7tb0                   1/1       Running   0          1h
kube-system   kube-scheduler-server-1            1/1       Running   0          1h
{% endhighlight %}

Finding all this took me quite some time as I was a pure Kubernetes newbee. So I do hope it really saves you time.

### Specific to minion
Run the join command as root (replace by your own specific values).

{% highlight bash %}
kubeadm join --token <token> <master-ip>:<master-port>
{% endhighlight %}

Login to master and check node as been added.

{% highlight bash %}
kubectl get nodes
{% endhighlight %}

You should see something like this
{% highlight bash %}
NAME       STATUS    AGE       VERSION
server-1   Ready     23h       v1.6.3
server-2   Ready     3m        v1.6.3
{% endhighlight %}

### Reset
If you want to start again, here is how to reset a node
{% highlight bash %}
kubectl drain <node name> --delete-local-data --force --ignore-daemonsets
kubectl delete node <node name>
# Then, on the node being removed, reset all kubeadm installed state:
kubeadm reset
{% endhighlight %}
