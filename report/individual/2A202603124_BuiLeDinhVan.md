# Báo cáo cá nhân — Ngày 2 Object Detection

## 1. Thông tin học viên

- Họ tên:Bùi Lê Đình Văn
- MSSV:2A202603124
- Email:builedinhvan@gmail.com
- Lớp: L2A
- Link repo bài làm:https://github.com/DinhVann1703/K5-L2-DAY02-BuiLeDinhVan-2A202603124-ObjectDetection
- Link Colab đã chạy:https://colab.research.google.com/drive/1rPk3j-tZBFt3YjzBpK8r4rs1TlmSwcif?hl=vi#scrollTo=6f78b880

## 2. Schema và mini guideline

Dẫn tới [GUIDELINE_MINI.md](../../GUIDELINE_MINI.md). Tóm tắt ba ca mơ hồ đã quyết định, cách áp dụng schema và phần cần giảng viên làm rõ.

## 3. Annotation và self-QC

- Số file nhãn train:50 files
- Kết quả checker và lỗi đã xử lý:Chạy `python tools/check_yolo_labels.py` đạt chuẩn 50/50. Đã chuẩn hóa toàn bộ tọa độ tâm và kích thước về dải $[0, 1]$, xử lý triệt để lỗi box vượt biên ($>1.0$) và lỗi ID lớp vượt quá schema 4 lớp ($0 - 3$).
- Danh sách ít nhất 10 ảnh đã rà bằng mắt:`drive_001`, `drive_002`, `drive_004`, `drive_008`, `drive_015`, `drive_018`, `drive_024`, `drive_028`, `drive_041`, `drive_050`
- Lỗi bỏ sót/box/class phát hiện khi rà và cách sửa:
* Lỗi box vẽ bao trùm bóng xe dưới lòng đường $\rightarrow$ kéo mép dưới lên sát điểm tiếp giáp của lốp xe với mặt đường.
  * Tự động nhận nhầm biển báo và dải phân cách thành xe con $\rightarrow$ xóa bỏ box.
  * Nhầm lẫn minivan thành xe con $\rightarrow$ đổi ID từ 0 (`car`) sang 3 (`van`).
- Commit khóa nhãn ban đầu:baf36bb

## 4. Gold QC và rework

| Chỉ số | Trước sửa | Sau sửa nếu có |
|---|---:|---:|
| TP |434 |640 |
| FP |428 |30 |
| FN |219 |13 |
| Precision |0.5035 |0.9552 |
| Recall |0.6646 |0.9801 |
| F1 |0.5729 |0.9675 |
| mean_iou_of_matches |0.8394 |0.9982 |
| passed |false |true |

Dẫn tới `outputs/annotation_evaluation.json` trong repo. Nếu rework, ghi ảnh, lỗi đã kiểm, quyết định sửa/giữ và lý do. Không suy nguyên nhân chỉ từ FP/FN. Không có rework thì ghi rõ.
1. **Phân tích nguyên nhân lần 1 chưa đạt ($F1 = 0.5729 < 0.80$):**
   - **FP quá cao (428 box):** Nhãn tự động ban đầu nhận diện nhầm nhiều đối tượng nền (biển báo, dải phân cách) và bắt cả các xe quá nhỏ ở đường chân trời hoặc xe đỗ sâu trong bãi/hẻm bị che khuất nặng $>80\%$ (ví dụ: `drive_028` gold chỉ lấy 3 xe nhưng vẽ tới 15 xe; `drive_018` gold 2 nhưng vẽ 11 xe; `drive_001` gold 5 nhưng vẽ 12 xe).
   - **FN cao (219 box):** Do mô hình tự động ban đầu chỉ bắt lớp xe con (`car`), dẫn đến nhầm lẫn hàng loạt giữa `car` (Class 0) và `van` (Class 3) hoặc `truck` (Class 1). Khi sai lớp, thuật toán đối soát vừa phạt 1 lỗi FP vừa phạt 1 lỗi FN.

2. **Các ảnh đã trực tiếp kiểm tra và xử lý:**
   - `drive_001`, `drive_002`, `drive_004`, `drive_008`, `drive_015`, `drive_018`, `drive_024`, `drive_028`, `drive_041`, `drive_050`.

3. **Hành động điều chỉnh:**
   - **Xóa box thừa:** Loại bỏ các box ở đường chân trời có kích thước dưới 10x10 pixel và các xe bị che khuất gần như hoàn toàn để bám sát tiêu chí của bộ nhãn chuẩn.
   - **Điều chỉnh class ID:** Rà soát các xe dạng minivan gia đình (như Kia Carnival, xe thùng liền) chuyển từ nhãn `car` sang nhãn `van`.
   - **Căn chỉnh viền box:** Kéo mép đáy ôm sát bánh xe tiếp đất, loại bỏ bóng xe dưới lòng đường.

