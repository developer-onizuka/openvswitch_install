# openvswitch_install

# 0. Goal

| VM | IP | Host | IP |
| --- | --- | --- | --- |
| master  | 192.168.33.100 | DELL precision T3620 (CentOS8 or Ubuntu)| 192.168.11.15 |
|         | 192.168.121.xxx (Internal IP in k8s cluster or Ubuntu)| DELL precision T3620 (CentOS8 or Ubuntu)| 192.168.11.15 |
| worker1 | 192.168.33.101 | DELL precision T3620 (CentOS8 or Ubuntu)| 192.168.11.15 |
|         | 192.168.121.xxx (Internal IP in k8s cluster)| DELL precision T3620 (CentOS8 or Ubuntu)| 192.168.11.15 |
| worker9 | 192.168.33.109 | DELL Optiplex 5050 (Ubuntu)| 192.168.11.23 |
|         | 192.168.121.xxx (Internal IP in k8s cluster)| DELL Optiplex 5050 (Ubuntu)| 192.168.11.23 |

# 1. CentOS8
```
$ sudo dnf install -y epel-release
$ sudo dnf install -y centos-release-openstack-train
$ sudo dnf install openvswitch
$ sudo systemctl enable --now openvswitch
$ sudo ovs-vsctl show
5c4bc60e-e5e6-450a-9a2a-53abd4cb3eb0
    ovs_version: "2.12.0"

$ ifconfig |grep virbr -A 1
virbr0: flags=4099<UP,BROADCAST,MULTICAST>  mtu 1500
        inet 192.168.122.1  netmask 255.255.255.0  broadcast 192.168.122.255
--
virbr1: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.121.1  netmask 255.255.255.0  broadcast 192.168.121.255
--
virbr2: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.33.1  netmask 255.255.255.0  broadcast 192.168.33.255
--
virbr3: flags=4163<UP,BROADCAST,RUNNING,MULTICAST>  mtu 1500
        inet 192.168.133.1  netmask 255.255.255.0  broadcast 192.168.133.255

$ sudo ovs-vsctl add-br br0
$ sudo ovs-vsctl show
5c4bc60e-e5e6-450a-9a2a-53abd4cb3eb0
    Bridge "br0"
        Port "br0"
            Interface "br0"
                type: internal
    ovs_version: "2.12.0"

$ sudo ovs-vsctl add-port br0 virbr2
$ sudo ovs-vsctl show
5c4bc60e-e5e6-450a-9a2a-53abd4cb3eb0
    Bridge "br0"
        Port "virbr2"
            Interface "virbr2"
        Port "br0"
            Interface "br0"
                type: internal
    ovs_version: "2.12.0"

$ sudo ovs-vsctl add-port br0 gre0 -- set interface gre0 type=gre options:remote_ip=192.168.11.23
$ sudo ovs-vsctl show
5c4bc60e-e5e6-450a-9a2a-53abd4cb3eb0
    Bridge "br0"
        Port "virbr2"
            Interface "virbr2"
        Port "br0"
            Interface "br0"
                type: internal
        Port "gre0"
            Interface "gre0"
                type: gre
                options: {remote_ip="192.168.11.23"}
    ovs_version: "2.12.0"
```
- How to enable/disable openvswitch daemon on CentOS/8
```
$ sudo systemctl disable --now openvswitch
$ sudo systemctl enable --now openvswitch
```

