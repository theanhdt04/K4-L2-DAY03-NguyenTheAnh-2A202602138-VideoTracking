# Báo cáo Ngày 3 — Tracking Annotation

Chép file này thành `reports/REPORT.md` rồi điền. Giữ nguyên các tiêu đề.

Họ tên / nhóm: `Nguyễn Thế Anh`
Ngày: `15/09/2026`

---

## 1. Quá trình gán nhãn

| Mục                               | Giá trị              |
| --------------------------------- | -------------------- |
| Công cụ                           | CVAT, export MOT 1.1 |
| Thời gian gán `clip_02` (warm-up) | `20` phút            |
| Thời gian gán `clip_01`           | `40` phút            |
| Số track đã vẽ trong `clip_01`    | 8                    |
| Số keyframe trung bình mỗi track  | trung bình 76.25     |

Ba tình huống khó nhất khi gán clip này, và bạn xử lý thế nào:

1. Xe ID1 đi sát mép trái và bị cắt ở frame 7-11; giữ bbox chạm rìa, không đoán phần ngoài ảnh.
2. Xe ID4 mới xuất hiện ở mép phải tại frame 50; bắt đầu track từ bbox xác định được và giữ phần nhìn thấy.
3. Ở frame 83, ID6 xuất hiện trong vùng gần ID4/ID5; giữ ID riêng theo chuyển động và bbox quan sát được, không nối nhầm track.

## 2. Tự kiểm và kiểm chéo

Ba lượt tua bắt được gì (lượt 1 nhìn ID, lượt 2 frame đầu/cuối, lượt 3 frame giữa):

- Lượt 1: Không có log tua thủ công; file clip_01 có 8 ID, không có ID switch trong phép chấm với gold.
- Lượt 2: Đối chiếu frame đầu/cuối bằng dữ liệu MOT; các track có bbox chạm biên được giữ đúng tại biên ảnh.
- Lượt 3: Đối chiếu frame giữa; các điểm cần soi thêm gồm frame 83 (ID6), frame 104-113 (cụm xe chồng lấn) và frame 133 (ID8 nhỏ ở phía dưới).

Kiểm chéo với: `REVIEW_PARTNER.md`; tài liệu hiện ghi 3 finding đều ở trạng thái `needs-review`, chưa có reviewer độc lập được định danh. Số lỗi bạn tìm được trong bản của bạn ấy: Chưa có dữ liệu. Số lỗi bạn ấy tìm được trong bản của bạn: 3 điểm cần review (ID5 frame 80, ID4 frame 50-53/149-151, ID8 frame 133-135/169-171).

Ca nào hai người quyết khác nhau, và luật nào còn thiếu trong `GUIDELINE_MINI.md`?

Peer review đã ghi nhận các điểm lệch entry/exit nhưng chưa kết luận đó là lỗi annotation vì chưa có reviewer độc lập và video/evidence hình ảnh kèm theo. Các luật xử lý hai xe chồng lấn, xe nhỏ/mờ, xe đứng yên và mật độ keyframe đã được bổ sung trong `GUIDELINE_MINI.md`.

## 3. Pre-gold lock và chấm trước/sau rework

| Evidence                                             | Giá trị                                                                                                           |
| ---------------------------------------------------- | ----------------------------------------------------------------------------------------------------------------- |
| SHA-256 từ `evidence/pre-gold/clip_01/manifest.json` | `0dd510b166969f162735346349b3bc3a51ee22b9d4da33136aefac057692ee08` (manifest hiện có tại `clip_01/manifest.json`) |
| Thời điểm khóa                                       | `2026-09-15T05:02:04Z` (UTC)                                                                                      |
| Số row / frame / track trước khi mở reference        | `610 / 190 / 8`                                                                                                   |

|                                    |    HOTA |    DetA |    AssA |    LocA |    IDF1 |    MOTA |    MOTP |      FP |      FN |    IDSW |
| ---------------------------------- | ------: | ------: | ------: | ------: | ------: | ------: | ------: | ------: | ------: | ------: |
| Bản pre-gold                       | Chưa có | Chưa có | Chưa có | Chưa có | Chưa có | Chưa có | Chưa có | Chưa có | Chưa có | Chưa có |
| Bản hiện tại (không có log rework) |   0.827 |   0.812 |   0.844 |   0.891 |   0.962 |   0.921 |   0.880 |      41 |       4 |       0 |

