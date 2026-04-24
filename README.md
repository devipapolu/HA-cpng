
🏷️ Title
**Highly Available PostgreSQL Cluster on Kubernetes using CloudNativePG, WAL Replication & HAProxy**

📝 Description
This project implements a highly available PostgreSQL database cluster on a multi-node Kubernetes environment using an operator-based approach.
The system ensures:
Continuous availability of database
Automatic failover when primary node fails
Self-healing by recreating failed pods
Data consistency using WAL-based replication
Load balancing using HAProxy

**🛠️ Tools & Technologies**

Kubernetes (kubeadm)
CloudNativePG
PostgreSQL
HAProxy
containerd (container runtime)
Flannel (CNI networking)
kubelet, kubeadm, kubectl
Linux (Ubuntu)

**🌐 Architecture**

Client
   ↓
HAProxy (Control Plane - 190)
   ↓
NodePort Service
   ↓
----------------------------------
| Kubernetes Cluster             |
|                                |
| Node 190 → Primary DB          |
| Node 188 → Replica DB          |
| Node 187 → Replica DB          |
----------------------------------

**⚙️ Steps to Complete This Project (with Commands & Explanation)**

🔹 Step 1: System Setup (All Nodes)

sudo apt update && sudo apt upgrade -y
👉 System packages update

sudo apt install -y curl wget git vim apt-transport-https ca-certificates software-properties-common
👉 Required tools install

🔹 Step 2: Install Container Runtime

sudo apt install -y containerd
👉 Containers run cheyadaniki runtime

sudo mkdir -p /etc/containerd
sudo containerd config default | sudo tee /etc/containerd/config.toml
👉 Default config create

sudo systemctl restart containerd
sudo systemctl enable containerd
👉 Service start & enable

🔹 Step 3: Install Kubernetes

sudo apt install -y kubelet kubeadm kubectl
sudo apt-mark hold kubelet kubeadm kubectl
👉 Kubernetes components install

sudo systemctl enable kubelet
sudo systemctl start kubelet

🔹 Step 4: Initialize Control Plane (190)

kubeadm init --pod-network-cidr=10.244.0.0/16
👉 Creates:

API Server
Scheduler
Controller Manager
etcd

🔹 Step 5: Join Worker Nodes
kubeadm join <MASTER-IP>:6443 --token <TOKEN> --discovery-token-ca-cert-hash sha256:<HASH>
👉 Worker nodes cluster lo join avutayi

🔹 Step 6: Install Networking (Flannel)
kubectl apply -f kube-flannel.yml
👉 Pod-to-pod communication enable
👉 Each node ki subnet assign

🔹 Step 7: Install CNPG

kubectl apply -f https://raw.githubusercontent.com/cloudnative-pg/cloudnative-pg/release-1.22/releases/cnpg-1.22.0.yaml
👉 PostgreSQL operator install

🔹 Step 8: Create Secret

kubectl create secret generic pg-secret \
--from-literal=username=postgres \
--from-literal=password=postgres
👉 Database credentials store

🔹 Step 9: Create PostgreSQL Cluster

apiVersion: postgresql.cnpg.io/v1
kind: Cluster
metadata:
  name: pg-cluster
spec:
  instances: 3

  storage:
    size: 1Gi

  bootstrap:
    initdb:
      database: appdb
      owner: appuser
      secret:
        name: pg-secret

  superuserSecret:
    name: pg-secret

command:
kubectl apply -f pg-cluster.yaml

👉 3 PostgreSQL pods create avutayi

🔹 Step 10: Expose Service
apiVersion: v1
kind: Service
metadata:
  name: pg-service
spec:
  type: NodePort
  selector:
    cnpg.io/cluster: pg-cluster
  ports:
    - port: 5432
      targetPort: 5432
      nodePort: 30096

command:
kubectl apply -f service.yaml

🔹 Step 11: Install HAProxy
sudo apt install -y haproxy

🔹 Step 12: Configure HAProxy
command:
sudo vim /etc/haproxy/haproxy.cfg

frontend postgres_front
    bind *:5432
    default_backend postgres_back

backend postgres_back
    balance roundrobin
    option tcp-check

    server node1 172.30.0.188:30096 check
    server node2 172.30.0.187:30096 check
    server node3 172.30.0.190:30096 check backup

command:
sudo systemctl restart haproxy

👉 Client → HAProxy → DB routing


🔥 METHOD 1: PostgreSQL lo check (BEST & EXACT)

Pod loki velli check chey:

kubectl exec -it <pod-name> -- psql -U postgres

Inside:
SELECT pg_is_in_recovery();

👉 Result:
false → 🟢 Primary
true → 🔵 Replica

👉 Ippudu HAProxy nundi request pampu
👉 Ye pod hit ayyindo logs lo chusi
👉 A pod lo pg_is_in_recovery() run chey

✔ Appudu exact ga telustundi:
HAProxy evariki traffic pampindi

🔥 METHOD 2: Pod logs compare cheyadam

Primary & replica pods logs open chey:

kubectl logs -f pod-primary
kubectl logs -f pod-replica

👉 HAProxy nundi query run chey

✔ Ye pod lo query kanipistundo → adhe receive chesindi


🧠 Internal Process (Control Plane & Worker Node)
🔹 STEP-BY-STEP POD CREATION (CONFUSION CLEAR 🔥)
1. User runs:
kubectl apply -f pg-cluster.yaml

👉 kubectl → API Server

2. API Server:
Validates YAML
Stores desired state in etcd
instances = 3

3. CNPG Operator:
👉 Watches API Server continuously
👉 Detects:

Required = 3 pods
Current = 0

👉 Sends request to API Server:
Create 3 PostgreSQL pods

4. API Server:
Creates Pod objects (not actual containers)

5. Scheduler:
Assigns pods to nodes:

pg-1 → node 190
pg-2 → node 188
pg-3 → node 187

6. kubelet (on each node):
👉 Watches API Server
👉 Sees assigned pod
👉 Calls container runtime
👉 Creates container

🔹 Worker Node Flow
kubelet → API Server watch chestundi
Pod assigned ayite detect chestundi
Container runtime ni call chestundi
Pod create chestundi
Flannel → IP assign chestundi
PostgreSQL start avutundi

🤖 Explanation about CNPG
👉 CloudNativePG
CNPG is a Kubernetes operator that:

PostgreSQL pods create chestundi
Primary & replicas maintain chestundi
Replication setup chestundi
Failover handle chestundi
Failed pods recreate chestundi
👉 Manual DB management avasaram ledu

🌐 Explanation about HAProxy
👉 HAProxy

HAProxy:

Load balancer ga work chestundi
Healthy node ki traffic pampistundi
Failed node ni avoid chestundi
Continuous DB access provide chestundi
🔥 Explanation about WAL'

👉 PostgreSQL uses WAL by default

WAL Flow
Write → Primary
      ↓
WAL create
      ↓
Stream → Replica
      ↓
Replica apply changes

👉 Full database copy kadu
👉 Only changes replicate

👉 WAL use cases:
Crash recovery
Replication
Data consistency

🎯 Conclusion
This project demonstrates:

Kubernetes orchestration
Operator-based automation
High availability architecture
Self-healing system
Distributed database

🚀 Future Updates
Monitoring → Prometheus + Grafana
Alerts → Slack / Email
Backup → WAL + S3
Security → TLS, RBAC
Auto scaling

🔥 Final Interview Line

👉
“Implemented a highly available PostgreSQL cluster on Kubernetes using CloudNativePG with WAL-based replication, automatic failover, and HAProxy load balancing.”
