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

Tailscale
it was so fun to use tailscale, feel opened a new door! next step is to go further to learn about ACL to limite the risk!


### Observability stack

Prometheus(deployed on k3s with PV on mini PC)
Grafana(deployed on k3s with PV on mini PC)
Loki(in progress) 


### plan


tbc


### log

**beginning and mid-august**

started the homelab, started with designing and building the infra, deploy monitoring etc...got a lot error, fiexd them, whent further, got more error...it's fine, can feel I become capable quickly after those errors!

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


**2025.08.27**

fixd music streaming based on tailscale. This is super cool service! both me and my partner benifited of it.

**2025.08.29**

The nas side was prepared, gonna start to migrate all the pv to nas（all the pv is hanging under my mini pc via nfs now before nas preprared）


**2025.08.31**
Got a new approch to migrate the pvs!!! Truenas CSI, sounds a perfect solution, sounds much better than manuelly use UID and GID for nfs!!!


**2025.09.01**

Harbor suddenly returned 502. I thought maybe the containers had crashed, so I decided to run docker compose down and then bring them back up. But before running up, I forgot to run ./prepare, which caused up to wipe out all my images. Harbor became accessible again, but everything was reset to zero. A huge operations disaster! I had no choice but to pull down all the images one by one, retag them, and push them back into the registry… I almost cried. I will never randomly run up again!!!!!!

**2025.09.02**

While learning Calico’s architecture, the lecturer mentioned that Calico’s VXLAN technology can perform encapsulation and decapsulation directly in kernel space. This allows it to bypass the interactions (copies) between user space and kernel space, letting operations run directly in kernel mode. I was a bit confused, so I went back to review the concepts of user space and kernel space. Suddenly I realized their architecture logic is very similar to aircraft systems.

(This summer I actually experienced a Boeing 737 flight simulator. I was shocked at how primitive the operating system looked. Later I learned that this was because it had to run on a highly stable RTOS. Meanwhile, other aircraft systems such as the in-flight entertainment system are more like user-space extensions for scalability. This deepened my understanding of ELISA’s work which I now see as providing a stable Linux kernel integration, or integrating existing RTOS solutions.)

Kernel space must remain absolutely stable, so it restricts the communication channels and enforces specific rules (syscalls, data copies, Ring 0 etc... I also revisited the concept of CPU privilege rings). User space normally runs in Ring 3, while kernel space operations run in Ring 0. To cross from Ring 3 to Ring 0, a syscall is required.

This suddenly made me realize: this architecture and interaction model is very similar to how in Kubernetes, all other components must go through the API server in order to read or write data in etcd! Following that logic, it becomes clear why the architecture is designed this way: for security and consistency.

**2025.09.09**

I’ve realized how important it is to balance rest and study. Last week I sat at my computer for too long without noticing, neglected exercise, and overworked my brain. My body sent an alarm: my neck hurt so badly I couldn’t lift my head, and I felt dizzy and exhausted. I decided to take it seriously, restarted my workouts, and after a few days of full rest plus more physical activity, I felt alive again.

This week our classes focused on Traefik. I had worked with Traefik before since it is the default Ingress Controller in K3s, but in my previous usage it was already abstracted by Kubernetes so I just had to call it without thinking too much. This time, the course broke down Traefik’s components and how it actually works, and I learned a lot.

What struck me most was the design idea of how Traefik listens to providers. It gave me a new perspective on how a standardized interface can be used across platforms. For example, with Docker in class: Traefik listens to the socket and consumes JSON-formatted data from it, then dynamically updates routing rules to enable reverse proxy and load balancing. In this way it builds an abstraction layer over the raw socket traffic, hiding away the traditional networking complexity. That is what allows it to work across hybrid environments and it was truly eye-opening.

Then it hit me: as long as the provider returns data in the format Traefik expects (JSON), I could connect it to any system I want. That thought made me both happy and excited, as if I had just discovered a truth (haha). And then it made sense why Kubernetes itself was designed so that everything is exposed as RESTful resources. Once HTTP/HTTPS is solved, you get incredible flexibility and portability.

While watching another video today, I also came across C/S vs. B/S architecture and suddenly realized there is an entire ecosystem and philosophy behind these patterns, which is fascinating.

Yesterday I also learned about the CAP theorem, and it connected so many dots in my earlier understanding of system design. For example, last week I had an insight about system layering in critical systems: the core layers follow CP (consistency plus partition tolerance), while the edge layers are more AP (availability plus partition tolerance). That revelation was liberating.

With this discovery, I have become even more interested in architecture. I have decided to systematically read some books on the topic. I was recommended Designing Data-Intensive Applications (DDIA) and Google’s Site Reliability Engineering. Hopefully I will get the chance to discuss and exchange thoughts with people who have read them too.



**2025.09.17**

My cluster went through another trial today.
To improve our home network speed, my boyfriend bought a new 10 Gbps router. To minimize disruption to the existing IP layout, we kept the old IP plan but re-cabled and changed the placement of servers and other devices. Since everything had to be moved physically, I had to shut down all the equipment before rearranging it.

I was extremely nervous. The memory of last time’s sudden power outage disaster was still vivid. But this time, since it was a planned shutdown, I could at least be sure that any service with persistent volumes would not lose data.

After moving the devices and powering them back on, Kubernetes quickly brought the cluster back up. All services returned, including Pi-hole, the one that had caused the previous disaster. This made me appreciate Kubernetes even more, it is just so convenient.

However, Harbor died again with a 502 error. Since Harbor is deployed directly with Docker Compose on the server, I did a compose down and compose up, but then the painful sight appeared: my images were wiped again.

For a moment, I felt complete despair. I had run prepare.sh beforehand, convinced that last time’s mistake would not repeat, but the images still vanished. That meant my previous experience was not enough to save me this time.

Starting troubleshooting from scratch, I discovered Harbor itself was fine. The real issue was that after restarting, the persistent volume was pointing to the wrong path. Later I found out it was actually a permission problem. PostgreSQL did not have the right permissions to access its directory, so it created a new one. Why did I not notice this earlier? Because last time I just used sudo to bypass the permission errors instead of fixing them properly.

The old images were still there, sitting in the original path under registry/docker/registry/v2/blobs/sha256/.

I went back into Kubernetes, retrieved the list of images in use (kubectl get pods -A -o jsonpath='{.spec.containers[*].image}'), and from that list re-pulled, re-tagged, and re-pushed them into Harbor. For Pi-hole, since it runs on an ARM node, I had to push multi-arch images.

I spent the entire evening on this, but in the end, it reinforced the Docker Compose knowledge I have been learning at school.
I have also become less afraid of shutting down servers or even power outages.

And now with the new setup, our home network is blazing fast. I could not be happier.

