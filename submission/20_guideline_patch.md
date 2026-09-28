# Guideline patch

- **Rule mới đề xuất:**
  1. Chuẩn hóa quy tắc bao bọc đối tượng xe hai bánh (`Bike`): Toàn bộ người điều khiển và hành khách ngồi sau (pillion passenger) cùng hành lý/ba lô gắn trên người phải được bao trọn trong một bounding box duy nhất có nhãn `Bike`. Chỉ gán `Pedestrian` độc lập khi hành khách đã bước xuống khỏi xe và đặt cả hai chân tiếp xúc mặt đường.
  2. Định lượng thuộc tính `edge_zone`: Gán `edge_zone=true` cho bất kỳ bounding box nào có tâm hình học nằm ngoài 75% bán kính vùng tròn quang học (optical lens circle radius) hoặc có ít nhất một cạnh chạm/cắt vào đường biên `lens_border`.
- **Áp dụng cho:** Class `Bike`, `Pedestrian`, attribute `edge_zone`, `occluded`, nhãn vùng `ignore_region` (lý do `lens_border` và `ego_body`).
- **Vì sao luật hiện tại (`docs/02-rules-vi.md`) không đủ:**
  Luật hiện tại chỉ nêu chung chung "vẽ trùm cả người lái" nhưng không làm rõ tình huống xe chở 2–3 người hoặc hàng cồng kềnh, dẫn đến việc chuyên viên gán nhãn có người gộp, có người tách thành Pedestrian gây xung đột taxonomy. Ngoài ra, việc xác định `edge_zone` bằng mắt thường trên thấu kính méo fisheye thiếu tính định lượng cơ học, dẫn đến tỷ lệ bất đồng thuận (disagreement rate) giữa các annotator lên tới 25% ở các vùng chuyển tiếp (mid-to-edge).
- **`rules_version` mới:** v1.1.0
- **Hiệu lực từ:** Vòng Rework (P5) và áp dụng bắt buộc cho toàn bộ các lô dữ liệu (batches) tiếp theo của dự án SVM 360.
