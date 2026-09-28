# Error analysis card

## Zone × block

| zone | block | what | count |
|---|---|---|---:|
| center | B2 | BOX_GEOMETRY | 1 |
| center | B2 | MISSING | 6 |
| center | B2 | SPURIOUS | 7 |
| center | B2 | WRONG_CLASS | 1 |
| edge | B2 | ATTRIBUTE | 1 |
| edge | B2 | MISSING | 1 |
| edge | B2 | SPURIOUS | 2 |
| mid | B2 | ATTRIBUTE | 1 |
| mid | B2 | BOX_GEOMETRY | 1 |
| mid | B2 | IGNORE_SCOPE | 1 |
| mid | B2 | MISSING | 2 |
| mid | B2 | SPURIOUS | 3 |

## Top defects
- SPURIOUS: 12 (ví dụ frame adasind_062370.jpg)
- MISSING: 9 (ví dụ frame adasind_062370.jpg)
- BOX_GEOMETRY: 2 (ví dụ frame adasind_062370.jpg)

## Phân tích của bạn

Hai bảng trên do `python3 lab11.py card` tính từ `findings.csv`; chạy lại lệnh sẽ cập nhật bảng và giữ nguyên mục này. Viết cho lỗi nổi bật nhất, dẫn frame/`object_ref`.

- Nguyên nhân khả dĩ (`why`) và vì sao bạn nghĩ vậy:
  + Đối với SPURIOUS (12 ca) và MISSING (9 ca) từ mô hình: Nguyên nhân gốc rễ là `E4_model_domain` (khoảng cách miền dữ liệu / domain shift của mô hình YOLO26m). Mô hình được huấn luyện trên ảnh phẳng tiêu chuẩn phối cảnh thẳng (pinhole perspective), hoàn toàn chưa được tinh chỉnh (fine-tune) trên ảnh thấu kính mắt cá góc siêu rộng (fisheye optical distortion). Trên các frame mật độ cao như `adasind_062370.jpg` và `adasind_117120.jpg`, biến dạng kéo giãn thấu kính và bóng đổ phức tạp khiến mô hình nhận nhầm bóng râm thành vật thể (SPURIOUS) hoặc không tách biệt được các xe máy đi sát nhau (MISSING).
  + Đối với BOX_GEOMETRY (2 ca) và ATTRIBUTE (2 ca) từ khâu gán nhãn thủ công: Nguyên nhân là `E1_annotator_error` và `E2_guideline_gap`. Trong không gian ảnh fisheye bị méo cong, người gán nhãn có xu hướng vẽ hộp bao nới lỏng quá mức nhằm bao trùm hết tay lái/bánh xe khiến box bị lỏng biên; đồng thời việc xác định ngưỡng thuộc tính `edge_zone` tại ranh giới vòng tròn quang học cần tiêu chuẩn định lượng cụ thể hơn trong guideline.
- Cách sửa và ai nhận việc (`owner`):
  + Đối với lỗi mô hình (`E4_model_domain`): `ai_team` phụ trách. Cần bổ sung pipeline biến đổi dữ liệu mô phỏng thấu kính mắt cá (Fisheye Distortion Data Augmentation) và nạp thêm tập dữ liệu SVM bốn camera để tái huấn luyện mô hình phát hiện vật thể.
  + Đối với lỗi gán nhãn thủ công (`E1_annotator_error`): `annotator` phụ trách. Thực hiện tinh chỉnh lại tọa độ box ôm sát pixel thực tế của vật thể theo quy chuẩn `R_BOX_TIGHT` và cập nhật bản `annotations-v2.xml` trong vòng rework.
  + Đối với quy chuẩn phân lớp (`E2_guideline_gap`): `guideline` phụ trách. Ban hành bản vá hướng dẫn `20_guideline_patch.md` nhằm chuẩn hóa tiêu chí gán nhãn hành khách ngồi sau xe máy và ngưỡng pixel gán nhãn `edge_zone`.
- Bằng chứng (ảnh trong `screenshots/`, dòng findings, rule):
  + Dòng findings `r3_diag` trên frame `adasind_062370.jpg` (vật thể `M3`, `M4`, `M5`, `M8`, `M10`): Mô hình sinh box ảo trên nền đường nhựa và bóng đổ.
  + Dòng findings `r1_craft` trên frame `adasind_062370.jpg` (`L2`): Vi phạm quy tắc hộp bao ôm sát `R_BOX_TIGHT`.
  + Ảnh chụp minh chứng giao diện CVAT và overlay so sánh đã được lưu trữ tại `submission/screenshots/cvat_b2_dense_overview.png` và `submission/screenshots/cvat_model_comparison.png`.