# 2. Ubuntu
```
$ sudo apt-get update
$ sudo apt-get install -y openvswitch-switch openvswitch-common

$ sudo ovs-vsctl add-br br0
$ sudo ovs-vsctl show
a69599ba-c200-4138-963f-abf09e94655b
    Bridge br0
        Port br0
            Interface br0
                type: internal
    ovs_version: "2.13.3
    
$ ip a |grep virbr
10: virbr0: <NO-CARRIER,BROADCAST,MULTICAST,UP> mtu 1500 qdisc noqueue state DOWN group default qlen 1000
    inet 192.168.122.1/24 brd 192.168.122.255 scope global virbr0
11: virbr0-nic: <BROADCAST,MULTICAST> mtu 1500 qdisc fq_codel master virbr0 state DOWN group default qlen 1000
13: virbr1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master ovs-system state UP group default qlen 1000
    inet 192.168.121.1/24 brd 192.168.121.255 scope global virbr1
14: virbr1-nic: <BROADCAST,MULTICAST> mtu 1500 qdisc fq_codel master virbr1 state DOWN group default qlen 1000
15: virbr2: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue master ovs-system state UP group default qlen 1000
    inet 192.168.33.1/24 brd 192.168.33.255 scope global virbr2
16: virbr2-nic: <BROADCAST,MULTICAST> mtu 1500 qdisc fq_codel master virbr2 state DOWN group default qlen 1000
17: vnet0: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel master virbr1 state UNKNOWN group default qlen 1000
18: vnet1: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel master virbr2 state UNKNOWN group default qlen 1000
19: virbr3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc noqueue state UP group default qlen 1000
    inet 192.168.133.1/24 brd 192.168.133.255 scope global virbr3
20: virbr3-nic: <BROADCAST,MULTICAST> mtu 1500 qdisc fq_codel master virbr3 state DOWN group default qlen 1000
21: vnet2: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel master virbr1 state UNKNOWN group default qlen 1000
22: vnet3: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel master virbr2 state UNKNOWN group default qlen 1000
23: vnet4: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel master virbr1 state UNKNOWN group default qlen 1000
24: vnet5: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel master virbr2 state UNKNOWN group default qlen 1000
25: vnet6: <BROADCAST,MULTICAST,UP,LOWER_UP> mtu 1500 qdisc fq_codel master virbr3 state UNKNOWN group default qlen 1000

$ sudo ovs-vsctl show
a69599ba-c200-4138-963f-abf09e94655b
    Bridge br0
        Port br0
            Interface br0
                type: internal
    ovs_version: "2.13.3"

$ sudo ovs-vsctl add-port br0 virbr2
$ sudo ovs-vsctl show
a69599ba-c200-4138-963f-abf09e94655b
    Bridge br0
        Port br0
            Interface br0
                type: internal
        Port virbr2
            Interface virbr2
    ovs_version: "2.13.3"

$ sudo ovs-vsctl add-port br0 gre0 -- set interface gre0 type=gre options:remote_ip=192.168.11.15
$ sudo ovs-vsctl show
a69599ba-c200-4138-963f-abf09e94655b
    Bridge br0
        Port br0
            Interface br0
                type: internal
        Port virbr2
            Interface virbr2
        Port gre0
            Interface gre0
                type: gre
                options: {remote_ip="192.168.11.15"}
    ovs_version: "2.13.3"
```
- How to enable/disable openvswitch daemon on Ubuntu
```
$ sudo sudo systemctl disable --now openvswitch-switch
$ sudo sudo systemctl enable --now openvswitch-switch
```

# 3. Ping between master and worker9
```
vagrant@master:~$ ping 192.168.33.109
PING 192.168.33.109 (192.168.33.109) 56(84) bytes of data.
64 bytes from 192.168.33.109: icmp_seq=1 ttl=64 time=3.52 ms
64 bytes from 192.168.33.109: icmp_seq=2 ttl=64 time=1.11 ms
64 bytes from 192.168.33.109: icmp_seq=3 ttl=64 time=1.06 ms
64 bytes from 192.168.33.109: icmp_seq=4 ttl=64 time=1.02 ms
^C
--- 192.168.33.109 ping statistics ---
4 packets transmitted, 4 received, 0% packet loss, time 3002ms
rtt min/avg/max/mdev = 1.016/1.678/3.524/1.065 ms
```

# 4. Change MTU size
```
sudo ip link set eth1 mtu 1450
```

# 5. Join the k8s cluster
```
token=$(sudo kubeadm token list |tail -n 1 |awk '{print $1}')
hashkey=$(openssl x509 -pubkey -in /etc/kubernetes/pki/ca.crt | openssl rsa -pubin -outform der 2>/dev/null | openssl dgst -sha256 -hex | sed 's/^.* //')
ssh vagrant@192.168.33.101 sudo kubeadm join 192.168.33.100:6443 --token $token --discovery-token-ca-cert-hash sha256:$hashkey
ssh vagrant@192.168.33.109 sudo kubeadm join 192.168.33.100:6443 --token $token --discovery-token-ca-cert-hash sha256:$hashkey
```

# 6. Install Istio
```
cd /home/vagrant
curl -L https://istio.io/downloadIstio | sh -
export PATH="$PATH:/home/vagrant/istio-1.12.0/bin"
echo y | istioctl install
kubectl label namespaces default istio-injection=enabled
kubectl get ns --show-labels
cd istio-1.12.0/samples/addons
kubectl apply -f .
```

