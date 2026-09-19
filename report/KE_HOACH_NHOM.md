# Kế Hoạch Nhóm 3 Người — Lab 7: Embedding & Vector Store

## 1. Mục tiêu chung

**Lớp:** K4-L3B  
**Chủ đề:** Quy trình Trả hàng/Hoàn tiền trên Shopee Việt Nam dành cho Người mua và Người bán.

Nhóm cần hoàn thành hai phần song song:

1. **Phần cá nhân (60 điểm/người):** mỗi thành viên tự hoàn thiện code trong `src/`, chạy test, chạy benchmark bằng chiến lược riêng và viết `REPORT_CANHAN.md`.
2. **Phần nhóm (40 điểm):** cùng xây dựng corpus 5–10 tài liệu, thống nhất 5 benchmark query, so sánh ba chiến lược và hoàn thiện `REPORT_NHOM.md`.

### Sản phẩm cuối cùng

- [ ] Toàn bộ test chạy thành công bằng `pytest tests/ -v` trên code của từng người.
- [ ] Có 5–10 tài liệu sạch trong `data/shopee-return-refund/`.
- [ ] Mỗi tài liệu có metadata đúng chuẩn L3B.
- [ ] `sources.csv` khớp một-một với các file tài liệu.
- [ ] Có đúng 5 benchmark query và 5 gold answer kiểm chứng được từ corpus.
- [ ] Ít nhất một query cần lọc `audience: buyer` hoặc `audience: seller`.
- [ ] Ba thành viên dùng ba chiến lược chunking khác nhau.
- [ ] Có ít nhất một chiến lược chunk theo heading/section.
- [ ] Hoàn thiện một `REPORT_NHOM.md` chung.
- [ ] Mỗi thành viên hoàn thiện một `REPORT_CANHAN.md` riêng.
- [ ] Chuẩn bị nội dung demo và phân công người trình bày.

---

## 2. Thành viên và vai trò

Điền tên thật trước khi bắt đầu:

| Thành viên | Họ tên | Vai trò điều phối | Chiến lược riêng |
|---|---|---|---|
| Người 1 | `[Điền tên]` | R1 — Data Lead | `FixedSizeChunker` có overlap |
| Người 2 | `[Điền tên]` | R2 — Benchmark Lead | `RecursiveChunker` |
| Người 3 | `[Điền tên]` | R3 — Strategy Lead | Custom chunker theo heading/section |

> Vai trò R1/R2/R3 là trách nhiệm điều phối thêm. Cả ba người vẫn phải tự hoàn thiện toàn bộ phần code cá nhân trong `src/` và tự chạy 5 benchmark query.

---

## 3. Quy trình Git của nhóm

### Nhánh làm việc

- `main`: chỉ chứa dữ liệu chung, báo cáo nhóm và các nội dung đã được cả nhóm duyệt.
- `member-1`: code và báo cáo cá nhân của Người 1.
- `member-2`: code và báo cáo cá nhân của Người 2.
- `member-3`: code và báo cáo cá nhân của Người 3.

Mỗi thành viên tạo nhánh riêng:

```bash
git switch -c member-1
git push -u origin member-1
```

Thay `member-1` bằng nhánh tương ứng. Không push trực tiếp lên `main`; dùng Pull Request để đưa dữ liệu và báo cáo nhóm vào `main`.

### Quy ước tránh xung đột

- R1 là người merge thay đổi trong `data/shopee-return-refund/` và `sources.csv`.
- R2 là người merge bảng benchmark trong `REPORT_NHOM.md`.
- R3 là người merge bảng so sánh chiến lược và kết quả baseline.
- Không merge file `src/` cá nhân của người này đè lên code của người khác.
- Trước khi làm việc, chạy `git pull`; trước khi push, kiểm tra `git status` và chạy test.
- Không commit `.env`, API key, dữ liệu cá nhân hoặc nội dung sau đăng nhập.

---

## 4. Phân công chi tiết

## Người 1 — R1 Data Lead

### Trách nhiệm nhóm

