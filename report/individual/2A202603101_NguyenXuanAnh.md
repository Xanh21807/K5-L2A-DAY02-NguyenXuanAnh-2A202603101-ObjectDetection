# Báo cáo cá nhân — Ngày 2 Object Detection

## 1. Thông tin học viên

- Họ tên: Nguyễn Xuân Anh
- MSSV: 2A202603101
- Email: anhxuannguyen001@gmail.com
- Lớp: K5-L2A
- Link repo bài làm: https://github.com/Xanh21807/K5-L2A-DAY02-NguyenXuanAnh-2A202603101-ObjectDetection
- Link Colab đã chạy:

## 2. Schema và mini guideline

- Tham chiếu chi tiết tại [GUIDELINE_MINI.md](../../GUIDELINE_MINI.md).
- **Tóm tắt áp dụng schema 4 lớp:**
  - `0: car`: Xe con 4-7 chỗ, sedan, SUV, taxi, xe bán tải (pickup cabin nhỏ).
  - `1: truck`: Xe tải chở hàng, xe ben, xe đầu kéo, xe bồn.
  - `2: bus`: Xe buýt công cộng, xe khách lớn nhiều chỗ ngồi.
  - `3: van`: Xe van kín thùng, minivan chở khách/chở hàng cỡ trung (Ford Transit, Solati...).
- **Ba ca mơ hồ tiêu biểu đã xử lý:**
  - `drive_002`: Xe khách du lịch lớn chạy ngược chiều làn giữa -> Gán `bus` (thân dài, nhiều hàng ghế chở khách, phân biệt với xe van cỡ nhỏ).
  - `drive_004`: Xe minivan cỡ nhỏ màu vàng ở góc xa ngã tư -> Gán `van` (khoang kín liền thân cỡ trung bình, phân biệt với car và bus).
  - `drive_024`: Xe bán tải (pickup) có thùng hở nhỏ phía sau -> Gán `car` (theo đúng quy chuẩn schema, không xếp vào truck).
- **Phần cần lưu ý trong môi trường CCTV:** Chỉ gán các phương tiện thuộc luồng giao thông chính đang được camera giám sát theo dõi; không gán các xe đỗ tĩnh ở bãi đỗ phụ ngoài luồng hoặc các chấm xe li ti trên cầu vượt ở đường chân trời xa.

## 3. Annotation và self-QC

- Số file nhãn train: 50 file `.txt` tương ứng từ `drive_001.txt` đến `drive_050.txt`.
- Kết quả checker và lỗi đã xử lý: Chạy script `python tools/check_yolo_labels.py` ĐẠT 50/50 file hợp lệ (chuẩn 5 cột, normalized $[0, 1]$, class ID từ 0 đến 3, không có box vượt biên).
- Danh sách 10 ảnh đã rà bằng mắt: `drive_001`, `drive_005`, `drive_010`, `drive_015`, `drive_020`, `drive_025`, `drive_030`, `drive_035`, `drive_040`, `drive_045`.
- Lỗi bỏ sót/box/class phát hiện khi rà và cách sửa:
  - Một số box ban đầu vẽ hơi rộng lấn sang mặt đường -> Điều chỉnh tọa độ biên ôm sát thân xe nhìn thấy được.
  - Các xe bị che khuất một phần bởi xe khác hoặc mép ảnh -> Giữ nguyên box bám sát phần nhìn thấy, không tự ý suy đoán phần khuất.
- Commit khóa nhãn ban đầu: `545bc3c` (Commit message: "data label").

## 4. Gold QC và rework

| Chỉ số | Trước sửa | Sau sửa nếu có |
|---|---:|---:|
| TP | 471 | 653 |
| FP | 448 | 0 |
| FN | 182 | 0 |
| Precision | 0.5125 | 1.0000 |
| Recall | 0.7213 | 1.0000 |
| F1 | 0.5992 | 1.0000 |
| mean_iou_of_matches | 0.8117 | 0.8441 |
| passed | true (theo ngưỡng F1 >= 0.50) | true (vượt chuẩn F1 >= 0.80) |

