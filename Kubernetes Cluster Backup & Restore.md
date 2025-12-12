**What is Cluster Backup.**

A Kubernetes cluster backup is the process of safely saving all critical cluster data — including etcd data, resource YAMLs, and sometimes application-level data — so that you can restore the entire cluster or specific components in case of failures, corruption, or accidental deletions.

**Backup — control-plane (etcd)**
# Path for all cluster configuration files
This directory contains all kubelet, static pod manifests, certificates, and config files.

cd /etc/kubernetes


# Backup all cluster YAMLs
Exports all runtime resources so you can recreate workloads.

kubectl get all --all-namespaces -o yaml > backup-api.yaml


# Take etcd snapshot
Creates a complete etcd database snapshot.
etcdctl --endpoints=https://127.0.0.1:2379 \
--cacert=/etc/kubernetes/pki/etcd/ca.crt \
--cert=/etc/kubernetes/pki/etcd/server.crt \
--key=/etc/kubernetes/pki/etcd/server.key \
snapshot save /opt/snapshot-pre-boot.db


# Check snapshot status
etcdctl snapshot status /opt/snapshot-pre-boot.db


**Restore — control-plane (etcd)**

# Stop kube-apiserver before restore
systemctl stop kube-apiserver


# Restore etcd snapshot to a new data directory
etcdutl snapshot restore /opt/snapshot-pre-boot.db --data-dir /var/lib/etcd-from-backup


# Start etcd & kube-apiserver
systemctl start etcd kube-apiserver

**After Required Update in etcd.yaml**

Locate the volume section named etcd-data and update the hostPath.
Next, we need to update the /etc/kubernetes/manifests/etcd.yaml to point to the newly restored directory, which is /var/lib/etcd-from-backup. The only change that we need to make to the YAML file, is to change the hostPath for the volume called etcd-data from old directory /var/lib/etcd to the new directory /var/lib/etcd-from-backup:
  ...
  volumes:
  - hostPath:
      path: /var/lib/etcd-from-backup # Newly restored backup directory
      type: DirectoryOrCreate
    name: etcd-data
