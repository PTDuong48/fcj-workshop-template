---
title: "Event 2"
date: 2024-01-01
weight: 2
chapter: false
pre: " <b> 4.2. </b> "
---



# FCAJ Community Day

### Tổng quan sự kiện

- **Tên Sự Kiện:** FCAJ Community Day
- **Thời gian tổ chức:** 23/05/2026
- **Địa điểm tổ chức:** Tầng 26 - Văn phòng AWS Việt Nam (Tòa nhà Bitexco, Số 2 Hải Triều, Phường Sài Gòn, TP.HCM)
- **Đơn vị tổ chức:** Cộng đồng FCAJ (với sự hỗ trợ của AWS Study Group).
- **Mục Đích Của Sự Kiện:**
  - Không chỉ là một buổi hội thảo nghe nhìn đơn thuần, mà là nơi để các thành viên kết nối và truyền cảm hứng cho nhau trong lĩnh vực công nghệ.
  - Cung cấp cái nhìn thực tế về thị trường việc làm trong kỷ nguyên AI.
  - Chia sẻ các kiến thức kỹ thuật từ AI, Cloud đến những trải nghiệm thực tế trong dự án và các cuộc thi.
  - Nội dung chính muốn truyền tải: Tinh thần chủ động học hỏi, trang bị kiến thức thực tế (case study) và sản phẩm thật thay vì chỉ dựa vào bằng cấp hay demo lý thuyết.
  - Giá trị dành cho người tham dự: Cơ hội kết nối với đối tác tiềm năng, học hỏi kinh nghiệm thực chiến từ các diễn giả đầu ngành và cập nhật các xu hướng công nghệ mới nhất như AI Agent, CloudFront pricing, và Multi-agent system.

### Danh Sách Diễn Giả

1. **Anh Nguyễn Gia Hưng:**
   - Chức vụ: Solution Architect tại AWS Việt Nam.
   - Vai trò: Người sáng lập FCAJ, diễn giả mở đầu về xu hướng thị trường.
2. **Anh Tịnh Trương:**
   - Chức vụ: Platform Engineer.
   - Đơn vị công tác: Got It (Gothamic).
3. **Anh Hải Anh:**
   - Đơn vị công tác: Pacific Việt Nam.
4. **Anh Nguyễn Tuấn Thịnh:**
   - Chức vụ: DevOps Engineer.
5. **Nhóm UTM (Uyển, Thảo, Mạch):** Đội ngũ đạt giải tại cuộc thi Hackathon.
6. **Diễn giả nữ (Chị Vy):**
   - Đơn vị công tác: VPBank (chia sẻ về hệ thống Multi-agent trong ngân hàng).

### Nội Dung Nổi Bật

- **Tổng quan vấn đề:**
  - Thị trường việc làm đang thay đổi mạnh mẽ do AI, khiến việc phát triển phần mềm rẻ hơn nhưng yêu cầu về vận hành (Platform Engineering) lại tăng cao.
  - Nỗi lo về chi phí Cloud tăng đột biến ("Bill spike") khi sử dụng mô hình Pay-as-you-go.
  - Sự thiếu hụt tính xác định (determinism) của các mô hình ngôn ngữ lớn (LLM) trong môi trường doanh nghiệp.
- **Giải pháp được giới thiệu:**
  - Sử dụng Amazon CloudFront Flat-rate pricing để cố định chi phí CDN hàng tháng.
  - Xây dựng hệ thống Multi-agent (Đa tác nhân) để giải quyết các bài toán phức tạp như đánh giá tín dụng.
  - Cung cấp ngữ cảnh (Context) đúng và đủ cho AI để tối ưu hóa kết quả đầu ra.
