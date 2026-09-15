# Peer review — Day 3

Đây là file `REVIEW_PARTNER.md`. Reviewer chỉ ghi finding; tác giả
tự sửa bài của mình và điền closure.

| Trường           | Giá trị                                                                  |
| ---------------- | ------------------------------------------------------------------------ |
| Author           | `Nguyễn Thế Anh`                                                         |
| Reviewer         | `Chưa có tên reviewer độc lập trong workspace; rà soát evidence hiện có` |
| Pair ID          | `Chưa có`                                                                |
| CVAT version     | `Chưa có thông tin`                                                      |
| Thời điểm review | `15/09/2026; đối chiếu các file hiện có trong workspace`                 |

Evidence lock được đối chiếu từ `clip_01/manifest.json`: SHA-256 `0dd510b166969f162735346349b3bc3a51ee22b9d4da33136aefac057692ee08`, khóa lúc `2026-09-15T05:02:04Z`, 610 row, 190 frame, 8 track. Snapshot tại `evidence/pre-gold/clip_01/gt.txt` chưa có trong workspace.

## Danh sách finding

Mỗi finding bắt buộc có `frame + ID + lỗi + sửa thế nào`. Nếu chưa thống nhất,
dùng `needs-review`; không ép tác giả sửa theo cảm tính.

|   # | CVAT frame |          MOT frame |  ID | Loại lỗi       | Quan sát + rule áp dụng                                                                                                                                                         | Cách sửa đề xuất                                                                                                                         | Closure: fixed / not-a-defect / needs-review |
| --: | ---------: | -----------------: | --: | -------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------- | -------------------------------------------- |
|   1 |         80 |                 80 |   5 | entry/timeline | `eval_vs_gold.json` ghi pred track 5 có bbox từ frame 80 trong khi track tham chiếu 6 bắt đầu ở frame 83; guideline yêu cầu bắt đầu từ frame đầu tiên xác định chắc chắn là xe. | Xem lại frame 80-82; nếu chưa đủ bằng chứng xe thì xóa bbox/đẩy entry về frame 83, nếu đủ thì cập nhật gold hoặc ghi rõ lý do giữ.       | needs-review                                 |
|   2 |  50 và 149 |   50-53 và 149-151 |   4 | entry/exit     | Evaluator ghi ID4 có bbox trước khi track tham chiếu 4 xuất hiện ở 50-53 và còn bbox sau khi track tham chiếu rời khung ở 149-151.                                              | Soi liên tục frame 49-54 và 148-152; giữ bbox chạm rìa nếu xe thực sự nhìn thấy, nếu không thì sửa điểm bắt đầu/kết thúc theo guideline. | needs-review                                 |
|   3 | 133 và 169 | 133-135 và 169-171 |   8 | entry/exit     | Evaluator ghi ID8 có bbox trước khi track tham chiếu 8 xuất hiện ở 133-135 và còn bbox sau khi rời khung ở 169-171.                                                             | Kiểm tra xe nhỏ ở mép dưới/phải; chỉ giữ frame khi nhận dạng được xe bốn bánh, không để bbox treo sau khi xe rời ảnh.                    | needs-review                                 |

## Reviewer checklist

| Hạng mục                                                   | PASS / FINDING / N/A | Frame–ID–evidence                                                                                                                  |
| ---------------------------------------------------------- | -------------------- | ---------------------------------------------------------------------------------------------------------------------------------- |
| Có tối thiểu 6 track hợp lệ; chỉ gồm xe bốn bánh           | PASS                 | `clip_01/gt.txt`: 8 track, 610 row; guideline chỉ cho phép vehicle. Chưa có evidence hình ảnh độc lập để xác minh class từng bbox. |
| Một xe giữ một ID; không reuse ID cho xe khác              | PASS                 | 8 ID duy nhất; `eval_vs_gold.json` ghi `IDSW=0`, không có fragmented/missed GT track.                                              |
| Occlusion ngắn giữ ID; crossing không đổi ID               | PASS                 | `eval_vs_gold.json`: `IDSW=0`; các ca chồng lấn cần tiếp tục dùng luật giữ ID theo quỹ đạo.                                        |
| Entry/exit đúng; không box treo sau khi xe rời khung       | FINDING              | Các finding 1-3: ID5, ID4 và ID8 có lệch entry/exit so với gold; cần soi video để kết luận.                                        |
| Bbox ôm phần nhìn thấy, không đoán phần bị che/ngoài khung | PASS                 | Các bbox kiểm tra được nằm trong frame; ví dụ ID4 frame 50 chạm mép phải với `x=956.95, w=3.05`, không vượt chiều rộng 960.        |
| Frame giữa hai keyframe không bị interpolation drift       | N/A                  | MOT export không lưu metadata keyframe; không thể xác minh quy trình interpolation từ file hiện có.                                |
| Export đúng MOT 1.1; frame bắt đầu từ 1; cột 2 là track ID | PASS                 | 610 row hợp lệ, frame 1-190, 9 cột mỗi row, frame đầu từ 1 và cột 2 là ID.                                                         |
| Mọi finding có cách sửa và closure do tác giả điền         | PASS                 | Cả 3 finding đều có frame, ID, cách sửa đề xuất và closure `needs-review`.                                                         |

## Self-QC attestation của reviewer

| Lượt                       | PASS / ĐÃ SỬA / NEEDS-REVIEW | Frame–ID–evidence                                                                                |
| -------------------------- | ---------------------------- | ------------------------------------------------------------------------------------------------ |
| 1 — identity/timeline      | NEEDS-REVIEW                 | Không có IDSW khi so với gold, nhưng cần soi entry sớm của ID5 tại frame 80 và các vùng ID4/ID8. |
| 2 — endpoint/scope         | NEEDS-REVIEW                 | Các lệch entry/exit của ID4 và ID8 cần xác minh bằng video, không kết luận chỉ từ evaluator.     |
| 3 — geometry/interpolation | PASS / N/A                   | MOT hợp lệ và bbox không vượt khung; metadata keyframe/interpolation không có trong export.      |

## Exit ticket

1. Finding quan trọng nhất và rule dùng để kết luận: `ID5 tại frame 80; cần xác minh rule bắt đầu track từ frame đầu tiên nhận dạng chắc chắn, vì evaluator ghi track tham chiếu bắt đầu ở frame 83.`
2. Một finding tác giả đóng là `not-a-defect`, kèm lý do (nếu có): `Chưa đóng finding nào; cả ba đều needs-review vì không có video/review partner độc lập trong workspace.`
3. Một rule cần Lab Coach làm rõ (nếu có): `Cần quy định rõ cách xử lý bbox rất nhỏ ở mép ảnh và tiêu chí phân biệt entry sớm với false positive, kèm frame mẫu.`