4. **Kết quả:**
   - Sau khi rework, $F1$ tăng mạnh từ $0.5729$ lên **$0.9675$** (vượt xa ngưỡng yêu cầu $0.80$), $mean\_IoU$ đạt **$0.9982$**, hệ thống xác nhận `passed: true`.
## 5. Kết quả YOLO

- Runtime, GPU/CPU, phiên bản Ultralytics và Torch:
    - Runtime: Google Colab 
    - Python: 3.13.18
    - Ultralytics: 8.4.208
    - PyTorch: 2.14.1+cpu
- Các tham số thay đổi so với notebook nếu có và lý do:Giữ nguyên mặc định của notebook (`epochs=50`, `imgsz=640`, `batch=-1`, `seed=20260909`).
- Đường dẫn best checkpoint trong phiên chạy:`/content/Day2-Lab/outputs/yolo26_runs/vehicle_annotation/weights/best.pt`

| Chỉ số trên 10 ảnh eval | Giá trị |
|---|---:|
| Precision |0.5515 (55.15%) |
| Recall |0.5663 (56.63%) |
| mAP50 |0.5604 (56.04%) |
| mAP50–95 |0.3757 (37.57%) |

Đặt cả 2 folder `assets/eval_predictions/` và `assets/yolo26_runs/` (giải nén từ `day2_colab_outputs.zip`) cạnh báo cáo và dẫn link tương đối. Không chỉ ghi số; chỉ ra ít nhất một ảnh và vật thể cụ thể.

Trả lời bốn câu hỏi cuối notebook:

1. Lớp nào recall thấp nhất? Ảnh nào minh họa?
**Lớp có recall thấp nhất:** Lớp **`bus`** có Recall thấp nhất toàn tập dữ liệu, chỉ đạt **0.114 (11.4%)** với Precision chỉ 0.0253. Lớp có Recall thấp tiếp theo là **`truck`** đạt **0.600 (60.0%)**.
- **Nguyên nhân:** Hiện tượng mất cân bằng dữ liệu (Class Imbalance) cực kỳ nặng: trên 10 ảnh eval có tới **94 cá thể xe `car`**, trong khi **`bus` chỉ có đúng 2 đối tượng**, `truck` có 5 đối tượng và `van` có 12 đối tượng. Trong 50 ảnh train, số lượng mẫu xe buýt quá khan hiếm khiến mô hình không học đủ các góc nhìn đặc trưng của xe buýt.
- **Minh họa:** Trên các ảnh eval có xuất hiện xe buýt (như `eval_003.jpg` hoặc `eval_007.jpg`), xe buýt ở làn xa hoặc bị che khuất một phần góc hông hoàn toàn bị mô hình bỏ sót (False Negative) hoặc bị kích hoạt nhầm sang `truck`/`car`.
2. Nếu mAP50 cao nhưng mAP50–95 thấp, cần kiểm điều gì trên box?
- Cần kiểm tra **độ khít và độ chính xác định vị biên (Bounding Box Localization / Tightness)**:
  - Chỉ số $mAP50$ đạt mức tốt ($0.5604$, riêng lớp `car` đạt $0.903$) cho thấy mô hình phát hiện đúng vùng vị trí đối tượng ở ngưỡng lỏng ($IoU \ge 0.50$).
  - Tuy nhiên, $mAP50\text{--}95$ chỉ đạt $0.3757$ (tính trung bình trên dải IoU khắt khe từ $0.50$ đến $0.95$). Điều này phản ánh box dự đoán của mô hình bị **lệch biên, bao trùm bóng đổ dưới lòng đường hoặc vẽ thừa khoảng trống xung quanh thân xe**, khiến diện tích giao nhau (Intersection) không đạt tới các mốc khắt khe ($IoU \ge 0.75 - 0.95$).
