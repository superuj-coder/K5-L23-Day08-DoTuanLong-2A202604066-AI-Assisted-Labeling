# Số đo review (det)

- Ảnh tính: 24 (batch,gold); box máy: 510; đối tượng sau duyệt: 489; ngưỡng accepted: IoU >= 0.9
- Đã **loại 23 đối tượng nằm quanh các bẫy** khỏi bảng này: lỗi nhân tạo không phản ánh chất lượng máy.

## 1. Bốn trạng thái (AI error profile)
| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---|---|---|
| accepted (giữ nguyên) | 469 | 469/510 = 92.0% | 510 box máy |
| edited (sửa) | 8 | 8/510 = 1.6% | 510 box máy |
| deleted (xoá) | 33 | 33/510 = 6.5% | 510 box máy |
| added (vẽ thêm) | 12 | 12/489 = 2.5% | 489 đối tượng sau duyệt |

Trong số deleted + added, **5 trường hợp là đổi lớp** (máy gán sai lớp, box đúng chỗ): đó không phải máy bỏ sót. Số vật máy **sót thật** ≈ added − đổi lớp = 7.

### Theo lớp
| Lớp | accepted | edited | deleted | added |
|---|---|---|---|---|
| person | 260 | 3 | 8 | 5 |
| bicycle | 7 | 0 | 1 | 1 |
| motorbike | 127 | 0 | 1 | 2 |
| car | 44 | 0 | 7 | 0 |
| bus | 3 | 0 | 0 | 2 |
| truck | 5 | 0 | 1 | 1 |
| three_wheeler | 23 | 5 | 15 | 1 |

## 2. So với gold (nhãn gán tay từ đầu)
Ảnh gold: 4; đối tượng gold: 24 (mẫu số của recall)
| | TP | FP | FN | Precision | Recall |
|---|---|---|---|---|---|
| Máy (pre-label) | 22 | 51 | 2 | 30.1% (KTC 95%: 21–41%) | 91.7% (KTC 95%: 74–98%) |

*Lạc quan: ngưỡng được chọn trên chính các ảnh gold này (in-sample), và chỉ có 4 ảnh. Đọc khoảng tin cậy, đừng đọc con số đơn.*


(Ảnh gold không nằm trong nhãn sau duyệt: không tính residual / new error. Đó là chủ ý: duyệt lại ảnh bạn vừa tự gán tay sẽ đo trí nhớ, không đo thiên kiến.)

## 3. Ảnh bẫy
- Bắt được 7/12 = 58.3% (KTC 95%: 32–81%) (ngưỡng đạt: 80%). Chỉ 12 bẫy nên khoảng tin cậy rất rộng: đọc cùng danh sách bẫy lọt, đừng kết luận chỉ từ %.
  - extra: 3/3
  - loose: 2/3
  - wrong_class: 1/3
  - missing: 1/3
  - lọt: img_005.jpg (wrong_class), img_008.jpg (missing), img_013.jpg (missing), img_023.jpg (loose), img_014.jpg (wrong_class)

## Gợi ý đọc số
- `deleted` cao → ngưỡng thấp hoặc ánh xạ nhãn sai; `added` cao → máy bỏ sót (xem lớp nào);
- `accepted` thấp (< 50%) → sửa mỗi box lâu hơn vẽ mới: đổi model/ngưỡng hoặc vẽ tay;
- bắt bẫy < 80% → duyệt lại lô, siết checklist (quét ảnh TRƯỚC khi nhìn nhãn máy).
