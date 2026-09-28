# Exit ticket

Đọc `docs/10-svm360-reading-vi.md` trước khi trả lời câu 1–2. Các câu về zone, `why`, rework, parking và sampling
đã nằm trong file tương ứng nên không hỏi lại ở đây.

1. Một vật ở vùng seam giữa hai camera thật xuất hiện với hai box khác nhau: đó là lỗi `DUPLICATE` hay cần một quy tắc riêng? Vì sao?
   - Đây **không phải lỗi `DUPLICATE` thuần túy** trên một ảnh đơn lẻ, mà là hiện tượng vật lý quang học bình thường khi vật thể đi vào vùng trường nhìn chồng lấn (overlapping field-of-view / seam line) giữa hai camera liền kề (ví dụ camera trước và camera gương bên trái).
   - Cần một **quy tắc riêng biệt (Cross-Camera Seam Association & Resolution Policy)** vì:
     + Trên mỗi camera đơn lẻ ở không gian 2D gốc (raw 2D image plane), việc gán nhãn vật thể độc lập là hoàn toàn hợp lệ và chính xác về mặt quang học cục bộ. Nếu vội vàng xóa một box và gán cờ DUPLICATE, camera đó sẽ bị mất dữ liệu huấn luyện (missing signal).
     + Khi chuyển lên không gian kết hợp 3D/BEV (Bird's Eye View), hệ thống dung hợp (Fusion module) cần một chính sách rõ ràng: hoặc chọn camera có góc nhìn trực diện hơn/độ phân giải cao hơn làm camera chính (primary observation), hoặc áp dụng thuật toán dung hợp đặc trưng có trọng số. Nếu chưa có thông tin hiệu chuẩn ngoại vi (extrinsic calibration) và đồng bộ thời gian chuẩn, tuyệt đối không được tự ý gộp hai box hoặc xóa bỏ box ở một trong hai camera.

2. Một vật đi qua nhiều frame trên cùng camera: khi nào giữ cùng track ID, khi nào thêm keyframe hoặc trạng thái Outside? Nêu bằng chứng sẽ cần trước khi nối track qua hai camera.
   - **Giữ cùng Track ID**: Khi vật thể di chuyển liên tục trong tầm nhìn của camera, không bị che khuất hoàn toàn (hoặc chỉ bị che khuất tạm thời dưới 0.5–1 giây / 5–10 frame) và quỹ đạo chuyển động có tính liên tục vật lý (smooth trajectory, vận tốc và gia tốc hợp lý).
   - **Thêm Keyframe**: Khi vật thể có sự thay đổi đột ngột về hình thái hình học (ví dụ: xe đang đi thẳng bắt đầu rẽ cua, thay đổi góc nhìn từ đuôi xe sang sườn xe, hoặc thay đổi trạng thái che khuất `occluded`/biến dạng `edge_zone`).
   - **Trạng thái Outside**: Đánh dấu `outside=true` ngay tại frame đầu tiên mà toàn bộ vật thể đi ra khỏi biên trường nhìn (hoặc bị che khuất vĩnh viễn không còn khả năng tái xuất hiện trong chuỗi ngắn).
   - **Bằng chứng cần thiết trước khi nối Track qua hai camera**:
     + (1) Bằng chứng đồng bộ thời gian phần cứng (hardware timestamp synchronization) với độ trễ cực nhỏ (< 10ms);
     + (2) Bằng chứng hình học không gian: Ma trận hiệu chuẩn ngoại vi (extrinsic calibration matrix) giữa hai camera đã được xác thực không bị trôi lệch, và vị trí chiếu lại (re-projection) của vật thể trên mặt phẳng mặt đất 3D trùng khớp với sai số dung sai cho phép (< 0.2m);
     + (3) Tính liên tục của vectơ vận tốc và hướng di chuyển (kinematic consistency) khi vật thể chuyển tiếp từ ranh giới rời camera này sang ranh giới bước vào camera kia.

3. Nhìn lại cả buổi: một chỗ bạn tin nhãn mình đúng nhưng reference hoặc người soát nghĩ khác (dẫn frame/`object_ref`), bạn đã xử lý thế nào, và nếu làm lại slice này bạn sẽ đổi gì trong cách làm?
   - **Tình huống thực tế**: Trên frame `adasind_062370.jpg` tại vị trí xe máy ở vùng chuyển tiếp mid-to-edge (`L1`/`R1`), ban đầu tôi có xu hướng vẽ hộp bao nới rộng một chút ra ngoài để bao trọn cả đầu mũ bảo hiểm và vệt mờ do chuyển động (motion blur) của bánh xe. Tuy nhiên, teaching reference yêu cầu hộp bao phải ôm sát chặt chẽ theo cấu trúc cứng nhìn thấy rõ của thân xe và lốp xe theo quy tắc `R_BOX_TIGHT`.
   - **Cách xử lý**: Tôi đã đối chiếu kỹ lưỡng lại hình ảnh gốc trên CVAT, đo đạc sai số IoU qua lệnh `local-quality` và overlay, chấp nhận điều chỉnh lại biên độ box theo đúng chuẩn reference ground truth trong vòng rework (P5) và ghi nhận bài học vào bảng `findings.csv` và `10_error_card.md`.
   - **Thay đổi nếu làm lại slice**: Nếu được thực hiện lại từ đầu, tôi sẽ: (1) Đọc kỹ clinic 6 ca dễ nhầm ngay từ P1 trước khi đặt bút vẽ; (2) Sử dụng công cụ zoom 400% trên CVAT để xác định chính xác điểm tiếp xúc của bánh xe với mặt đường; (3) Tự kiểm tra sơ bộ với bộ lọc threshold IoU trước khi xuất bản draft để giảm thiểu số lần phải tinh chỉnh thủ công.
