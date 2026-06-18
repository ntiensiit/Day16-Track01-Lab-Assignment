# Lab 2 — JTBD Project Analysis

**Người làm:** Nguyễn Tiến Sỉ — 2A202600681

## Dự án / sản phẩm

**Tên mô tả:** Synthetic-to-Real Dataset Pipeline

## Project slice

Dự án tập trung vào workflow tạo dữ liệu huấn luyện có nhãn đáng tin, kiểm tra chất lượng dữ liệu và đo hiệu quả trên tập kiểm thử giữ riêng.

## Job executor

Người trực tiếp hoàn thành job là kỹ sư phụ trách dữ liệu và mô hình nhận diện.

## Core JTBD

Tạo và kiểm chứng bộ dữ liệu nhận diện vật thể có nhãn đáng tin khi cần huấn luyện hệ thống hoạt động tốt trên ảnh camera thật trong bối cảnh cảnh vật thay đổi.

## Job stories

1. Khi thêm vật thể mới hoặc đổi bối cảnh, tôi muốn tạo nhanh dữ liệu có nhãn để không phải chụp và gán nhãn lại từ đầu.
2. Khi kết quả trên ảnh thật chưa tốt, tôi muốn so sánh các biến thể dữ liệu bằng cùng một benchmark để chọn hướng cải thiện dựa trên số đo.
3. Khi ảnh tạo ra trông tốt nhưng có thể sai nhãn, tôi muốn kiểm tra chất lượng trước khi dùng để huấn luyện.

## Current alternatives

- Chụp ảnh thật và gán nhãn thủ công: đáng tin nhưng chậm.
- Dữ liệu mô phỏng đơn giản: nhanh nhưng còn khoảng cách với ảnh thật.
- Thuê ngoài gán nhãn: giảm tải nội bộ nhưng tốn chi phí và cần review.
- Public dataset: rẻ nhưng thường không khớp bối cảnh.

## JTBD lite map

| Step | Pain |
|---|---|
| Define | Dễ scope quá rộng |
| Locate | Dữ liệu phân tán |
| Prepare | Nhiều bước dễ lỗi |
| Confirm | Khó kiểm tra nhãn ở quy mô lớn |
| Execute | Pipeline dài |
| Monitor | Log rời rạc |
| Modify | Tốn thời gian thử-sai |
| Conclude | Dễ diễn giải sai nếu metric không rõ |

**2 bước đau nhất:** Confirm và Conclude.

## AI leverage point

AI nên hỗ trợ tạo dữ liệu, kiểm tra chất lượng và tóm tắt kết quả benchmark. Giá trị chính không phải là tạo ảnh đẹp, mà là tạo dữ liệu có kiểm chứng.

## Product hypothesis

Nếu sản phẩm giúp người dùng tạo, lọc và benchmark dữ liệu nhanh hơn, thì họ sẽ chuyển từ workflow thủ công sang pipeline của nhóm vì giảm vòng lặp chụp–gán nhãn và có bằng chứng định lượng để ra quyết định.

## Assumptions to validate

1. Người dùng chính đúng là kỹ sư phụ trách dữ liệu/model.
2. Pain tạo và gán nhãn dữ liệu đủ lớn để họ đổi workflow.
3. Dữ liệu tạo ra cải thiện model ổn định trên ảnh thật.
4. Quality gate bắt được lỗi quan trọng.
5. User chấp nhận độ phức tạp kỹ thuật của MVP.

## Verdict cuối

Dự án nên định vị là validation-first dataset pipeline. GenAI realism là điểm khác biệt, nhưng phải được chứng minh bằng quality gate và benchmark thay vì chỉ dựa vào ảnh nhìn đẹp.

## Checklist

- [x] Có project slice + market context.
- [x] Có job executor + core JTBD.
- [x] Có 3 job stories.
- [x] Có JTBD lite map + pain points.
- [x] Có AI leverage point + product hypothesis.
- [x] Có assumptions to validate + verdict cuối.
- [x] Không còn tên repo dự án, link repo dự án hoặc mã đề tài.
