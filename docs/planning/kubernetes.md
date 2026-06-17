# Kế hoạch cài đặt và cấu hình kubernetes

## Tổng quan

Việc lập kế hoạch cài đặt và cấu hình StarlingX giúp triển khai nhanh chóng, đảm bảo đáp ứng yêu cầu vận hành hiện tại và khả năng mở rộng trong tương lai.

Quá trình lập kế hoạch hỗ trợ:

* Đáp ứng yêu cầu của container workload.
* Đáp ứng nhu cầu quản trị và vận hành cloud.
* Tích hợp với hạ tầng Data Center hoặc Telecom hiện có.
* Chuẩn bị cho việc mở rộng hệ thống trong tương lai.

## Các nội dung cần lập kế hoạch

### Network Planning

Thiết kế mạng cho hệ thống StarlingX.

### Storage Planning

Lập kế hoạch lưu trữ và dung lượng dữ liệu.

### Node Installation Planning

Lập kế hoạch cài đặt và triển khai các node.

### Node Resource Planning

Xác định tài nguyên CPU, RAM, Disk và Network cho từng node.

## Network Planning
### Network Requirements

StarlingX sử dụng nhiều loại mạng khác nhau tùy theo quy mô hệ thống và các tính năng được triển khai.

#### Các loại mạng chính

- **PXE Boot Network (tùy chọn)**: Dùng để cài đặt và khởi động node qua mạng.
- **Management Network**: Mạng nội bộ bắt buộc, phục vụ giao tiếp giữa các thành phần của hệ thống.
- **Cluster Host Network**: Mạng bắt buộc cho Kubernetes, hỗ trợ giao tiếp giữa các container và các node trong cụm.
- **OAM Network**: Mạng quản trị và vận hành, dùng để truy cập hệ thống từ bên ngoài.
- **External Network (tùy chọn)**: Cung cấp kết nối từ ứng dụng hoặc dịch vụ ra mạng ngoài.

#### Lưu ý

- Có thể gộp nhiều mạng trên cùng một giao diện vật lý bằng VLAN hoặc Multi-Netted Interface.
- Với hệ thống ít node, nên sử dụng subnet nhỏ để tiết kiệm địa chỉ IP và dễ quản lý.
- Mỗi node cần được cấp địa chỉ IP cho các mạng được sử dụng.
- Hệ thống có 2 controller yêu cầu hỗ trợ IP Multicast trên Management Network để đảm bảo đồng bộ giữa các controller.

### Networks for a System with Dedicated Storage

Đối với hệ thống sử dụng **Dedicated Storage**, StarlingX khuyến nghị triển khai đầy đủ các mạng để đảm bảo hiệu năng và khả năng mở rộng.

#### Cấu hình mạng khuyến nghị

- **Management Network (10GE)**
    - Quản lý hạ tầng nội bộ.
    - Truyền tải lưu lượng I/O giữa các node và Storage (Ceph).

- **OAM Network**
    - Quản trị và vận hành hệ thống từ bên ngoài.

- **Cluster Host Network**
    - Kết nối container-to-container trong Kubernetes.
    - Có thể dùng cho kết nối workload ra mạng ngoài.

- **External Network**
    - Cung cấp kết nối từ ứng dụng hoặc dịch vụ ra bên ngoài.

- **PXE Boot Network (tùy chọn)**
    - Cần khi Management Network sử dụng VLAN Tag.
    - Hoặc khi Management Network được chia sẻ với các thiết bị khác.

#### Lưu ý

- Nếu Cluster Host Network không được sử dụng cho kết nối bên ngoài, có thể dùng OAM Network hoặc các cổng mạng khác trên Controller và Worker.
- Với hệ thống tải trung bình, OAM Network có thể được gộp chung với Management Network để giảm số lượng giao diện mạng cần sử dụng.

### IP Support

StarlingX hỗ trợ cả **IPv4** và **IPv6** cho các mạng trong hệ thống.

