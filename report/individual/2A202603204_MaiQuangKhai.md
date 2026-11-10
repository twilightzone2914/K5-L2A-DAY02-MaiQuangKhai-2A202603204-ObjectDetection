# Báo cáo cá nhân — Ngày 2 Object Detection

## 1. Thông tin học viên

- Họ tên: Mai Quang Khải
- MSSV: 2A202603204
- Email: khaikhaipq69@gmail.com
- Lớp: K5-L2A
- Link repo bài làm: https://github.com/twilightzone2914/K5-L2A-DAY02-MaiQuangKhai-2A202603204-ObjectDetection.git
- Link Colab đã chạy: https://colab.research.google.com/drive/1aKnNs1A-WFMkwUDjgo\_uUludFQk7xkKb?usp=sharing

## 2. Schema và mini guideline

Tài liệu quy tắc gán nhãn chi tiết được trình bày tại [GUIDELINE\_MINI.md](../../GUIDELINE_MINI.md).

- **Cách áp dụng schema 4 lớp (`0: car`,** **`1: truck`,** **`2: bus`,** **`3: van`)**:
    - `car` (`0`): Xe con, sedan, hatchback, SUV, crossover, taxi và xe bán tải cabin nhỏ; đặc trưng bởi phần mũi/nắp ca-pô nhô rõ phía trước và trần thấp hơn `van`/`bus`.
    - `truck` (`1`): Xe tải chở hàng (thùng kín, thùng bạt, thùng lửng) và xe đầu kéo; đặc trưng bởi phần đầu cabin tách biệt rõ với thùng hàng phía sau và khung gầm tải trọng lớn.
    - `bus` (`2`): Xe buýt công cộng, xe khách, minibus chở khách; thân xe hình hộp chữ nhật dài và cao, đầu phẳng, hai bên hông có dãy nhiều ô cửa kính liên tiếp.
    - `van` (`3`): Xe van thương mại chở hàng nhẹ, minivan/MPV chở người và phương tiện thương mại đầu ngắn cải tạo chuyên dụng (như xe cứu hộ giao thông có nửa thân trước là cabin van liền khối, nửa sau gắn sàn cần cẩu kéo); thân xe hình hộp liền khối, mũi ngắn/tù hơn `car` và ngắn hơn `bus` (thường chỉ có 2–3 khoang cửa sổ hông).
- **Tóm tắt 3 ca mơ hồ đã quyết định trong** [**GUIDELINE\_MINI.md**](../../GUIDELINE_MINI.md):
    - **Ca 1 (`drive_050.jpg`) — Vật thể bị che khuất / bị cắt bởi rìa khung hình**: Xe buýt tuyến 106 và chiếc xe con màu đỏ ở tiền cảnh bị cắt ngang thân dưới bởi mép dưới ảnh (`y_max = 640`), đồng thời một số xe ở làn giữa bị xe phía trước che khuất một phần. Quyết định: Vẫn gán nhãn `bus` và `car` khi phần lộ ra $\\ge 20\\text{px} \\times 20\\text{px}$ và rõ đặc trưng lớp; chỉ vẽ box bao quanh **phần diện tích thực tế nhìn thấy được** và dừng đúng tại mép ảnh hoặc ranh giới bị che khuất.
    - **Ca 2 (`drive_020.jpg`) — Vật thể bị mờ nhòe (blurry)**: Hai chiếc xe ở làn đường giữa bị nhòe chuyển động (motion blur) do di chuyển tốc độ cao, trong khi luồng phương tiện ở góc xa phía trên đường chân trời bị mờ và rất nhỏ ($\< 20\\text{px}$). Quyết định: Giữ lại và gán nhãn `car` cho hai xe bị motion blur ở gần vì kích thước vẫn lớn ($\\ge 20\\text{px}$) và rõ phom dáng sedan/hatchback; **bỏ qua không gán nhãn** các phương tiện ở xa quá mờ có chiều rộng hoặc chiều cao $\< 20\\text{px}$.
    - **Ca 3 (`drive_006.jpg`) — Môi trường đặc biệt (mặt đường ướt gây bóng phản chiếu)**: Trời mưa làm mặt đường nhựa ướt phản chiếu bóng tối và hình ảnh thân xe rất đậm ngay dưới gầm chiếc xe trắng và xe đỏ ở tiền cảnh. Quyết định: Dừng cạnh dưới của bounding box đúng tại điểm tiếp xúc thực tế của lốp xe/cản xe với mặt đường, **loại bỏ hoàn toàn** phần bóng nước phản chiếu để tránh làm giãn chiều cao box và lệch tâm `y_center`.