- Tạo `data/urls.csv` từ `scripts/urls.example.csv`.
- Tạo và quản lý thư mục `data/shopee-return-refund/`.
- Kiểm tra `robots.txt` qua crawler mẫu; không vượt CAPTCHA hoặc giới hạn truy cập.
- Làm sạch menu, footer, banner, nội dung quảng cáo và phần không liên quan.
- Kiểm tra metadata của tất cả tài liệu.
- Tạo và giữ `sources.csv` khớp một-một với các file `.md`.
- Điền Data Inventory và Metadata Schema trong `REPORT_NHOM.md`.

### Hai tài liệu phụ trách ban đầu

1. Những quy định chung về Trả hàng/Hoàn tiền:  
   `https://help.shopee.vn/portal/4/article/188931`
2. Hướng dẫn gửi yêu cầu Trả hàng/Hoàn tiền:  
   `https://help.shopee.vn/portal/4/article/79233`

Metadata gợi ý:

```yaml
audience: buyer
category: return-conditions
language: vi
```

```yaml
audience: buyer
category: return-request-process
language: vi
```

### Chiến lược riêng

- Dùng `FixedSizeChunker`.
- Thử tối thiểu hai cấu hình, ví dụ:
  - `chunk_size=500`, `overlap=50`
  - `chunk_size=700`, `overlap=100`
- Chọn cấu hình tốt hơn dựa trên độ mạch lạc và kết quả top-3.

### Điều kiện hoàn thành

- [ ] Tài liệu đã làm sạch và đọc lại bằng mắt.
- [ ] Metadata đủ và đúng nội dung.
- [ ] `sources.csv` không thiếu hoặc thừa dòng.
- [ ] Có bảng kiểm kê tài liệu trong báo cáo nhóm.
- [ ] Có kết quả benchmark cho cấu hình FixedSize đã chọn.

---

## Người 2 — R2 Benchmark Lead

### Trách nhiệm nhóm

- Viết đúng 5 benchmark query dùng chung.
- Viết gold answer ngắn gọn, không suy đoán ngoài corpus.
- Ghi `doc_id` hoặc chunk chứa căn cứ của từng gold answer.
- Đảm bảo câu hỏi đa dạng: điều kiện, thời hạn, quy trình, bằng chứng và nghĩa vụ.
- Thiết kế ít nhất một query cần metadata filter.
- Thu kết quả top-3 và câu trả lời agent của cả ba thành viên.
- Chấm từng câu theo thang 0–2 điểm và tổng hợp vào `REPORT_NHOM.md`.

### Hai tài liệu phụ trách ban đầu

1. Quy trình Shopee xử lý yêu cầu Trả hàng/Hoàn tiền:  
   `https://help.shopee.vn/portal/4/article/190242`
2. Thời gian nhận tiền hoàn và cách kiểm tra:  
   `https://help.shopee.vn/portal/4/article/189473`

Metadata gợi ý:

```yaml
audience: buyer
category: dispute-process
language: vi
```

```yaml
audience: buyer
category: refund-timeline
language: vi
```

### Chiến lược riêng

- Dùng `RecursiveChunker`.
- Ghi rõ thứ tự separator và base case.
- Kiểm tra các bảng/thời hạn không bị tách khỏi tiêu đề giải thích.

### Khung 5 benchmark query cần hoàn thiện

| # | Dạng câu hỏi | Audience dự kiến | Yêu cầu |
|---|---|---|---|
| 1 | Điều kiện được trả hàng | `buyer` | Có gold answer và nguồn |
| 2 | Thời hạn gửi yêu cầu | `buyer` | Phân biệt loại sản phẩm/đơn hàng nếu nguồn có nêu |
| 3 | Bằng chứng cần cung cấp | `buyer` | Kiểm tra top-3 |
| 4 | Thời gian nhận tiền hoàn | `buyer` | Phân biệt phương thức thanh toán |
| 5 | Thời hạn hoặc nghĩa vụ phản hồi | `seller` | Bắt buộc chạy A/B có filter và không filter |

### Điều kiện hoàn thành

- [ ] Cả 5 gold answer đều trích được từ corpus.
- [ ] Không dùng kiến thức ngoài tài liệu để viết gold answer.
- [ ] Có ít nhất một truy vấn lọc `audience: seller`.
- [ ] Có bảng điểm của cả ba chiến lược.
- [ ] Có ít nhất một failure case và giải thích nguyên nhân.