> Tất cả các mạng phải sử dụng cùng một họ địa chỉ (IPv4 hoặc IPv6), ngoại trừ **PXE Boot Network**, mạng này luôn sử dụng IPv4.

#### Hỗ trợ IPv4 và IPv6

| Network | IPv4 | IPv6 | Ghi chú |
|----------|------|------|----------|
| PXE Boot Network | ✔ | ✘ | Chỉ hỗ trợ IPv4 do StarlingX chưa hỗ trợ IPv6 UEFI Boot. |
| Management Network | ✔ | ✔ | Hỗ trợ cả IPv4 và IPv6. |
| OAM Network | ✔ | ✔ | Hỗ trợ cả IPv4 và IPv6. |
| Cluster Host Network | ✔ | ✔ | Hỗ trợ cả IPv4 và IPv6 cho Kubernetes Cluster. |

#### Lưu ý

- PXE Boot Network phải là mạng **untagged**.
- Nếu sử dụng PXE Boot Network, mạng này được dùng để cài đặt và khởi động các node mới.
- Management, OAM và Cluster Host Network có thể được cấu hình bằng IPv4 hoặc IPv6.
- Không nên kết hợp IPv4 và IPv6 giữa các mạng trong cùng một hệ thống.

### PXE Boot Network

PXE Boot Network là mạng tùy chọn được sử dụng để khởi động và cài đặt các node mới thông qua PXE Boot.

Thông thường, StarlingX sử dụng **Management Network** cho PXE Boot nên không cần mạng PXE riêng.

#### Khi nào cần PXE Boot Network?

Cần triển khai PXE Boot Network khi:

- Management Network sử dụng VLAN Tag.
- Management Network cần hỗ trợ IPv6.
- Management Network không phù hợp cho PXE Boot.

Trong các trường hợp này, PXE Boot Network phải:

- Sử dụng **IPv4**.
- Là mạng **untagged**.

#### Lợi ích

- Tách biệt lưu lượng PXE Boot khỏi Management Network.
- Hỗ trợ các cấu hình mạng đặc biệt.
- Đơn giản hóa việc cài đặt và bổ sung node mới vào hệ thống.

> Lưu ý: StarlingX hiện không hỗ trợ PXE Boot qua IPv6, do đó PXE Boot Network luôn phải sử dụng IPv4.

### Cluster Host Network

Cluster Host Network là mạng vật lý bắt buộc cho Kubernetes trong StarlingX, cung cấp:

- Giao tiếp giữa các node trong cụm.
- Mạng nội bộ giữa container, pod và service.
- Hỗ trợ quản lý và điều khiển Kubernetes.

StarlingX sử dụng **Calico CNI** để xây dựng các mạng logic Kubernetes trên nền Cluster Host Network.

#### Đặc điểm

- Tất cả các node phải kết nối vào Cluster Host Network.
- Thường chia sẻ giao diện mạng với Management Network.
- Hỗ trợ giao tiếp container-to-container trong toàn cụm.

#### Kết nối ra bên ngoài

Workload có thể kết nối ra ngoài theo hai cách:

1. Thông qua **OAM Network** hoặc các cổng mạng được cấu hình riêng.
2. Sử dụng trực tiếp **Cluster Host Network** làm mạng external.

Các dịch vụ được publish thông qua:

- Kubernetes NodePort
- Ingress Controller
- Load Balancer

#### High Availability (HA)

HA cho ứng dụng có thể được triển khai bằng:

- Load Balancer bên ngoài.
- Nhiều bản ghi DNS trỏ tới các Controller/Worker Nodes.

#### Hỗ trợ BGP

Khi Cluster Host Network được sử dụng làm mạng external, Calico có thể:

- Quảng bá route thông qua BGP.
- Advertise Service IP hoặc Ingress IP.
- Kết nối với Router/Switch BGP bên ngoài.

Điều này cho phép workload Kubernetes được truy cập trực tiếp từ mạng bên ngoài mà không cần cấu hình route thủ công.

### Storage Network

Storage Network là mạng tùy chọn, chỉ cần thiết khi StarlingX sử dụng **NetApp Trident Cluster** làm backend lưu trữ bên ngoài.