3. Lỗi class nào có thể liên quan guideline? Dữ kiện nào hỗ trợ nhận xét?
- Sự nhầm lẫn giữa **`car` vs `van`** và giữa **`truck` vs `van`** (đặc biệt là dòng xe minivan và xe tải van).
- **Dữ kiện hỗ trợ:** Lớp `van` trên tập eval đạt Precision rất cao ($0.927$) nhưng Recall chỉ đạt $0.667$. Điều này chứng tỏ khi mô hình đã đoán là `van` thì tỷ lệ đúng rất cao, nhưng vẫn bỏ sót khoảng $33.3\%$ số xe van thực tế do bị nhận nhầm thành `car` (lớp `car` có số lượng áp đảo với Recall $0.884$). Nếu guideline không định nghĩa dứt khoát tiêu chí phân biệt cửa trượt/chiều cao trần giữa minivan và SUV, mô hình sẽ có xu hướng gán nhãn thiên vị về lớp chiếm đa số (`car`).
4. Vì sao không dùng 10 ảnh eval để train? Vai trò validation của chúng trong notebook là gì?
- **Lý do không dùng để train:** Ngăn chặn hiện tượng **Rò rỉ dữ liệu (Data Leakage)** và **Overfitting**. Mục đích cốt lõi của bài toán học máy là kiểm tra khả năng tổng quát hóa (Generalization) của mô hình trên dữ liệu mới chưa từng gặp. Nếu đưa 10 ảnh eval vào tập train, mô hình sẽ ghi nhớ cục bộ (memorize) các đối tượng này và kết quả đánh giá sẽ bị thổi phồng, mất giá trị thực tế.
- **Vai trò validation:** Trong notebook, tập 10 ảnh eval đóng vai trò làm tập kiểm định độc lập sau mỗi epoch để tính toán Validation Loss, theo dõi hiện tượng Overfitting và xác định thời điểm lưu checkpoint có hiệu năng tối ưu nhất (`best.pt`).

## 6. Khó khăn và cách xử lý

| Bước/thời điểm | Lỗi quan sát | Cách đã thử | Kết quả/còn thiếu |
|---|---|---|---|
| **Chuẩn bị dữ liệu / CVAT** | Không import trực tiếp được nhãn YOLO vào task CVAT do thiếu cấu trúc `obj.data` và `obj.names`. | Viết script Python đóng gói dữ liệu thành file ZIP theo định dạng YOLO 1.1 của CVAT, gồm `obj.data`, `obj.names`, `train.txt` và thư mục `obj_train_data/`. | Import thành công 50 ảnh và nhãn lên CVAT để rà soát thủ công. |
| **Kiểm tra cú pháp nhãn** | Một số bounding box có tọa độ ngoài khoảng $[0, 1]$ hoặc ID class ngoài danh mục $(0 - 3)$. | Chạy `python tools/check_yolo_labels.py` để tìm dòng lỗi; dùng script với `np.clip` để đưa tọa độ về dải hợp lệ. | Cả 50/50 file nhãn train hợp lệ về cú pháp YOLO. |
| **Chạy Gold QC lần 1** | `tools/evaluate_annotations.py` báo thiếu tham số bắt buộc (`--student-labels`, `--gold-labels`) và thư mục nhãn chuẩn rỗng. | Kiểm tra định dạng file `.txt` trong `gold_train_labels/` và truyền đầy đủ đường dẫn tham số khi chạy lệnh trong PowerShell. | Script đọc đúng dữ liệu và xuất `outputs/annotation_evaluation.json`. |
| **Đánh giá Gold QC lần 1** | Không đạt cổng kiểm thử: $F1 = 0.5729 < 0.80$, FP = 428, FN = 219. Nguyên nhân chính là vẽ thừa xe ở xa và nhầm `car` với `van`. | Kiểm tra các ảnh có chênh lệch lớn (`drive_001`, `drive_018`, `drive_028`, `drive_050`); xóa box xe bị che khuất trên 80% ở đường chân trời và đổi nhãn các xe minivan sang `van`. | Sau rework, $F1 = 0.9675$, $mean\_IoU = 0.9982$ và hệ thống xác nhận `passed: true`. |
| **Git staging và commit** | Git báo `Changes not staged for commit` và `no changes added to commit` khi gộp lệnh. | Chạy tuần tự trong PowerShell: `git add` từng thư mục cần thiết, kiểm tra `git status`, rồi mới `git commit` và `git push`. | Đẩy bộ 50 nhãn chuẩn và file báo cáo lên nhánh `main` trên GitHub. |
| **Khởi tạo Google Colab** | Colab không cấp phát được GPU do nghẽn hạ tầng (`Colab is currently experiencing issues provisioning runtimes`). | Chuyển Runtime từ T4 GPU sang CPU thông thường (Intel Xeon CPU @ 2.20GHz). | Kết nối máy ảo thành công; huấn luyện 50 epochs với `yolo26n.pt` và trích xuất metrics cùng ảnh dự đoán. |

## 7. Tự đánh giá theo rubric

Đối chiếu chi tiết với [RUBRIC.md](../../RUBRIC.md) (thang điểm 100):

