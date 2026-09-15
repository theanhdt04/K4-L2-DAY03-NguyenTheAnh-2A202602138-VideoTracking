# Mini annotation guideline — Ngày 3 (tracking)

> Điền file này **trong lúc gán nhãn**, không phải sau khi xong. Mỗi lần bạn dừng
> lại nghĩ "cái này tính sao nhỉ?" thì đó là một dòng phải ghi vào đây.
>
> Đây là tài liệu mà người gán nhãn tiếp theo sẽ đọc để làm giống bạn. Nếu hai
> người trong nhóm gán khác nhau, gần như luôn là vì file này chưa nói rõ — chứ
> không phải vì ai kém.

Nhóm / tên: `Nguyễn Thế Anh`
Clip: `clip_01`, `clip_02`

---

## 1. Phạm vi: gán cái gì, không gán cái gì

Một lớp duy nhất: **`vehicle`** — xe bốn bánh (xe con, van, xe buýt, xe tải).

| Gán                           | Không gán                                           |
| ----------------------------- | --------------------------------------------------- |
| xe con, SUV, taxi, xe bán tải | người đi bộ                                         |
| van, minivan                  | xe đạp                                              |
| xe buýt, minibus              | **xe máy / mô tô**                                  |
| xe tải, xe đầu kéo            | xe trong ảnh quảng cáo, trong gương, dưới bóng nước |

Bổ sung của nhóm (nếu có): `không`

## 2. Luật ID — phần quan trọng nhất

| Tình huống                       | Luật của nhóm                                                                                                           | Vì sao                                                                                                                                                                                                                                                                                                  |
| -------------------------------- | ----------------------------------------------------------------------------------------------------------------------- | ------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Xe bị che một phần rồi hiện lại  | giữ nguyên ID nếu bị che **dưới 25 frame** (mặc định của lab: 25 frame = 2 giây @ 12.5 fps)                             | `khớp với frame rate thực tế của clip_01 — trong khoảng đó người xem vẫn còn nhớ đây là cùng một xe, nối ID lại không phải đoán mò.`                                                                                                                                                                    |
| Xe bị che lâu hơn ngưỡng trên    | `tạo track mới, không cố nối lại ID cũ`                                                                                 | `chưa gặp ca thực tế nào vượt ngưỡng trong clip_01/clip_02 — cả 15 track (8 ở clip_01, 7 ở clip_02) đều liên tục, không đứt đoạn — nhưng để nhất quán với dòng "xe rời khung quay lại" bên dưới: quá 25 frame thì độ tin cậy vị trí/hình dạng giảm mạnh, nối nhầm ID rủi ro cao hơn lợi ích giữ ID cũ.` |
| Xe rời khung hình rồi quay lại   | mặc định: **track mới**                                                                                                 | `không có gì đảm bảo là cùng một xe khi nó rời khung rồi quay lại — nhất là khi nhiều xe cùng model/màu đi qua camera — gán lại ID cũ là đoán mò, không kiểm chứng được.`                                                                                                                               |
| Hai xe cắt nhau / chồng lên nhau | giữ ID của từng xe theo quỹ đạo trước và sau vùng chồng; nếu không đủ bằng chứng thì không đổi ID chỉ vì bbox giao nhau | ưu tiên liên tục về vị trí, kích thước và hướng chuyển động; không gán lại ID theo màu/kiểu xe đơn lẻ. Nếu một xe bị che nhưng còn nhận ra, bbox chỉ ôm phần nhìn thấy.                                                                                                                                 |

## 3. Luật bbox

| Tình huống                             | Luật của nhóm                                                                                                                                                          |
| -------------------------------------- | ---------------------------------------------------------------------------------------------------------------------------------------------------------------------- |
| Xe bị cắt bởi rìa ảnh                  | bbox chạm đúng rìa, không đoán phần ngoài ảnh                                                                                                                          |
| Xe bị xe khác che một phần             | bbox ôm phần **nhìn thấy được**                                                                                                                                        |
| Xe vừa xuất hiện, còn rất nhỏ / rất mờ | bắt đầu track từ frame đầu tiên xác định chắc chắn là xe bốn bánh; không dùng ngưỡng pixel cứng, nhưng phải có hình dạng hoặc phần xe đủ để phân biệt với vật thể khác |
| Xe đang đỗ, không di chuyển            | vẫn gán bbox ở mọi frame xe còn nhìn thấy và giữ nguyên ID; không bỏ qua chỉ vì xe không có chuyển động                                                                |
| Keyframe đặt dày ở đâu                 | đặt dày ở frame xuất hiện/rời khung, mép ảnh, che khuất, hai xe chồng lấn và lúc hình dạng/kích thước đổi nhanh; trong vùng ổn định có thể nội suy                     |

## 4. Ít nhất ba ca mơ hồ đã gặp thật

Ghi **frame cụ thể** và **ID cụ thể**, không ghi chung chung.

### Ca 1

- Clip / frame / ID: `clip_01 / 7-11 / ID1`
- Tình huống: xe đi sát mép trái và bbox bị cắt bởi rìa ảnh.
- Quyết định: giữ ID1, đặt x=0 và chỉ vẽ phần nằm trong ảnh.
- Lý do: bbox không được đoán phần ngoài ảnh; đây là đúng luật bbox chạm rìa.

### Ca 2

- Clip / frame / ID: `clip_01 / 50 / ID4`
- Tình huống: xe ID4 vừa xuất hiện từ mép phải với bbox rất hẹp.
- Quyết định: bắt đầu track tại frame đầu tiên nhận ra chắc chắn là xe và giữ bbox chạm rìa.
- Lý do: không gán phần xe còn nằm ngoài ảnh; file nhãn ghi ID4 từ frame 50.

### Ca 3

- Clip / frame / ID: `clip_01 / 83 / ID6`
- Tình huống: ID6 xuất hiện gần vùng ID4/ID5 đang cùng có mặt, dễ bị nối nhầm khi xe chồng lấn.
- Quyết định: tạo và giữ ID6 riêng, không đổi ID các xe đang có.
- Lý do: ID được quyết định theo quỹ đạo và bằng chứng của từng xe, không chỉ theo vị trí giao nhau của bbox.

## 5. Sửa gì sau khi chấm với gold và sau khi kiểm chéo

Luật nào trong file này hoá ra còn thiếu hoặc còn mơ hồ? Viết lại cho rõ:

- Sau khi chấm với gold, giữ nguyên luật không đổi ID chỉ vì hai bbox giao nhau; cần xem lại theo chuỗi frame trước/sau vùng che khuất.
- Manifest pre-gold hiện có tại `clip_01/manifest.json`, ghi SHA-256 `0dd510b166969f162735346349b3bc3a51ee22b9d4da33136aefac057692ee08`, thời điểm khóa `2026-09-15T05:02:04Z`, 610 row, 190 frame và 8 track; snapshot vật lý tại đường dẫn `evidence/pre-gold/clip_01/gt.txt` chưa có trong workspace.
- `REVIEW_PARTNER.md` đã ghi 3 finding về entry/exit của ID5, ID4 và ID8; cả 3 vẫn là `needs-review`, nên chưa ghi nhận thay đổi annotation sau rework. Các lần sau phải lưu closure `fixed`/`not-a-defect` cùng frame và ID cụ thể.
