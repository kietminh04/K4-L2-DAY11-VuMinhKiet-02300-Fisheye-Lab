# Hướng Dẫn Thực Hiện Lab 11: SVM 360° Fisheye & Parking Lot Labeling (Từ Đầu Đến Lúc Nộp)

Tài liệu này hướng dẫn chi tiết từng bước quy trình thực hành **Day 11 Lab — Surround View Monitoring (SVM) 360° & Fisheye Camera** (thời lượng 240 phút) trên hệ thống CVAT local và công cụ Python `lab11.py` trên Windows.

---

## 1. Mục Tiêu & Cấu Trúc Tổng Quan Lab 11

### 1.1. Bản chất hệ thống SVM 360° & Thấu kính Fisheye
- **Surround View Monitoring (SVM)**: Hệ thống quan sát 360° toàn cảnh quanh xe tự hành gồm **bốn camera mắt cá** (Front, Rear, Left, Right). 
- **Đặc thù thấu kính Fisheye**: Góc nhìn siêu rộng (FOV ≥ 180°–190°) nhưng gây ra **biến dạng quang học cong méo cực mạnh (barrel distortion)**, đặc biệt ở vành đai mép ảnh (edge zone).
- **Nguyên tắc vàng khi gán nhãn**: Bám sát phần vật thể thực tế nhìn thấy trên ảnh gốc méo; **tuyệt đối không tự ý "nắn thẳng" vật thể theo trực giác thế giới phẳng**, không vẽ xuyên qua vật cản che khuất và không suy đoán vùng mặt đường bị khuất mắt.

### 1.2. Năm chuẩn đầu ra (Core Outcomes) sau bài Lab
1. **Phân biệt ranh giới ô đỗ & lối đi**: Vẽ đúng polyline `parking_line` (vạch sơn chia từng ô đỗ riêng lẻ) và polygon `free_space` (mặt đường lối xe chạy khả dụng trong bãi).
2. **Gán nhãn 6 class trên ảnh Fisheye**: Thành thạo bounding box cho `Car`, `Bus`, `Truck`, `ThreeWheeler`, `Bike`, `Pedestrian` theo luật chiều cao ≥ 40 px, quy tắc gộp `Rider + Bike`, cờ `truncated` / `occluded` và vùng loại trừ `ignore_region`.
3. **Quy trình kiểm soát chất lượng chuẩn công nghiệp**: Tự soát nhãn (`selfqc`), khóa bản export (`lock`), thực hiện QA mù độc lập (`qa`) và truy vết bất đồng về đúng ảnh, đối tượng, mã quy tắc.
4. **Chẩn đoán lỗi ba phía (Triaging & Diagnosis)**: Phân tích báo cáo đối chiếu Người (`L` - Learner) – Tham chiếu (`R` - Teaching Reference) – Mô hình AI (`M` - Model YOLO26m), điền nhật ký phân loại nguyên nhân (`findings.csv`) và đo lường biến thiên sửa đổi (`rework/delta.md`).
5. **Thiết kế phân bổ dữ liệu & Kế hoạch Gold Set**: Phân bổ 200 frame cho 4 camera (`45_sampling_plan.csv`), xây dựng quy trình kiểm chứng `46_gold_set_plan.md` và giải quyết bài toán chồng lấn vùng nhìn (`seam`) / theo dõi đối tượng liên camera (`50_exit_ticket.md`).

---

## 2. Nguồn Dữ Liệu & Danh Mục File Cần Nộp (`submission/`)

### 2.1. Nguồn dữ liệu sử dụng trong bài
| Nguồn dữ liệu | Vị trí thư mục | Nhiệm vụ thực hiện | Giới hạn cần lưu ý |
| :--- | :--- | :--- | :--- |
| **Ảnh bãi đỗ xe** | `assets/parking/` | Gán nhãn trên `parking-lot-core.jpg`; đối chiếu vai trò vạch tiền cảnh vs lối xe chạy với `parking-lot-contrast.png`. | Ảnh camera thường, chưa qua calibration hay chứng nhận an toàn động học. |
| **Ảnh Fisheye ADASIND** | `assets/images/` | Gán nhãn bài hiệu chuẩn `C0` (3 frame) và `Slice được giao` (3 frame). | Dữ liệu trích từ 1 camera mắt cá; mỗi slice đại diện cho một điều kiện môi trường. |
| **Tình huống giả lập 50k frame** | Tài liệu / Slide 11 | Thiết kế bảng phân bổ lấy mẫu 200 frame và kế hoạch xây dựng bộ nhãn vàng (Gold Set). | Tình huống bài toán thiết kế dữ liệu; repo không chứa đủ 50.000 frame thực tế. |

### 2.2. Danh mục các file bắt buộc trong `submission/`
Lệnh `python lab11.py check` sẽ kiểm tra tự động toàn bộ cấu trúc thư mục sau:

