# Escalation ticket

## Ticket 1

- **Frame:** adasind_062370.jpg
- **Ảnh chụp:** submission/screenshots/escalation_ticket_1.png
- **Expected impact:**
  Xung đột phân lớp nghiêm trọng giữa class `Bike` và `ThreeWheeler`/hàng hóa chở thêm tại các đô thị đông đúc. Nếu mô hình học theo các box không nhất quán (chỗ gộp hàng, chỗ bỏ rơi hàng hóa cồng kềnh nhô ra ngoài thân xe), hệ thống xe tự hành ADAS có nguy cơ tính toán sai kích thước bao (bounding volume), dẫn đến va quẹt với phần hàng nhô ra khi căn khoảng cách vượt xe ở cự ly hẹp (< 0.5m).
- **Owner:** guideline
- **Recommendation:**
  1. Hội đồng quy chuẩn dữ liệu (Guideline Committee) cần ban hành phụ lục hướng dẫn có ảnh minh họa thực tế về "Xe chở hàng quá khổ và xe tự chế" trong giao thông đô thị đặc thù.
  2. Quy định rõ: Nếu kiện hàng gắn liền cố định trên giá xe, bounding box phải bao trọn toàn bộ kiện hàng và gán attribute `occluded=true` nếu hàng che mất biển số/đèn hậu; nếu hàng kéo lê rời phía sau cần tách thành vật thể rời hoặc vùng cảnh báo an toàn.
  3. Cập nhật bài kiểm tra định kỳ (calibration benchmark) cho đội ngũ gán nhãn trước khi giải phóng đợt gán nhãn quy mô lớn.
