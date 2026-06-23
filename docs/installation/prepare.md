# Chuẩn bị môi trường

Phần này mô tả các bước chuẩn bị máy chủ vật lý và môi trường ảo hóa trước khi triển khai **StarlingX Standard với Dedicated Storage**.

## Yêu cầu về host vật lý

Máy chủ cài đặt cần đáp ứng tối thiểu:

| Thành phần | Yêu cầu                               |
| ---------- | ------------------------------------- |
| CPU        | x86_64 hỗ trợ Hardware Virtualization |
| vCPU       | 8 Cores                               |
| RAM        | 32 GB                                 |
| Disk       | 500 GB                                |
| Network    | 1 NIC có kết nối Internet             |


## Chuẩn bị máy ảo

Tạo các máy ảo:

* dedicatedstorage-controller-0
* dedicatedstorage-controller-1
* dedicatedstorage-storage-0
* dedicatedstorage-storage-1
* dedicatedstorage-worker-0
* dedicatedstorage-worker-1


