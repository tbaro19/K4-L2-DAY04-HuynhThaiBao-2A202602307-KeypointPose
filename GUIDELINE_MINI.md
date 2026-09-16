# Mini guideline  |  người gán: Huỳnh Thái Bảo (2A202602307)  |  ngày: 16/09/2026

> Điền file này **trong lúc** gán nhãn, không phải sau khi xong. Mỗi lần bạn dừng lại
> hơn 10 giây để phân vân, đó là một dòng phải ghi vào đây.

## 1. Luật bắt buộc (đã thống nhất cả lớp - không sửa)

- Bộ 17 điểm COCO, đúng tên, đúng thứ tự. Lấy từ file `.SVG` chung.
- Mọi người trong ảnh đều có **đủ 17 điểm**. Điểm không dùng được thì gắn cờ, không xoá.
- Trái/phải tính theo **cơ thể người**, không theo bức ảnh.
- Bị che, còn trong khung -> `v = 1`, **vẫn đặt chấm** ở vị trí ước lượng.
- Ra ngoài mép ảnh -> `v = 0`, **không** đặt chấm.
- Không dùng `Hidden` (`h`) - nó không được lưu vào file.

## 2. Luật của nhóm bạn (phải điền)

| Tình huống | Luật nhóm bạn chọn | Vì sao |
| --- | --- | --- |
| Hông của người mặc quần áo dài | Chấm tại vị trí ước lượng của mào chậu / khớp háng giải phẫu (ngang đáy thắt lưng, đối xứng cột sống). Gán `v = 1` nếu áo/váy trùm kín không thấy nếp hông; gán `v = 2` nếu quần ôm sát lộ rõ đường cong cơ thể. | Trang phục rộng che mất bề mặt xương chậu thật, nhưng bộ xương người luôn có tỷ lệ cố định nối từ vai qua cột sống xuống xương chậu. Đặt `v = 1` bảo toàn cấu trúc skeleton cho mô hình học tư thế đứng/ngồi. |
| Tai bị tóc hoặc mũ bảo hiểm che một phần | Nếu nhìn thấy vành tai hoặc phần dái tai -> gán `v = 2`. Nếu tóc phủ kín hoặc đội mũ bảo hiểm/mũ len trùm tai nhưng đầu vẫn trong khung -> gán `v = 1`, chấm ước lượng đối xứng qua sống mũi/mắt ngang tầm đuôi mắt. | Tai gắn chặt với cấu trúc hộp sọ cứng cố định. Dù bị tóc che thì vị trí tương đối so với mắt và mũi không thay đổi, giúp mô hình pose học được góc xoay của đầu (head yaw/pitch). |
| Người bị cắt ở mép ảnh (chỉ thấy từ hông trở lên) | Các khớp từ đầu đến thắt lưng gán bình thường (`v = 2` hoặc `v = 1`). Mọi khớp từ đầu gối, cổ chân bị mép viền ảnh cắt mất ra ngoài khung hình bắt buộc gán `v = 0` (tọa độ `0 0 0`, không đặt chấm). | Tránh lỗi nghiêm trọng: ép đặt điểm ra ngoài biên ảnh hoặc đặt tại mép viền sẽ làm mô hình học sai kích thước chi thể và bóp méo phân phối tọa độ chuẩn hóa [0, 1]. |
| Cổ tay nằm sau tay lái / sau thân mình | Chiếu theo trục cẳng tay từ khuỷu tay hướng về phía bàn tay, đặt chấm ước lượng tại vị trí khớp cổ tay giải phẫu và đánh dấu cờ `v = 1`. Tuyệt đối không xóa điểm hay để `v = 0` nếu vùng cổ tay còn trong khung hình. | Cẳng tay định hướng rõ ràng vị trí cổ tay. Dùng `v = 1` giữ nguyên kết nối xương khuỷu tay - cổ tay (tránh gãy chuỗi động học skeleton). |
| Hai người chồng lên nhau (Occlusion giữa người với người) | Lần lượt gán trọn vẹn từng người một (bật/tắt layer hoặc filter từng skeleton trên CVAT). Khớp của người bị che khuất bởi cơ thể người đứng trước phải gán `v = 1` theo đúng trục giải phẫu của người đó, không chấm nhảy sang khớp của người đứng trước. | Tránh lỗi `nham_nguoi` (ID switch / cross-person keypoint matching). Mô hình nếu học điểm của người A gán sang người B sẽ dự đoán skeleton bị biến dạng kéo dài bất thường. |
| Người nhỏ đến mức nào thì không gán nữa | Chỉ gán người có chiều cao bounding box >= 40 pixels hoặc chiều dài thân mình nhìn rõ được từ vai đến hông. Người quá nhỏ ở hậu cảnh (dưới 30 pixels, mờ nhòe không phân biệt được đầu - thân) bỏ qua không gán skeleton. | Ở độ phân giải dưới 30px, sai số 2-3 pixel ước lượng đã vượt quá bán kính dung sai OKS (kích thước người quá nhỏ làm $\sigma \cdot s$ cực bé), khiến việc gán nhãn không còn ý nghĩa khoa học và gây nhiễu cho loss function. |