| Thư mục / File | Nguồn sinh ra | Bạn cần làm gì |
| :--- | :--- | :--- |
| `submission/00_setup/` | Tool sinh khi chạy `doctor`, `mode` | Điền thông tin camera, vùng thân xe ego và viền đen thấu kính vào `sensor_context.md`. |
| `submission/parking/` | Tool sinh khi nạp export parking | Chứa `annotations.xml`. Điền đầy đủ nhận xét vào `observations.md` (giải thích 2 vạch đã chọn, vạch bị loại, ranh giới free_space). |
| `submission/p1_calib/` | Tool sinh khi khóa và so sánh C0 | Chứa `annotations.xml`, `lock.txt`, `reference.txt`, `compare.md`, `compare.html`. |
| `submission/r1_craft/` | Tool sinh khi làm slice chính | Chứa `annotations.xml` (bản cuối), `lock.txt`, `selfqc.md` (đã soát đủ 9 mục kiểm tra). |
| `submission/r2_qa/` | Tool sinh khi chạy `qa` | Chứa `qa_overlay.html`, `qa_review.md` (đánh giá mù dựa trên rule, ghi nhận xét có thể kiểm chứng). |
| `submission/r3_diag/` | Tool sinh khi chạy chẩn đoán | Chứa `local_quality.md`, `local_quality_conflicts.csv`, `model_compare.md`, `iou_sweep.md`, `zone_table.md`. Điền phần Nhận xét trong `zone_table.md`. |
| `submission/rework/` | Tool sinh khi rework | Chứa `annotations-v2.xml`, `lock2.txt`, `delta.md` (đối chiếu số lượng matched, missing, spurious trước/sau sửa). |
| `submission/findings.csv` | File bảng mẫu do tool quản lý | Ghi nhận ít nhất **12 dòng findings** (đủ 3 vai `r1_craft`, `r2_qa`, `r3_diag`, phủ 4 loại cell, 8 dòng có M, phủ 3 zone `center/mid/edge`). |
| `submission/10_error_card.md` | Sinh từ lệnh `card` | Điền mục **Phân tích của bạn**: Phân tích lỗi theo bảng zone × block, nguyên nhân gốc rễ và đề xuất khắc phục. |
| `submission/20_guideline_patch.md` | Tool tạo mẫu | Đề xuất chỉnh sửa/bổ sung rule gán nhãn mới và đánh số phiên bản quy chuẩn. |
| `submission/30_escalation_ticket.md` | Tool tạo mẫu | Lập vé chuyển tiếp cho ca mơ hồ/ngoại lệ nghiêm trọng, kèm đường dẫn ảnh minh chứng trong `screenshots/`. |
| `submission/40_decision_log.csv` | File bảng mẫu | Ghi nhận ít nhất **4 quyết định** kỹ thuật, trong đó phải có ít nhất **1 ca `status=escalated`**. |
| `submission/45_review_plan.md` | Tool tạo mẫu | Lập kế hoạch review 2 lát cắt khó của ADASIND và phương pháp kiểm tra độ phủ. |
| `submission/45_sampling_plan.csv` | File bảng mẫu | Phân bổ đúng **8 tổ hợp** (Front/Rear/Left/Right × Normal/Hard), frames là số nguyên dương, **tổng chính xác 200 frame**. |
| `submission/46_gold_set_plan.md` | Tool tạo mẫu | Kế hoạch xây dựng Gold Set cho 4 camera: tiêu chí chọn ca, nguy cơ sai sót, quy trình review độc lập, chính sách seam. |
| `submission/50_exit_ticket.md` | Tool tạo mẫu | Trả lời đầy đủ **3 câu hỏi cốt lõi**: Bài toán Seam & 2 box; Identity & Tracking; Phân xử 1 bất đồng thực tế. |
| `submission/screenshots/` | Thư mục ảnh chụp | Lưu ít nhất **2 ảnh chụp màn hình minh chứng** (được dẫn link từ `30_escalation_ticket.md` hoặc nhận xét). |

---

## 3. Quy Trình Thực Hiện Từng Bước (P0 – P6) Trên Windows

> [!NOTE]
> Mọi lệnh dưới đây đều thực hiện tại thư mục gốc của repository bằng **PowerShell** trên Windows 11.  
> Luôn đặt mã hóa chuẩn UTF-8 `$env:PYTHONIOENCODING="utf-8"` để tránh lỗi hiển thị tiếng Việt trên terminal.

```powershell
$env:PYTHONIOENCODING="utf-8"
```

---

### Bước 1: Khởi động hệ thống CVAT & Nhận diện Slice (P0)

1. **Bật CVAT Local**:
   - Mở Docker Desktop trên máy.
   - Tại thư mục chứa cấu hình CVAT (ví dụ `D:\Code\Courses\cvat`), chạy lệnh:
     ```powershell
     docker compose start
     ```
   - Truy cập trình duyệt tại `http://localhost:8080/` và đăng nhập tài khoản admin (`admin` / mật khẩu của bạn).

2. **Kiểm tra môi trường**:
   - Tại thư mục repo Lab 11, chạy:
     ```powershell
     python lab11.py doctor
     ```
   - Đảm bảo terminal báo `✓` ở các mục kiểm tra kết nối CVAT và Python (kết quả lưu tại `submission/00_setup/doctor.txt`).

