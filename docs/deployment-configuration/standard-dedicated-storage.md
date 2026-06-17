# Standard Configuration with Dedicated Storage

## Tổng quan

Mô hình Standard Configuration with Dedicated Storage tách riêng các node theo chức năng:

* Controller Nodes
* Compute Nodes
* Storage Nodes

Storage được triển khai trên các node riêng biệt, cung cấp khả năng mở rộng và hiệu năng cao nhất.

---

## Kiến trúc

![Standard Configuration with Dedicated Storage](../images/standard-dedicated-storage.png)

*Hình: Standard Configuration với Dedicated Storage.*

---

## Thành phần

* **Controller Nodes**

    * Quản lý và điều phối hệ thống.

* **Compute Nodes**

    * Chạy workload Kubernetes/OpenStack.

* **Storage Nodes**

    * Triển khai Ceph Storage Cluster.
    * Cung cấp Persistent Storage cho hệ thống.

---

## Đặc điểm

* Controller, Compute và Storage tách biệt.
* Hỗ trợ mở rộng độc lập từng thành phần.
* Hiệu năng và dung lượng lưu trữ cao.
* Phù hợp cho môi trường Production quy mô lớn.

---

## Kết luận

Đây là mô hình triển khai đầy đủ nhất của StarlingX, cung cấp khả năng mở rộng, hiệu năng và độ sẵn sàng cao cho các hệ thống Production.
