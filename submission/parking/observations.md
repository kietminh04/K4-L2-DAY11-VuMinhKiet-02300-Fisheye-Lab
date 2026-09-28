# Quan sát vạch ô đỗ

- Hai vạch `parking_line` đã vẽ (mô tả vị trí trong ảnh):
  1. Vạch trung tâm tiền cảnh: polyline từ (408.5, 652.0) đến (528.0, 719.0), là vạch sơn trắng phân định mép trái của ô đỗ xe ngay hàng đầu tiên gần góc nhìn camera.
  2. Vạch bên phải tiền cảnh: polyline từ (750.5, 635.0) đến (958.0, 685.0), là vạch sơn trắng phân định mép phải của ô đỗ xe liền kề ở hàng đầu tiên.
- Một vạch/dấu sơn hoặc biên **không** vẽ, và vì sao:
  Vạch sơn biên dài phía sau (khu vực x=50 đến x=300, y ≈ 500–520) gần hàng cây và hàng rào xa không được vẽ, vì đây là vạch dẫn hướng lối đi/ranh giới phân khu giao thông trong bãi, không phải vạch sơn chia một ô đỗ xe (stall) riêng lẻ. Ngoài ra các vạch đỗ ở các hàng quá xa bị mờ và suy giảm độ tương phản cũng được bỏ qua theo đúng nguyên tắc không suy diễn khi không thấy rõ ranh giới ô.
- Polygon `free_space` dừng ở đâu; có phần bị che nào không:
  Polygon `free_space` bao trọn lòng đường xe chạy nội bộ (aisle/driveway) giữa hàng ô đỗ tiền cảnh (y ≈ 630) và hàng ô đỗ phía trên (y ≈ 580), kéo dài ngang khung hình từ x=80 đến x=920. Vùng này hoàn toàn là mặt nhựa đường trống, không bị xe cộ hay vật thể nào che khuất; biên polygon dừng chính xác tại ranh giới trước khi chạm vào các vạch ô đỗ.
- Ca chưa chắc cần hỏi người soát (nếu không có, ghi “không có”):
  không có