3. **Chọn chế độ làm bài & Nhận mã Slice**:
   - Nếu làm **Cá nhân** (Solo):
     ```powershell
     python lab11.py mode --members kiet.vm227128
     python lab11.py status
     ```
   - Nếu làm **Nhóm (2–3 người)**:
     ```powershell
     python lab11.py mode --members an,binh,kiet --self kiet
     python lab11.py status
     ```
   - Ghi lại mã **Slice của bạn** (ví dụ: `B1-edge`, `C2-mid`, v.v.). Mã này sẽ thay thế cho `SLICE_CUA_BAN` trong các lệnh sau.

4. **Điền thông tin cảm biến**:
   - Mở file `submission/00_setup/sensor_context.md`, ghi nhận xét về vị trí lắp đặt camera theo quan sát thực tế trên ảnh, các vùng cản trước/sau xe chủ (ego bumper) và vòng tròn viền đen quang học của ống kính mắt cá.

---

### Bước 2: P0 — Gán nhãn Vạch Ô Đỗ & Lòng Đường Bãi Đỗ (Phút 0–40)

> [!IMPORTANT]
> **Quy chuẩn phân định**:
> - `parking_line` (Polyline): Chỉ vẽ cho phần vạch sơn nhìn thấy phân định ranh giới từng ô đỗ riêng lẻ. Dừng đường tại chỗ sơn bị đứt hoặc bị bánh xe che khuất.
> - `free_space` (Polygon): Vùng mặt đường trống nhìn thấy của lối xe chạy chính trong bãi đỗ. Đa giác không được đâm xuyên qua xe đang đỗ, gờ bó vỉa hè (`curb`), thân cây hoặc vùng khuất tầm mắt.

1. **Chuẩn bị dữ liệu**:
   ```powershell
   python lab11.py parking
   ```
   - Mở xem 2 ảnh: `assets/parking/parking-lot-core.jpg` (ảnh chính đưa vào CVAT) và `assets/parking/parking-lot-contrast.png` (ảnh đối chứng phân biệt vạch ô đỗ vs lối xe chạy).

2. **Thao tác trên CVAT Local**:
   - Tạo Task mới trên CVAT:
     - Name: `Day11 · parking_line · public-sample`
     - Labels: Chuyển sang tab **Raw**, dán toàn bộ nội dung file `assets/parking/labels.json` vào rồi bấm **Save**.
     - Files: Chọn đúng file `assets/parking/parking-lot-core.jpg`.
     - Bấm **Submit & Open** → Mở Job.
   - **Vẽ nhãn**:
     - Chọn `Draw new polyline` → Chọn nhãn `parking_line` → Vẽ ít nhất **2 vạch chia ô đỗ khác nhau**. Nhấn phím `N` để kết thúc mỗi đường.
     - Chọn `Draw new polygon` → Chọn nhãn `free_space` → Vẽ ít nhất **1 đa giác vùng lòng đường di chuyển**. Viền khít ranh giới, không đâm xuyên xe hay vỉa hè.
   - Nhấn `Ctrl + S` để lưu.

3. **Export & Nạp vào hồ sơ**:
   - Trên thanh menu CVAT: Chọn **Menu → Export job dataset → CVAT for images 1.1** (tắt cờ *Save images*), tải file ZIP về máy.
   - Nạp file ZIP vào bài làm:
     ```powershell
     python lab11.py parking --file "C:\Users\...\Downloads\parking-export.zip"
     ```
   - Lệnh sẽ lưu nhãn vào `submission/parking/annotations.xml`.

4. **Điền tài liệu & Phác thảo kế hoạch**:
   - Mở `submission/parking/observations.md`: Ghi rõ vị trí 2 vạch ô đỗ đã chọn, 1 vạch sơn bị bạn loại bỏ và lý do, ranh giới vùng free_space dừng ở đâu, các trường hợp nghi ngờ.
   - Mở `submission/45_sampling_plan.csv`: Phác thảo trước 8 dòng phân bổ cho 4 camera × 2 điều kiện normal/hard (tổng 200 frame).
   - Ghi nhận những ý tưởng đầu tiên vào `submission/46_gold_set_plan.md`.

---

### Bước 3: P1 — Hiệu Chuẩn Cách Gán Nhãn Trên C0 (Phút 40–70)

Tập `C0` gồm 3 frame mắt cá giúp thống nhất cách áp dụng bộ quy chuẩn R01–R11 trước khi gán nhãn slice chính thức.

1. **Lấy thông tin và tạo task C0**:
   ```powershell
   python lab11.py cvat C0
   ```
   - Trên CVAT: Tạo task mới theo tên lệnh in ra (bắt buộc chứa chuỗi `raw_fisheye`).
   - Labels: Tab **Raw**, dán toàn bộ nội dung `assets/labels.json` vào rồi Save (kiểm tra Constructor có đủ 6 class: `Car`, `Bus`, `Truck`, `ThreeWheeler`, `Bike`, `Pedestrian` và nhãn `ignore_region`).
   - Files: Chọn đúng các ảnh lệnh liệt kê.
   - Tại trang chi tiết Task: Chọn **Actions → Upload annotations → CVAT 1.1**, nạp file `assets/prefill/C0.xml`, sau đó mở Job.