# 7. Install Metallb-system
```
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.11.0/manifests/namespace.yaml
kubectl apply -f https://raw.githubusercontent.com/metallb/metallb/v0.11.0/manifests/metallb.yaml
kubectl get nodes -o wide
cat <<EOF | kubectl create -f -
apiVersion: v1
kind: ConfigMap
metadata:
  namespace: metallb-system
  name: config
data:
  config: |
    address-pools:
    - name: default
      protocol: layer2
      addresses:
      - 192.168.121.220-192.168.121.250
EOF
kubectl describe configmap config -n metallb-system
```

# 8. Install GPU operator
- https://github.com/developer-onizuka/gpu-operator3
```
curl -fsSL -o get_helm.sh https://raw.githubusercontent.com/helm/helm/master/scripts/get-helm-3 && chmod 700 get_helm.sh && ./get_helm.sh
helm repo add nvidia https://nvidia.github.io/gpu-operator && helm repo update
helm install --wait --generate-name nvidia/gpu-operator
```

# 9. Run face_recognizer
```
git clone https://github.com/developer-onizuka/openvswitch_install
cd openvswitch_install
kubectl apply -f face_recognizer_istio.yaml
```
```
# kubectl get pods -o wide
NAME                                                              READY   STATUS    RESTARTS      AGE   IP              NODE      NOMINATED NODE   READINESS GATES
facerecognizer-test-67d544f588-gpgb7                              2/2     Running   0             26s   10.10.215.15    worker9   <none>           <none>
facerecognizer-test-67d544f588-w6p62                              2/2     Running   0             26s   10.10.235.151   worker1   <none>           <none>
gpu-operator-1637593355-node-feature-discovery-master-75db6gr6c   2/2     Running   1 (36m ago)   38m   10.10.219.66    master    <none>           <none>
gpu-operator-1637593355-node-feature-discovery-worker-fxlrq       2/2     Running   3 (37m ago)   38m   10.10.215.4     worker9   <none>           <none>
gpu-operator-1637593355-node-feature-discovery-worker-pkjwr       2/2     Running   4 (37m ago)   38m   10.10.235.140   worker1   <none>           <none>
gpu-operator-1637593355-node-feature-discovery-worker-zrgsl       2/2     Running   1 (36m ago)   38m   10.10.219.67    master    <none>           <none>
gpu-operator-5f8b7c4f59-b4rnv                                     2/2     Running   1 (37m ago)   38m   10.10.219.65    master    <none>           <none>

# kubectl get services
NAME                                                    TYPE        CLUSTER-IP      EXTERNAL-IP   PORT(S)    AGE
facerecognizer-svc                                      ClusterIP   10.109.55.27    <none>        5000/TCP   4s
gpu-operator                                            ClusterIP   10.109.95.20    <none>        8080/TCP   32m
gpu-operator-1637593355-node-feature-discovery-master   ClusterIP   10.101.39.243   <none>        8080/TCP   34m
kubernetes                                              ClusterIP   10.96.0.1       <none>        443/TCP    57m
```

# 10. Istio Ingress Contoller
```
cat <<EOF | kubectl apply -f -
apiVersion: networking.k8s.io/v1
kind: IngressClass
metadata:
  name: istio
spec:
  controller: istio.io/ingress-controller
```

# 11. Ingress for kiali and grafana
```
cat <<EOF | kubectl apply -f -
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: myingress
  namespace: istio-system
spec:
  ingressClassName: istio
  defaultBackend:
    resource:
      apiGroup: myingress.com
      kind: StorageBucket
      name: static-assets
  rules:
  - host: myingress.com
    http:
      paths:
      - path: /kiali
        pathType: Prefix
        backend: 
          service:
            name: kiali
            port:
              number: 20001
      - path: /grafana
        pathType: Prefix
        backend: 
          service:
            name: grafana
            port:
              number: 3000
EOF
```

