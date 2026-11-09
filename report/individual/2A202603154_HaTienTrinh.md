# Báo cáo cá nhân — Ngày 2 Object Detection

## 1. Thông tin học viên

- Họ tên: Hà Tiến Trịnh
- MSSV: 2A202603154
- Email: 26ai.trinhht@vinuni.edu.vn
- Lớp: L2A
- Link repo bài làm: https://github.com/HaTienTrinh/K5-L2-Day02-HaTienTrinh-2A202603154-ObjectDetection/blob/main/report/individual/MSSV_HoTen.md
- Link Colab đã chạy: https://colab.research.google.com/drive/12sw2ooRwbD-EgBAL_7DJmuP_Q2uIGXDl?usp=sharing

## 2. Schema và mini guideline

Tóm tắt 3 ca mơ hồ đã quyết định:
- drive_001.jpg: Xe SUV trắng bị cột đèn che dọc giữa thân và hàng xe đỗ bên lề trái → Vẽ 1 box car thống nhất cho xe trắng; ở vòng đầu gán car cho xe đỗ lề trái nhưng sau khi đối chiếu Gold QC phát hiện lề trái thuộc vùng Ignore Region của UA-DETRAC nên đã lược bỏ khi rework.
- drive_005.jpg: Xe tải cứu hộ màu trắng ở giữa đường và dòng xe xếp hàng rất xa dưới gầm cầu vượt → Gán truck cho xe cứu hộ; chỉ gán các xe còn rõ hình khối ở phía trên và bỏ qua các chấm xe quá mờ ở cuối hàng.
- drive_015.jpg: Xe tải nhỏ chở các bao trắng ở làn trên bên phải và mảnh nóc xe bị cắt sát mép dưới cùng → Gán truck cho xe chở hàng; bỏ qua phần nóc xe quá mảnh ở sát rìa dưới.

Phần cần giảng viên làm rõ: Quy ước ánh xạ ID lớp thực tế trong bộ gold_train_labels và dataset/labels/eval (được xuất từ UA-DETRAC với thứ tự thực tế trên ảnh là 0: xe con, 1: xe van, 2: xe tải, 3: xe buýt) cũng như ranh giới vùng Ignore Region của UA-DETRAC ở hậu cảnh xa.

## 3. Annotation và self-QC

- Số file nhãn train: 50 file .txt
- Kết quả checker và lỗi đã xử lý: đạt 50/50 file nhãn hợp lệ
- Danh sách ít nhất 10 ảnh đã rà bằng mắt: drive_001.jpg, drive_002.jpg, drive_003.jpg, drive_005.jpg, drive_008.jpg, drive_009.jpg, drive_015.jpg, drive_024.jpg, drive_041.jpg, drive_042.jpg, drive_046.jpg.
- Lỗi bỏ sót/box/class phát hiện khi rà và cách sửa: Phát hiện một số box ở sát rìa ảnh cần chỉnh lại cạnh cho khít phần nhìn thấy, một số xe nhỏ ở quá xa bị gán dư hoặc nhầm giữa car và van.
- Commit khóa nhãn ban đầu: 8a277d2 

## 4. Gold QC và rework

| Chỉ số | Trước sửa | Sau sửa nếu có |
|---|---:|---:|
| TP | 416 | 511 |
| FP | 337 | 113 |
| FN | 237 | 142 |
| Precision | 0.5525 | 0.8189 |
| Recall | 0.6371 | 0.7825 |
| F1 | 0.5917 | 0.8003 |
| mean_iou_of_matches | 0.8527 | 0.8579 |
| passed | false | true |

Dẫn tới `outputs/annotation_evaluation.json` trong repo. Nếu rework, ghi ảnh, lỗi đã kiểm, quyết định sửa/giữ và lý do. Không suy nguyên nhân chỉ từ FP/FN. Không có rework thì ghi rõ.