2. **Rà soát và chỉnh sửa theo 11 Quy tắc Vàng (R01–R11)**:
   - **Phạm vi kích thước (R01)**: Chỉ vẽ Bounding Box cho đối tượng thuộc 6 class có chiều cao nhìn thấy $\ge 40\text{ px}$. Bỏ qua vật thể nhỏ hơn $40\text{ px}$ ở xa.
   - **Hình học Fisheye (R02)**: Box chữ nhật 2D phải bám sát mép viền thực tế của đối tượng trên ảnh cong méo; **không nắn thẳng hoặc suy đoán phần bị che khuất**.
   - **Rider & Bike (R04)**: Người ngồi lái gắn liền trên xe hai bánh được gộp chung thành một box nhãn `Bike`. Người dắt xe hoặc đứng cạnh xe thì vẽ 1 box `Pedestrian` riêng và 1 box `Bike` riêng. Người ngồi trong cabin ô tô/xe buýt **không vẽ box riêng**.
   - **Thuộc tính cắt biên & che khuất (R05)**:
     - `truncated = true`: Đối tượng bị đường tròn quang học hoặc mép viền ảnh cắt cụt một phần.
     - `occluded = true`: Đối tượng bị xe khác, người hoặc cột biển báo che khuất tầm nhìn.
     - Hai cờ này độc lập với nhau.
   - **Vùng loại trừ (Ignore Region - R06/R07)**:
     - Nhãn polygon `ignore_region` đi kèm thuộc tính `reason`: `ego_body` (phủ kín nắp ca-pô, cản trước/sau xe chủ), `lens_border` (phủ 4 góc đen ngoài tầm nhìn thấu kính), `crowd_or_group`, `unreadable`, `privacy_or_policy`.
     - **Quy tắc 50%**: Tuyệt đối không để một box đối tượng nằm $\ge 50\%$ diện tích bên trong polygon `ignore_region`.
     - Đối với frame `adasind_006840.jpg` và `adasind_271039.jpg`: Nếu không nhìn thấy thân xe chủ thì **không vẽ thêm polygon `ego_body`**.

3. **Export, Khóa & So Sánh Reference**:
   - Lưu task (`Ctrl + S`), export định dạng **CVAT for images 1.1** (tắt *Save images*).
   - Chạy tuần tự 3 lệnh:
     ```powershell
     python lab11.py lock calib "C:\Users\...\Downloads\c0-export.zip"
     python lab11.py reference calib
     python lab11.py compare calib
     ```
   - Mở `submission/p1_calib/compare.html` và `compare.md` trên trình duyệt để đối chiếu trực quan khác biệt giữa nhãn của bạn và Teaching Reference.

4. **Ghi nhận Findings**:
   - Mở file `submission/findings.csv`, ghi ít nhất **3 finding đầu tiên** phát hiện được từ so sánh C0 (chỉ rõ frame, đối tượng, mã rule liên quan).

---

### Bước 4: P2 — Gán Nhãn Slice Chính, Đối Chứng K12 & Tự Soát Self-QC (Phút 70–125)

1. **Khởi tạo Task cho Slice được giao**:
   ```powershell
   python lab11.py cvat SLICE_CUA_BAN
   ```
   *(Thay `SLICE_CUA_BAN` bằng mã thật, ví dụ `B1-edge`).*
   - Tạo task trên CVAT theo đúng tên lệnh yêu cầu, nạp nhãn `assets/labels.json`, chọn đúng 3 ảnh của slice và upload prefill `assets/prefill/<slice>.xml`.

2. **Gán nhãn & Làm bài tập đối chứng K12**:
   - Mở từng frame, soát toàn bộ box có sẵn từ prefill, vẽ bổ sung đối tượng còn thiếu, xóa đối tượng ngoài phạm vi (< 40 px), chỉnh sửa box bị lỏng/chệch mép viền.
   - Soát các polygon `lens_border` và vẽ thêm `ego_body` nếu frame có lộ cản/nắp ca-pô xe chủ.
   - **Đối chứng K12 cho 4 đối tượng** (do lệnh `cvat` gợi ý):
     - Vẽ thêm 1 polygon viền sát theo thân nhìn thấy của đối tượng, cùng class với box.
     - Giữ phím `Ctrl`, click chuột chọn cả Box và Polygon đó, nhấn phím `G` để gán chúng cùng một `group_id`. (Công cụ `lab11.py fill` sẽ tính tỷ lệ diện tích polygon / box để đánh giá mức độ phình viền do méo fisheye).

3. **Xuất bản nháp (Draft) & Tự kiểm tra Self-QC**:
   - Lưu trên CVAT, export file nháp dạng ZIP.
   - Chạy các lệnh kiểm tra:
     ```powershell
     python lab11.py draft "C:\Users\...\Downloads\r1-draft.zip"
     python lab11.py fill
     python lab11.py selfqc r1_craft
     ```
   - Mở file `submission/r1_craft/selfqc.md`, đọc các cảnh báo tự động và tự rà soát trung thực **đủ 9 tiêu chí kiểm tra**:
     1. Phạm vi kích thước ($\ge 40\text{ px}$).
     2. Đầy đủ `lens_border` và `ego_body`.
     3. Đúng lớp đối tượng (phân biệt rõ Van/Minibus vs Car/Bus).
     4. Gộp đúng Rider + Bike.
     5. Độ khít hình học box trên ảnh méo.
     6. Thuộc tính `truncated` và `occluded`.
     7. Không bỏ sót, không vẽ trùng lặp.
     8. Lý do `ignore_region` hợp lệ, không nuốt $> 50\%$ box.
     9. Đúng tên task và định dạng.
   - Sau khi kiểm tra kỹ trên ảnh, đổi các ô `- [ ]` thành `- [x]`.