---

## Người 3 — R3 Strategy Lead

### Trách nhiệm nhóm

- Chạy `ChunkingStrategyComparator().compare()` trên 2–3 tài liệu.
- Tổng hợp số chunk, độ dài trung bình và đánh giá độ mạch lạc.
- Viết custom chunker chia theo heading/section.
- Kiểm tra chiến lược của ba thành viên không trùng nhau.
- Tổng hợp ưu, nhược điểm và giải thích chiến lược tốt nhất.
- Chuẩn bị luồng demo retrieval cho nhóm.

### Hai tài liệu phụ trách ban đầu

1. Điều khoản dịch vụ Shopee Mall:  
   `https://help.shopee.vn/portal/4/article/77262`
2. Quy chế hoạt động sàn Shopee:  
   `https://help.shopee.vn/portal/4/article/77245`

Chỉ giữ các phần liên quan trực tiếp đến trả hàng, hoàn tiền, khiếu nại và nghĩa vụ người bán. Nếu một trang chứa nội dung cho cả hai đối tượng, tách thành file riêng theo audience.

Metadata gợi ý:

```yaml
audience: seller
category: seller-return-obligations
language: vi
```

```yaml
audience: seller
category: seller-rights-and-duties
language: vi
```

### Chiến lược riêng

Custom chunker ưu tiên các heading như:

- Điều kiện áp dụng
- Thời hạn phản hồi
- Quyền của người bán
- Nghĩa vụ của người bán
- Xử lý khiếu nại
- Hoàn tiền

Mỗi chunk cần giữ tiêu đề cùng nội dung bên dưới để truy vấn vẫn hiểu được ngữ cảnh.

### Điều kiện hoàn thành

- [ ] Có code custom chunker chạy được.
- [ ] Có baseline của ba chunker có sẵn.
- [ ] Có bảng so sánh ba thành viên.
- [ ] Có nhận xét vì sao heading chunking phù hợp hoặc không phù hợp.
- [ ] Có demo một query thường và một query dùng metadata filter.

---

## 5. Công việc cá nhân bắt buộc cho cả ba người

Mỗi người tự hoàn thiện các phần sau trên nhánh cá nhân:

### Code

- [ ] `SentenceChunker.chunk`
- [ ] `RecursiveChunker.chunk`
- [ ] `RecursiveChunker._split`
- [ ] `compute_similarity`
- [ ] `ChunkingStrategyComparator.compare`
- [ ] `EmbeddingStore._make_record`
- [ ] `EmbeddingStore._search_records`
- [ ] `EmbeddingStore.add_documents`
- [ ] `EmbeddingStore.search`
- [ ] `EmbeddingStore.get_collection_size`
- [ ] `EmbeddingStore.search_with_filter`
- [ ] `EmbeddingStore.delete_document`
- [ ] `KnowledgeBaseAgent.__init__`
- [ ] `KnowledgeBaseAgent.answer`

### Kiểm thử

```bash
pytest tests/ -v
```

Không chỉ dán dòng cuối. Mỗi người nên lưu toàn bộ output test để đưa vào báo cáo cá nhân.

### Báo cáo cá nhân

- [ ] Giải thích cosine similarity.
- [ ] Tính bài toán số lượng chunk.
- [ ] Giải thích hướng tiếp cận khi code.
- [ ] Dán kết quả test.
- [ ] Dự đoán và đo similarity cho 5 cặp câu.
- [ ] Chạy cùng 5 benchmark query của nhóm.
- [ ] Ghi top-1, score, độ liên quan và câu trả lời agent.
- [ ] Nêu điều học được từ chiến lược của thành viên khác.

---

## 6. Chuẩn dữ liệu thống nhất

Mỗi file `.md` phải có front matter tối thiểu:

```yaml
---
doc_id: ten-tai-lieu-khong-dau
title: Tên tài liệu
source_url: https://help.shopee.vn/...
retrieved_at: YYYY-MM-DD
document_version: "not-stated"
audience: buyer
category: return-conditions
language: vi
---
```

Quy tắc:

