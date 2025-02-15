---

## Table of Contents
1. [Deploy a New Kubernetes Master Node](#deploy-a-new-kubernetes-master-node)
2. [Copy the Backup to the Test Machine](#copy-the-backup-to-the-test-machine)
3. [Stop etcd on the Test Cluster](#stop-etcd-on-the-test-cluster)
4. [Remove Old etcd Data](#remove-old-etcd-data)
5. [Restore etcd Backup on the Test Cluster](#restore-etcd-backup-on-the-test-cluster)
6. [Start etcd](#start-etcd)
7. [Verify etcd is Running](#verify-etcd-is-running)
8. [Restart Kubernetes Components](#restart-kubernetes-components)
9. [Verify Kubernetes is Restored](#verify-kubernetes-is-restored)
10. [Final Steps: Testing in the New Environment](#final-steps-testing-in-the-new-environment)
11. [Summary of Key Commands](#summary-of-key-commands)

---

## Deploy a New Kubernetes Master Node

1. On the test machine, initialize the Kubernetes master node:
   ```
   sudo kubeadm init --pod-network-cidr=192.168.1.0/16
   ```
2. Wait for the initialization to complete.
3. Move your kubeconfig:
   ```
   mkdir -p $HOME/.kube
   sudo cp -i /etc/kubernetes/admin.conf $HOME/.kube/config
   sudo chown $(id -u):$(id -g) $HOME/.kube/config
   ```

---

## Copy the Backup to the Test Machine

1. From your production environment, transfer the etcd backup to the test machine:
   ```
   scp /backup/etcd-snapshot.db user@your-test-node:/home/user/
   ```
2. On the test machine, move the backup to the desired location:
   ```
   sudo mv /home/user/etcd-snapshot.db /backup/
   ```

---

## Stop etcd on the Test Cluster

1. Stop the etcd service:
   ```
   sudo systemctl stop etcd
   ```
2. Verify that etcd has stopped:
   ```
   sudo systemctl status etcd
   ```

---

## Remove Old etcd Data

1. Move the old etcd data:
   ```
   sudo mv /var/lib/etcd /var/lib/etcd.old
   ```
2. Create a new directory for etcd:
   ```
   sudo mkdir -p /var/lib/etcd
   ```

---

## Restore etcd Backup on the Test Cluster

1. Restore the etcd backup:
   ```
   ETCDCTL_API=3 etcdctl snapshot restore /backup/etcd-snapshot.db --data-dir=/var/lib/etcd
   ```

---

## Start etcd

1. Restart the etcd service:
   ```
   sudo systemctl restart etcd
   ```
2. Verify the status of etcd:
   ```
   sudo systemctl status etcd
   ```

---

## Verify etcd is Running

1. Check the etcd endpoint status:
   ```
   ETCDCTL_API=3 etcdctl --endpoints=https://127.0.0.1:2379
     --cacert=/etc/kubernetes/pki/etcd/ca.crt
     --cert=/etc/kubernetes/pki/etcd/server.crt
     --key=/etc/kubernetes/pki/etcd/server.key
     endpoint status --write-out=table
   ```

---

## Restart Kubernetes Components

1. Restart the Kubernetes components:
   ```
   sudo systemctl restart kube-apiserver kube-controller-manager kube-scheduler kubelet
   ```

---

## Verify Kubernetes is Restored

1. Check the cluster status:
   ```
   kubectl get nodes
   ```
2. Verify that all pods are restored:
   ```
   kubectl get pods -A
   ```

---

## Final Steps: Testing in the New Environment

1. Access services:
   ```
   kubectl get svc -A
   ```
2. Check deployments:
   ```
   kubectl get deployments -A
   ```
3. If some pods fail, check their logs:
   ```
   kubectl logs -f <pod-name> -n <namespace>
   ```

---

## Summary of Key Commands

| Step                            | Command                                                                                |
| ------------------------------- | -------------------------------------------------------------------------------------- |
| Copy backup to test environment | `scp /backup/etcd-snapshot.db user@your-test-node:/home/user/`                         |
| Stop etcd                       | `sudo systemctl stop etcd`                                                             |
| Remove old etcd data            | `sudo mv /var/lib/etcd /var/lib/etcd.old && sudo mkdir -p /var/lib/etcd`               |
| Restore etcd backup             | `etcdctl snapshot restore /backup/etcd-snapshot.db --data-dir=/var/lib/etcd`           |
| Restart etcd                    | `sudo systemctl restart etcd`                                                          |
| Verify etcd status              | `etcdctl endpoint status --write-out=table`                                            |
| Restart Kubernetes components   | `sudo systemctl restart kube-apiserver kube-controller-manager kube-scheduler kubelet` |
| Verify cluster                  | `kubectl get nodes`                                                                    |

---
