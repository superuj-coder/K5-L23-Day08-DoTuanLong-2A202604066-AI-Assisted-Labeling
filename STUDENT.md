# Thông tin học viên

- Họ và tên: Đỗ Tuấn Long
- MSHV: 2A202604066
- Lớp (L2 / L3 / khác): L2
- Tài khoản GitHub: https://github.com/superuj-coder
- Repo bài nộp (URL): https://github.com/superuj-coder/K5-L23-Day08-DoTuanLong-2A202604066-AI-Assisted-Labeling
- Commit chốt bài (hash): 0317de2ec5747aa0afca7cda20586eee430725f2
- Máy: Win 10, CPU
- CVAT (Docker): v2.82.1

## Kết quả tóm tắt (điền cuối bài; số lấy từ `submission/qc/*`, kèm mẫu số)

- Máy: accepted / edited / deleted: 469/510 = 92.0% / 8/510 = 1.6% / 33/510 = 6.5%; added: 12/489 = 2.5%.
- So với gold: precision / recall của **máy** 30.1% / 91.7% (4 ảnh, 24 đối tượng gold; kết quả in-sample).
- Bắt bẫy: 7/12 = 58.3%.
- Thời gian: gán tay 1.25 phút/ảnh (5 phút/4 ảnh); duyệt có pre-label 0.83 phút/ảnh (20 phút/24 ảnh).
- Lỗi QC: 60.0% theo ảnh (3/5), 3.8% theo đối tượng (4/104).
- **Quyết định:** Rework — bắt bẫy dưới 80% và QC thấy lỗi ở 3/5 ảnh trong mẫu nhỏ.