- `doc_id` duy nhất và nên trùng tên file.
- `audience` chỉ dùng `buyer`, `seller` hoặc `both`.
- Ngoài `audience`, phải có ít nhất một trường lọc hữu ích.
- Không bịa `document_version`; không thấy thì dùng `not-stated`.
- Chỉ phần dưới front matter được dùng làm `Document.content`.
- Nếu một trang chứa quy định buyer và seller khác nhau, tách thành hai file.
- Không đưa PDF/HTML thô, menu, quảng cáo hoặc footer vào corpus.
- Không dùng trang có tiêu đề “bản trước đây” làm căn cứ cho chính sách hiện hành.

### Header của `sources.csv`

```csv
doc_id,file_path,title,source_url,retrieved_at,document_version,license_or_permission
```

---

## 7. Các checkpoint thực hiện

## Checkpoint 1 — Khởi động

- [ ] Điền tên ba thành viên và tạo ba nhánh.
- [ ] Cài dependency bằng Python 3.11.
- [ ] Chạy test ban đầu và lưu kết quả.
- [ ] Chốt sáu URL dự kiến.
- [ ] Tạo `data/urls.csv`.

## Checkpoint 2 — Dữ liệu

- [ ] Có ít nhất 5 file `.md` sạch.
- [ ] Mọi file đủ metadata.
- [ ] Có cả `buyer` và `seller`.
- [ ] `sources.csv` khớp một-một.
- [ ] R1 đã review nội dung và metadata.

Ưu tiên đủ 5 tài liệu chất lượng trước; chỉ tăng lên 6–10 nếu còn thời gian.

## Checkpoint 3 — Code cá nhân

- [ ] Mỗi người đã hoàn thành TODO trong `src/`.
- [ ] Mỗi người chạy `pytest tests/ -v`.
- [ ] Không còn `NotImplementedError` trong luồng bắt buộc.

## Checkpoint 4 — Thiết kế chiến lược

- [ ] R3 có baseline trên 2–3 tài liệu.
- [ ] Ba người chốt tham số/thuật toán riêng.
- [ ] Custom heading chunker chạy được.

## Checkpoint 5 — Benchmark

- [ ] R2 chốt 5 query và gold answer.
- [ ] Cả ba chạy đúng cùng corpus và cùng 5 query.
- [ ] Lưu top-3, score và agent answer.
- [ ] Chạy A/B filtered và unfiltered cho query seller.
- [ ] Ghi nhận ít nhất một failure case.

## Checkpoint 6 — Báo cáo và demo

- [ ] R1 hoàn thiện phần dữ liệu của báo cáo nhóm.
- [ ] R2 hoàn thiện phần benchmark và bảng điểm.
- [ ] R3 hoàn thiện phần so sánh chiến lược và demo.
- [ ] Cả nhóm review `REPORT_NHOM.md`.
- [ ] Mỗi người hoàn thiện `REPORT_CANHAN.md` riêng.
- [ ] Chạy test lần cuối trước khi nộp.

---

## 8. Kịch bản demo đề xuất

1. Giới thiệu chủ đề và corpus trong 30–45 giây.
2. Cho xem metadata `buyer` và `seller` trong hai tài liệu mẫu.
3. Trình bày ngắn ba chiến lược chunking.
4. Chạy một query dành cho buyer và hiển thị top-3.
5. Chạy query dành cho seller khi chưa filter.
6. Chạy lại với `metadata_filter={"audience": "seller"}`.
7. So sánh kết quả, nêu chiến lược tốt nhất và một failure case.
8. Kết luận điều nhóm học được về chunking và metadata.

---

## 9. Definition of Done

Bài chỉ được xem là hoàn thành khi:

- Code của từng người vượt qua test bắt buộc.
- Corpus có nguồn minh bạch, nội dung sạch và đủ metadata.
- Gold answer đều có căn cứ trong corpus.
- Kết quả benchmark có thể chạy lại được.
- Báo cáo giải thích được **tại sao** một chiến lược tốt hơn, không chỉ liệt kê điểm.
- Phân biệt rõ phần chung của nhóm và đóng góp cá nhân.
- Không có API key, dữ liệu cá nhân hoặc tài liệu không được phép chia sẻ trong Git.

