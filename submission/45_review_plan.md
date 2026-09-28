# Kế hoạch review từ lỗi quan sát được

Từ `findings.csv` và `zone_table.md`, chọn **hai lát cắt của bài ADASIND một camera** cần review trước. Bảng này
giải thích dữ liệu thật bạn vừa làm; nó không thay cho kế hoạch bốn camera giả lập ở `45_sampling_plan.csv`.

| Lát cắt / frame | Số ca và loại lỗi | Vì sao review trước | Bằng chứng cần giữ |
|---|---|---|---|
| Lát cắt mật độ cao vùng trung tâm (Center Zone trên adasind_062370.jpg và adasind_117120.jpg) | 13 ca lỗi (6 Missing do che khuất chéo và 7 Spurious do bóng đổ/hoa văn đường) | Vùng center tập trung lưu lượng giao thông trực diện và là hướng di chuyển chính của xe ego; lỗi bỏ sót xe máy hoặc nhận diện sai ở cự ly 5–15m có rủi ro va chạm trực tiếp cao nhất. | Ảnh crop so sánh model vs ground truth, ma trận IoU ngưỡng 0.5, log phân loại lỗi MISSING/SPURIOUS trong findings.csv. |
| Lát cắt biến dạng quang học vùng rìa (Edge Zone trên adasind_069450.jpg) | 3 ca lỗi (1 Missing xe ba bánh sát mép thấu kính, 2 Spurious tại viền mờ thấu kính) | Vùng rìa thấu kính mắt cá chịu độ méo phi tuyến cực đại và suy giảm độ phân giải góc; dễ xảy ra tranh chấp nhãn giữa vật thể ngoài lề đường và vật cản trong luồng chạy. | Overlay hiển thị đường biên thấu kính (lens_border circle), tọa độ góc cực (polar coordinates) và cờ thuộc tính edge_zone=true. |

Giới hạn của kết luận từ ba frame ADASIND:
Ba frame tĩnh trong slice B2-dense chỉ đại diện cho một góc nhìn camera trước duy nhất, trong một điều kiện thời tiết nắng ráo ban ngày tại một tuyến phố đô thị cụ thể. Mẫu quan sát này không đủ độ bao phủ để đại diện cho toàn bộ phân phối dữ liệu (data distribution) thực tế của hệ thống xe tự hành, bao gồm các kịch bản trời mưa ướt kính lái, ban đêm ánh đèn rọi ngược, các camera hông chịu góc nhìn xiên hẹp và camera sau với điểm mù cản thấp.

## Chuyển sang kế hoạch bốn camera giả lập

Cách soát độ phủ của 200 frame ở `45_sampling_plan.csv` (kể cả tránh đếm nhiều frame liền nhau trong cùng cảnh như nhiều ca độc lập), và vì sao kế hoạch đó chỉ giúp tìm ca cần soi, chưa đo được tỷ lệ lỗi:
- Để bảo đảm tính đại diện và độ phủ thực chất của 200 frame, quy trình lấy mẫu bắt buộc phải áp dụng kỹ thuật lọc cửa sổ thời gian (time-window stride filtering, tối thiểu cách nhau 3–5 giây hoặc > 30 mét di chuyển) và phân tầng theo thuộc tính kịch bản (scenario stratification: thời tiết, mật độ, loại cung đường). Tuyệt đối không trích xuất các frame liên tiếp trong cùng một chuỗi video ngắn (burst frames) vì sự tương đồng tự tương quan cao (temporal autocorrelation) sẽ làm sai lệch đánh giá độ đa dạng của mẫu.
- Kế hoạch lấy mẫu tập trung vào ca khó (hard slices chiếm 45% tổng mẫu) đóng vai trò như một bộ lọc nhắm mục tiêu (targeted diagnostic probing) để phát hiện sớm các điểm gãy tiềm ẩn của hệ thống (edge cases, failure modes, boundary defects). Do mẫu bị lệch có chủ đích về phía các tình huống khắc nghiệt, tỷ lệ lỗi quan sát được trên tập 200 frame này chỉ phản ánh mức độ nghiêm trọng của ca khó, hoàn toàn không thể dùng làm đại lượng thống kê suy diễn tỷ lệ lỗi trung bình (true population error rate) của toàn bộ 50.000 frame trong vận hành thường nhật.