- **Công nghệ/Dịch vụ/Công cụ:** AWS CloudFront, Amazon QuickSight, Amazon Bedrock, Terraform, Obsidian (cho AI Second Brain), và các mô hình LLM (GPT, Llama, Claude).
- **Demo hoặc Case Study:**
  - Dự án UTMorpho: Công cụ chỉnh sửa UI trực tiếp bằng AI được xây dựng trong 36 giờ Hackathon.
  - Hệ thống đánh giá tín dụng cho Startup: Sử dụng Multi-agent để phân tích dữ liệu phi truyền thống cho ngân hàng.
- **Các điểm đáng chú ý:** Tầm quan trọng của Security & Compliance trong doanh nghiệp lớn; AI không chỉ là chatbot mà còn phải biết hành động (Agentic AI).

### Những Gì Học Được

- **Tư duy và phương pháp:**
  - AI Mindset & AI Adoption: Cách áp dụng AI vào quy trình doanh nghiệp một cách hiệu quả.
  - Working Backward: Bắt đầu từ nhu cầu khách hàng để xây dựng hệ thống.
- **Kiến thức kỹ thuật:** Hiểu về tham số Temperature (t=0 không hoàn toàn định tính trên Cloud do tối ưu hóa phần cứng), cơ chế catching đa tầng của CDN và bảo mật VPC Origin.
- **Best practices:** Luôn kiểm tra (Testing) nhiều lần, sử dụng JSON Mode để ổn định format đầu ra của AI và thực hiện xoay vòng API Key thường xuyên.
- **Kinh nghiệm thực tế:** Trong doanh nghiệp, "Ship in thì Ship out" (Dữ liệu rác đầu vào sẽ cho kết quả rác đầu ra), cần sự can thiệp của con người để kiểm chứng tri thức AI.

### Ứng Dụng Vào Công Việc

- **Có thể áp dụng:** Thiết kế hệ thống AI có khả năng xử lý song song (parallel processing) và có cơ chế giám sát (audit trail).
- **Công nghệ muốn thử nghiệm:** Triển khai hạ tầng bằng Terraform (IaC) để dễ dàng quản lý và tái tạo môi trường.
- **Ý tưởng cải thiện quy trình:** Tích hợp AI Agent để tự động hóa việc tóm tắt cuộc họp và gửi email hành động tiếp theo cho các bên liên quan.

### Trải Nghiệm Trong Event

- **Học hỏi từ diễn giả:** Những câu chuyện thực chiến "xương máu" về việc copy code từ GPT gây lỗi trên production.
- **Trải nghiệm thực hành:** Xem demo trực tiếp về cách AI stream mã nguồn HTML và cho phép kéo thả chỉnh sửa giao diện.
- **Giao lưu và kết nối:** Khuyến khích người tham dự chủ động bắt chuyện với người ngồi cạnh vì họ có thể là đối tác tương lai.
- **Điều ấn tượng nhất:** Sự nhiệt huyết của cộng đồng, tinh thần không ngại chia sẻ cả những thất bại và bài học thực tế.

### Bài Học Rút Ra

- **Kiến thức quan trọng nhất:** AI là mô hình xác suất (probabilistic), không phải định tính (deterministic), nên cần thiết kế hệ thống để xử lý sự ngẫu nhiên này.
- **Kinh nghiệm thực tế:** Đừng chỉ đưa demo, hãy đưa ra sản phẩm có giá trị thực tế cho người dùng cuối.
- **Định hướng phát triển tiếp theo:** Tập trung vào kiến thức Kỹ thuật phần mềm (Software Engineering) vững chắc bên cạnh kỹ năng AI để có thể triển khai sản phẩm thực tế thay vì chỉ làm trong phòng thí nghiệm.

### Một số hình ảnh khi tham gia sự kiện

![Hình ảnh sự kiện 1](event2-1.png)

![Hình ảnh sự kiện 2](event2-2.png)

![Hình ảnh sự kiện 3](event2-3.png)

![Hình ảnh sự kiện 4](event2-4.png)

![Hình ảnh sự kiện 5](event2-5.png)