# X. Vagrantfiles
- Master/Worker1 (Ubuntu)
```
# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  config.vm.box = "generic/ubuntu2004"
  #config.vm.box = "roboxes/ubuntu2004"
#---------- master ----------
  config.vm.define "master_192.168.33.100" do |server|
    server.vm.network "private_network", ip: "192.168.33.100"
    server.vm.hostname = "master"
    server.vm.provider "libvirt" do |kvm|
      kvm.memory = 8192 
      kvm.cpus = 2
      kvm.machine_type = "q35"
      kvm.cpu_mode = "host-passthrough"
      kvm.kvm_hidden = true
      kvm.pci :bus => '0x06', :slot => '0x10', :function => '0x0'
    end
    server.vm.provision "shell", inline: <<-SHELL
      sudo ip link set eth1 mtu 1450
      sudo swapoff -a
      sudo sed -i "/swap/d" /etc/fstab
      sudo cat <<EOF > /etc/modprobe.d/blacklist-nouveau.conf
blacklist nouveau
options nouveau modeset=0
EOF
      sudo update-initramfs -u 
      sudo apt-get update
      sudo apt-get install -y curl
      sudo apt-get install -y docker.io
      sudo cat <<EOF > /etc/docker/daemon.json
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "storage-driver": "overlay2"
}
EOF
      sudo systemctl enable docker
      sudo systemctl daemon-reload
      sudo systemctl restart docker
      sleep 120
      sudo apt-get install -y sshpass
      ssh-keygen -t rsa -N '' -f ~/.ssh/id_rsa <<< y
      cat <<EOF > ~/.ssh/config
host 192.168.33.*
   StrictHostKeyChecking no
EOF
      chmod 600 ~/.ssh/config
      sshpass -p "vagrant" ssh-copy-id -i ~/.ssh/id_rsa.pub vagrant@192.168.33.101
      #sshpass -p "vagrant" ssh-copy-id -i ~/.ssh/id_rsa.pub vagrant@192.168.33.102
      sshpass -p "vagrant" ssh-copy-id -i ~/.ssh/id_rsa.pub vagrant@192.168.33.109
      curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
      sudo cat <<EOF | sudo tee /etc/apt/sources.list.d/kubernetes.list
deb https://apt.kubernetes.io/ kubernetes-xenial main
EOF
      sudo apt-get update
      sudo apt-get install -y -q kubelet kubectl kubeadm
      sudo kubeadm init --pod-network-cidr=10.10.0.0/16 --apiserver-advertise-address=192.168.33.100
      mkdir -p $HOME/.kube
      sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
      sudo chown $(id -u):$(id -g) $HOME/.kube/config
      sudo kubectl taint nodes --all node-role.kubernetes.io/master-
      token=$(sudo kubeadm token list |tail -n 1 |awk '{print $1}')
      hashkey=$(openssl x509 -pubkey -in /etc/kubernetes/pki/ca.crt | openssl rsa -pubin -outform der 2>/dev/null | openssl dgst -sha256 -hex | sed 's/^.* //')
      ssh vagrant@192.168.33.101 sudo kubeadm join 192.168.33.100:6443 --token $token --discovery-token-ca-cert-hash sha256:$hashkey
      #ssh vagrant@192.168.33.102 sudo kubeadm join 192.168.33.100:6443 --token $token --discovery-token-ca-cert-hash sha256:$hashkey
      ssh vagrant@192.168.33.109 sudo kubeadm join 192.168.33.100:6443 --token $token --discovery-token-ca-cert-hash sha256:$hashkey
      sudo kubectl label node worker1 node-role.kubernetes.io/node=worker1
      #sudo kubectl label node worker2 node-role.kubernetes.io/node=worker2
      sudo kubectl label node worker9 node-role.kubernetes.io/node=worker9
      sudo kubectl apply -f https://docs.projectcalico.org/manifests/calico.yaml
      #sudo kubectl apply -f https://github.com/antrea-io/antrea/releases/download/v1.2.3/antrea.yml
      #sudo kubectl apply -f https://raw.githubusercontent.com/coreos/flannel/master/Documentation/kube-flannel.yml
      kubectl set env daemonset/calico-node -n kube-system IP_AUTODETECTION_METHOD=interface=eth0,eth1
      cat <<EOF > /etc/rc.local
#!/bin/sh
sudo ip link set eth1 mtu 1450
EOF
      chmod 775 /etc/rc.local
      sudo sed -i "/swap/d" /etc/fstab
      sudo cat /etc/fstab
      sudo sed -i '2i \ \ "insecure-registries":["192.168.11.15:5000"],' /etc/docker/daemon.json
    SHELL
  end
#---------- worker1 ----------
  config.vm.define "worker1_192.168.33.101" do |server|
    server.vm.network "private_network", ip: "192.168.33.101"
    server.vm.hostname = "worker1"
    server.vm.provider "libvirt" do |kvm|
      kvm.memory = 16384
      kvm.cpus = 2
      kvm.machine_type = "q35"
      kvm.cpu_mode = "host-passthrough"
      kvm.kvm_hidden = true
      kvm.pci :bus => '0x06', :slot => '0x10', :function => '0x4'
      kvm.pci :bus => '0x01', :slot => '0x00', :function => '0x0'
    end
    server.vm.provision "shell", inline: <<-SHELL
      sudo ip link set eth1 mtu 1450
      sudo swapoff -a
      sudo sed -i "/swap/d" /etc/fstab
      sudo cat <<EOF > /etc/modprobe.d/blacklist-nouveau.conf
blacklist nouveau
options nouveau modeset=0
EOF
      sudo update-initramfs -u 
      sudo apt-get update
      sudo apt-get install -y curl
      sudo apt-get install -y docker.io
      sudo cat <<EOF > /etc/docker/daemon.json
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "storage-driver": "overlay2"
}
EOF
      sudo systemctl enable docker
      sudo systemctl daemon-reload
      sudo systemctl restart docker
      curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
      sudo cat <<EOF | sudo tee /etc/apt/sources.list.d/kubernetes.list
deb https://apt.kubernetes.io/ kubernetes-xenial main
EOF
      sudo apt-get update
      sudo apt-get install -y -q kubelet kubectl kubeadm
      cat <<EOF > /etc/rc.local
#!/bin/sh
sudo ip link set eth1 mtu 1450
EOF
      chmod 775 /etc/rc.local
      sudo sed -i "/swap/d" /etc/fstab
      sudo sed -i '2i \ \ "insecure-registries":["192.168.11.15:5000"],' /etc/docker/daemon.json
    SHELL
  end
end
```
- Worker9 (Ubuntu)
```
# -*- mode: ruby -*-
# vi: set ft=ruby :

Vagrant.configure("2") do |config|
  config.vm.box = "generic/ubuntu2004"
  #config.vm.box = "roboxes/ubuntu2004"
#---------- worker9 ----------
  config.vm.define "worker9_192.168.33.109" do |server|
    server.vm.network "private_network", ip: "192.168.33.109"
    server.vm.hostname = "worker9"
    server.vm.provider "libvirt" do |kvm|
      kvm.memory = 16384
      kvm.cpus = 2
      kvm.machine_type = "q35"
      kvm.cpu_mode = "host-passthrough"
      kvm.kvm_hidden = true
      kvm.pci :bus => '0x01', :slot => '0x00', :function => '0x0'
    end
    server.vm.provision "shell", inline: <<-SHELL
      sudo ip link set eth1 mtu 1450
      sudo swapoff -a
      sudo sed -i "/swap/d" /etc/fstab
      sudo cat <<EOF > /etc/modprobe.d/blacklist-nouveau.conf
blacklist nouveau
options nouveau modeset=0
EOF
      sudo update-initramfs -u 
      sudo apt-get update
      sudo apt-get install -y curl
      sudo apt-get install -y docker.io
      sudo cat <<EOF > /etc/docker/daemon.json
{
  "exec-opts": ["native.cgroupdriver=systemd"],
  "log-driver": "json-file",
  "log-opts": {
    "max-size": "100m"
  },
  "storage-driver": "overlay2"
}
EOF
      sudo systemctl enable docker
      sudo systemctl daemon-reload
      sudo systemctl restart docker
      curl -s https://packages.cloud.google.com/apt/doc/apt-key.gpg | sudo apt-key add -
      sudo cat <<EOF | sudo tee /etc/apt/sources.list.d/kubernetes.list
deb https://apt.kubernetes.io/ kubernetes-xenial main
EOF
      sudo apt-get update
      sudo apt-get install -y -q kubelet kubectl kubeadm
      cat <<EOF > /etc/rc.local
#!/bin/sh
sudo ip link set eth1 mtu 1450
EOF
      chmod 775 /etc/rc.local
      sudo sed -i "/swap/d" /etc/fstab
      sudo sed -i '2i \ \ "insecure-registries":["192.168.11.15:5000"],' /etc/docker/daemon.json
    SHELL
  end
#--------------------
end
```

