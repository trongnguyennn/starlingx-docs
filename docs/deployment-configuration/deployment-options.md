# Các Tùy Chọn Triển Khai StarlingX (Deployment Options)

StarlingX hỗ trợ nhiều cấu hình triển khai khác nhau để phù hợp với từng nhu cầu về quy mô, hiệu năng và tính sẵn sàng cao (HA). 

---

## 1. Bảng so sánh các options triển khai

| Tùy chọn cấu hình | Số lượng Server chính | Chức năng tích hợp | Khả năng mở rộng (Worker Nodes) | Tính năng Sẵn sàng cao (HA) |
| :--- | :---: | :--- | :---: | :---: |
| **All-in-one Simplex** | 1 | Controller + Worker + Storage | Không (Chỉ gồm 1 node duy nhất) | Không |
| **All-in-one Duplex** | 2 | Controller + Worker + Storage | Mở rộng lên đến **50** worker nodes | **Có** (Bảo vệ HA giữa 2 server chính) |
| **Standard (Storage trên Controller)** | 2 | Controller + Storage | Mở rộng lên đến **200** worker nodes | **Có** (Cụm Controller 2 node HA) |
| **Standard (Storage chuyên dụng)** | 2 + (2 đến 9) | Controller (riêng) + Ceph Storage (riêng) | Mở rộng lên đến **200** worker nodes | **Có** (Cụm Controller HA + Cụm Ceph HA) |