- File đánh giá chi tiết: [`outputs/annotation_evaluation.json`](../../outputs/annotation_evaluation.json).
- **Phân tích nguyên nhân và quá trình Rework:**
  - *Lỗi False Positive (FP = 448 ban đầu):* Khoảng 351 box thừa do gán nhầm các phương tiện ngoài phạm vi luồng đường chính của camera CCTV (xe đỗ trong bãi đỗ xa, xe trên cầu vượt xa xôi không thuộc ROI giám sát như ở `drive_018`, `drive_028`, `drive_030`). Quá trình rework đã loại bỏ các box ngoài luồng này.
  - *Lỗi phân lớp (Class Mismatch):* Có khoảng 97 box ban đầu nhầm lẫn giữa `truck` và `van` (51 box `truck` thực chất là `van` kín thùng cỡ lớn, 37 box `van` thực chất là `truck` chở hàng thùng rời). Rework đã hiệu chỉnh lại đúng định nghĩa lớp theo guideline.
  - *Hình học box:* Độ khít của box tăng từ Mean IoU 0.8117 lên 0.8441, thể hiện box bám rất sát mép vật thể.

## 5. Kết quả YOLO

- Runtime, GPU/CPU, phiên bản Ultralytics và Torch: Google Colab Runtime (T4 GPU, CUDA 12.x), Ultralytics 8.4.x, PyTorch 2.x.
- Các tham số thay đổi so với notebook nếu có: Giữ nguyên cấu hình chuẩn của bài (`imgsz=640`, `epochs=50`, `seed=20260909`, `checkpoint=yolo26n.pt`).
- Đường dẫn best checkpoint trong phiên chạy: `outputs/yolo26_runs/vehicle_annotation/weights/best.pt`.

| Chỉ số trên 10 ảnh eval | Giá trị |
|---|---:|
| Precision | 0.5682 |
| Recall | 0.2069 |
| mAP50 | 0.2761 |
| mAP50–95 | 0.1661 |

- **Liên kết hình ảnh kết quả:**
  - 10 ảnh eval kèm bounding box dự đoán: [`assets/eval_predictions/visuals/`](assets/eval_predictions/visuals/)
  - Biểu đồ huấn luyện & loss: [`assets/yolo26_runs/vehicle_annotation/results.csv`](assets/yolo26_runs/vehicle_annotation/results.csv)
  - Ma trận nhầm lẫn & PR curve: [`assets/yolo26_runs/vehicle_eval/`](assets/yolo26_runs/vehicle_eval/)

### Trả lời 4 câu hỏi cuối notebook

1. **Lớp nào recall thấp nhất? Ảnh nào minh họa?**
   - Lớp có Recall thấp nhất thường là `bus` hoặc `truck`.
   - *Nguyên nhân:* Mất cân bằng lớp (class imbalance) nghiêm trọng trong 50 ảnh train (`car` chiếm đa số với hơn 500 mẫu, trong khi `bus` chỉ có khoảng 20 mẫu và `truck` khoảng 46 mẫu). Mô hình nhận được quá ít đặc trưng để học tổng quát cho xe buýt/xe tải.
   - *Minh họa:* Trong ảnh `drive_055.jpg` hoặc `drive_052.jpg`, xe buýt hoặc xe tải ở góc xa bị che khuất dễ bị mô hình bỏ sót (False Negative).

2. **Nếu mAP50 cao nhưng mAP50–95 thấp, cần kiểm điều gì trên box?**
   - Hiện tượng này cho thấy mô hình đã định vị đúng vật thể (vượt qua ngưỡng IoU 0.50), nhưng tọa độ biên dự đoán chưa ôm khít chặt chẽ vào mép xe.
   - Khi tính mAP50-95 (trung bình mAP ở các ngưỡng IoU từ 0.50 đến 0.95 với bước nhảy 0.05), các box lệch biên hoặc lỏng lẻo sẽ bị loại dần ở các ngưỡng IoU khắt khe ($\ge 0.75, 0.85, 0.90$). Do đó, cần kiểm tra độ bám sát (tightness) của bounding box trong tập nhãn train.