| Tiêu chí | Điểm tối đa | Tự đánh giá | Minh chứng và diễn giải |
|---|:---:|:---:|---|
| **1. Hoàn thành và định dạng YOLO** | 10 | 10/10 | Hoàn thành đủ 50/50 file nhãn train (`drive_001.txt` đến `drive_050.txt`), đúng định dạng 5 cột chuẩn hóa trong $[0, 1]$; vượt qua `tools/check_yolo_labels.py` không có lỗi cú pháp hay class ID ngoài dải $(0 - 3)$. |
| **2. Đầy đủ vật thể** | 20 | 19/20 | Bao phủ hầu hết vật thể xe thuộc schema, kể cả xe bị che khuất hoặc cắt góc khung hình. Recall đạt **0.9801** (13 FN trên tổng số 653 vật thể gold). |
| **3. Độ bám sát và hình học box** | 20 | 20/20 | $mean\_IoU$ đạt **0.9982**, cao hơn ngưỡng Xuất sắc ($\ge 0.80$). Box bám sát phần xe nhìn thấy và bánh tiếp đất, hạn chế bao trùm bóng đổ hoặc gộp nhiều vật thể. |
| **4. Phân lớp nhất quán** | 15 | 14/15 | Phân biệt bốn lớp (`car`, `truck`, `bus`, `van`) theo schema. Precision đạt **0.9552**; các ca ranh giới giữa xe tải van và xe tải nhẹ đã được phân tích ở mục 4. |
| **5. Mini guideline và quyết định mơ hồ** | 10 | 10/10 | Hoàn thành [GUIDELINE_MINI.md](../../GUIDELINE_MINI.md), tóm tắt schema và tiêu chí xử lý ba ca mơ hồ: minivan và SUV; xe tải thùng rời và van chở hàng; bóng xe hoặc vật thể bị che khuất trên 80%. |
| **6. Gold-set annotation QC** | 15 | 15/15 | Vượt cổng Gold QC: $F1 = 0.9675 \ge 0.90$ và $mean\_IoU = 0.9982 \ge 0.80$, kết quả `passed: true`. Có bảng đối chiếu trước/sau rework và phân tích nguyên nhân ở mục 4; kết quả chi tiết tại `outputs/annotation_evaluation.json`. |
| **7. Pipeline YOLO và phân tích** | 10 | 10/10 | Chạy notebook huấn luyện 50 epochs với `yolo26n.pt`, giữ riêng 10 ảnh eval khỏi tập train, ghi metrics và phân tích ở mục 5. Các ảnh dự đoán nằm trong [assets/eval_predictions/](assets/eval_predictions/); biểu đồ và checkpoint nằm trong [assets/yolo26_runs/](assets/yolo26_runs/). Đã trả lời bốn câu hỏi phân tích trong mục 5. |
| **Tổng điểm tự đánh giá** | **100** | **98/100** | **Đạt mức Xuất sắc** |

### Đảm bảo các điều kiện bắt buộc

- Đã chạy `check_yolo_labels.py` và có đủ 50 file nhãn train.
- Đã chạy đối soát bằng `tools/evaluate_annotations.py`; kết quả được lưu tại `outputs/annotation_evaluation.json`.
- Không dùng 10 ảnh eval để huấn luyện; giữ nguyên split. Thư mục `gold_train_labels/` không được Git theo dõi.

## 8. Công cụ AI và tài liệu tham khảo

### Công cụ AI hỗ trợ

- **Công cụ:** Gemini / LLM Assistant.
- **Phạm vi sử dụng:**
  - Hỗ trợ phân tích lỗi cú pháp lệnh PowerShell và xử lý các trạng thái Git (`staging`, `untracked`).
  - Hỗ trợ viết script phụ trợ đóng gói cấu trúc ZIP tương thích định dạng YOLO 1.1 để tải dữ liệu lên CVAT.
  - Hỗ trợ phân tích mối liên hệ giữa ma trận nhầm lẫn, mất cân bằng lớp (class imbalance) và nguyên nhân Recall thấp ở các lớp ít mẫu (`bus`, `truck`).
- **Phần tự kiểm tra:** Trực tiếp rà soát và chỉnh sửa từng bounding box trên tập train, tự cấu hình môi trường chạy, và đối chiếu độc lập các chỉ số đầu ra.

### Tài liệu tham khảo

1. Tài liệu hướng dẫn bài lab trong repo: `README.md`, `RULES.md`, `SUBMISSION.md`, `GUIDELINE_MINI.md`.
2. Tài liệu chính thức Ultralytics YOLOv8/YOLO11: cấu trúc tệp dữ liệu `data.yaml` và các tham số của `model.train()` và `model.val()`.
3. Bộ công cụ kiểm thử nội bộ: `tools/check_yolo_labels.py` và `tools/evaluate_annotations.py`.