*Lưu ý hình ảnh mẫu:* Khi thực hiện trên CVAT, chụp màn hình các trường hợp che khuất điển hình (đặc biệt là tư thế ngồi lái xe, góc chụp nghiêng của đầu, và người bị cắt nửa thân dưới ở mép dưới) để làm tài liệu chuẩn hóa trực quan cho cả nhóm.

## 3. Ba ca mơ hồ đã gặp (bắt buộc, ghi ít nhất 3)

### Ca 1 - ảnh `train_04.jpg`, người thứ `2`, khớp `left_wrist`

- **Mơ hồ ở chỗ nào:** Người thứ 2 đứng sát cạnh người thứ 1, hai cánh tay có vùng giao thoa gần nhau. Cổ tay trái của người thứ 2 bị che khuất một phần bởi bàn tay và thân mình của người thứ 1, rất dễ bị chấm nhầm vào cổ tay hoặc mép áo người thứ 1.
- **Bạn quyết thế nào:** Gán `v = 1`, đặt chấm ước lượng nằm dọc theo hướng xương cẳng tay trái của người thứ 2, giữ khoảng cách độc lập với skeleton của người thứ 1.
- **Vì sao:** Bằng chứng thị giác từ cẳng tay trái cho thấy hướng đi rõ ràng của xương trụ và xương quay, khớp vẫn nằm trọn vẹn bên trong ảnh chứ không vượt ra khỏi mép khung hình.
- **Nếu người khác quyết ngược lại thì model học sai cái gì:** Nếu chấm sang cánh tay người thứ 1 sẽ gây lỗi `nham_nguoi`, model sẽ học thói quen "kéo nối xương chéo" giữa hai người đứng cạnh nhau; nếu đặt `v = 0` thì model sẽ bỏ sót khớp cổ tay bị che khuất thông thường.

### Ca 2 - ảnh `train_01.jpg`, người thứ `1`, khớp `left_ear`

- **Mơ hồ ở chỗ nào:** Người quay góc nghiêng sang phải (profile view), toàn bộ tai trái bị che khuất hoàn toàn bởi mái tóc và hộp sọ từ góc chụp ngược lại.
- **Bạn quyết thế nào:** Gán `v = 1`, đặt chấm ước lượng tại vị trí đối xứng giải phẫu của tai trái so với tai phải qua trục mắt - mũi.
- **Vì sao:** Toàn bộ vùng đầu của người nằm gọn ở trung tâm bức ảnh. Khớp tai tuy bị che khuất tầm nhìn nhưng vật lý vẫn nằm nguyên trong khung hình, do đó bắt buộc là `v = 1` chứ không thể là `v = 0`.
- **Nếu người khác quyết ngược lại thì model học sai cái gì:** Nếu gán `v = 0` (lỗi xóa khớp bị che), model sẽ học sai rằng người chụp nghiêng thì tai bị biến mất khỏi không gian hình học, làm mất thông tin ước lượng góc quay đầu 3D.

### Ca 3 - ảnh `train_15.jpg`, người thứ `2`, khớp `left_elbow` và `left_wrist`

- **Mơ hồ ở chỗ nào:** Người mặc áo khoác tối màu, cánh tay gập về phía trước và bị thân người cùng đạo cụ cầm tay che khuất vùng nếp gấp khuỷu tay.
- **Bạn quyết thế nào:** Gán `v = 1` cho cả hai khớp; xác định vị trí khuỷu tay dựa trên phần bả vai trái thả xuống và đỉnh nhọn của nếp gấp ống tay áo, sau đó định vị cổ tay hướng về phía vật đang cầm.
- **Vì sao:** Dù bóng tối và nếp nhăn quần áo làm mờ biên giới cơ thể, nhưng chiều dài tương đối của xương cánh tay người trưởng thành xấp xỉ bằng khoảng cách từ vai đến thắt lưng.
- **Nếu người khác quyết ngược lại thì model học sai cái gì:** Nếu đặt lệch ra ngoài thân áo sẽ tạo ra lỗi `truot_han` hoặc `lech_nhe` lớn, khiến model dự đoán xương cánh tay bị dị tật biến dạng cong hoặc kéo dài quá mức.