Phân tích nguyên nhân (không chỉ nhìn số FP/FN mà mở lại ảnh đối chiếu):
Độ bám sát hình học của box ngay từ đầu đã rất tốt (mean_iou_of_matches = 0.8527 >= 0.70), không bị lỗi box lỏng.
Nguyên nhân 1 (Lỗi ánh xạ ID hệ thống giữa data.yaml và bộ nhãn UA-DETRAC): Trên CVAT đã khởi tạo đúng thứ tự 0: car, 1: truck, 2: bus, 3: van theo data.yaml. Tuy nhiên khi mở ảnh đối chiếu với gold_train_labels (ví dụ drive_009, drive_041, drive_042) và cả dataset/labels/eval (drive_051, drive_052, drive_053), phát hiện bộ nhãn gốc từ UA-DETRAC đang mang ID 1 cho xe van, 2 cho xe tải, 3 cho xe buýt. Vì không được phép sửa data.yaml và gold_train_labels, đã thực hiện ánh xạ đồng bộ ID trong dataset/labels/train/ (1 -> 2, 2 -> 3, 3 -> 1) để khớp hệ quy chiếu với tập Gold và tập Eval, giúp TP tăng ngay từ 416 lên 513.
Nguyên nhân 2 (Gán dư xe trong vùng Ignore Region và hậu cảnh quá xa của UA-DETRAC): Mở lại các ảnh drive_001 (4 xe đỗ lề trái), drive_002, drive_003, drive_008, drive_009, drive_012, drive_020, drive_024, drive_031, drive_033, drive_038, drive_046 (4 chấm xe ở sát mép trên y ~ 0.03), drive_007, drive_045, drive_049 (làn xe bên trái x < 0.30). Quyết định: Xóa thủ công trên CVAT các box quá nhỏ ở xa và các box nằm trong vùng Ignore Region của UA-DETRAC, giúp giảm FP từ 337 xuống 113 và nâng F1 lên 0.8003 (passed: true).

## 5. Kết quả YOLO

- Runtime, GPU/CPU, phiên bản Ultralytics và Torch: Google Colab GPU (T4 GPU), Python 3, phiên bản Ultralytics & PyTorch mới nhất.
- Các tham số thay đổi so với notebook nếu có và lý do: Không thay đổi (giữ nguyên mặc định của notebook).
- Đường dẫn best checkpoint trong phiên chạy: /content/Day2-Lab/outputs/yolo26_runs/vehicle_annotation/weights/best.pt

| Chỉ số trên 10 ảnh eval | Giá trị |
|---|---:|
| Precision | 0.8848 |
| Recall | 0.5651 |
| mAP50 | 0.7018 |
| mAP50–95 | 0.4670 |

Đặt cả 2 folder `assets/eval_predictions/` và `assets/yolo26_runs/` (giải nén từ `day2_colab_outputs.zip`) cạnh báo cáo và dẫn link tương đối. Không chỉ ghi số; chỉ ra ít nhất một ảnh và vật thể cụ thể.

Log huấn luyện 50 epochs:
 [assets/yolo26_runs/vehicle_annotation/results.csv]
 (assets/yolo26_runs/vehicle_annotation/results.csv)

Biểu đồ đánh giá trên tập eval:
- [Confusion Matrix](assets/yolo26_runs/vehicle_eval/confusion_matrix.png)
- [PR Curve](assets/yolo26_runs/vehicle_eval/BoxPR_curve.png)
- [Recall Curve](assets/yolo26_runs/vehicle_eval/BoxR_curve.png)

Ảnh dự đoán trên tập eval (nhận xét cụ thể):
- [assets/eval_predictions/visuals/drive_051.jpg](assets/eval_predictions/visuals/drive_051.jpg): Model nhận diện chính xác hầu hết các xe car với độ tự tin cao (0.77 – 0.93) và phát hiện đúng chiếc xe tải thùng trắng ở góc trên bên phải (bus theo ID của tập eval), nhưng bị cột đèn che dọc phía trước làm giảm confidence.
- [assets/eval_predictions/visuals/drive_058.jpg](assets/eval_predictions/visuals/drive_058.jpg): Model bắt rất tốt hàng xe con (car 0.91 – 0.97) và dàn xe buýt xếp hàng bên trái (van 0.54 – 0.84 theo ID tập eval), nhưng chiếc xe chuyên dụng màu trắng-xanh ở làn bên phải bị dự đoán nhầm thành car 0.86.

