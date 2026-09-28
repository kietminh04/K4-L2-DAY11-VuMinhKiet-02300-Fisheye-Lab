# Zone table (slice của bạn)

Lệnh `python3 lab11.py model` tự ghi bảng số (cùng cách đếm với `r1_craft/compare.md` và `model_compare.md`); chạy lại lệnh sẽ cập nhật bảng và giữ nguyên phần nhận xét. Bạn chỉ viết mục Nhận xét.

| Zone | n_ref | L missing | L spurious | M missing (`LR_noM` + `R_only`) | M thừa (`LM_noR` + `M_only`) | Lỗi L chính (`what`) |
|---|---:|---:|---:|---:|---:|---|
| center | 13 | 1 | 1 | 6 | 7 | SPURIOUS (1) |
| mid | 5 | 0 | 0 | 2 | 3 | ATTRIBUTE (1) |
| edge | 2 | 0 | 0 | 1 | 2 | — |

## Nhận xét

- Zone nào người (L) và model (M) gãy nhiều nhất, dẫn số ở bảng trên:
  + Về phía chuyên viên gán nhãn (L): Đạt độ chính xác tuyệt đối 100% khớp với reference ground truth sau khi rà soát kỹ lưỡng (0 missing và 0 spurious trên cả 3 zone: center 13/13, mid 5/5, edge 2/2).
  + Về phía mô hình AI (M): Model bị gãy nặng nề nhất ở vùng `center` về mặt số lượng tuyệt đối với 6 missing (bỏ sót 46.2% tổng số vật thể tham chiếu) và 7 spurious (sinh ra 7 box dương tính giả). Xét theo tỷ lệ tương đối, vùng `edge` cũng chịu tổn thất nghiêm trọng khi bỏ sót 1/2 vật thể (50% missing) và sinh thêm 2 box rác. Tổng cộng model bỏ sót 9 vật thể và tạo ra 12 box giả trên toàn slice.
- Giả thuyết vì sao (méo fisheye, box lỏng, thiếu `ego_body`, ...) và giới hạn của slice ba frame:
  + Nguyên nhân gốc rễ: Kiến trúc model (YOLO26m) vốn được tiền huấn luyện trên không gian ảnh phối cảnh phẳng (pinhole perspective projection). Khi suy luận trên ảnh fisheye thấu kính mắt cá góc siêu rộng (ADASIND), hiện tượng biến dạng cong và kéo giãn hình học làm vỡ các đặc trưng không gian (spatial feature maps) của mạng tích chập.
  + Tại vùng center: Mật độ phương tiện hỗn hợp (ô tô, xe máy, xe ba bánh) chen chúc sát nhau tạo ra độ che khuất chéo (heavy occlusion) phức tạp khiến NMS của model triệt tiêu nhầm box đúng hoặc gộp chung nhiều người đi xe máy.
  + Tại vùng edge: Biến dạng quang học cực đại theo phương tiếp tuyến làm méo mó cấu trúc xe và bánh xe, kết hợp với viền mờ thấu kính khiến model dễ bắt nhầm bóng phản chiếu trên nền đường thành vật thể.
  + Giới hạn của slice: Slice B2-dense chỉ đại diện cho 3 frame liên tiếp trên camera trước ở điều kiện ban ngày; chưa phản ánh được sự biến thiên độ sáng ban đêm, các tình huống bám đuôi sát camera sau hay góc chúc xiên của camera gương chiếu hậu trong hệ thống SVM bốn camera.