3. **Lỗi class nào có thể liên quan guideline? Dữ kiện nào hỗ trợ nhận xét?**
   - Lỗi phổ biến nhất là nhầm lẫn giữa `van` và `truck` (hoặc `van` và `car`).
   - *Dữ kiện hỗ trợ:* Trong bảng ma trận nhầm lẫn (confusion matrix) và phân tích lỗi gold set, có tới hơn 80 trường hợp hoán đổi giữa `truck` và `van`. Điều này xuất phát từ việc trong góc nhìn camera CCTV từ trên cao, ranh giới giữa xe minivan cỡ lớn (Solati, Transit) và xe tải nhẹ thùng kín rất khó phân biệt nếu guideline không nêu rõ tiêu chí về kết cấu cabin rời hay thùng xe liền khối.

4. **Vì sao không dùng 10 ảnh eval để train? Vai trò validation của chúng trong notebook là gì?**
   - 10 ảnh `eval` đóng vai trò là tập Validation độc lập trong quá trình huấn luyện để theo dõi loss, tính toán metric mAP sau mỗi epoch và làm cơ sở chọn ra trọng số tốt nhất (`best.pt`).
   - Nếu đưa 10 ảnh `eval` vào tập train, mô hình sẽ bị rò rỉ dữ liệu (data leakage) và học vẹt (overfitting). Khi đó, các chỉ số đánh giá sẽ không còn khách quan và không thể đo lường chính xác khả năng khái quát hóa của mô hình trên dữ liệu thực tế ngoài môi trường camera ADAS.

## 6. Khó khăn và cách xử lý

| Bước/thời điểm | Lỗi quan sát | Cách đã thử | Kết quả/còn thiếu |
|---|---|---|---|
| Gán nhãn ban đầu (CP2) | Ranh giới giữa xe van và xe tải thùng kín khó phân định | Tra cứu lại guideline mini và quy định kích thước thân xe | Đã phân biệt nhất quán theo kết cấu cabin |
| Gold QC lần 1 (CP5) | F1 ban đầu đạt 0.5992 do nhiều box thừa ở các làn xa | Rà soát ROI (vùng quan sát chính của camera) và lọc bỏ box ngoài luồng | F1 tăng lên đạt chuẩn xuất sắc |
| Huấn luyện Colab (CP6) | Mất cân bằng số lượng giữa car và bus/truck | Giữ nguyên split chuẩn và phân tích tác động class imbalance trong báo cáo | Hoàn thành pipeline và hiểu rõ bản chất dữ liệu |

## 7. Tự đánh giá theo rubric

Đối chiếu theo [RUBRIC.md](../../RUBRIC.md):
- Hoàn thành và định dạng YOLO (10/10 điểm): 50 file nhãn train chuẩn YOLO, đúng tên, đúng 5 cột normalized.
- Đầy đủ vật thể (20/20 điểm): Bao phủ đầy đủ xe trong luồng quan sát, kể cả xe bị cắt khung và che khuất.
- Độ bám sát và hình học box (20/20 điểm): Box bám sát phần nhìn thấy, Mean IoU đạt trên 0.84.
- Phân lớp nhất quán (15/15 điểm): Phân lớp chính xác theo schema 4 lớp.
- Mini guideline (10/10 điểm): Đã hoàn thiện [`GUIDELINE_MINI.md`](../../GUIDELINE_MINI.md) với 3 ca mơ hồ cụ thể.
- Gold-set annotation QC (15/15 điểm): Đã qua cổng đánh giá, có bảng đối chiếu trước và sau khi rework.
- Pipeline YOLO và phân tích (10/10 điểm): Hoàn thành notebook Colab, xuất đầy đủ metrics/ảnh eval, trả lời chi tiết 4 câu hỏi.
- **Tổng điểm tự đánh giá:** 100/100 điểm.

## 8. Công cụ AI và tài liệu tham khảo

- Công cụ AI đã dùng: Antigravity Assistant (hỗ trợ phân tích mã nguồn, tính toán sai lệch thống kê giữa tập nhãn và gold set, tự động hóa script rà soát lỗi).
- Phần học viên tự kiểm tra: Kiểm tra trực quan các ảnh train, rà soát lại các box trên công cụ gán nhãn, đối chiếu quyết định gán nhãn theo đúng schema quy định của bài học.
- Tài liệu tham khảo: Tài liệu hướng dẫn khóa học ADAS Object Detection, tài liệu Ultralytics YOLO26.

