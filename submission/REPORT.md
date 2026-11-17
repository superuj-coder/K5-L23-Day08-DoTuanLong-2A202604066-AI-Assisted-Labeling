# Báo cáo Day 08 — 2A202604066 - Đỗ Tuấn Long

## 1. Model và recipe (Part 2–3)

| Lớp | Model + cỡ | Ngưỡng | Lý do (run_id trong `model_log.csv`) |
|---|---|---|---|
| person / bicycle / motorbike / car / bus / truck | YOLO26m | motorbike=0.35; các lớp khác dùng mặc định 0.3 | `r05`; ngưỡng motorbike đạt recall 100% trên 5 vật gold. Các lớp person và car không đạt recall 90% với ngưỡng ≤0.5; bicycle, bus, truck không đủ mẫu để chọn riêng. |
| three_wheeler | YOLOE-26s | 0.15 | `r06`; ngưỡng gợi ý đạt recall 100%, precision 67% trên 6 vật gold. |

- Lệnh gộp: `python scripts/merge_prelabels.py work/prelabel/yoloe_all/pre.coco.json work/prelabel/y26_all/pre.coco.json --thr "motorbike=0.35,three_wheeler=0.15" --conf 0.3 --iou 0.6 --split "batch,gold" --out work/prelabel/merged_review/pre.coco.json`
- SAM đã chọn: SAM 2.1 (`sam2.1_b.pt`), chạy qua YOLO26m + SAM trong `r04`; có một ảnh mask được ghi nhận ở `masks/tuktuk_img013.jpg`.
- Lỗi không giải quyết chỉ bằng đổi ngưỡng: nhầm lớp, ví dụ xe ba bánh bị gán thành car/truck; cần sửa lớp sau khi đối chiếu ảnh và taxonomy. Thống kê theo lớp cho thấy three_wheeler có 15 box bị xóa và 5 box chỉnh sửa trong batch, nhưng không có đối chiếu từng lỗi với ảnh trong log.
- Box điểm cao/thấp cụ thể chưa được ghi lại trong `model_log.csv`; không đủ dữ liệu để nêu ảnh ví dụ.
- Giới hạn: threshold sweep dùng 4 ảnh gold; chọn và chấm trên cùng tập nên kết quả lạc quan. Mẫu nhỏ, nhất là lớp ít gặp, khiến ngưỡng chỉ là ước lượng thô.

## 2. AI error profile (từ `qc/review_stats.md`)

| Trạng thái | Số | Tỉ lệ | Mẫu số |
|---|---:|---:|---|
| accepted | 469 | 92.0% | 510 box máy |
| edited | 8 | 1.6% | 510 box máy |
| deleted | 33 | 6.5% | 510 box máy |
| added | 12 | 2.5% | 489 đối tượng sau duyệt |

- Lớp cần chú ý nhất là `three_wheeler`: 15 box bị xóa, 5 chỉnh sửa và 1 thêm. Đây là số theo lớp, chưa phân biệt hết lỗi phân loại với box thừa.
- Trên 4 ảnh gold (24 đối tượng), máy có precision 30.1% (22 TP, 51 FP) và recall 91.7% (22 TP, 2 FN). Khoảng tin cậy 95% lần lượt 21–41% và 74–98%; kết quả in-sample do chọn ngưỡng trên cùng gold.

## 3. Thiên kiến tự động hóa và ảnh bẫy

- Bắt được 7/12 bẫy = 58.3% (KTC 95%: 32–81%), dưới ngưỡng mục tiêu 80%. Theo loại: extra 3/3, loose 2/3, wrong_class 1/3, missing 1/3. Bẫy lọt: wrong_class ở `img_005.jpg`, missing ở `img_008.jpg` và `img_013.jpg`, loose ở `img_023.jpg`, wrong_class ở `img_014.jpg`.
- Kết quả cho thấy tỉ lệ accepted cao (469/510 = 92.0%). Bẫy missing và wrong_class bị lọt phù hợp với rủi ro không quét toàn ảnh hoặc quá tin nhãn máy; nguyên nhân cụ thể chưa được ghi lại nên đây là diễn giải, không phải kết luận chắc chắn.

## 4. Máy có đáng dùng không?

