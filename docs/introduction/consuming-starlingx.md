# Cách sử dụng StarlingX

## 1. Tổng quan

StarlingX hiện đã sẵn sàng để sử dụng, tuy nhiên phiên bản mã nguồn mở vẫn tồn tại một số hạn chế về tính năng.

Một số tính năng quan trọng như:

* Secure Boot
* Live Software Update (cập nhật phần mềm không downtime)

chưa được hỗ trợ đầy đủ trong bản community.

Nguyên nhân là do:

* Cộng đồng **không cung cấp các image đã được ký số (signed images)**
* Trong khi các tính năng bảo mật nâng cao yêu cầu image phải được ký

Việc cung cấp signed images thường do:

* Nhà cung cấp thương mại (vendor)
* Hoặc chính người dùng / doanh nghiệp tự thực hiện

---

## 2. Các cách triển khai StarlingX

### 2.1 Triển khai từ mã nguồn mở (Open Source)

Có thể sử dụng trực tiếp phần mềm mã nguồn mở.

* Có thể tải ISO từ mirror (ví dụ: Wind River)
* Hoặc tự build image

Lưu ý:

* Các image này **không được ký**
* Không hỗ trợ:

  * Secure Boot
  * Live Software Update

Khuyến nghị:

* Sử dụng **release image** → đã được test và validate
* Developer có thể dùng **daily build** → để phát triển

---

### 2.2 Triển khai phiên bản nội bộ (Internal Build)

Doanh nghiệp có thể tự xây dựng phiên bản StarlingX riêng:

Bao gồm:

* Kiểm thử (acceptance testing)
* Tùy biến theo nhu cầu
* Ký số image nội bộ

Từ đó có thể:

* Kích hoạt Secure Boot
* Triển khai Live Software Update (patch)

Phù hợp với:

* Doanh nghiệp lớn
* Tổ chức có đội platform riêng

---

### 2.3 Sử dụng bản từ nhà cung cấp (Vendor)

Bạn có thể sử dụng sản phẩm StarlingX từ vendor thương mại.

Vendor sẽ cung cấp:

* Signed images
* Signed software updates
* Hỗ trợ kỹ thuật (support)
* Có thể bổ sung thêm tính năng

Ưu điểm:

* Đầy đủ tính năng bảo mật
* Có support SLA

Nhược điểm:

* Chi phí cao
* Phụ thuộc vendor

---

## 3. Kết luận

Có 3 mô hình sử dụng StarlingX:

1. Open Source → miễn phí, phù hợp lab/dev, hạn chế tính năng
2. Internal Build → linh hoạt, phù hợp doanh nghiệp lớn
3. Vendor Solution → đầy đủ tính năng, có support

Việc lựa chọn phụ thuộc vào:

* Nhu cầu bảo mật
* Khả năng vận hành
* Nguồn lực kỹ thuật