4. **Sửa lỗi trên CVAT & Khóa bản cuối (Lock R1)**:
   - Quay lại CVAT chỉnh sửa triệt để các lỗi phát hiện qua Self-QC.
   - Lưu và export lại ra file ZIP mới (đây là bản nộp chính thức).
   - Khóa bản cuối:
     ```powershell
     python lab11.py lock r1_craft "C:\Users\...\Downloads\r1-final.zip"
     ```
   - **Ghi lại Mã Khóa dạng `XXXX-XXXX`** mà lệnh trả về trên màn hình.  
   *(Tuyệt đối không sửa file `submission/r1_craft/annotations.xml` bằng tay sau khi đã khóa).*
   - **Nghỉ giải lao 15 phút** (phút 125–140) để mắt nghỉ ngơi trước khi bước vào pha Cold Review / QA mù.

---

### Bước 5: P3 — QA Mù Bằng Ảnh & Guideline (Phút 140–165)

QA mù là quá trình đánh giá chất lượng mà người kiểm tra **chưa hề được xem Teaching Reference hay kết quả của Model AI**.
- Nếu làm **Cá nhân**: Bạn thực hiện **Cold Review** (tự soát lại bài của chính mình sau thời gian nghỉ giải lao).
- Nếu làm **Nhóm**: Bạn nhận file XML/ZIP đã khóa và mã khóa từ bạn cùng nhóm theo phân công trong `submission/00_setup/team.json`.

1. **Khởi tạo môi trường QA**:
   ```powershell
   python lab11.py qa --slice SLICE_CUA_BAN --file "submission/r1_craft/annotations.xml" --code MA_KHOA_CUA_BAN
   ```
   *(Nếu review bài bạn khác: truyền đường dẫn file và mã khóa của bạn đó).*

2. **Thực hiện đánh giá**:
   - Mở file `submission/r2_qa/qa_overlay.html` trên trình duyệt.
   - Soi từng frame, đối chiếu từng box với quy chuẩn R01–R11.
   - Mở `submission/r2_qa/qa_review.md`: Điền rõ frame, mã đối tượng (`object_ref`), mã quy tắc vi phạm (`rule_id`) và nhận xét khách quan có thể kiểm chứng được. Xóa hết các dòng `TODO`.
   - Mở `submission/findings.csv`: Thêm ít nhất **3 dòng finding cho pha QA** với các trường:
     - `round`: `r2_qa`
     - `cell`: `L_only`
     - `rule_id`: Điền mã rule vi phạm (ví dụ `R01`, `R04`, `R05`)
     - `why`: **BẮT BUỘC ĐỂ TRỐNG** (vì ở pha QA mù, ta chỉ ghi nhận hiện tượng quan sát, chưa kết luận nguyên nhân).

---

### Bước 6: P4 — Chẩn Đoán Lỗi Ba Phía: Nhãn – Reference – Model (Phút 165–200)

Ở pha này, ta mở toàn bộ dữ liệu đối chiếu 3 nguồn:
- **`L` (Learner)**: Nhãn do bạn vẽ.
- **`R` (Teaching Reference)**: Nhãn tham chiếu chuẩn của khóa học.
- **`M` (Model Output)**: Dự đoán từ mô hình YOLO26m chạy inference trên ảnh fisheye.

1. **Chạy bộ công cụ phân tích tự động**:
   ```powershell
   python lab11.py reference r1_craft
   python lab11.py compare r1_craft
   python lab11.py local-quality
   python lab11.py model
   python lab11.py iou-sweep --iou 0.3,0.5,0.7
   ```
   *(Lệnh này cũng tự động tạo các khung file mẫu `10_error_card.md`, `20_guideline_patch.md`, `30_escalation_ticket.md`, `45_review_plan.md`, `50_exit_ticket.md`).*

2. **Đọc hiểu các báo cáo phân tích**:
   - `submission/r1_craft/compare.html`: Nhìn trực quan các box lệch nhau giữa bạn và Reference.
   - `submission/r3_diag/local_quality.md`: Đọc chỉ số $\text{IoU} \ge 0.5$, $\text{Precision} = \frac{\text{TP}}{\text{TP}+\text{FP}}$, $\text{Recall} = \frac{\text{TP}}{\text{TP}+\text{FN}}$, danh sách class yếu.
   - `submission/r3_diag/local_quality_conflicts.csv`: Danh sách các ca xung đột hình học và nhãn lớp.
   - `submission/r3_diag/model_compare.html`: Đối chiếu xem vật thể nào cả 3 bên cùng thấy (`LRM`), bên nào chỉ có người thấy (`LR_noM`), hay Model bị ảo giác (`M_only`).
   - `submission/r3_diag/zone_table.md`: Phân bố lỗi theo 3 vùng khoảng cách từ tâm thấu kính (`center`, `mid`, `edge`).