Trả lời bốn câu hỏi cuối notebook:

1. Lớp nào recall thấp nhất? Ảnh nào minh họa?
- Lớp có Recall và mAP50 thấp nhất là bus (ID 2 — đạt Recall tối đa 0.50, mAP50 = 0.513, chỉ đúng 1/2 mẫu trong tập eval) và tiếp theo là truck (ID 1 — đạt Recall tối đa 0.60, mAP50 = 0.602, đúng 3/5 mẫu). Trong khi đó car (mAP50 = 0.866) và van (mAP50 = 0.827) cao hơn nhiều do hiện tượng mất cân bằng lớp (class imbalance).
- Ảnh minh họa: [assets/eval_predictions/visuals/drive_058.jpg](assets/eval_predictions/visuals/drive_058.jpg) — phương tiện mang nhãn ID 2 ở làn bên phải (x=0.9375, y=0.3968) bị model dự đoán nhầm thành car 0.86 khiến Recall của lớp này chỉ đạt 50%.

2. Nếu mAP50 cao nhưng mAP50–95 thấp, cần kiểm điều gì trên box?
- Hiện tại mAP50 = 0.7018 khá tốt nhưng mAP50–95 = 0.4670 thấp hơn đáng kể. Vì mAP50–95 tính trung bình qua các ngưỡng IoU khắt khe từ 0.50 đến 0.95, khoảng cách lớn này cho thấy model xác định đúng vị trí tương đối và đúng lớp của xe (vượt ngưỡng IoU 0.50), nhưng 4 cạnh bounding box chưa thực sự bám khít tuyệt đối ở các ngưỡng IoU cao (0.75 – 0.95). Cần kiểm tra lại độ khít (tightness) của các cạnh box trên những vật thể bị che khuất một phần, bị cắt ở rìa ảnh, hoặc các xe nhỏ ở xa trong tập train.

3. Lỗi class nào có thể liên quan guideline? Dữ kiện nào hỗ trợ nhận xét?
- Lỗi nhầm lẫn giữa car với truck (ID 1, 2 mẫu bị đoán thành car trong confusion matrix) và giữa car với bus (ID 2, 1 mẫu bị đoán thành car).
- Dữ kiện hỗ trợ: (1) Các dòng xe SUV cỡ lớn, minivan, xe bán tải hoặc xe tải nhỏ nhìn từ góc camera CCTV trên cao ở khoảng cách xa có hình khối rất giống car; (2) Sự lệch quy ước ánh xạ ID giữa bảng tên trong data.yaml và bộ nhãn gốc UA-DETRAC trong gold_train_labels/labels/eval.
4. Vì sao không dùng 10 ảnh eval để train? Vai trò validation của chúng trong notebook là gì?
- Không được đưa 10 ảnh eval vào tập train để tránh rò rỉ dữ liệu (data leakage): nếu model đã học trực tiếp trên các ảnh này thì điểm mAP khi đánh giá sẽ cao giả tạo (memorization/overfitting), không phản ánh khả năng tổng quát hóa trên cảnh mới.
- Trong notebook này, 10 ảnh eval đóng vai trò là tập validation (val) dùng sau mỗi epoch để theo dõi loss/mAP và tự động chọn ra checkpoint tốt nhất (best.pt), đồng thời dùng để đánh giá cuối cùng (do đó đây là tập validation, không phải một tập test độc lập hoàn toàn).

## 6. Khó khăn và cách xử lý