Qua cổng (`IDF1 >= 0.80`, `MOTA >= 0.75`, `MOTP >= 0.70`): **có** đối với nhãn hiện có; chưa có bằng chứng pre-gold/rework để xác nhận lịch sử thay đổi.

Sau khi đọc danh sách lỗi, bạn đã sửa cụ thể những gì? Ghi theo frame và ID:

| Loại lỗi              | Frame                                | ID      | Đã sửa thế nào                                                                                                 |
| --------------------- | ------------------------------------ | ------- | -------------------------------------------------------------------------------------------------------------- |
| Entry/exit cần review | 80, 50-53, 149-151, 133-135, 169-171 | 5, 4, 8 | Chưa sửa; giữ trạng thái `needs-review` theo `REVIEW_PARTNER.md` vì chưa có evidence hình ảnh/reviewer độc lập |
|                       |                                      |         |                                                                                                                |
|                       |                                      |         |                                                                                                                |

## 4. Kết quả model: ByteTrack control vs ReID treatment

Cấu hình từ `outputs/model_run_config.json`:

| Mục                                | Giá trị                                                                              |
| ---------------------------------- | ------------------------------------------------------------------------------------ |
| Python / ultralytics / torch / lap | `3.13.15 / 8.4.145 / 2.11.0+cu128 / 0.5.13`                                          |
| weights / hai tracker              | `yolo26n.pt / bytetrack.yaml / /content/Day3-Lab/configs/trackers/botsort-reid.yaml` |
| conf / IoU / imgsz / classes       | `0.25 / 0.70 / 960 / [2, 5, 7]`                                                      |
| device                             | `0`                                                                                  |

| So sánh                   |  HOTA |  DetA |  AssA |  LocA |  IDF1 |  MOTA |  MOTP |  FP |  FN | IDSW |
| ------------------------- | ----: | ----: | ----: | ----: | ----: | ----: | ----: | --: | --: | ---: |
| bạn vs gold               | 0.827 | 0.812 | 0.844 | 0.891 | 0.962 | 0.921 | 0.880 |  41 |   4 |    0 |
| ByteTrack control vs gold | 0.709 | 0.649 | 0.776 | 0.846 | 0.875 | 0.749 | 0.823 |  88 |  54 |    2 |
| BoT-SORT + ReID vs gold   | 0.763 | 0.711 | 0.820 | 0.872 | 0.900 | 0.792 | 0.860 |  91 |  26 |    2 |
| ReID vs bạn               | 0.782 | 0.728 | 0.841 | 0.913 | 0.883 | 0.764 | 0.906 |  85 |  57 |    2 |

## 5. Phân tích — năm câu hỏi

**1. MOTA của bạn cao hơn hay thấp hơn IDF1? Nếu MOTA cao mà IDF1 thấp thì điều đó nói gì, và vì sao MOTA không phạt nặng lỗi ID?**

MOTA của nhãn tôi là 0.921, thấp hơn IDF1 là 0.962. Đây không phải trường hợp MOTA cao nhưng IDF1 thấp. Nhãn có 41 FP, 4 FN và 0 IDSW khi so với gold; MOTA bị chi phối bởi FP/FN/IDSW và mức khớp vị trí, còn IDF1 tập trung vào các cặp identity đúng. Vì IDSW chỉ là một thành phần nhỏ trong MOTA, MOTA có thể không phạt nặng lỗi ID; ở kết quả này không có IDSW nên chênh lệch chủ yếu không đến từ lỗi ID.

**2. ByteTrack control và BoT-SORT + ReID treatment khác nhau thế nào ở IDF1, AssA và IDSW? Dẫn một frame sequence để giải thích treatment tốt hơn, tệ hơn hoặc không đổi đáng kể. Nhắc rõ đây không cô lập causal effect của ReID vì hai tracker implementation khác.**