3. **Hoàn thiện các dòng chẩn đoán trong `submission/findings.csv`**:
   Thêm các dòng `round=r3_diag` để giải quyết toàn bộ các bất đồng, điền đầy đủ các cột:
   - **`cell`**: 
     - `LRM`: Cả 3 nguồn (Người, Reference, Model) đều phát hiện.
     - `LR_noM`: Người và Reference thấy, Model bỏ sót.
     - `LM_noR`: Người và Model thấy, Reference bỏ sót.
     - `RM_noL`: Reference và Model thấy, Người bỏ sót.
     - `L_only`, `R_only`, `M_only`: Chỉ duy nhất nguồn đó phát hiện.
   - **`what` (Hiện tượng lỗi)**: `MISSING` (bỏ sót), `SPURIOUS` (vẽ thừa), `WRONG_CLASS` (sai lớp), `BOX_GEOMETRY` (hình học lỏng/chệch), `DUPLICATE` (trùng lặp), `ATTRIBUTE` (sai thuộc tính cờ), `IGNORE_SCOPE` (vi phạm vùng ignore).
   - **`why` (Giả thuyết phân xử)**:
     - `E0_reference_defect`: Reference bị lỗi/thiếu sót (có bằng chứng rõ ràng).
     - `E1_annotator_error`: Lỗi do người học vẽ sai hoặc không nắm vững rule.
     - `E2_guideline_gap`: Quy chuẩn hướng dẫn chưa rõ ràng, gây mơ hồ.
     - `E3_data_defect`: Ảnh bị mờ nhòe, chói sáng, quá tối hoặc vỡ hạt nặng.
     - `E4_model_domain`: Model AI chưa thích nghi tốt với miền dữ liệu fisheye góc rộng.
     - `E5_unresolved`: Chưa đủ bằng chứng để phân xử, cần thử nghiệm thêm.
   - **`severity`**: `P0` (sai phạm vi/lỗi ignore nghiêm trọng), `P1` (thiếu/thừa/sai class/lệch box), `P2` (sai thuộc tính cờ), `P3` (ghi chép/trình bày).
   - **`owner`**: `annotator`, `guideline`, `data_ops`, `ai_team`, `qa`.
   - **`action`**: `rework` (sửa lại nhãn), `keep_with_reason` (giữ nguyên nhãn kèm lập luận), `escalate` (gửi vé chuyển tiếp cho Lead).

4. **Viết nhận xét Zone**:
   - Mở `submission/r3_diag/zone_table.md`: Viết mục **Nhận xét** phân tích xem vùng nào (`edge` hay `center`) có nhiều lỗi nhất, nguyên nhân do biến dạng quang học fisheye hay do kích thước vật thể.

5. **Kiểm tra hợp lệ bảng Findings**:
   ```powershell
   python lab11.py triage
   ```
   - Đảm bảo lệnh báo `✓ Findings hợp lệ` (không còn ô trống bất hợp lệ hay sai cấu trúc CSV).

---

### Bước 7: P5 — Sửa Nhãn Rework & Đo Lường Biến Thiên Delta (Phút 200–215)

1. **Rà soát danh sách cần sửa**:
   - Lọc trong `submission/findings.csv` các dòng có `action=rework` và `severity=P0` hoặc `P1`.
2. **Sửa trên CVAT**:
   - Mở lại Job của slice trên CVAT, chỉnh sửa chính xác các box/polygon cần sửa dựa trên phân tích ở P4.
3. **Export & Khóa bản Rework**:
   - Lưu trên CVAT, export file ZIP mới.
   - Chạy lệnh khóa và tính toán delta:
     ```powershell
     python lab11.py lock rework "C:\Users\...\Downloads\rework-export.zip"
     python lab11.py rework
     ```
4. **Đối chiếu kết quả**:
   - Mở file `submission/rework/delta.md`: Xem bảng biến thiên số lượng `matched` (tăng lên), `missing` và `spurious` (giảm xuống) giữa bản vẽ ban đầu (`r1_craft`) và bản sau sửa (`rework`).

---

### Bước 8: P6 — Bàn Giao Hồ Sơ, Error Card & Kế Hoạch 4 Camera (Phút 215–240)

1. **Tạo Error Card**:
   ```powershell
   python lab11.py card
   ```
   - Mở `submission/10_error_card.md`: Viết phần **Phân tích của bạn** dựa trên bảng thống kê zone × block tự sinh.

2. **Hoàn thiện các tài liệu bàn giao**:
   - `submission/20_guideline_patch.md`: Đề xuất 1 quy tắc mới hoặc làm rõ rule cũ (ví dụ: bổ sung quy định gán nhãn xe kéo đẩy tay, xe ba gác tự chế, hoặc tiêu chuẩn crop viền biển hiệu).
   - `submission/30_escalation_ticket.md`: Lập vé chuyển tiếp cho 1 trường hợp cực khó trong slice của bạn; chỉ rõ frame, tác động dự kiến (`expected impact`), đề xuất xử lý và **dẫn link tới ảnh chụp màn hình trong `submission/screenshots/`**.
   - `submission/40_decision_log.csv`: Ghi nhận ít nhất **4 quyết định**, trong đó bắt buộc có ít nhất **1 ca có `status=escalated`** (phù hợp với finding có `action=escalate`).
   - `submission/45_review_plan.md`: Lập kế hoạch review cho 2 lát cắt dữ liệu khó của ADASIND, giải thích cách kiểm tra độ phủ.