| Bước/thời điểm | Lỗi quan sát | Cách đã thử | Kết quả/còn thiếu |
|---|---|---|---|
| CP2 – CP3 (Gán nhãn & Self-QC) | Các phương tiện ở quá xa dưới gầm cầu hoặc bị cột đèn/xe khác che khuất khó phân biệt lớp và xác định biên box. | Thiết lập quy tắc rõ trong GUIDELINE_MINI.md, chạy check_yolo_labels.py và rà lại bằng mắt trên CVAT. | 50/50 file đạt chuẩn cấu trúc YOLO, mean_iou_of_matches đạt > 0.85. |
| CP5 (Chấm lần 1 với Gold set) | mean_iou_of_matches đạt 0.8527 nhưng F1 chỉ đạt 0.5917 (FP=337, FN=237). | Mở ảnh đối chiếu tọa độ giữa dataset/labels/train và gold_train_labels, phát hiện ID 3 lớp truck/bus/van trong bộ nhãn UA-DETRAC gốc được đánh số theo quy ước 1=van, 2=truck, 3=bus. | Ánh xạ đồng bộ ID trong dataset/labels/train/ (1->2, 2->3, 3->1), giúp TP tăng từ 416 lên 513 và F1 tăng lên 0.7297. |
| CP5 (Vòng Rework trên CVAT) | Vẫn còn nhiều lỗi FP do gán cả các xe nằm trong vùng Ignore Region và hậu cảnh quá xa của bộ UA-DETRAC (như lề trái drive_001, làn trái drive_007, drive_045, drive_049 và vùng xa ở drive_002, drive_003). | Mở lại từng ảnh trên CVAT để xóa thủ công các box thừa ở vùng Ignore/hậu cảnh xa rồi xuất lại và đồng bộ ID. | Giảm FP xuống 113, nâng F1 lên 0.8003 (passed: true), đủ điều kiện sang bước Train YOLO. |

## 7. Tự đánh giá theo rubric

- Hoàn thành và định dạng YOLO (10/10đ): Đã hoàn thành đủ 50 file .txt cùng basename, đúng 5 cột chuẩn hóa, vượt qua check_yolo_labels.py.
- Đầy đủ vật thể (20/20đ): Đã gán đầy đủ các phương tiện thuộc schema (Recall đạt 0.7825 với 511 TP).
- Độ bám sát và hình học box (20/20đ): Box bám rất sát vật thể, mean_iou_of_matches đạt 0.8579 (vượt mức Xuất sắc >= 0.80).
- Phân lớp nhất quán (15/15đ): Phân biệt rõ 4 lớp và không gán vật thể ngoài phạm vi schema.
- Mini guideline và quyết định mơ hồ (10/10đ): Đã điền đầy đủ định nghĩa 4 lớp, quy tắc chung và 3 ca mơ hồ (drive_001, drive_005, drive_015) trong GUIDELINE_MINI.md.
- Gold-set annotation QC (15/15đ): Đạt cả 2 ngưỡng (F1 = 0.8003 >= 0.80 và mean_iou = 0.8579 >= 0.70, passed: true) kèm giải trình chi tiết các vòng rework.
- Pipeline YOLO và phân tích (10/10đ): Đã chạy trọn vẹn notebook trên Google Colab với yolo26n.pt, giữ nguyên split eval, trích xuất đầy đủ assets/eval_predictions/ và assets/yolo26_runs/ và trả lời 4 câu hỏi phân tích.
## 8. Công cụ AI và tài liệu tham khảo

Công cụ đã sử dụng:
- CVAT (Computer Vision Annotation Tool): Dùng để vẽ bounding box thủ công, rà soát (Self-QC) và chỉnh sửa (Rework) bộ nhãn 50 ảnh
- Google Colab & Ultralytics YOLO (yolo26n.pt): Dùng để huấn luyện và đánh giá mô hình.
- Trợ lý AI (Antigravity / Gemini): Hỗ trợ chẩn đoán nguyên nhân lệch quy ước Class ID giữa data.yaml và bộ nhãn gốc UA-DETRAC, chạy lệnh đồng bộ ID lớp sau khi xuất từ CVAT và góp ý bố cục trình bày báo cáo.

Tài liệu tham khảo: 
- Tài liệu hướng dẫn nội bộ của bài lab và tài liệu Ultralytics YOLO.