#### Mục đích

- Kết nối các Controller Node và Worker Node tới NetApp Trident Cluster.
- Hỗ trợ truy cập và quản lý lưu trữ từ bên ngoài hệ thống StarlingX.

#### Đặc điểm

- Có thể sử dụng kết nối **10 GbE**.
- Hỗ trợ **VLAN Tagging** để chia sẻ giao diện với:
  - Management Network
  - OAM Network
- Có thể sử dụng toàn bộ hoặc một phần dải IP của subnet.
- Hỗ trợ:
  - Dynamic IP (DHCP)
  - Static IP

#### Lưu ý

- Chỉ triển khai khi sử dụng NetApp Trident Storage Backend.
- Các nguyên tắc thiết kế tương tự như Management Network.
- Không cần cấu hình Storage Network nếu sử dụng các giải pháp lưu trữ tích hợp như Ceph trong StarlingX.

### Internal Management Network Overview

Internal Management Network là mạng quản lý nội bộ dành riêng cho từng cụm StarlingX và được sử dụng cho giao tiếp giữa các thành phần của hệ thống.

> Một mạng Internal Management chỉ được sử dụng cho một cụm StarlingX duy nhất. Không hỗ trợ chia sẻ giữa nhiều cụm.

#### Vai trò

Trong quá trình cài đặt StarlingX, mạng này được sử dụng cho các dịch vụ:

- BOOTP
- DHCP
- PXE Boot

Các dịch vụ này giúp khởi tạo và đưa các node vào trạng thái hoạt động.

#### Yêu cầu

- Bắt buộc phải sẵn sàng trước khi cài đặt hệ thống.
- Mỗi node cần ít nhất một cổng mạng:
  - 1 GbE hoặc 10 GbE
- Cổng mạng phải:
  - Hỗ trợ PXE Boot.
  - Có thể được sử dụng làm thiết bị khởi động chính.

#### VLAN và IPv6

Internal Management Network có thể được cấu hình dưới dạng VLAN-tagged.

Trong trường hợp:

- Management Network sử dụng VLAN Tag.
- Hoặc cần hỗ trợ IPv6.

thì phải triển khai thêm:

- **PXE Boot Network riêng**
- Sử dụng IPv4
- Untagged trên cùng giao diện vật lý

#### Lưu ý

> Internal Management Network không được sử dụng trong mô hình **All-in-One Simplex (AIO-SX)**.


### Kubernetes Internal Management Network Planning

Internal Management Network là mạng riêng tư, chỉ được sử dụng bởi các node trong cụm StarlingX.

> Mạng này không được sử dụng trong mô hình All-in-One Simplex (AIO-SX).

#### Yêu cầu và khuyến nghị

- Được sử dụng cho PXE Boot các node mới.
- Phải là mạng **untagged** nếu dùng cho PXE Boot.
- Chỉ hỗ trợ **IPv4** cho PXE Boot.

Nếu:

- Management Network sử dụng VLAN Tag.
- Hoặc cần hỗ trợ IPv6.

thì phải triển khai thêm một **PXE Boot Network riêng** sử dụng IPv4.

#### Giao diện mạng

- Có thể sử dụng cổng 1GbE hoặc 10GbE.
- Giao diện phải:
  - Hỗ trợ Network Boot (PXE).
  - Được cấu hình làm thiết bị khởi động chính trong BIOS.

#### Địa chỉ IP

- Khuyến nghị sử dụng dải IP riêng (RFC 1918):
  - 10.0.0.0/8
  - 172.16.0.0/12
  - 192.168.0.0/16

- Nên sử dụng subnet mặc định do StarlingX đề xuất.
- Có thể chỉ định một dải IP riêng cho StarlingX hoặc để StarlingX quản lý toàn bộ subnet.

#### Static IP Allocation

Nếu sử dụng IP tĩnh:

- Node mới sẽ không tự động được thêm vào hệ thống.
- Cần thêm thủ công bằng lệnh:

```bash
system host-add




