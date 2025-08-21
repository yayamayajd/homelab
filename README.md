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

**Current Network** 

Nginx on the mini PC handles load balancing and redirection:
   web
    |
   Nginx on mini pc(443 TLS termination)--->Harbor
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


**plan**


tbc


**log**


tbc