So với ByteTrack, ReID tăng IDF1 từ 0.875 lên 0.900 và AssA từ 0.776 lên 0.820; IDSW giữ nguyên ở 2. Một chuỗi cho thấy lỗi bị dịch chuyển thay vì biến mất: ByteTrack có các switch tại frame 59 (gold ID4, track 14 -> 15) và frame 94 (gold ID5, track 23 -> 32), còn ReID có switch tại frame 87 (gold ID5, track 17 -> 18) và frame 113 (gold ID6, track 24 -> 31). Vì vậy ReID tốt hơn tổng thể về association nhưng không giải quyết mọi ca. Đây là system comparison, không cô lập causal effect của ReID vì hai run dùng hai tracker implementation khác nhau.

**3. DetA, FP và FN đổi thế nào? Lỗi còn lại là detector hay association?**

DetA tăng từ 0.649 lên 0.711. FN giảm mạnh từ 54 xuống 26, nhưng FP tăng nhẹ từ 88 lên 91. AssA cũng tăng từ 0.776 lên 0.820 và IDF1 tăng, cho thấy association được cải thiện; đồng thời FP và các ghost track còn lại cho thấy detector/track initiation vẫn có lỗi. Do đó lỗi còn lại là cả detection và association, nhưng phần thiếu bbox (FN) giảm rõ hơn phần lỗi liên kết.

**4. Một chỗ bạn đúng và ReID sai (frame, ID, vì sao):**

Frame 104 là một ví dụ model sai theo diagnostics: ReID tạo track 24, nhưng phép so `ReID vs bạn` đánh dấu đây là ghost prediction không khớp track tham chiếu nào. Nhãn của tôi vẫn giữ các track xe đã có ở vùng này, thay vì thêm track 24. Đây là bằng chứng định lượng từ evaluator, không phải kết luận dựa trên việc nhìn model như gold.

**5. Một chỗ ReID làm bạn xem lại annotation (frame, ID, vì sao), hoặc lý do evidence cho thấy model sai:**

Chưa có đủ evidence hình ảnh để khẳng định ReID đúng còn annotation cần sửa. Peer review hiện cũng chỉ ghi `needs-review` cho các điểm entry/exit của annotation. Các diagnostics ReID như frame 87 với model track 18 và frame 113 với model track 31 không tự chứng minh annotation sai; vì vậy không sửa nhãn chỉ từ output model.

## 6. Nếu phải gán thêm 10 clip nữa

Bạn sẽ sửa gì trong `GUIDELINE_MINI.md`, và đổi gì trong quy trình làm việc của mình?

Các luật còn thiếu đã được bổ sung trong `GUIDELINE_MINI.md`, gồm chồng lấn, xe nhỏ/mờ, xe đứng yên và mật độ keyframe. Quy trình hiện có manifest pre-gold với hash/thời điểm/row-frame-track; bước tiếp theo là lưu snapshot đúng đường dẫn `evidence/pre-gold/clip_01/gt.txt`, ghi log ba lượt tua và đóng 3 finding bằng `fixed` hoặc `not-a-defect`. Các lần chạy model tiếp theo sẽ giữ nguyên detector input và ghi đầy đủ cấu hình để phân biệt lỗi detector với association.

## 7. Tệp đã nộp

- [x] `annotations/clip_01/gt.txt`
- [x] `annotations/clip_02/gt.txt`
- [x] `evidence/pre-gold/clip_01/gt.txt` và `manifest.json` (manifest tương ứng hiện ở `clip_01/manifest.json`; snapshot đúng đường dẫn chưa có)
- [x] `GUIDELINE_MINI.md` đã điền
- [x] `outputs/eval_vs_gold.json`
- [x] `outputs/model_bytetrack_clip_01.txt`
- [x] `outputs/model_reid_clip_01.txt`
- [x] `outputs/model_run_config.json`
- [x] `outputs/eval_bytetrack_vs_gold.json`, `outputs/eval_reid_vs_gold.json`, `outputs/eval_reid_vs_me.json`
- [x] `REVIEW_PARTNER.md` (3 finding, đều `needs-review`)
- [x] `reports/REPORT.md` (file này)
