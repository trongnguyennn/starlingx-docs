# Cài đặt StarlingX trên VirtualBox

Phần này hướng dẫn triển khai StarlingX trên các máy ảo **VirtualBox**, thay thế cho môi trường mặc định sử dụng **libvirt/KVM**.

## Mục đích

Triển khai môi trường StarlingX Lab trên VirtualBox để học tập, thử nghiệm và đánh giá tính năng.

## Nội dung triển khai

* Chuẩn bị môi trường VirtualBox.
* Tạo các VM Controller, Worker và Storage.
* Khởi động Controller Node để bắt đầu cài đặt.
* Tùy chỉnh các tham số cài đặt.

---

## Chuẩn bị

Trước khi triển khai cần chuẩn bị:

### Máy chủ cài đặt

* Windows hoặc Linux.
* Đủ tài nguyên CPU, RAM và Disk để chạy nhiều VM.

### VirtualBox

Cài đặt VirtualBox trên máy chủ.

### VirtualBox Extension Pack

Bắt buộc cài đặt VirtualBox Extension Pack để hỗ trợ:

* PXE Boot
* Intel Network Adapter

Điều này cho phép Worker và Storage Node có thể boot qua mạng từ Controller Node.
### Lưu ý
- StarlingX cung cấp bộ script tạo VM trong repository [tools](https://opendev.org/starlingx/tools).
- Các script này có thể chưa được cập nhật theo khuyến nghị mới nhất của StarlingX.
- Nên kiểm tra tài liệu phiên bản hiện tại trước khi sử dụng.

## Tạo VM Controller, Worker và Storage Hosts

Đối với mỗi node StarlingX, cần tạo một máy ảo (VM) trong VirtualBox với cấu hình phù hợp theo vai trò của node.

### OS Type và Memory Settings

Cấu hình hệ điều hành cho tất cả các VM:

| Tham số | Giá trị              |
| ------- | -------------------- |
| Type    | Linux                |
| Version | Other Linux (64-bit) |

Phân bổ RAM

| Node Type       | RAM              |
| --------------- | ---------------- |
| Controller Node | 16384 MB (16 GB) |
| Worker Node     | 8192 MB (8 GB)   |
| Storage Node    | 4096 MB (4 GB)   |
| All-in-One Node | 20480 MB (20 GB) |

### Mô hình ví dụ

```text
Controller-0 : 16 GB RAM
Controller-1 : 16 GB RAM

Worker-0     : 8 GB RAM
Worker-1     : 8 GB RAM

Storage-0    : 4 GB RAM
Storage-1    : 4 GB RAM
```

### Disk Settings

Sử dụng cấu hình mặc định của VirtualBox:

* Disk Controller: Mặc định (IDE)
* Disk Format: VDI

#### Controller Nodes

Mỗi Controller cần tối thiểu **2 ổ đĩa**:

| Disk   | Dung lượng |
| ------ | ---------- |
| Disk 1 | 240 GB     |
| Disk 2 | 10 GB      |

> Nếu sử dụng Analytics, nên tăng Disk 2 lên **30 GB**.

#### Worker Nodes

| Disk      | Dung lượng |
| --------- | ---------- |
| Root Disk | 80 GB      |

> Với mô hình AIO, khuyến nghị sử dụng **100 GB**.

Lưu ý:

* Khi cấu hình Local Storage, hệ thống cung cấp khoảng **12 GB** cho VM Instances.
* Có thể bổ sung thêm ổ đĩa để mở rộng Local Storage nếu cần.

#### Storage Nodes

Mỗi Storage Node cần tối thiểu **2 ổ đĩa**:

| Disk      | Dung lượng |
| --------- | ---------- |
| Root Disk | 80 GB      |
| OSD Disk  | ≥ 10 GB    |

Lưu ý:

* Mỗi OSD yêu cầu một ổ đĩa riêng.
* Dung lượng OSD phụ thuộc vào nhu cầu lưu trữ và số lượng VM dự kiến triển khai.

#### Installation ISO

Gắn file ISO StarlingX vào ổ CD/DVD của VM:

1. Chọn **Storage** trong cấu hình VM.
2. Chọn **Empty CD-ROM**.
3. Nhấn biểu tượng CD/DVD.
4. Chọn file ISO StarlingX.

> Chỉ gắn ISO cho **controller-0**.

Các node còn lại:

* controller-1
* worker nodes
* storage nodes

sẽ thực hiện **PXE Network Boot** từ controller-0.

### System Settings

Cấu hình các thông số hệ thống trong VirtualBox cho từng VM.

#### Motherboard Settings

Vào:

```text
System → Motherboard
```

Cấu hình Boot Order:

1. Floppy
2. CD/DVD
3. Hard Disk
4. Network

> Bắt buộc bật tùy chọn **Network Boot (PXE)** để các node có thể khởi động qua mạng.

---

#### Processor Settings

Vào:

```text
System → Processor
```

Cấu hình CPU theo vai trò của node:

| Node Type       | vCPU |
| --------------- | ---- |
| Controller Node | 4    |
| Worker Node     | 3    |
| Storage Node    | 1    |

---

#### Lưu ý cho Worker Node

Cấu hình mặc định chỉ đủ tài nguyên để chạy một VM Instance.

Nếu muốn chạy nhiều VM hơn:

* Tăng số lượng vCPU cho Worker Node.

Nếu Worker được cấp phát **hơn 4 vCPU**, cần giới hạn số CPU dành cho **vSwitch** trước khi unlock node:

```bash
system host-cpu-modify worker-0 -f vswitch -p0 1
```

Nếu không thực hiện bước này:

* vSwitch có thể khởi động thất bại.
* Dịch vụ quan trọng bị lỗi.
* Node sẽ liên tục reboot (reboot loop).

### Network Settings

Cấu hình mạng cho các VM trong VirtualBox.

#### OAM Network

Có hai lựa chọn cho mạng OAM:

##### Option 1: Host-Only Network (Khuyến nghị)

* Tạo Host-Only Adapter trong VirtualBox.
* Controller truy cập mạng ngoài thông qua Router VM.
* Cấu hình:

| Tham số      | Giá trị       |
| ------------ | ------------- |
| IPv4 Address | 10.10.10.254  |
| Netmask      | 255.255.255.0 |
| DHCP Server  | Disabled      |

##### Option 2: NAT Network

* Cho phép Controller truy cập Internet trực tiếp thông qua NAT của VirtualBox.

---

#### Controller Nodes

| Adapter   | Cấu hình                              |
| --------- | ------------------------------------- |
| Adapter 1 | Host-Only hoặc NAT Network (OAM)      |
| Adapter 2 | Internal Network: `intnet-management` |

#### Worker Nodes

| Adapter   | Cấu hình                    |
| --------- | --------------------------- |
| Adapter 1 | `intnet-unused`             |
| Adapter 2 | `intnet-management`         |
| Adapter 3 | `intnet-data1` (virtio-net) |
| Adapter 4 | `intnet-data2` (virtio-net) |

Lưu ý:

* Promiscuous Mode: **Allow All**
* Có thể bổ sung NIC để mô phỏng LAG.

#### Storage Nodes

| Adapter   | Cấu hình            |
| --------- | ------------------- |
| Adapter 1 | `intnet-unused`     |
| Adapter 2 | `intnet-management` |

---

#### Boot Priority

Tất cả các VM (Controller, Worker, Storage) phải cấu hình PXE Boot trên Interface 2:

```bash
VBoxManage modifyvm <vm-name> --nicbootprio2 1
```

Ví dụ:

```bash
VBoxManage modifyvm controller-0 --nicbootprio2 1
```

Điều này cho phép các node:

* PXE Boot từ Controller.
* Tự động cài đặt hệ điều hành qua mạng.

---

#### Debug PXE Boot

Nếu PXE Boot không hoạt động:

1. Khởi động VM.
2. Nhấn `F12`.
3. Chọn `LAN Boot`.
4. Nhấn `Ctrl+B` để vào iPXE CLI.

Lệnh:

```text
autoboot
```
sẽ kiểm tra từng interface và thử thực hiện Network Boot.





