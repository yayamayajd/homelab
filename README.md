# homelab

building my homelab to host my projects! having lots of fun with infra experiments!



## Purpose ##

The idea is to build a personal playground for learning — a place where I can host infrastructure, programming projects, CI/CD pipelines, and other experiments. By getting hands-on, I hope to deepen my knowledge step by step. At the same time, I also want this setup to serve daily needs such as streaming, storage, and a centralized IoT system. In short, this is meant to be my own base.



## Resource

-	**Hardware**: one mini PC as the main server, one Raspberry Pi 4, one all-flash NAS, and an L2 switch

-	**Future plan**: add another mini PC and combine it with the NAS for high availability



## Structure

### K3s cluster:

-	Mini PC (x86) running Ubuntu LTS 24, acting as the main server
-	Raspberry Pi (ARM) running Ubuntu Server, forming a heterogeneous cluster with the mini PC
-	The mini PC handles compute workloads, while the Raspberry Pi is used to experiment with IoT edge applications and DNS resolution
-	Due to limited hardware, etcd HA is postponed for now, but I plan to gradually expand later


### NAS: 

running **TrueNAS**, continuing its role as a NAS:
- providing NFS persistent volumes for Kubernetes applications
- storing important household data
- sharing files with Apple devices at home via SMB




### Architecture (evolving)

**mini pc**

-	Harbor as a private container registry
-	Docker for building, pulling, tagging images, and pushing them to Harbor
-	K3s master node and control plane, switched from SQLite to etcd

**raspberryPi**：

-	Pi-hole acting as DNS server and ad-blocker

**Current Network**:

Nginx on the mini PC handles load balancing and redirection.

   Web
    |
   Nginx on miniPC(443 TLS termination)--->Harbor
                                      |--->Traefik Ingress(k3s)
                                                |-->services(Cluster ip)
                                                        |-->Pods(Deployment,Daemoset)


**DNS resolution path**:

    client(http/https request)
       |
    DNS server (Pi-hole, returns mini PC’s IP)
       |
    nginx on mini PC

**Remote access**

Tailscale (in progress) 


### Observability stack

Prometheus(deployed on k3s with PV on mini PC)
Grafana(deployed on k3s with PV on mini PC)
Loki(in progress) 


### plan


tbc


### log


**2025.08.24**

Since the beginning of this month, after I got my mini PC, I have been working on the infra. From choosing the OS, setting up the network, to practicing Pod deployment, I met many errors, but also learned a lot through hands-on work.
I kept notes of the errors I faced. If similar issues happen again, I can quickly find the cause. I plan to write in my log about my infra experience and thoughts.

Today I will start with the choice of operating system.

For the mini PC as the network center server, I first ruled out Windows. Then I hesitated for a long time between installing Linux directly on bare metal, or using Proxmox and then running Linux VMs on it. Proxmox makes it easy to start VMs and containers, but it also adds another layer of virtual networking, and it takes some system resources itself. Since I plan to mainly run a Kubernetes cluster on this server, and all apps will be deployed inside Pods, I finally gave up Proxmox and chose to install Linux directly. About the Linux distribution, I also thought a lot. At a meetup, people suggested Talos, which is designed for Kubernetes. At the ELISA workshop, a Volvo DevOps engineer suggested NixOS, which is good for understanding the installation and execution chain of apps. In the end, I chose Ubuntu, which I know best. For a master node of the cluster, I want something I can control and keep stable.

For the Raspberry Pi, I also chose Ubuntu. But installing the system took me much more time. The problem was that I didn’t have a micro HDMI cable for Raspberry Pi, so I could not connect it to a monitor. I decided to try a headless installation. I wanted it to install by itself after power on, and then I could directly ssh into it. So I had to pre-configure the user, password, ssh status, and static IP.

About the choice of Kubernetes version:

From the start, I planned to set up a k3s cluster at home. Why k3s instead of full k8s? First, my home resources are too limited for the full version. Full k8s would be too heavy. I didn’t choose microk8s or minikube either. Minikube is only single node, and microk8s does not support heterogeneous clusters as well as k3s. k3s is the best solution for edge devices. I have one Raspberry Pi, which I originally bought to run monitoring because my old NAS was too weak. But with the hardware upgrade of my homelab, the old NAS is now retired, so my plan for the Raspberry Pi has also changed. I want to add it to the cluster as a lightweight worker node. So k3s is my best choice.

For storage, I use a new NAS with TrueNAS installed. It provides NFS for Persistent Volumes, so that the mini PC is in charge of computing, while the NAS is in charge of storage.


**2025.08.25**

fied tailscale, now i can visit home network from outside!