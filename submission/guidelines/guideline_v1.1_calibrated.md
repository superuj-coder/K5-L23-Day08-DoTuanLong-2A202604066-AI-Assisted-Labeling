# Label contract — Phố châu Á — Bổ sung v1.1 — 2A202604066 - Đỗ Tuấn Long

> Chỉ ghi các bổ sung rút ra từ review/QC; guideline v1.0 vẫn là hợp đồng nền.

## 1. Quy tắc review để giảm bỏ sót (CAL-01)

Trước khi bật nhãn máy, quét toàn bộ ảnh và tự đếm sơ bộ đối tượng theo lớp. Sau đó bật nhãn và kiểm tra theo thứ tự: vật bị bỏ sót, vật/nhãn thừa hoặc sai lớp, cuối cùng là độ khít của box. Không dùng confidence cao làm lý do bỏ qua một vùng ảnh. Ví dụ các bẫy bị lọt trong ảnh `img_005`, `img_008`, `img_013`, `img_014`, `img_023` cho thấy cần giữ bước quét ảnh độc lập.

## 2. Mỗi xe ba bánh chỉ có một nhãn phương tiện (CAL-02)

Một phương tiện vật lý chỉ nhận một nhãn phương tiện. Xe ba bánh/tuk-tuk/xích lô thuộc `three_wheeler`, không gán thêm `car` hoặc `truck` cho chính chiếc xe đó. Người lái/người ngồi trên xe vẫn là đối tượng `person` riêng nếu nhìn thấy đủ theo guideline. Ví dụ QC: `img_003`, `img_006`, `img_014`.

## 3. Xác minh nhãn máy `three_wheeler` (CAL-03)

Không chấp nhận một box `three_wheeler` chỉ vì confidence cao. Đối chiếu hình dạng và số bánh nhìn thấy với định nghĩa lớp; nếu vật thực tế là car/truck thì sửa lớp, nếu không có vật phù hợp thì xóa box. Trong review batch, `three_wheeler` có 15 box deleted và 5 edited; đây là tín hiệu cần kiểm tra lớp kỹ hơn, không phải bằng chứng rằng mọi box bị xóa đều sai lớp. Ví dụ bẫy box loose bị lọt: `img_023`.

## 4. Câu hỏi còn mở

Các câu hỏi còn lại trong mục 6 của guideline v1.0 (xe điện, cụm xe dày, hàng nhô ra, người trong xe có mui, kính xe, box chồng lấn, ngưỡng confidence) chưa được kiểm chứng bằng ca cụ thể trong dữ liệu lần này; tiếp tục đánh dấu `can_review` khi gặp và không tự đặt quy tắc mới.

## 5. Lịch sử thay đổi

| Phiên bản | Thay đổi | Lý do |
|---|---|---|
| v1.0 | Bản guideline gốc | — |
| v1.1 | Thêm bước quét toàn ảnh trước nhãn máy; nhắc mỗi xe ba bánh chỉ có một nhãn phương tiện; yêu cầu xác minh thủ công nhãn `three_wheeler`. | CAL-01, CAL-02, CAL-03; review stats và QC |
