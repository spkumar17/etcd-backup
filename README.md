# 🚀 Etcd Backup and Restore: Safeguard Your Kubernetes Cluster State 🛡️


This guide explains the process to back up and restore etcd, the key-value store that stores Kubernetes cluster data. The steps ensure that all critical cluster state information is preserved and can be restored in case of data loss.

---

## Prerequisites

1. **Access to etcd**:
   - Ensure you have access to the etcd server running in your cluster.
   - The commands should be run on the control plane node where etcd is running.

2. **Install `etcdctl`**:
   - Use `etcdctl` version compatible with the etcd server (etcd v3 API is recommended).
   ```bash
   export ETCDCTL_API=3
   ```

3. **Certificates and Keys**:
   - Ensure you have the following files:
     - `ca.crt`: Certificate Authority file.
     - `server.crt`: Server certificate.
     - `server.key`: Server private key.
   
---

## 1. Backing Up etcd

1. **Run the Backup Command**:
   ```bash
   sudo etcdctl --endpoints=https://127.0.0.1:2379 \
     --cacert=/etc/kubernetes/pki/etcd/ca.crt \
     --cert=/etc/kubernetes/pki/etcd/server.crt \
     --key=/etc/kubernetes/pki/etcd/server.key \
     snapshot save /path/to/backup/etcd-backup.db
   ```

2. **Verify the Backup**:
   Check the snapshot details (size, revision, etc.):
   ```bash
   sudo etcdctl --write-out=table snapshot status /path/to/backup/etcd-backup.db
   ```

---

## 2. Restoring etcd

1. **Restore the Snapshot**:
   ```bash
   sudo etcdctl snapshot restore /path/to/backup/etcd-backup.db \
     --data-dir /var/lib/etcdrestore
   ```

2. **Update etcd Configuration**:
   - Edit the etcd manifest file (`/etc/kubernetes/manifests/etcd.yaml`) to point to the restored data directory:
     ```yaml
     - --data-dir=/var/lib/etcdrestore
     ```
   - Save the file and exit. The etcd Pod will restart automatically and use the restored data.

---

## 3. Restoring etcd in a Cluster

For clusters with multiple etcd nodes or running API servers:

1. **Stop API Servers**:
   - Stop all Kubernetes API server instances to prevent stale data or conflicts:
     ```bash
     sudo systemctl stop kube-apiserver
     ```

2. **Restore etcd on All Nodes**:
   - Perform the restoration process on all etcd nodes using the backup file.

3. **Restart API Servers**:
   - Start the API server instances:
     ```bash
     sudo systemctl start kube-apiserver
     ```

4. **Restart Kubernetes Components**:
   - Restart the kube-scheduler, kube-controller-manager, and kubelet to ensure they don’t rely on stale data:
     ```bash
     sudo systemctl restart kube-scheduler
     sudo systemctl restart kube-controller-manager
     sudo systemctl restart kubelet
     ```

---

## Additional Notes

### **What is Backed Up?**
- Secrets, ConfigMaps, Persistent Volume Claims (PVCs), cluster state, and metadata required for the cluster to function.

### **What is Not Backed Up?**
- Logs, metrics, and application data outside etcd must be backed up separately.

### **Backup Frequency**
- Take regular backups to minimize potential data loss.

### **Testing Restores**
- Test the restoration process in a non-production environment to validate its reliability.

---

## References
- [Official etcdctl Documentation](https://kubernetes.io/docs/tasks/administer-cluster/configure-upgrade-etcd/)

---

### Example Command Summary

- **Backup Command**:
  ```bash
  sudo etcdctl --endpoints=https://127.0.0.1:2379 \
    --cacert=/etc/kubernetes/pki/etcd/ca.crt \
    --cert=/etc/kubernetes/pki/etcd/server.crt \
    --key=/etc/kubernetes/pki/etcd/server.key \
    snapshot save /path/to/backup/etcd-backup.db
  ```

- **Restore Command**:
  ```bash
  sudo etcdctl snapshot restore /path/to/backup/etcd-backup.db \
    --data-dir /var/lib/etcdrestore
  
