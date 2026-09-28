# Sensor context

- Rig: Camera mắt cá góc siêu rộng (fisheye lens, trường nhìn FOV ~180°–190°) được gắn tại vị trí phía trước xe thử nghiệm ADASIND (khu vực lưới tản nhiệt/cản trước hoặc kính chắn gió trên giá thử nghiệm ADAS), hướng thẳng về phía trước để giám sát toàn cảnh giao thông đô thị cận cảnh và tầm nhìn rộng.
- `ego_body`: Ở mép dưới cùng của khung hình (khu vực đáy ảnh sát viền), có thể quan sát thấy phần rìa nắp ca-pô/thanh cản trước hoặc gá gắn camera của xe ego; không thấy tay lái hay khoang lái bên trong vì cảm biến được lắp đặt phía ngoài ngoại thất xe.
- Vòng kính (lens circle): Vòng tròn quang học của thấu kính mắt cá nằm tập trung ở trung tâm bức ảnh, chiếm khoảng 80%–85% diện tích khung hình (kích thước frame 1280×720/1280×960). Bốn góc ảnh xuất hiện viền đen tối góc (optical vignetting / mask đen) do giới hạn vật lý của cảm biến khi chụp vòng tròn thấu kính góc siêu rộng.
