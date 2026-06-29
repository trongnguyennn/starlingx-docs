# Bootstrap và Cấu hình StarlingX R8.0 (VirtualBox)

## 1. Bootstrap Controller-0

### Đăng nhập lần đầu

```text
Username: sysadmin
Password: sysadmin
```

Hệ thống sẽ yêu cầu đổi mật khẩu.

---

### Cấu hình OAM tạm thời

> Chỉ cần khi controller-0 cần truy cập Internet để bootstrap.

```bash
export CONTROLLER0_OAM_CIDR=10.10.10.3/24
export DEFAULT_OAM_GATEWAY=10.10.10.1

sudo ip address add $CONTROLLER0_OAM_CIDR dev enp7s1
sudo ip link set up dev enp7s1
sudo ip route add default via $DEFAULT_OAM_GATEWAY dev enp7s1
```

---

### Tạo file bootstrap

```bash
cd ~

cat <<EOF > localhost.yml
system_mode: duplex

dns_servers:
  - 8.8.8.8
  - 8.8.4.4

external_oam_subnet: 10.10.10.0/24
external_oam_gateway_address: 10.10.10.1
external_oam_floating_address: 10.10.10.2
external_oam_node_0_address: 10.10.10.3
external_oam_node_1_address: 10.10.10.4

admin_username: admin
admin_password: <admin-password>
ansible_become_pass: <sysadmin-password>
EOF
```

---

### Chạy Bootstrap

```bash
ansible-playbook \
/usr/share/ansible/stx-ansible/playbooks/bootstrap.yml
```

Thời gian thực hiện:

```text
5 - 10 phút
```

---

## 2. Cấu hình Controller-0

### Nạp môi trường quản trị

```bash
source /etc/platform/openrc
```

### Gán OAM và Management Network

```bash
OAM_IF=enp7s1
MGMT_IF=enp7s2

system host-if-modify controller-0 lo -c none

IFNET_UUIDS=$(system interface-network-list controller-0 | \
awk '{if ($6=="lo") print $4;}')

for UUID in $IFNET_UUIDS; do
    system interface-network-remove ${UUID}
done

system host-if-modify controller-0 $OAM_IF -c platform
system interface-network-assign controller-0 $OAM_IF oam

system host-if-modify controller-0 $MGMT_IF -c platform
system interface-network-assign controller-0 $MGMT_IF mgmt
system interface-network-assign controller-0 $MGMT_IF cluster-host
```

---

### Cấu hình NTP

```bash
system ntp-modify \
ntpservers=0.pool.ntp.org,1.pool.ntp.org
```

---

### Kích hoạt Ceph

> Bắt buộc nếu sử dụng Persistent Storage hoặc OpenStack.

```bash
system storage-backend-add ceph --confirmed
```

---

### Label OpenStack (nếu triển khai OpenStack)

```bash
system host-label-assign controller-0 \
openstack-control-plane=enabled
```

---

### Unlock Controller-0

```bash
system host-unlock controller-0
```

Controller-0 sẽ reboot.

---

## 3. Cài Controller-1

Khởi động VM:

```text
controller-1
```

PXE boot từ controller-0.

Kiểm tra:

```bash
system host-list
```

Gán role:

```bash
system host-update 2 personality=controller install_output=graphical console=
```

---

### Cấu hình Controller-1

```bash
OAM_IF=enp7s1

system host-if-modify controller-1 $OAM_IF -c platform
system interface-network-assign controller-1 $OAM_IF oam
system interface-network-assign controller-1 mgmt0 cluster-host
```

### Label OpenStack

```bash
system host-label-assign controller-1 \
openstack-control-plane=enabled
```

### Unlock

```bash
system host-unlock controller-1
```

---

## 4. Cài Storage Nodes

Khởi động:

```text
storage-0
storage-1
```

Gán role:

```bash
system host-update 3 personality=storage install_output=graphical console=
system host-update 4 personality=storage install_output=graphical console=
```

---

### Gán Cluster Host Network

```bash
for NODE in storage-0 storage-1; do
   system interface-network-assign \
   $NODE mgmt0 cluster-host
done
```

---

### Thêm OSD

Storage-0:

```bash
HOST=storage-0
DISKS=$(system host-disk-list ${HOST})
TIERS=$(system storage-tier-list ceph_cluster)

system host-stor-add ...
```

Storage-1:

```bash
HOST=storage-1
DISKS=$(system host-disk-list ${HOST})
TIERS=$(system storage-tier-list ceph_cluster)

system host-stor-add ...
```

---

### Unlock Storage

```bash
for STORAGE in storage-0 storage-1; do
   system host-unlock $STORAGE
done
```

---

## 5. Cài Worker Nodes

Khởi động:

```text
worker-0
worker-1
```

Gán role:

```bash
system host-update 5 personality=worker hostname=worker-0 install_output=graphical console=

system host-update 6 personality=worker hostname=worker-1 install_output=graphical console=
```

---

### Gán Cluster Host Network

```bash
for NODE in worker-0 worker-1; do
   system interface-network-assign \
   $NODE mgmt0 cluster-host
done
```

---

### Label OpenStack

```bash
for NODE in worker-0 worker-1; do
  system host-label-assign \
  $NODE openstack-compute-node=enabled

  system host-label-assign \
  $NODE openvswitch=enabled

  system host-label-assign \
  $NODE sriov=enabled
done
```

