# Tạo Máy Ảo VirtualBox cho StarlingX

## Chuẩn bị mạng VirtualBox

### OAM Network

Tạo Host-Only Network:

```text
Tools → Network → Host-only Networks → Create
```

Cấu hình:

| Tham số      | Giá trị       |
| ------------ | ------------- |
| Name         | vboxnet0      |
| IPv4 Address | 10.10.10.254  |
| Netmask      | 255.255.255.0 |
| DHCP Server  | Disable       |

### Internal Networks

Không cần tạo trước, VirtualBox sẽ tự tạo khi khai báo tên:

```text
intnet-management
intnet-unused
intnet-data1
intnet-data2
```

---

# Tạo Controller Nodes

## controller-0

### General

| Tham số | Giá trị              |
| ------- | -------------------- |
| Name    | controller-0         |
| Type    | Linux                |
| Version | Other Linux (64-bit) |

### Hardware

| Tài nguyên | Giá trị |
| ---------- | ------- |
| vCPU       | 4       |
| RAM        | 16 GB   |

### Storage

| Disk   | Dung lượng |
| ------ | ---------- |
| Disk 1 | 240 GB     |
| Disk 2 | 10 GB      |

### Network

| Adapter   | Type              | Network           |
| --------- | ----------------- | ----------------- |
| Adapter 1 | Host-Only Adapter | vboxnet0          |
| Adapter 2 | Internal Network  | intnet-management |

### Boot Order

```text
Floppy
Optical
Hard Disk
Network
```

### ISO

Gắn file ISO:

```text
starlingx-intel-x86-64-cd.iso
```

---

## controller-1

Cấu hình giống controller-0:

| Tài nguyên | Giá trị |
| ---------- | ------- |
| vCPU       | 4       |
| RAM        | 16 GB   |
| Disk 1     | 240 GB  |
| Disk 2     | 10 GB   |

### Network

| Adapter   | Type              | Network           |
| --------- | ----------------- | ----------------- |
| Adapter 1 | Host-Only Adapter | vboxnet0          |
| Adapter 2 | Internal Network  | intnet-management |

> Không gắn ISO cho controller-1.

---

# Tạo Storage Nodes

## storage-0

### Hardware

| Tài nguyên | Giá trị |
| ---------- | ------- |
| vCPU       | 1       |
| RAM        | 4 GB    |

### Storage

| Disk   | Dung lượng  |
| ------ | ----------- |
| Disk 1 | 80 GB       |
| Disk 2 | 10 GB (OSD) |

### Network

| Adapter   | Type             | Network           |
| --------- | ---------------- | ----------------- |
| Adapter 1 | Internal Network | intnet-unused     |
| Adapter 2 | Internal Network | intnet-management |

---

## storage-1

Cấu hình giống storage-0.

---

# Tạo Worker Nodes

## worker-0

### Hardware

| Tài nguyên | Giá trị |
| ---------- | ------- |
| vCPU       | 3       |
| RAM        | 8 GB    |

### Storage

| Disk   | Dung lượng |
| ------ | ---------- |
| Disk 1 | 80 GB      |

### Network

| Adapter   | Type             | Network           |
| --------- | ---------------- | ----------------- |
| Adapter 1 | Internal Network | intnet-unused     |
| Adapter 2 | Internal Network | intnet-management |
| Adapter 3 | Internal Network | intnet-data1      |
| Adapter 4 | Internal Network | intnet-data2      |

---

## worker-1

Cấu hình giống worker-0.

---

# Thiết lập PXE Boot

Thực hiện trên máy Host:

```bash
VBoxManage modifyvm controller-0 --nicbootprio2 1
VBoxManage modifyvm controller-1 --nicbootprio2 1

VBoxManage modifyvm storage-0 --nicbootprio2 1
VBoxManage modifyvm storage-1 --nicbootprio2 1

VBoxManage modifyvm worker-0 --nicbootprio2 1
VBoxManage modifyvm worker-1 --nicbootprio2 1
```

---

# Kiểm tra Topology

```text
vboxnet0 (OAM)
│
├── controller-0
└── controller-1

intnet-management
│
├── controller-0
├── controller-1
├── storage-0
├── storage-1
├── worker-0
└── worker-1

intnet-data1
│
├── worker-0
└── worker-1

intnet-data2
│
├── worker-0
└── worker-1
```

---

# Bắt đầu cài đặt

1. Khởi động `controller-0`.
2. Chọn:

```text
Graphics Controller Node Install
```

3. Sau khi cài xong và bootstrap thành công:

   * Khởi động controller-1
   * Khởi động storage-0, storage-1
   * Khởi động worker-0, worker-1

4. Nhấn:

```text
F12 → LAN Boot
```

để các node PXE boot từ controller-0.
