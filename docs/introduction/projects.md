# Projects

StarlingX được xây dựng từ nhiều **sub-projects** cung cấp các dịch vụ và công cụ quản lý cần thiết cho Edge Cloud. Tài liệu API và Release Notes của từng project được cung cấp riêng cho từng thành phần.

## Bare Metal

Quản lý hạ tầng vật lý, bao gồm provisioning, inventory và vận hành các node trong hệ thống StarlingX.

## Clients

Bộ CLI và API client dùng để quản trị và tương tác với StarlingX.

## Config

Quản lý cấu hình hệ thống như host, network, storage và các dịch vụ nền tảng.

## Distributed Cloud

Cho phép quản lý nhiều cloud/site từ một System Controller trung tâm, hỗ trợ đồng bộ cấu hình và quản lý tập trung.

## Distributed Cloud Client

CLI/API client dành riêng cho các chức năng Distributed Cloud.

## Fault Management

Thu thập, quản lý và hiển thị alarm, sự kiện và trạng thái lỗi của hệ thống.

## High Availability

Cung cấp các cơ chế High Availability (HA) cho controller và các dịch vụ nền tảng.

## Horizon Plugin (GUI)

Giao diện Web tích hợp với Horizon để quản lý và vận hành StarlingX.

## Integration

Các thành phần tích hợp StarlingX với OpenStack, Kubernetes và các dịch vụ liên quan.

## NFV

Các tính năng hỗ trợ Network Function Virtualization (NFV) cho môi trường Telco và Edge Computing.

## Software Updates

Quản lý patch, update và upgrade hệ thống StarlingX.

## Upstream

Các thành phần được đồng bộ hoặc phát triển dựa trên các dự án upstream như OpenStack và Kubernetes.

## Project Structure

```text
StarlingX
├── Bare Metal
├── Clients
├── Config
├── Distributed Cloud
├── Distributed Cloud Client
├── Fault Management
├── High Availability
├── Horizon Plugin (GUI)
├── Integration
├── NFV
├── Software Updates
└── Upstream
```
