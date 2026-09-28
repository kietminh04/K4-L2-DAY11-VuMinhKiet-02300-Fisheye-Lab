# Đối chiếu chất lượng cục bộ — rectangle

Teaching reference, không phải gold set đã phê duyệt; không có điểm đạt tự động.
Nguồn: export r1_craft đã khóa SHA256 `5df89977a413cb1395fdc1a1776fa2568327b427ba81bec9422813e460be9626`; slice `B2-dense`.
Ghép hình học greedy một-một theo IoU ≥ 0.50, rồi so class; H ≥ 40 px.
Box trái nằm chủ yếu trong ignore_region reference không tính. Polygon, polyline, track không được chấm.
Đây là phép tính offline của lab, không phải báo cáo hay kết quả tương đương CVAT Premium.

Frame được tính: adasind_062370.jpg, adasind_069450.jpg, adasind_117120.jpg. Frame thiếu trong export: không.
TP=20; FP=0; FN=0; số lần đối chiếu=20; mean IoU của TP=1.000.

| Chỉ số | Micro | Macro | Nhãn thấp nhất |
|---|---:|---:|---:|
| accuracy | 1.000 | 1.000 | 1.000 |
| precision | 1.000 | 1.000 | 1.000 |
| recall | 1.000 | 1.000 | 1.000 |
| jaccard | 1.000 | 1.000 | 1.000 |
| dice | 1.000 | 1.000 | 1.000 |

| Nhãn | TP | FP | FN | Accuracy | Precision | Recall | Jaccard | Dice |
|---|---:|---:|---:|---:|---:|---:|---:|---:|
| Bike | 4 | 0 | 0 | 1.000 | 1.000 | 1.000 | 1.000 | 1.000 |
| Car | 4 | 0 | 0 | 1.000 | 1.000 | 1.000 | 1.000 | 1.000 |
| Pedestrian | 4 | 0 | 0 | 1.000 | 1.000 | 1.000 | 1.000 | 1.000 |
| ThreeWheeler | 7 | 0 | 0 | 1.000 | 1.000 | 1.000 | 1.000 | 1.000 |
| Truck | 1 | 0 | 0 | 1.000 | 1.000 | 1.000 | 1.000 | 1.000 |

| Frame | TP | FP | FN | Accuracy | Precision | Recall |
|---|---:|---:|---:|---:|---:|---:|
| adasind_062370.jpg | 9 | 0 | 0 | 1.000 | 1.000 | 1.000 |
| adasind_069450.jpg | 5 | 0 | 0 | 1.000 | 1.000 | 1.000 |
| adasind_117120.jpg | 6 | 0 | 0 | 1.000 | 1.000 | 1.000 |

Confusion matrix: hàng = teaching reference; cột = export đã khóa.
`<missing>` là thiếu box; `<extra>` là box thừa. Xem `local_quality_confusion.csv`.

| Reference \ Export | Bike | Car | Pedestrian | ThreeWheeler | Truck | <missing> |
|---|---:|---:|---:|---:|---:|---:|
| Bike | 4 | 0 | 0 | 0 | 0 | 0 |
| Car | 0 | 4 | 0 | 0 | 0 | 0 |
| Pedestrian | 0 | 0 | 4 | 0 | 0 | 0 |
| ThreeWheeler | 0 | 0 | 0 | 7 | 0 | 0 |
| Truck | 0 | 0 | 0 | 0 | 1 | 0 |
| <extra> | 0 | 0 | 0 | 0 | 0 | 0 |

Chi tiết xung đột trong `local_quality_conflicts.csv`; dữ liệu máy đọc trong `local_quality.json`.
Mismatching label đóng góp một FP cho class vẽ và một FN cho class reference; attribute khác được báo riêng.
Micro accuracy đếm mỗi cặp ghép sai class là một lần đối chiếu; Jaccard đếm cả FP và FN.
Macro/worst bỏ nhãn không xuất hiện ở cả hai phía; chỉ số không có mẫu là N/A.