- **Phần cần giảng viên làm rõ**:
    - **Sự lệch ánh xạ ID lớp giữa** **`data.yaml`** **và bộ nhãn đối chiếu (`gold_train_labels/`,** **`dataset/labels/eval/`)**: Theo `README.md` và `data.yaml`, thứ tự lớp chuẩn là `0: car, 1: truck, 2: bus, 3: van`. Tuy nhiên khi đối chiếu trực quan trên ảnh của `gold_train_labels/` và `dataset/labels/eval/`, các lớp `1, 2, 3` trong nhãn gốc của bộ dữ liệu đang được mã hóa theo thứ tự `1: van, 2: truck, 3: bus` (ví dụ xe buýt 2 tầng ở `drive_052.jpg` và xe buýt ở `drive_053.jpg` mang nhãn `3`, xe tải thùng kín ở `drive_051.jpg` mang nhãn `2`, còn xe minivan ở `drive_056.jpg` mang nhãn `1`).
    - **Quy ước ngưỡng kích thước tối thiểu cho xe ở xa trên camera CCTV**: Với góc máy CCTV treo cao nhìn dọc đại lộ, các xe ở tít xa chân trời bị suy giảm độ phân giải mạnh ($\< 15\\text{–}20\\text{px}$); cần xác nhận ngưỡng chuẩn (trong bài làm đang áp dụng ngưỡng $\\text{width} \\ge 20\\text{px}$ và $\\text{height} \\ge 20\\text{px}$ trên ảnh $640 \\times 640$).

## 3. Annotation và self-QC

- **Số file nhãn train**: `50/50` file (`dataset/labels/train/drive_001.txt` đến `drive_050.txt`), tổng cộng **744 bounding boxes**.
- **Kết quả checker và lỗi đã xử lý**:
    - Chạy lệnh `python3 tools/check_yolo_labels.py`: `ĐẠT cấu trúc: 50/50 file nhãn hợp lệ. Chờ ground truth để chấm chất lượng.`
    - Toàn bộ 50 file đều đúng định dạng 5 cột chuẩn YOLO (`class x_center y_center width height`), `class_id` thuộc `{0, 1, 2, 3}`, tọa độ nằm trong khoảng hợp lệ `[0, 1]`, không có file rỗng hoặc dòng trắng thừa.
