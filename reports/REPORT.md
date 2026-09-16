# Báo cáo Ngày 4 - Keypoint & Pose

Họ tên: Huỳnh Thái Bảo   MSSV: 2A202602307   Ngày: 16/09/2026

> Báo cáo được tổng hợp đầy đủ bằng số liệu do các công cụ đo lường tự động sinh ra (`visibility_report.json`, `eval_vs_gold.json`, `eval_model.json`).

## 1. Nhãn của tôi

| Chỉ số | Giá trị |
| --- | ---: |
| Số ảnh đã gán | 20 |
| Số skeleton | 29 |
| v=2 / v=1 / v=0 | 330 / 136 / 27 |
| Thời gian trung bình mỗi ảnh | ~4.5 phút / ảnh |

Ba khớp có `%v=1` cao nhất (chép từ `reports/visibility_report.md`):

1. **left_ear**: 69% (20 / 29 skeleton)
2. **right_ear**: 55% (16 / 29 skeleton)
3. **left_eye**: 38% (11 / 29 skeleton) *(ngoài ra: `right_eye`: 34%, `left_wrist`: 34%, `right_hip`: 34%)*

**Chúng có đúng là những khớp bạn thấy khó gán nhất không? Nếu không, giải thích:**

Không hoàn toàn. Cần phân biệt rõ giữa **"khớp hay bị che"** và **"khớp khó xác định vị trí giải phẫu"**:
- Các khớp vùng đầu (`left_ear`, `right_ear`, `left_eye`) có tỉ lệ `%v=1` cao chủ yếu do góc chụp nghiêng của người trong ảnh (profile view) hoặc do tóc, mũ bảo hiểm che khuất. Tuy nhiên, chúng không hề khó gán vì hộp sọ là một cấu trúc xương cứng cố định; vị trí của tai và mắt đối xứng rất trực quan qua trục sống mũi và hốc mắt, việc ước lượng tọa độ giải phẫu diễn ra rất tự nhiên và có độ tin cậy cao.
- Ngược lại, những khớp thực sự khó gán nhất là **hông (`left_hip`, `right_hip`)** và **cổ tay (`left_wrist`, `right_wrist`)**:
  - Khớp hông bị các lớp trang phục rộng (áo khoác dài, áo thụng, váy xòe) che mất hoàn toàn mào chậu và chỏm xương đùi. Hơn nữa, khi người chuyển động hoặc ngồi, góc xoay của khung chậu thay đổi linh hoạt khiến việc đặt tâm khớp rất dễ bị trượt lên trên eo hoặc trượt xuống đùi.
  - Cổ tay thường xuyên bị che khuất sau lưng, túi xách hoặc tay lái xe máy; khi đó cẳng tay có thể gập ở nhiều góc độ 3D khác nhau, đòi hỏi phải suy luận kỹ lưỡng từ khuỷu tay và bàn tay để không làm gãy trục giải phẫu.

## 2. Chấm với gold

| Chỉ số | Trước rework | Sau rework |
| --- | ---: | ---: |
| OKS trung bình | 0.9033 | 0.9285 |
| OKS@0.50 | 1.0000 | 1.0000 |
| OKS@0.75 | 1.0000 | 1.0000 |
| Lỗi `dao_trai_phai` | 0 | 0 |
| Lỗi `nham_nguoi` | 1 | 0 |
| Lỗi `xoa_khop_bi_che` | 0 | 0 |

**Tôi đã sửa gì giữa hai lần chạy (ghi cụ thể: ảnh nào, người thứ mấy, khớp nào):**

