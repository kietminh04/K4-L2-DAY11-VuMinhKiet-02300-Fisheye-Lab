# Quan sát vạch ô đỗ

- Hai vạch `parking_line` đã vẽ (mô tả vị trí trong ảnh):
  1. Đã vẽ chi tiết 10 đoạn vạch `parking_line` phân định các ô đỗ riêng lẻ: bao gồm các vạch ô đỗ ở hàng tiền cảnh (vạch chéo trái x ≈ 408–528, y ≈ 652–719; vạch chéo phải x ≈ 701–958, y ≈ 625–685; vạch biên trái x ≈ 28, y ≈ 683–717; vạch biên phải x ≈ 918–960, y ≈ 596–602).
  2. Toàn bộ các vạch phân cách ô đỗ ở hàng thứ hai phía trên (x từ 65 đến 828, y từ 500 đến 570) cũng được vẽ bám sát từng vạch sơn trắng nhìn thấy rõ nét trên mặt bãi đỗ.
- Một vạch/dấu sơn hoặc biên **không** vẽ, và vì sao:
  Vạch sơn biên dài phía sau (khu vực hàng cây xa sát chân tường rào x=50 đến x=300, y ≈ 450–490) và các đường ranh giới khu vực không được vẽ, vì đây là vạch giới hạn chu vi bãi/dẫn hướng luồng xe chạy nội bộ, không đóng vai trò phân định ranh giới một ô đỗ xe (stall) riêng lẻ. Ngoài ra các vệt mờ do phản chiếu ánh sáng ở phía xa cũng được loại bỏ theo nguyên tắc không suy đoán ngoài vùng nhìn thấy rõ.
- Polygon `free_space` dừng ở đâu; có phần bị che nào không:
  Polygon `free_space` bao trọn toàn bộ mặt phẳng lòng đường xe chạy nội bộ (driveway/aisle) kéo dài ngang suốt chiều rộng khung hình (từ x=2 đến x=960, y dao động từ 523 đến 682) nằm giữa hàng ô đỗ tiền cảnh và hàng ô đỗ phía trên. Vùng này hoàn toàn là bề mặt nhựa đường thông thoáng, không có xe cộ đậu lấn, không có người đi bộ hay vật cản; polygon dừng chính xác tại mép đầu các vạch phân ô đỗ và mép khung hình.
- Ca chưa chắc cần hỏi người soát (nếu không có, ghi “không có”):
  không có