3. **Hoàn thiện Kế hoạch Sampling & Gold Set 4 Camera**:
   - `submission/45_sampling_plan.csv`:
     - Điền đầy đủ **đúng 8 dòng**:
       - `front,normal`
       - `front,hard`
       - `rear,normal`
       - `rear,hard`
       - `left,normal`
       - `left,hard`
       - `right,normal`
       - `right,hard`
     - Cột `frames` phải là các số nguyên dương và **tổng của cả 8 dòng phải bằng đúng 200 frame**.
     - Mỗi dòng điền rõ nguy cơ (`risk`) và lý do phân bổ (`rationale`).
   - `submission/46_gold_set_plan.md`:
     - Viết chiến lược xây dựng Gold Set cho từng camera (Front, Rear, Left, Right).
     - Nêu rõ các ca dễ sai (chói đèn xe, ban đêm, bóng cây, vật thể sát cản xe chủ).
     - Quy trình review độc lập 2 vòng và cơ chế phân xử bất đồng.
     - **Chính sách Seam (vùng nhìn chồng giữa 2 camera)**: Khi đối tượng nằm ở ranh giới giao nhau giữa camera Trước và camera Sườn, quy định rõ camera nào chịu trách nhiệm chính để tránh duplicate box.

4. **Điền Exit Ticket & Chụp ảnh minh chứng**:
   - Mở `submission/50_exit_ticket.md`: Trả lời đầy đủ **3 câu hỏi lớn**:
     1. Xử lý vùng chồng lấn (Seam) và hiện tượng nhân đôi box giữa 2 camera lân cận.
     2. Điều kiện duy trì ID tracking, đánh dấu keyframe và bật thuộc tính `Outside` khi xe rẽ thoát khỏi khung nhìn.
     3. Phân tích 1 bất đồng thực tế sâu sắc nhất trong bài lab và cách bạn đã giải quyết.
   - **Chụp màn hình**: Chụp ít nhất **2 ảnh màn hình** minh chứng (ví dụ: ca xung đột trên CVAT, bảng so sánh HTML, hoặc vết cắt occlusion) và lưu vào thư mục `submission/screenshots/` (định dạng `.png` hoặc `.jpg`).

---

## 4. Kiểm Thử Hồ Sơ Tự Động Trước Khi Nộp (`lab11.py check`)

Trước khi commit và nộp bài, chạy chuỗi 3 lệnh kiểm toán cổng chất lượng:

```powershell
python lab11.py triage
python lab11.py status
python lab11.py check
```

### Bảng tiêu chuẩn kiểm duyệt tự động của `python lab11.py check`:
Khi chạy lệnh `check`, hệ thống sẽ quét toàn bộ hồ sơ và chỉ trả về **Exit Code 0 (`✓ Hồ sơ hình thức đầy đủ`)** khi thỏa mãn 100% các điều kiện sau:

- [x] Có đầy đủ `submission/parking/annotations.xml` và file `observations.md` đã điền hết nhận xét.
- [x] Đã hoàn thành và khóa pha `p1_calib` (có đủ xml, lock, reference, compare).
- [x] Bản `r1_craft` đã khóa bản export cuối cùng; file `selfqc.md` đã tích đủ 9 mục `- [x]`.
- [x] Pha `r2_qa` có `qa_review.md` (không còn chữ `TODO`).
- [x] Bảng `findings.csv` có **ít nhất 12 dòng**, gồm:
  - Ít nhất **3 dòng** cho mỗi vai (`r1_craft`, `r2_qa`, `r3_diag`).
  - Xuất hiện ít nhất **4 loại cell** khác nhau (ngoài `na`).
  - Có ít nhất **8 dòng chứa mô hình `M`** trong cell (`LRM`, `LM_noR`, `RM_noL`, `M_only`).
  - Bằng chứng phủ đủ **3 vùng không gian thấu kính**: `center`, `mid`, `edge`.
  - Các dòng `r2_qa` có `rule_id`, cột `why` để trống.
  - Các dòng `r3_diag` có đầy đủ `why`, `severity`, `owner`, `action`.
- [x] File `submission/40_decision_log.csv` có **ít nhất 4 quyết định**, trong đó có **ít nhất 1 ca `status=escalated`**.
- [x] Thư mục `submission/screenshots/` có **ít nhất 2 ảnh chụp**.
- [x] File `submission/rework/delta.md` ghi nhận số liệu trước/sau sửa đổi.
- [x] File `submission/45_sampling_plan.csv` có **đúng 8 tổ hợp**, số nguyên dương **cộng đúng 200 frame**.
- [x] Các file `10_error_card.md`, `20_guideline_patch.md`, `30_escalation_ticket.md`, `45_review_plan.md`, `46_gold_set_plan.md`, `50_exit_ticket.md` đã có nội dung phân tích hoàn chỉnh, **tuyệt đối không còn sót bất kỳ chữ `TODO` nào**.

---

## 5. Quy Chuẩn Đặt Tên & Hướng Dẫn Nộp Bài