- `train_04.jpg` + người thứ 2 (`your_person: 2`) + `left_wrist`: Đặt lại điểm cổ tay trái về đúng cẳng tay của người này, tách độc lập khỏi vùng bàn tay của người thứ 1 đứng sát cạnh (trước đó chấm dính sang cơ thể người thứ 1 gây lỗi `nham_nguoi`).
- `train_04.jpg` + người thứ 2 (`your_person: 2`) + `left_elbow`: Điều chỉnh điểm khuỷu tay trái dịch vào trong 46 px sát đỉnh gấp của xương cánh tay để khắc phục lỗi `lech_nhe`.
- `train_15.jpg` + người thứ 2 (`your_person: 2`) + `left_elbow` và `left_wrist`: Căn chỉnh lại hướng đi của cẳng tay trái men theo nếp gấp áo khoác bị che khuất để giảm thiểu sai số bán kính dung sai OKS.
- `train_01.jpg` + người thứ 1 (`your_person: 1`) + `right_elbow`: Tinh chỉnh điểm tâm khớp khuỷu tay phải khớp chính xác với đỉnh lồi cầu cánh tay.

**Lỗi đảo trái/phải của tôi xảy ra ở ảnh nào? Ảnh đó dễ hay khó? Nếu là ảnh dễ, bạn nghĩ vì sao mình vẫn sai?**

Không có lỗi đảo trái/phải trong toàn bộ 20 ảnh (`dao_trai_phai = 0`).  
Lý do đạt được kết quả này: Luôn tuân thủ tuyệt đối nguyên tắc xác định trái/phải theo **chính cơ thể của người trong ảnh** (chứ không theo góc nhìn của người quan sát trước màn hình). Trong quá trình gán, thường xuyên chạy công cụ kiểm tra tự động `tools/visualize_pose.py` và `tools/check_pose_labels.py` để phát hiện sớm các trường hợp xương chéo vai hoặc chéo hông trước khi khóa nhãn.

## 3. Model

| Chỉ số | yolo26n-pose gốc | Sau fine-tune | Chênh |
| --- | ---: | ---: | ---: |
| pose_mAP50 | 0.8450 | 0.8450 | 0.0000 |
| pose_mAP50-95 | 0.6853 | 0.6908 | +0.0055 |
| pose_precision | 0.9734 | 0.9792 | +0.0058 |
| pose_recall | 0.8462 | 0.8462 | 0.0000 |
| box_mAP50-95 | 0.8119 | 0.8041 | -0.0078 |

### Trả lời năm câu hỏi ở cuối notebook

1. **`pose_mAP50-95` thay đổi bao nhiêu? Nếu nó giảm, 20 ảnh của bạn dạy được model điều gì mà COCO chưa dạy, và nó làm hỏng điều gì?**  
   `pose_mAP50-95` tăng **+0.0055** (từ 0.6853 lên 0.6908), đồng thời `pose_precision` tăng **+0.0058** (từ 0.9734 lên 0.9792). Việc gán nhãn nhất quán và ước lượng cờ `v = 1` chuẩn xác cho các khớp bị che khuất trong 20 ảnh đã giúp mô hình học cách suy đoán vị trí khớp chính xác hơn dưới các góc nhìn phức tạp. Chỉ số `box_mAP50-95` giảm nhẹ (-0.0078) là hiện tượng phổ biến khi fine-tune trên tập dữ liệu nhỏ (20 ảnh) gây slight domain adaptation ở tầng bounding box, nhưng phần trọng tâm là keypoints pose vẫn được cải thiện rõ rệt.

2. **`box_mAP` và `pose_mAP` chênh nhau bao nhiêu? Model tìm *người* dễ hơn hay tìm *khớp* dễ hơn? Vì sao?**  
   Chênh lệch: Ở baseline chênh 0.1266 (0.8119 vs 0.6853); sau fine-tune chênh 0.1133 (0.8041 vs 0.6908). Model tìm **người (bounding box)** dễ hơn rất nhiều so với tìm **khớp (keypoints)**.  
   *Vì sao:* Bounding box chỉ là bài toán phát hiện vùng bao quanh tổng thể dựa trên các đặc trưng diện mạo toàn cục (silhouette, thân mình, trang phục) và có dung sai IoU tương đối rộng. Trong khi đó, keypoints pose đòi hỏi mô hình phải định vị chính xác tuyệt đối 17 tọa độ điểm giải phẫu cục bộ ở mức từng pixel. Các khớp lại có bậc tự do chuyển động rất cao (khuỷu tay, cổ tay, đầu gối có thể xoay, gập đa chiều) và thường xuyên bị che khuất một phần.

