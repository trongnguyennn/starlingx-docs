# Tài liệu thuật ngữ StarlingX (STX 6.0)
---

## 1. Các thành phần chính

### 1.1 Node (Nút)

Là một server vật lý hoặc máy ảo trong hệ thống.

---

### 1.2 Controller Node

Node điều khiển, chịu trách nhiệm:

* Quản lý toàn bộ hệ thống
* Chạy control plane của Kubernetes
* Chạy các service OpenStack (Keystone, Glance, Cinder...)

Hỗ trợ HA:

* Active-Active
* Active-Standby

---

### 1.3 Worker Node

Node dùng để chạy workload:

* Container (Kubernetes)
* Máy ảo (OpenStack)

Trong OpenStack:

* Đóng vai trò Compute Node
* Xử lý networking (vSwitch, routing)

---

### 1.4 Storage Node

Node cung cấp lưu trữ:

* Sử dụng Ceph
* Hỗ trợ:
    * Block storage (Cinder)
    * Object storage
    * Image storage (Glance)

---

### 1.5 All-in-one Node

Một node đảm nhiệm tất cả:

* Controller
* Worker
* Storage

→ Thường dùng cho lab hoặc môi trường nhỏ.

---

### 1.6 Bare Metal

Node chạy trực tiếp trên phần cứng:

* Không có hypervisor
* Không có Kubernetes kubelet

---

## 2. Networking trong StarlingX

### 2.1 Management Network

* Mạng nội bộ (private)
* Dùng cho:
    * Quản lý hệ thống
    * Monitoring
    * Traffic Ceph

→ Bắt buộc trên tất cả node

---

### 2.2 OAM Network (Operations, Administration, Management)

* Mạng truy cập từ bên ngoài
* Dùng cho:
    * API
    * Horizon (UI OpenStack)
    * SSH, SNMP

→ Chỉ cần trên Controller

---

### 2.3 Data Network

* Dùng cho workload:
    * VM
    * Container

Hỗ trợ:

* SR-IOV
* PCI passthrough

→ Chủ yếu dùng trên Worker

---

### 2.4 IPMI Network

* Mạng quản lý phần cứng
* Dùng cho:
    * Remote power
    * Console

---

### 2.5 PXE Network

* Dùng để boot qua mạng
* Triển khai OS tự động

---

### 2.6 Infra Network (Deprecated)

* Đã deprecated
* Trước đây dùng cho storage

---

### 2.7 Node Interfaces
Các kiểu cấu hình NIC:

* Single NIC
* Bonding (LAG)
* VLAN tagging

---

### 2.8 Shared NIC

Một NIC dùng chung nhiều mạng:

* Management
* OAM
* Data
* PXE

---

## 3. Khái niệm Cloud

### 3.1 Kubernetes Cluster

* Nhóm các node chạy container
* Có control plane chung

---

### 3.2 Virtual Machine (VM)

* Máy ảo chạy trên hypervisor

---

### 3.3 Virtualization

* Ảo hóa tài nguyên:
    * CPU
    * RAM
    * Network
    * Storage

---

### 3.4 Edge Computing

* Đặt tài nguyên gần người dùng
* Giảm độ trễ
* Tăng hiệu năng

---

### 3.5 IoT (Internet of Things)

* Thiết bị kết nối internet
* Hoạt động tự động

---

## 4. Vận hành & Lifecycle

### 4.1 Deployment Tools

* Công cụ cài đặt và quản lý hệ thống

---

### 4.2 Precheck

* Kiểm tra trước khi triển khai:
    * Health
    * Cấu hình

---

### 4.3 Preinstallation

* Chuẩn bị trước khi cài đặt

---

### 4.4 Prestaging

* Load sẵn package trước khi deploy

---

### 4.5 Upgrade / Downgrade

* Nâng cấp / hạ cấp version

---

### 4.6 Restore

* Khôi phục từ backup

---

### 4.7 Rollback

* Quay lại trạng thái trước đó khi lỗi

---

## 5. Lưu trữ (Storage)

* Sử dụng Ceph
* Hỗ trợ replication:
    * 2 bản
    * 3 bản

→ Đảm bảo HA