- Thời gian log: gán tay gold 5 phút / 4 ảnh = 1.25 phút/ảnh; review 20 phút / 24 ảnh = 0.83 phút/ảnh. So sánh có giới hạn vì gold được gán trước, giúp làm quen cảnh và taxonomy; ngoài ra batch review có pre-label để sửa.
- Kết luận: có thể dùng pre-label để tăng tốc như bản nháp, nhưng bộ kết quả này chưa đủ cơ sở để release. Dù accepted là 92.0%, chỉ bắt được 7/12 bẫy (58.3%) và precision trên gold là 30.1% (51 FP trên 73 dự đoán ghép/không ghép theo báo cáo).
- Ba quy tắc giảm thiên kiến cho lần review sau:
  1. Ẩn toàn bộ nhãn máy và quét ảnh trước ở cả 24/24 ảnh; ghi số lượng vật ước tính theo lớp trước khi bật nhãn.
  2. Rà riêng toàn ảnh để tìm vật bị bỏ sót, sau đó mới kiểm tra nhãn và độ khít của từng box; không dùng confidence cao làm lý do bỏ qua.
  3. Sau mỗi 10 ảnh nghỉ 3–5 phút, rồi kiểm tra lại các ảnh `can_review` và mọi box nhỏ hơn 20 px theo guideline.

## 5. Mask SAM

| Ảnh | SAM | Số click | Mask gồm người lái/hàng hóa? | Lỗi quan sát | File trong `masks/` |
|---|---|---:|---|---|---|
| `img_013` | SAM 2.1 (`sam2.1_b.pt`) | 3 (2 dương, 1 âm theo lệnh đã log) | Có | Không | `tuktuk_img013.jpg` |
| `img_003` | SAM 2.1 (`sam2.1_b.pt`) | 8 (4 dương, 4 âm) | Có | Không có lỗi | `img_003.jpg` |
| `img_010` | SAM 2.1 (`sam2.1_b.pt`) | 8 (4 dương, 4 âm) | Có | Không có lỗi | `img_010.jpg` |

- Đã có đủ 3 mask. Quy tắc đối chiếu là theo phần nhìn thấy của vật, không gồm bóng, và không gộp các vật chạm nhau.
- Interactor SAM trong CVAT: chưa ghi nhận.

## 6. Calibration (Part 6)

- Đã ghi 3 quyết định: `CAL-01` (bẫy lọt/quét toàn ảnh), `CAL-02` (mỗi xe ba bánh một nhãn phương tiện, người là nhãn riêng), `CAL-03` (xác minh lớp three_wheeler thay vì dựa vào confidence).
- Đã thêm quy tắc bổ sung vào `submission/guidelines/guideline_v1.1_calibrated.md`; các câu hỏi v1.0 khác chưa gặp trong ca này vẫn được đánh dấu chưa kiểm chứng.

## 7. QC và quyết định

- Mẫu QC: 5 ảnh ngẫu nhiên, 104 đối tượng; người kiểm tra: Đỗ Tuấn Long. Nguồn: `qc/qc_sample.csv` và `qc/qc_review.csv`.
- Lỗi theo đối tượng: 4/104 = 3.8%; ảnh có lỗi: 3/5 = 60.0%. Các số này tính từ phiếu QC, không phải ước lượng chắc chắn cho toàn bộ batch.
- Lỗi quan sát được: sai lớp 3 trường hợp (`img_003`, ID 2; `img_006`, ID 3; `img_014`, ID 4) và thừa 1 trường hợp (`img_003`, ID 1). Không ghi nhận lỗi ở `img_031` (ID 5) và `img_035` (ID 6).
- Cận trên Wilson 95% của tỉ lệ ảnh lỗi là 88%; mẫu 5 ảnh quá nhỏ để chứng minh ngưỡng lỗi 2%.
- Script kiểm tra được cấu trúc COCO, lớp, kích thước box và thống kê; không xác nhận box có ôm sát vật, vật bị cả người lẫn máy bỏ sót, hay người review có quét ảnh đúng quy trình.

**Quyết định: Rework**, ngày 2026-11-17. Lý do: bắt bẫy 7/12 (58.3%), dưới mục tiêu 80%; QC ghi nhận lỗi ở 3/5 ảnh, tỉ lệ cao hơn ngưỡng 2% và có lỗi sai lớp lặp lại. Kết quả mẫu nhỏ, không chứng minh được tỉ lệ lỗi toàn batch, nhưng đủ để chưa Release.

Việc cần làm tiếp: nhờ người khác rà các ảnh QC có lỗi và các ảnh bẫy bị lọt; sửa nhãn trong CVAT nếu cần, export lại, rồi chạy lại `review_stats.py`, `build_release.py` và QC. Giữ 58.3% là kết quả bắt bẫy của vòng review đã đo; chỉ báo cáo kết quả vòng sau riêng nếu có thực hiện lại. Part 5 đã đủ số mask và người dùng đã duyệt hai mask mới.

## 8. Ghi chú cá nhân

- Có dùng ChatGPT/Codex để rà repo, giải thích thứ tự workflow; các số liệu trong báo cáo được đối chiếu với file đầu ra hiện có.
- Sử dụng model trên colab
- Cache `data/cache/`: model log có các lượt chạy YOLO26 và YOLOE trên CUDA; không thấy ghi nhận dùng cache.