- **Danh sách ít nhất 10 ảnh đã rà bằng mắt**: `drive_004.jpg`, `drive_006.jpg`, `drive_014.jpg`, `drive_018.jpg`, `drive_020.jpg`, `drive_028.jpg`, `drive_034.jpg`, `drive_038.jpg`, `drive_045.jpg`, `drive_046.jpg`, `drive_050.jpg`.
- **Lỗi bỏ sót/box/class phát hiện khi rà và cách sửa**:
    - **Lỗi bỏ sót vật thể (FN)**: Khi rà lại `drive_050.jpg` (Frame #5 trên CVAT), phát hiện bản nháp đầu tiên chỉ mới vẽ 1 box cho xe buýt tuyến 106 ở tiền cảnh và bỏ sót 24 xe con phía sau $\\rightarrow$ đã quét bổ sung và gán đầy đủ 25 phương tiện trên `drive_050.jpg`.
    - **Lỗi nhầm lớp (Class)**: Khi sử dụng hỗ trợ pre-annotation từ `yolo26m.pt` (huấn luyện trên COCO chỉ có `car`, `truck`, `bus`, không có lớp `van`), các xe minivan/MPV và chiếc xe cứu hộ giao thông cải tạo `公安` (xuất hiện ở `drive_006.jpg`, `drive_014.jpg`, `drive_038.jpg`, `drive_046.jpg`) bị gán nhầm thành `car` hoặc `truck` $\\rightarrow$ đã rà soát thủ công trên CVAT để chuyển các phương tiện này về đúng nhãn `van` (`class 3`).
    - **Lỗi trùng box và độ chặt box (Tightness)**: Một số phương tiện cỡ trung bị mô hình gợi ý sinh ra 2 box chồng lấp khác lớp (`car` và `truck` có $\\text{IoU} \> 0.65$) hoặc cạnh đáy box bị kéo trùm xuống vùng bóng phản chiếu trên mặt đường ướt (`drive_006.jpg`, `drive_038.jpg`) $\\rightarrow$ đã áp dụng lọc khử trùng lặp liên lớp ($\\text{IoU} \> 0.65$), loại bỏ toàn bộ box nhiễu $\< 20\\text{px}$ ở chân trời và chỉnh tay các cạnh box bám sát thân xe thật.
- **Commit khóa nhãn ban đầu**: `64f6362` (`64f63622e25adeb900801744a64890f4c60434db` — commit message: `training data`).

## 4. Gold QC và rework

| Chỉ số | Trước sửa | Sau sửa nếu có |
| --- | ---: | ---: |
| TP | 412 | — |
| FP | 332 | — |
| FN | 241 | — |
| Precision | 0.5538 | — |
| Recall | 0.6309 | — |
| F1 | 0.5898 | — |
| mean\_iou\_of\_matches | 0.8534 | — |
| passed | false | — |

Kết quả chi tiết từng ảnh được lưu tại [outputs/annotation\_evaluation.json](../../outputs/annotation_evaluation.json).

- **Phân tích kết quả kiểm định với** **`gold_train_labels/`**:
    1. **Chất lượng đóng khung hình học (Box Tightness) rất tốt**: Chỉ số `mean_iou_of_matches` đạt **0.8534** (vượt xa ngưỡng yêu cầu `0.70`), khẳng định các bounding box được vẽ rất sát biên phương tiện.
    2. **Nguyên nhân phát sinh** **`FP = 332`** **và** **`FN = 241`** **làm** **`F1 = 0.5898`**:
        - **Lệch thứ tự ID lớp giữa** **`data.yaml`** **và** **`gold_train_labels/`**: Khi đối chiếu cặp box trùng vị trí ($\\text{IoU} \\ge 0.50$) giữa nhãn học viên và `gold_train_labels/`, có tới **114 phương tiện khớp chặt vị trí nhưng lệch** **`class_id`** (bị script `evaluate_annotations.py` tính đồng thời thành `114 FP` + `114 FN`). Cụ thể: **52 xe buýt** học viên gán đúng `2: bus` theo `data.yaml` nhưng trong `gold_train_labels/` lại mang ID `3`; **28 xe van** học viên gán `3: van` nhưng trong `gold_train_labels/` mang ID `1`; và **9 xe tải** học viên gán `1: truck` nhưng trong `gold_train_labels/` mang ID `2`.
        - **Khác biệt về phạm vi gán xe đỗ nền và xe rất nhỏ ở xa**: Trên các ảnh có bãi đỗ phụ hoặc gầm cầu vượt như `drive_007.jpg` (`student: 23, gold: 13`), `drive_018.jpg` (`student: 10, gold: 2`), `drive_028.jpg` (`student: 12, gold: 3`), học viên gán đầy đủ các xe con đỗ bên lề có kích thước rõ ràng ($\\ge 20\\text{px}$), trong khi `gold_train_labels/` bỏ qua các xe đỗ này. Ngược lại, ở các ảnh đại lộ đông xe như `drive_024.jpg` (`gold: 30, student: 21`), `drive_026.jpg` (`gold: 39, student: 25`), `drive_027.jpg` (`gold: 31, student: 24`), bộ gold gán cả các cụm xe mờ $\< 20\\text{px}$ ở sát đường chân trời mà học viên đã chủ động loại bỏ theo quy tắc trong [GUIDELINE\_MINI.md](../../GUIDELINE_MINI.md).
- **Quyết định về rework**: **Không thực hiện rework đổi ngược class ID chạy theo** **`gold_train_labels/`**, mà quyết định **giữ nguyên bộ nhãn 50 ảnh tuân thủ đúng schema chính thức của** **`data.yaml`** **(`0: car, 1: truck, 2: bus, 3: van`)** để đảm bảo mô hình YOLO26n học đúng tên lớp thực tế khi huấn luyện và suy luận trên ảnh.

## 5. Kết quả YOLO

- **Runtime, GPU/CPU, phiên bản Ultralytics và Torch**: Chạy trên Google Colab GPU (`CUDA:0`), `Ultralytics 8.4.208`, `PyTorch 2.x` (bật AMP mixed precision).
- **Các tham số thay đổi so với notebook nếu có và lý do**: Giữ nguyên toàn bộ siêu tham số huấn luyện mặc định trong notebook (`model="yolo26n.pt"`, `epochs=50`, `imgsz=640`, `batch=-1`, `seed=20260909`); chỉ cập nhật biến `REPO_URL` trỏ tới repository GitHub cá nhân để Colab tự động clone nhãn.
- **Đường dẫn best checkpoint trong phiên chạy**: `/content/Day2-Lab/outputs/yolo26_runs/vehicle_annotation/weights/best.pt` (đạt fitness tốt nhất tại Epoch 42/50).

| Chỉ số trên 10 ảnh eval | Giá trị |
| --- | ---: |
| Precision | 0.4640 |
| Recall | 0.3191 |
| mAP50 | 0.3150 |
| mAP50–95 | 0.2174 |

Toàn bộ kết quả trực quan và biểu đồ huấn luyện được đặt tại hai thư mục [assets/eval\_predictions/](assets/eval_predictions/) và [assets/yolo26\_runs/](assets/yolo26_runs/) (bao gồm [results.png](assets/yolo26_runs/vehicle_annotation/results.png), [confusion\_matrix.png](assets/yolo26_runs/vehicle_eval/confusion_matrix.png), [val\_batch0\_labels.jpg](assets/yolo26_runs/vehicle_eval/val_batch0_labels.jpg) và [val\_batch0\_pred.jpg](assets/yolo26_runs/vehicle_eval/val_batch0_pred.jpg)).

- **Nhận xét chi tiết trên ảnh và vật thể cụ thể**:
    - Trên lớp chiếm đa số là `car` (94/113 vật thể của tập `eval`), mô hình đạt kết quả cao: **`Precision = 0.7299`,** **`Recall = 0.7766`,** **`mAP50 = 0.8133`,** **`mAP50–95 = 0.5425`**. Minh họa rõ nhất ở ảnh ban đêm [drive\_051.jpg](assets/eval_predictions/visuals/drive_051.jpg), mô hình phát hiện chính xác và bám rất sát biên cả 8 xe con đang lưu thông qua ngã tư (độ tự tin `0.63–0.88`, như chiếc sedan màu đỏ ở tiền cảnh đạt `car 0.88`), chỉ bỏ sót chiếc xe tải thùng kín màu trắng ở góc trên bên phải bị cột đèn giao thông che chắn một phần.
    - Ở ảnh [drive\_052.jpg](assets/eval_predictions/visuals/drive_052.jpg), mô hình phát hiện tốt chuỗi 11 xe `car` ở cả hai chiều đường (`0.46–0.88`), nhưng tại ngưỡng vẽ `conf=0.25` đã bỏ sót chiếc xe buýt 2 tầng ở giữa đường và nhận nhầm người đi xe máy/xe đạp ở sát mép trái dưới thành `car 0.36`.
    - Khi đối chiếu ghép cặp giữa nhãn tập `eval` ([val\_batch0\_labels.jpg](assets/yolo26_runs/vehicle_eval/val_batch0_labels.jpg)) và dự đoán của mô hình ([val\_batch0\_pred.jpg](assets/yolo26_runs/vehicle_eval/val_batch0_pred.jpg)) trên các ảnh [drive\_053.jpg](assets/eval_predictions/visuals/drive_053.jpg), [drive\_056.jpg](assets/eval_predictions/visuals/drive_056.jpg), [drive\_058.jpg](assets/eval_predictions/visuals/drive_058.jpg) và [drive\_060.jpg](assets/eval_predictions/visuals/drive_060.jpg), mô hình đã **dự đoán đúng ngữ nghĩa thực tế các xe buýt là** **`bus`** (ví dụ hai xe buýt trong `drive_053.jpg` được dự đoán là `bus 0.5` và `bus 0.3`; xe buýt ở đỉnh ảnh `drive_060.jpg` được dự đoán là `bus 0.8`). Tuy nhiên, do file nhãn ground-truth của tập `eval` đang gán các xe buýt này là `van` (`class 3`), phép chấm tự động coi các dự đoán đúng này là sai lớp, làm chỉ số `mAP` trung bình toàn cục bị kéo xuống `0.3150`.

### Trả lời bốn câu hỏi cuối notebook:

1. **Lớp nào recall thấp nhất? Ảnh nào minh họa?**
    - **Trả lời**: Trên bảng chấm tự động theo nhãn `eval`, hai lớp có Recall bằng `0.0000` là **`truck`** (`P = 1.0000, R = 0.0000, mAP50 = 0.3495`) và **`van`** (`P = 0.0000, R = 0.0000, mAP50 = 0.0220`), kế tiếp là **`bus`** (`R = 0.5000`), trong khi **`car`** đạt Recall cao nhất (`R = 0.7766`).
    - **Ảnh minh họa**:
        - Với phương tiện tải/buýt thực tế bị bỏ sót ở ngưỡng `conf=0.25`: Ảnh [drive\_051.jpg](assets/eval_predictions/visuals/drive_051.jpg) (chiếc xe tải thùng kín màu trắng ở góc phải trên bị hai cột đèn che khuất phần đầu) và ảnh [drive\_052.jpg](assets/eval_predictions/visuals/drive_052.jpg) (chiếc xe buýt 2 tầng ở giữa làn đường).
        - Với hiện tượng Recall bằng `0` do lệch nhãn ground-truth tập `eval`: Ảnh [drive\_053.jpg](assets/eval_predictions/visuals/drive_053.jpg) và [drive\_060.jpg](assets/eval_predictions/visuals/drive_060.jpg) trong [val\_batch0\_labels.jpg](assets/yolo26_runs/vehicle_eval/val_batch0_labels.jpg) — các xe buýt cỡ lớn được ground-truth tập `eval` gán nhãn `van`, còn mô hình dự đoán đúng là `bus` (thể hiện trên [confusion\_matrix.png](assets/yolo26_runs/vehicle_eval/confusion_matrix.png) với **9/12 mẫu** **`True = van`** **được mô hình dự đoán là** **`Predicted = bus`** và **3/5 mẫu** **`True = truck`** **được dự đoán là** **`Predicted = van`**).
2. **Nếu mAP50 cao nhưng mAP50–95 thấp, cần kiểm điều gì trên box?**
    - **Trả lời**: `mAP50` chỉ yêu cầu $\\text{IoU} \\ge 0.50$ là tính đúng (dung sai vị trí rộng), trong khi `mAP50–95` lấy trung bình trên các ngưỡng khắt khe từ `0.50` đến `0.95` (bước nhảy `0.05`). Nếu `mAP50` cao nhưng `mAP50–95` thấp (ví dụ ở lớp `car`, `mAP50 = 0.8133` nhưng `mAP50–95 = 0.5425`), cần kiểm tra lại **độ bám sát biên (tightness) và tính nhất quán hình học của bounding box**:
        - Box có bị rộng quá mức (bao gồm cả khoảng trống nền đường, bóng đổ hoặc bóng phản chiếu dưới mặt đường ướt) hay bị cắt hụt một phần đầu/đuôi xe không?
        - Cách xử lý vật thể bị che khuất (occlusion) hoặc bị cắt ở rìa ảnh giữa tập `train` và `eval` có nhất quán không (chỉ đóng khung phần nhìn thấy hay ước lượng toàn bộ thân xe)?
3. **Lỗi class nào có thể liên quan guideline? Dữ kiện nào hỗ trợ nhận xét?**
    - **Trả lời**:
        - **Thứ nhất (quan trọng nhất)**: Sự nhầm lẫn chéo có hệ thống giữa 3 lớp `truck`, `bus`, `van` khi đánh giá trên `eval`. Dữ kiện hỗ trợ từ [confusion\_matrix.png](assets/yolo26_runs/vehicle_eval/confusion_matrix.png) và [val\_batch0\_labels.jpg](assets/yolo26_runs/vehicle_eval/val_batch0_labels.jpg): Có `9/12` vật thể mang nhãn `van` trong `eval` được mô hình dự đoán là `bus`, và `3/5` vật thể mang nhãn `truck` trong `eval` được dự đoán là `van` (cùng `2/5` dự đoán là `car`), bởi vì học viên gán nhãn theo đúng bảng `names` trong `data.yaml` (`1: truck, 2: bus, 3: van`) còn nhãn có sẵn của tập `eval` bị hoán vị ID thành `1: van, 2: truck, 3: bus`.
        - **Thứ hai**: Ranh giới hình thái giữa **`car`** **(SUV/crossover cỡ lớn) và** **`van`** **(minivan/MPV)** trên ảnh CCTV độ phân giải $640 \\times 640$. Trong tập train, số lượng mẫu `car` áp đảo so với `van`, đồng thời khi nhìn từ góc cao phía trước/sau, các mẫu MPV 7 chỗ và SUV có tỷ lệ khung hình rất gần nhau khiến mô hình có xu hướng thiên lệch dự đoán về lớp `car` (`2/5` mẫu minivan trong `eval` được dự đoán là `car`).
4. **Vì sao không dùng 10 ảnh eval để train? Vai trò validation của chúng trong notebook là gì?**
    - **Trả lời**:
        - **Không dùng 10 ảnh** **`eval`** **để train** nhằm ngăn chặn hoàn toàn hiện tượng **rò rỉ dữ liệu (data leakage)**. Với tập dữ liệu nhỏ (50 ảnh train), nếu đưa 10 ảnh `eval` vào quá trình cập nhật trọng số (gradient descent), mô hình sẽ ghi nhớ (overfit) trực tiếp bối cảnh và vị trí phương tiện của 10 ảnh này, làm các chỉ số `Precision`, `Recall`, `mAP` mất khả năng phản ánh năng lực tổng quát hóa (generalization) trên dữ liệu chưa từng gặp.
        - **Vai trò validation của 10 ảnh** **`eval`** **trong notebook**: Được cố định làm tập kiểm định độc lập (`val: images/eval` trong `data.yaml`) để: (1) theo dõi sự hội tụ của hàm mất mát (`val/box_loss`, `val/cls_loss`) sau mỗi epoch nhằm chọn ra checkpoint tốt nhất (`best.pt` tại Epoch 42) mà không bị overfit vào tập train; và (2) làm thước đo chuẩn cố định để đánh giá chất lượng bộ nhãn của học viên thông qua hiệu năng thực tế của mô hình phát hiện vật thể.

## 6. Khó khăn và cách xử lý

| Bước/thời điểm | Lỗi quan sát | Cách đã thử | Kết quả/còn thiếu |
| --- | --- | --- | --- |
| Gán nhãn trên CVAT (Task #1) | Thứ tự frame `0..49` trong CVAT không xếp theo thứ tự bảng chữ cái `drive_001.jpg`..`drive_050.jpg` (ví dụ Frame #0 là `drive_020.jpg`, Frame #5 là `drive_050.jpg`). | Truy xuất metadata `/api/tasks/1/data/meta` để ánh xạ chính xác chỉ số `frame_id` sang tên file `drive_xxx.jpg` khi đẩy pre-annotation và xuất nhãn. | Ánh xạ chính xác 100% cả 50/50 ảnh sang đúng file `drive_001.txt`–`drive_050.txt`. |
| Chạy pre-annotation với `yolo26m.pt` trên GPU local (RTX 3050 4GB VRAM) | Lỗi `torch.OutOfMemoryError` ở bước warmup khi truyền danh sách 50 đường dẫn ảnh vào `model.predict(source=list_of_paths)` do Ultralytics xếp chồng cả 50 ảnh thành một tensor `(50, 3, 640, 640)`. | Chuyển sang truyền đường dẫn thư mục `source='dataset/images/train'` kèm `batch=1, stream=True` để xử lý tuần tự từng ảnh. | Chạy hoàn tất 50 ảnh trong \~7 giây trên GPU mà không bị tràn VRAM. |
| Khử nhiễu pre-annotation và bổ sung lớp `van` | `yolo26m.pt` (COCO) không có lớp `van` và là kiến trúc NMS-free nên đôi khi sinh 2 box trùng nhau khác lớp (`car` và `truck`) trên cùng một xe; riêng `drive_050.jpg` ban đầu mới gán 1 box `bus` nên bị script bỏ qua. | Viết bộ lọc hậu xử lý loại bỏ mọi box $\< 20\\text{px}$, khử trùng lặp liên lớp với $\\text{IoU} \> 0.65$, quét bổ sung 24 box cho `drive_050.jpg` và rà soát thủ công trên CVAT để gán lại nhãn `van` (bao gồm xe cứu hộ `公安`). | Thu được 744 box sạch trên 50 file, đạt `50/50` qua `check_yolo_labels.py`. |
| Đánh giá Gold QC (`CP5`) và đánh giá YOLO trên tập `eval` (`CP6`) | `mean_iou_of_matches` đạt rất cao (`0.8534`), nhưng `F1` đạt `0.5898` và trên tập `eval` mô hình dự đoán đúng `bus` nhưng bị chấm thành sai lớp. | Phân tích ma trận nhầm lẫn và trực quan hóa nhãn `gold_train_labels/` cùng `[val_batch0_labels.jpg](assets/yolo26_runs/vehicle_eval/val_batch0_labels.jpg)`, phát hiện các lớp `1, 2, 3` trong nhãn gốc bị hoán vị (`1=van, 2=truck, 3=bus`) so với `data.yaml` (`1=truck, 2=bus, 3=van`). | Giữ nguyên nhãn của học viên theo đúng `data.yaml` và giải trình chi tiết bằng số liệu/hình ảnh trong Mục 4 và Mục 5. |

## 7. Tự đánh giá theo rubric

Đối chiếu với [RUBRIC.md](../../RUBRIC.md):

- **1. Mini guideline và hiểu schema (20 điểm)**: **Đã hoàn thành đầy đủ**. Đã định nghĩa rõ 4 lớp kèm điểm phân biệt trong [GUIDELINE\_MINI.md](../../GUIDELINE_MINI.md), nêu đủ 4 quy tắc đóng khung chung (bao gồm ngưỡng $\\ge 20\\text{px}$ và loại bỏ bóng nước phản chiếu) và ghi lại 3 ca mơ hồ tách biệt có mã ảnh cụ thể (`drive_050.jpg`, `drive_020.jpg`, `drive_006.jpg`).
- **2. Độ đầy đủ và hợp lệ của nhãn YOLO (20 điểm)**: **Đã hoàn thành đầy đủ**. Có đủ `50/50` file `.txt` trong `dataset/labels/train/`, vượt qua 100% kiểm tra cú pháp của `python3 tools/check_yolo_labels.py`, và đã thực hiện self-QC bằng mắt trên hơn 10 ảnh cùng khóa nhãn ở commit `64f6362`.
- **3. Chất lượng nhãn so với gold và xử lý rework (25 điểm)**: **Đã có đầy đủ kết quả chạy và phân tích chuyên sâu**. File [outputs/annotation\_evaluation.json](../../outputs/annotation_evaluation.json) đạt `mean_iou_of_matches = 0.8534` ($\\ge 0.70$). Chỉ số `F1 = 0.5898` (`passed = false`) đã được chứng minh bằng dữ kiện định lượng là do sự hoán vị ID lớp `1, 2, 3` trong `gold_train_labels/` (114 box trùng khớp vị trí $\\text{IoU} \\ge 0.50$ nhưng khác ID lớp giữa `data.yaml` và bộ gold) cùng khác biệt tiêu chí gán xe đỗ nền/xe mờ $\< 20\\text{px}$.
- **4. Huấn luyện và đánh giá YOLO26 trên Colab (20 điểm)**: **Đã hoàn thành đầy đủ**. Đã chạy trọn vẹn notebook trên Google Colab, thu được metrics thực tế, giải nén đầy đủ cả hai thư mục [assets/eval\_predictions/](assets/eval_predictions/) và [assets/yolo26\_runs/](assets/yolo26_runs/) cạnh báo cáo, và trả lời chi tiết 4 câu hỏi phân tích dựa trên ảnh dự đoán và confusion matrix.
- **5. Báo cáo cá nhân và tính trung thực học thuật (15 điểm)**: **Đã hoàn thành đầy đủ**. Báo cáo đổi tên đúng chuẩn `2A202603204_MaiQuangKhai.md`, ghi số liệu thật khớp 100% với JSON/CSV, không commit `gold_train_labels/` hay file trọng số `.pt` lên repository.

## 8. Công cụ AI và tài liệu tham khảo

- **Công cụ đã sử dụng**:
    - **CVAT (Computer Vision Annotation Tool) \&** **`cvat-mcp`**: Khởi tạo Task #1 (`ATGT`) tại local (`http://127.0.0.1:8080`) để vẽ, rà soát và tinh chỉnh trực quan 50 ảnh train.
    - **Ultralytics** **`yolo26m.pt`** **(Pre-annotation hỗ trợ khởi tạo box)**: Chạy suy luận gợi ý bounding box ban đầu cho 33 frame chưa gán thủ công với ngưỡng lọc kích thước $\\ge 20\\text{px} \\times 20\\text{px}$ và khử trùng lặp liên lớp $\\text{IoU} \> 0.65$.
    - **Trợ lý AI (Antigravity)**: Hỗ trợ viết script đồng bộ nhãn từ CVAT API sang định dạng YOLO `.txt`, phân tích đối chiếu ma trận nhầm lẫn giữa `data.yaml` và nhãn `eval`/`gol.`
    - **Phần học viên đã tự kiểm tra trực tiếp**: Tự tay gán nhãn các frame đầu tiên, trực tiếp duyệt từng frame trên giao diện CVAT cho toàn bộ 50 ảnh để chỉnh lại lớp `van` (xe minivan, xe cứu hộ ), co sát cạnh box loại bỏ bóng nước mặt đường ướt, xóa các box gợi ý sai và kiểm tra trực quan kết quả dự đoán trên 10 ảnh `eval`.
- **Tài liệu tham khảo ngoài repository**:
    - Tài liệu chính thức Ultralytics YOLO: https://docs.ultralytics.com/
    - Tài liệu hướng dẫn sử dụng CVAT: https://docs.cvat.ai/