### 5.1. Nộp bài Cá nhân (Individual Submission)
1. **Kiểm tra trạng thái Git**:
   ```powershell
   git status
   ```
2. **Commit và Push lên GitHub cá nhân**:
   ```powershell
   git add .
   git commit -m "Hoàn thành toàn bộ hồ sơ Lab 11 SVM Fisheye & Parking"
   git push origin main
   ```
3. Đảm bảo repo cá nhân của bạn có tên dạng:  
   `K4-DAY11-VuMinhKiet-20227128` (hoặc `2A202602300`) và đang ở trạng thái **Public**.
4. Gửi đường link repo GitHub của bạn lên hệ thống nộp bài của lớp (Canvas / Google Classroom / VLearn).

---

### 5.2. Nộp bài theo Nhóm (Team Submission)
Nếu lớp quy định nộp bài theo nhóm 2–3 người:
1. Mỗi thành viên vẫn thực hiện toàn bộ các bước P0–P6 trong **repo thực hành cá nhân riêng** để công cụ `lab11.py check` xác nhận thành công.
2. Trưởng nhóm tạo một repository tổng hợp riêng đặt tên theo chuẩn:  
   `K4-DAY11-TenNhom` (ở chế độ **Public**).
3. Cấu trúc repo nhóm chỉ cần 2 file:
   ```text
   K4-DAY11-TenNhom/
   ├── README.md
   └── TEAMMATES.md
   ```
4. **Điền `TEAMMATES.md`**: Sao chép mẫu từ repo bài tập sang repo nhóm, điền đầy đủ bảng thông tin:
   - Họ và tên, MSSV, Tên định danh trong lệnh `mode`.
   - Mã Slice được phân công của từng người.
   - Vòng phân công QA (Ai review bài của ai).
   - Phần việc, commit chốt bài và **Link dẫn thẳng tới commit trong repo cá nhân của từng thành viên**.
5. **Điền `README.md` của nhóm**:
   - Tên nhóm, danh sách thành viên, đại diện nộp bài.
   - Bảng tổng hợp liên kết trực tiếp tới file `submission/manifest.json`, bản nhãn đã khóa, file QA review, file delta và exit ticket của từng người.
   - Tóm tắt ngắn cách thức nhóm thảo luận, thống nhất bảng phân bổ 200 frame và xử lý các bất đồng kỹ thuật.
6. Trưởng nhóm nộp link repo tổng hợp `K4-DAY11-TenNhom` lên cổng nộp bài của lớp.

---

## 6. Bảng Tra Cứu Sự Cố Nhanh (Troubleshooting Quick Sheet)

| Hiện tượng lỗi | Nguyên nhân gốc rễ | Cách khắc phục dứt điểm |
| :--- | :--- | :--- |
| `python lab11.py doctor` báo kết nối CVAT `✗` | Container CVAT chưa được khởi động trong Docker. | Mở Docker Desktop, vào thư mục CVAT chạy `docker compose start`. Sau đó chạy lại lệnh `doctor`. |
| Lệnh `lock` báo sai số lượng frame hoặc thiếu ảnh | Bạn chọn nhầm ảnh trên CVAT hoặc export từ sai Job. | Mở lại CVAT, kiểm tra kỹ danh sách ảnh của task theo đúng lệnh `lab11.py cvat <slice>`. Export lại đúng định dạng CVAT 1.1 (tắt Save images). |
| `lab11.py reference` báo chưa khóa (Not locked) | File export chưa được ghi nhận qua lệnh `lock`. | Chạy lệnh `python lab11.py lock ...` trước, đảm bảo có mã khóa in ra, rồi mới chạy lệnh `reference`. |
| `lab11.py triage` báo lỗi trường trong `findings.csv` | Thừa/thiếu cột, sai chính tả giá trị enum (ví dụ gõ sai `P1` thành `p1`, `r1_craft` thành `r1`), hoặc nội dung chứa dấu phẩy chưa bọc ngoặc kép `"..."`. | Mở `submission/findings.csv` bằng trình soạn thảo văn bản (VS Code), kiểm tra đúng header, bọc dấu ngoặc kép các chuỗi có dấu phẩy, chuẩn hóa chữ hoa/thường theo đúng danh mục ở Mục 3 (Bước 6). |
| `lab11.py check` báo thiếu dòng có mô hình `M` | Cột `cell` trong `findings.csv` chưa đủ 8 dòng thuộc nhóm: `LRM`, `LM_noR`, `RM_noL`, `M_only`. | Rà soát lại báo cáo `model_compare.html`, bổ sung các dòng phân tích đối chiếu có liên quan đến Model AI vào `findings.csv`. |
| `lab11.py check` báo tổng sampling khác 200 | Cột `frames` trong `submission/45_sampling_plan.csv` cộng lại không đúng 200. | Kiểm tra lại tổng 8 dòng của 4 camera, điều chỉnh số nguyên dương ở các ô sao cho tổng cộng chính xác bằng 200. |
| Terminal in chữ tiếng Việt bị lỗi font loằng ngoằng | Bảng mã console Windows mặc định dùng CP1258/CP437. | Chạy lệnh `$env:PYTHONIOENCODING="utf-8"` hoặc `chcp 65001` trước khi thực thi script Python. |
