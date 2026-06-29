# Install StarlingX OpenStack (stx-openstack)

## Điều kiện tiên quyết

Trước khi cài OpenStack, hệ thống StarlingX phải được cấu hình hoàn chỉnh:

* Các node đã được gán đúng role OpenStack (controller, worker, storage...).
* Đã cấu hình vSwitch.
* Đã tạo `nova-local` volume group trên các node chạy Compute.
* Cluster Kubernetes hoạt động ổn định.

---

# 1. Tăng dung lượng Docker Storage

Mặc định filesystem `docker_lv` chỉ có 30GB, không đủ để cài đặt OpenStack.

Kiểm tra:

```bash
system host-fs-list controller-0
```

Tăng dung lượng Docker Storage:

```bash
system host-fs-modify controller-0 docker=60
```

> Khuyến nghị tối thiểu: **60GB**

Kiểm tra lại:

```bash
system host-fs-list controller-0
```

---

# 2. Tải gói stx-openstack

Tải package từ StarlingX Mirror:

```text
release/<version>/debian/openstack/outputs/
```

Ví dụ:

```text
stx-openstack-<version>-debian-stable-versioned.tgz
```

---

# 3. Upload OpenStack Application

Upload package vào StarlingX:

```bash
system application-upload stx-openstack-<version>-debian-stable-versioned.tgz
```

Lệnh này sẽ:

* Nạp OpenStack Manifest
* Nạp Helm Charts
* Tạo Helm Overrides mặc định
* Tự động cấu hình các dịch vụ OpenStack theo trạng thái hiện tại của hệ thống

Kiểm tra:

```bash
system application-list
```

Kết quả mong muốn:

```text
stx-openstack    uploaded
```

---

# 4. (Tuỳ chọn) Cấu hình Host-Based Ceph

Mặc định stx-openstack sử dụng Rook-Ceph.

Nếu hệ thống dùng Host-Based Ceph thì cần cập nhật image `ceph-config-helper` trước khi Apply Application.

Ví dụ:

```bash
system helm-override-update \
stx-openstack \
cinder \
openstack \
--reuse-values \
--set images.tags.cinder_storage_init=<ceph-helper-image>
```

Thực hiện tương tự cho:

* cinder
* glance
* libvirt
* nova

---

# 5. Cấu hình Proxy (nếu có)

Nếu hệ thống sử dụng Proxy:

```bash
system docker-proxy-modify
```

Đảm bảo biến:

```text
HTTP_PROXY
HTTPS_PROXY
NO_PROXY
```

đã được cấu hình chính xác trước khi Apply Application.

---

# 6. Apply OpenStack Application

Kích hoạt OpenStack:

```bash
system application-apply stx-openstack
```

Kiểm tra trạng thái:

```bash
watch -n 5 system application-list
```

Ví dụ:

```text
+---------------+-----------+---------+
| application   | version   | status  |
+---------------+-----------+---------+
| stx-openstack | 1.0-0     | applied |
+---------------+-----------+---------+
```

---

# 7. Theo dõi quá trình cài đặt

Kiểm tra Pod OpenStack:

```bash
kubectl get pods -n openstack
```

Kiểm tra Helm Release:

```bash
helm list -n openstack
```

Kiểm tra Application:

```bash
system application-list
```

---

# 8. Xác nhận OpenStack hoạt động

Kiểm tra endpoint:

```bash
openstack endpoint list
```

Kiểm tra service:

```bash
openstack service list
```

Kiểm tra compute:

```bash
openstack compute service list
```

Kiểm tra image:

```bash
openstack image list
```

---

# Kiến trúc sau khi cài đặt

```text
+----------------------+
|  StarlingX Platform  |
+----------------------+
          |
          v
+----------------------+
| Kubernetes Cluster   |
+----------------------+
          |
          v
+----------------------+
|   stx-openstack      |
+----------------------+
| Keystone             |
| Glance               |
| Nova                 |
| Neutron              |
| Cinder               |
| Horizon              |
| MariaDB              |
| RabbitMQ             |
+----------------------+
          |
          v
+----------------------+
| Ceph Storage Backend |
+----------------------+
```

## Thời gian cài đặt
```text
| Hệ thống        | Thời gian    |
| --------------- | ------------ |
| VirtualBox Lab  | 10 - 30 phút |
| Physical Server | 5 - 10 phút  |
```
## Kiểm tra hoàn tất

```bash
system application-list
```

Kết quả:

```text
stx-openstack   applied
```

và tất cả Pod trong namespace `openstack` ở trạng thái:

```text
Running
Completed
```