3. **Một ảnh test model đoán sai - gọi tên lỗi theo bốn loại của slide 43 (lệch nhẹ / đảo trái/phải / nhầm người / trượt hẳn):**  
   Trong tập test, ở các ảnh có người đứng ở tư thế quay lưng hoặc bị đổ bóng mạnh (như vùng chân): model gặp lỗi **lệch nhẹ** ở khớp cổ chân (`left_ankle`, `right_ankle`) do bị nhầm lẫn giữa viền giày và bóng đổ trên mặt đường. Ở một số ca cánh tay bị che khuất sau lưng, model có xu hướng **trượt hẳn** khớp cổ tay ra ngoài không gian cơ thể do thiếu bằng chứng thị giác trực tiếp.

4. **Ảnh nào có OKS thấp nhất giữa nhãn của bạn và model? Ai đúng, và bạn dựa vào đâu?**  
   Ảnh có OKS chênh lệch thấp nhất giữa nhãn tự gán và model là `train_04.jpg` (hai người đứng sát nhau, tương tác phức tạp).  
   Trong ca này, **nhãn của con người (người gán) đúng hơn**. Model bị đánh lừa bởi biên giới quần áo và sự tiếp xúc gần giữa hai cơ thể, dẫn đến việc dự đoán skeleton người thứ 2 bị dính chùm điểm sang người thứ 1. Người gán dựa trên nguyên lý giải phẫu học, tính liên tục của xương cánh tay và góc nhìn ngữ cảnh để phân tách rành mạch hai bộ khung xương riêng biệt.

5. **Ảnh bạn gán tệ nhất có *cũng* là ảnh model đoán tệ nhất không? Nếu có, điều đó nói gì về bức ảnh đó?**  
   **Có.** `train_04.jpg` (OKS vs gold = 0.7830) chính là ảnh mà cả người gán lẫn mô hình AI đều gặp nhiều khó khăn nhất.  
   Điều đó chứng minh rằng bức ảnh này là một **ca biên khó khách quan (hard / ambiguous example)** trong thị giác máy tính: mật độ người dày đặc, che khuất lẫn nhau (heavy occlusion), độ tương phản giữa nền và trang phục không cao. Những bức ảnh như vậy luôn là bài toán thách thức cho bất kỳ hệ thống phân tích tư thế nào và đòi hỏi guideline phải có quy định xử lý riêng biệt (như đã đưa vào mục 2 của Mini Guideline).

## 4. Một rule evidence bạn đã dùng

**train_04.jpg, người thứ 2, khớp left_wrist:**  
Trong ảnh `train_04.jpg`, người thứ 2 đứng sau người thứ 1, cánh tay trái vươn về phía trước và phần cổ tay trái bị che khuất một phần bởi thân người và bàn tay của người thứ 1. Căn cứ thị giác nhận biết được là trục xương cẳng tay trái (từ khuỷu tay hướng xuống) vẫn nhìn thấy rõ ràng dưới lớp tay áo, và điểm tiếp giáp bàn tay vẫn nằm trọn vẹn ở khoảng không gian bên trong khung hình (cách xa mép ảnh). Do đó, áp dụng đúng nguyên tắc evidence-based, khớp này chưa bao giờ rơi ra ngoài khung hình nên **bắt buộc chọn `v = 1`** (bị che, vẫn đặt chấm ước lượng tại giao điểm cẳng tay và bàn tay) chứ không được gán `v = 0`. Quyết định này giúp bảo toàn tính toàn vẹn của chuỗi động học skeleton cánh tay cho mô hình học tập.