---

### Tạo Nova Ephemeral Storage

```bash
for NODE in worker-0 worker-1; do
   system host-fs-add ${NODE} instances=10
done
```

---

### Unlock Worker

```bash
for NODE in worker-0 worker-1; do
   system host-unlock $NODE
done
```

---

## 6. Kiểm tra Cluster

```bash
system host-list
```

Kỳ vọng:

```text
controller-0  unlocked-enabled
controller-1  unlocked-enabled
storage-0     unlocked-enabled
storage-1     unlocked-enabled
worker-0      unlocked-enabled
worker-1      unlocked-enabled
```

---

## Mô hình triển khai

```text
Controller Cluster
├── controller-0
└── controller-1

Storage Cluster
├── storage-0
└── storage-1

Worker Nodes
├── worker-0
└── worker-1

Ceph Cluster
├── OSD: storage-0
├── OSD: storage-1
├── MON: controller-0
└── MON: controller-1
```

## 7. Lưu ý
### Lỗi Kubernetes OOM do kubepods bị giới hạn 500Mi
Triệu chứng
```bash
kubectl get pods -A
```
Bị timeout: Unable to connect to the server: TLS handshake timeout

Các pod restart liên tục:
```text
calico-node
calico-kube-controllers
coredns
cert-manager
dex
flux source-controller
helm-controller
```
Kiểm tra
```bash
sudo dmesg -T | egrep -i "oom|killed|memory|disk|error|fail" | tail -50
sudo crictl inspect <container-id> | grep -i -A5 -B5 "reason\|exitCode\|message"
```
Kernel log:
```text
Memory cgroup out of memory
Killed process coredns
Killed process cainjector
Killed process dex
```
Kiểm tra
```bash
cat /sys/fs/cgroup/memory/k8s-infra-stx/kubepods/memory.limit_in_bytes
```
Kết quả:
```text
524288000
```
Tương đương: 500 MiB

Trong khi host có:
```bash
free -h
13Gi RAM
```
Root Cause
Cgroup: /k8s-infra-stx/kubepods bị giới hạn: 500Mi
Toàn bộ workload Kubernetes phải dùng chung 500Mi RAM.

Dẫn đến:

OOM
↓
Calico crash
↓
CoreDNS crash
↓
API timeout
↓
Controller-1 không join được

Workaround

Tăng tạm giới hạn:
```bash
echo 8589934592 > \
/sys/fs/cgroup/memory/k8s-infra-stx/kubepods/memory.limit_in_bytes

echo 8589934592 | sudo tee /sys/fs/cgroup/memory/k8s-infra-stx/kubepods/memory.limit_in_bytes
```
Kiểm tra:
```bash
cat /sys/fs/cgroup/memory/k8s-infra-stx/kubepods/memory.limit_in_bytes
8589934592
```
Sau đó các pod hồi phục:
```bash
kubectl get pods -A
```
Lưu ý: Sau reboot cần kiểm tra lại:
```bash
cat /sys/fs/cgroup/memory/k8s-infra-stx/kubepods/memory.limit_in_bytes
```
Nếu quay về: 524288000 thì cần điều tra service hoặc cấu hình StarlingX đang ghi đè giá trị này.

### Lỗi cấu hình mạng
```bash
sudo grep server /etc/cni/net.d/calico-kubeconfig
```
Kết quả
```text
    server: https://[10.96.0.1]:443
```
Sửa lại
```bash
sudo sed -i 's#https://\[10\.96\.0\.1\]:443#https://10.96.0.1:443#g' /etc/cni/net.d/calico-kubeconfig
sudo systemctl restart kubelet
```
Kiểm tra lại:
```bash
sudo grep server /etc/cni/net.d/calico-kubeconfig
```

Phải thành:
```text
server: https://10.96.0.1:443
```
### Lỗi filesystem /usr đang mount read-only, nên sed -i không tạo được file tạm trong cùng thư mục.
Kiểm tra mount
```bash
mount | grep ' /usr '
```
Remount /usr thành read-write
```bash
sudo mount -o remount,rw /usr
```

### Lỗi pull image khi bootstap
Kiểm tra:
```bash
grep -R "kubernetes-entrypoint" \
/usr/share/ansible/stx-ansible/playbooks/roles/common/load-images-information/vars/
```
Xử lý:
```bash
sudo find /usr/share/ansible/stx-ansible/playbooks/roles/common/load-images-information/vars \
-name system-images.yml \
-exec sed -i \
's#quay.io/stackanetes/kubernetes-entrypoint:v0.3.1#quay.io/airshipit/kubernetes-entrypoint:latest-ubuntu_focal#g' {} \;
```

### Lỗi node bị degraded do colectd
Thực hiện lệnh
```bash
sudo systemctl status systemd-journald
sudo ls -l /var/log/collectd.log /var/log/daemon.log /var/log/syslog 2>/dev/null
sudo grep -i collectd /var/log/daemon.log /var/log/syslog /var/log/collectd.log 2>/dev/null | tail -100
```

```bash
sudo cp /etc/systemd/system/collectd.service /etc/systemd/system/collectd.service.bak.$(date +%F_%H%M)

sudo sed -i 's/^Type=notify/Type=simple/' /etc/systemd/system/collectd.service

sudo systemctl daemon-reload
sudo systemctl reset-failed collectd
sudo systemctl restart collectd
sudo systemctl status collectd
```