
# Ansible Role for Kubernetes Cluster Setup

This Ansible role installs and configures a highly available Kubernetes cluster using [kubeadm](https://kubernetes.io/docs/reference/setup-tools/kubeadm), automating the entire installation process. It demonstrates how to use Ansible for Kubernetes setup. For production environments, it is recommended to use [Kubespray](https://kubespray.io).

High availability features include **multiple master nodes** to prevent a single point of failure, **kube-vip** for a failover virtual IP address ensuring control plane accessibility, **Longhorn** for distributed block storage and data resilience, and an **Nginx ingress controller** for load balancing and reliable ingress traffic routing.

## Requirements

Install ansible, ipaddr and netaddr:

```
pip install -r requirements.txt
```

Download the role form GitHub:

```
ansible-galaxy install git+https://github.com/amine-baaa/ansible-role-kubernetes.git
```

## Role Variables

This role accept this variables:

| Var   | Required |  Default | Desc |
| ------- | ------- | ----------- |
| `kubernetes_subnet`       | `yes`       |  `192.168.25.0/24` | Subnet where Kubernetess will be deployed. If the VM or bare metal server has more than one interface, Ansible will filter the interface used by Kubernetes based on the interface subnet |
| `disable_firewall`       | `no`       | `no`       | If set to yes Ansible will disable the firewall.   |
| `kubernetes_version`       | `no`       | `1.24.3`       | Kubernetes version to install  |
| `kubernetes_cri`       | `no`       | `containerd`       | Kubernetes [CRI](https://kubernetes.io/docs/concepts/architecture/cri/) to install.   |
| `kubernetes_cni`       | `no`       | `flannel`       | Kubernetes [CNI](https://github.com/containernetworking/cni) to install.  |
| `kubernetes_dns_domain`       | `no`       | `cluster.local`       | Kubernetes default DNS domain  |
| `kubernetes_pod_subnet`       | `no`       | `10.244.0.0/16`       | Kubernetes pod subnet  |
| `kubernetes_service_subnet`       | `no`       | `10.96.0.0/12`       | Kubernetes service subnet  |
| `kubernetes_api_port`       | `no`       | `6443`       | kubeapi listen port  |
| `setup_vip`       | `no`       | `no`       | Setup kubernetes VIP addres using [kube-vip](https://kube-vip.io/)   |
| `kubernetes_vip_ip`       | `no`       | `192.168.25.225`       | **Required** if setup_vip is set to *yes*. Vip ip address for the control plane  |
| `kubevip_version`       | `no`       | `v0.4.3`       | kube-vip container version  |
| `install_longhorn`       | `no`       | `no`       | Install [Longhorn](#longhorn), Cloud native distributed block storage for Kubernetes.  |
| `longhorn_version`       | `no`       | `v1.3.1`       | Longhorn release.  |
| `install_nginx_ingress`       | `no`       | `no`       | Install [nginx ingress controller](#nginx-ingress-controller)  |
| `nginx_ingress_controller_version`       | `no`       | `controller-v1.3.0`       | nginx ingress controller version  |
| `nginx_ingress_controller_http_nodeport`       | `no`       | `30080`       | NodePort used by nginx ingress controller for the incoming http traffic  |
| `nginx_ingress_controller_https_nodeport`       | `no`       | `30443`       |  NodePort used by nginx ingress controller for the incoming https traffic  |
| `enable_nginx_ingress_proxy_protocol`       | `no`       | `no`       | Enable  nginx ingress controller proxy protocol mode |
| `enable_nginx_real_ip`       | `no`       | `no`       | Enable nginx ingress controller real-ip module |
| `nginx_ingress_real_ip_cidr`       | `no`       | `0.0.0.0/0`       | **Required** if enable_nginx_real_ip is set to *yes* Trusted subnet to use with the real-ip module  |
| `nginx_ingress_proxy_body_size`       | `no`       | `20m`       | nginx ingress controller max proxy body size  |
| `sans_base`       | `no`       | `[list of values, see defaults/main.yml]`       | list of ip addresses or FQDN uset to sign the kube-api certificate  |










- Use [Vagrant](https://www.vagrantup.com) and VirtualBox to test the role by bringing up an example infrastructure. After downloading this repository, start the virtual machines with:

```
vagrant up
```

## Using this role

To use this role you follow the example in the [examples/] dir. Adjust the hosts.ini file with your hosts and run the playbook:

```
user@mintrrr:~$ ansible-playbook -i hosts-ubuntu.ini site.yml -e kubernetes_init_host= 

PLAY [kubemaster]  

TASK [Gathering Facts]

TASK [ansible-role-kubernetes : include_tasks]  

TASK [ansible-role-kubernetes : Install required system packages]

TASK [ansible-role-kubernetes : Add Google GPG apt Key]  

TASK [ansible-role-kubernetes : Add K8s Repository]

TASK [ansible-role-kubernetes : Add Docker GPG apt Key]  

TASK [ansible-role-kubernetes : shell]

TASK [ansible-role-kubernetes : Add Docker Repository]

TASK [ansible-role-kubernetes : setup]

TASK [ansible-role-kubernetes : include_tasks]  

TASK [ansible-role-kubernetes : disable ufw]

TASK [ansible-role-kubernetes : Install iptables-legacy]   

TASK [ansible-role-kubernetes : Remove zram-generator-defaults]  

TASK [ansible-role-kubernetes : disable firewalld]

TASK [ansible-role-kubernetes : Put SELinux in permissive mode, logging actions that would be blocked.] 

TASK [ansible-role-kubernetes : Disable SELinux]   

TASK [ansible-role-kubernetes : Install openssl]   

TASK [ansible-role-kubernetes : load overlay kernel module]   

TASK [ansible-role-kubernetes : load br_netfilter kernel module]       


TASK [ansible-role-kubernetes : Add KUBELET_ROOT_DIR env var]

TASK [ansible-role-kubernetes : Add KUBELET_ROOT_DIR env var, set value]

TASK [ansible-role-kubernetes : Install longhorn]

TASK [ansible-role-kubernetes : Install longhorn storageclass]

TASK [ansible-role-kubernetes : include_tasks]  

TASK [ansible-role-kubernetes : Check if ingress-nginx is installed]      

TASK [ansible-role-kubernetes : Install ingress-nginx]

TASK [ansible-role-kubernetes : render nginx_ingress_config.yml]       

TASK [ansible-role-kubernetes : Apply nginx ingress config]   

```


```
kubectl get nodes  
```


```
kubectl get pods --all-namespaces
```


We can also inspect the service of the nginx ingress controller:

```
kubectl get svc -n ingress-nginx
```

From an external machine we can test the ingress controller:

```
user@mintrrr:~$ curl -v http://[HOST]:[HTTP_PORT]
> 
< 
```