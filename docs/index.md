# Tài liệu Triển khai Hệ thống StarlingX

Chào mừng bạn đến với tài liệu hướng dẫn kỹ thuật chuẩn triển khai **StarlingX** — nền tảng điện toán đám mây phân tán mã nguồn mở, tối ưu cho môi trường Edge và mạng Viễn thông thế hệ mới.

Tài liệu này được biên soạn nhằm cung cấp quy trình từng bước (Step-by-Step) giúp các kỹ sư hệ thống từ giai đoạn khảo sát phần cứng đến vận hành thực tế (Day-2) một cách chuyên nghiệp.

---

## 📌 Tổng quan Hệ thống tài liệu

Để việc triển khai đạt hiệu quả cao nhất, tài liệu được phân chia thành các cấu phần mạch lạc theo đúng vòng đời dự án:

```text
  🎨 1. Kiến trúc & Phần cứng ──► ⚙️ 2. Chuẩn bị Môi trường ──► 🚀 3. Cài đặt Nền tảng
                                                                        │
  🎯 Môi trường Vận hành <─── 📊 5. Vận hành Day-2 <─── 🛠️ 4. Cấu hình Hệ thống ┘
```

### 🧭 Lộ trình Triển khai chi tiết

1. **[Chương 1: Giới thiệu & Kiến trúc](01-architecture.md)**
    * Bối cảnh ra đời, mục đích và động cơ của dự án StarlingX.
    * Lựa chọn mô hình triển khai: *All-in-One (Simplex/Duplex)* hoặc *Distributed Cloud*.
    * Yêu cầu cấu hình phần cứng tối thiểu (CPU, RAM, Storage, NICs).

2. **[Chương 2: Chuẩn bị Môi trường](02-prerequisites.md)**
    * Cấu hình BIOS/UEFI tiêu chuẩn (Bật Intel VT-d, SR-IOV, Hyper-Threading).
    * Quy hoạch dải IP (OAM, Management, Cluster-host, Cluster-pod).
    * Chuẩn bị USB/ISO boot cài đặt StarlingX.

3. **[Chương 3: Cài đặt Nền tảng](03-installation.md)**
    * Khởi động và cài đặt hệ điều hành nền tảng lên Node gốc (Controller-0).
    * Khởi tạo file cấu hình Bootstrap (`localhost.yml`).
    * Chạy script cấu hình tự động cho Controller đầu tiên.

4. **[Chương 4: Cấu hình Hệ thống & Tài nguyên](04-configuration.md)**
    * Tích hợp thêm các Node bổ sung (Controller-1, Worker Nodes, Storage Nodes).
    * Cấu hình mạng nâng cao (vSwitch, SR-IOV, Bonded Interfaces).
    * Cấu hình phân vùng lưu trữ tích hợp Ceph Storage.

5. **[Chương 5: Vận hành & Bảo trì (Day-2)](05-operations.md)**
    * Quy trình Sao lưu (Backup) và Khôi phục (Restore) hệ thống.
    * Hướng dẫn áp dụng bản vá (Patching) và nâng cấp (Upgrades).
    * Giám sát tài nguyên và xử lý các cảnh báo hệ thống (Fault Management).

---

## ⚡ Các mô hình triển khai hỗ trợ

Tùy thuộc vào hạ tầng thực tế, hệ thống tài liệu này hỗ trợ các kịch bản cấu hình sau:

*   **All-in-One (AIO) Simplex:** Cấu hình tiết kiệm nhất. Toàn bộ các chức năng Controller, Worker, Storage chạy trên **01 Server vật lý**. Phù hợp cho môi trường Lab/PoC.
*   **All-in-One (AIO) Duplex:** Cung cấp tính sẵn sàng cao (High Availability). Chạy trên **02 Server vật lý**, tự động đồng bộ và dự phòng lỗi (Failover). Phù hợp cho trạm Edge quy mô nhỏ.
*   **Multi-Node / Distributed Cloud:** Mô hình sản xuất (Production). Phân tách tường minh các node chức năng chuyên biệt, quản lý tập trung từ xa hàng nghìn Subcloud biên.

---

!!! info "Thông tin Phiên bản"
    *   **Mục tiêu áp dụng:** StarlingX Platform (Khuyên dùng các bản release ổn định gần nhất).
    *   **Hệ điều hành nền:** Debian-based StarlingX OS.
    *   **Yêu cầu người đọc:** Có kiến thức cơ bản về Linux CLI, Networking (VLAN/Routing), Kubernetes và ảo hóa KVM.

---

👉 **Bắt đầu ngay:** Hãy bấm vào **[Chương 1: Giới thiệu & Kiến trúc](01-architecture.md)** ở menu bên cạnh để tìm hiểu sâu hơn về động cơ công nghệ và mô hình kiến trúc của StarlingX.
