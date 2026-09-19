# Báo Cáo Nhóm — Lab 7: Embedding & Vector Store

**Nhóm:** Shopee Return & Refund
**Thành viên:** Phạm Xuân Quý (2A202602745), [Thành viên 2], [Thành viên 3]
**Ngày:** 2026-09-19

> **Nộp 1 bản / nhóm.** Phần cá nhân (hướng tiếp cận, kết quả riêng, dự đoán…) mỗi thành viên nộp riêng trong `REPORT_CANHAN.md`. Chi tiết thang điểm: `docs/SCORING.md`.

**Tổng điểm phần nhóm: 40** = Lựa chọn tài liệu (10) + Thiết kế chiến lược (15) + Chất lượng truy xuất (10) + Thuyết trình (5).

---

## 1. Lựa chọn tài liệu (Document Set Quality) — Nhóm (10 điểm)

### Chủ đề (Domain) & Lý Do Chọn

**Chủ đề:** Quy trình Trả hàng/Hoàn tiền trên Shopee Việt Nam dành cho Người mua và Người bán.

**Tại sao nhóm chọn chủ đề này?**
> Chủ đề có nhiều điều kiện, ngoại lệ, thời hạn và quy trình cụ thể nên phù hợp để đánh giá chất lượng retrieval. Việc có tài liệu dành cho cả người mua và người bán cũng giúp nhóm kiểm chứng tác dụng của metadata filtering, đặc biệt khi cùng một câu hỏi có thể truy xuất nhầm mốc thời gian của đối tượng khác.

### Danh sách tài liệu (Data Inventory)

| # | Tên tài liệu | Nguồn (Source URL) | Ngày lấy / Phiên bản | Số ký tự | Metadata đã gán |
|---|--------------|------------|--------------------|----------|-----------------|
| 1 | Quy định chung về trả hàng và hoàn tiền | [Shopee #188931](https://help.shopee.vn/portal/4/article/188931) | 2026-09-19 / `not-stated` | 6.050 | `audience: buyer`, `category: return-conditions`, `language: vi` |
| 2 | Hướng dẫn gửi yêu cầu trả hàng và hoàn tiền | [Shopee #79233](https://help.shopee.vn/portal/4/article/79233) | 2026-09-19 / `not-stated` | 2.272 | `audience: buyer`, `category: return-request-process`, `language: vi` |
| 3 | Quy trình xử lý yêu cầu trả hàng và hoàn tiền | [Shopee #190242](https://help.shopee.vn/portal/4/article/190242) | 2026-09-19 / `not-stated` | 7.846 | `audience: buyer`, `category: dispute-process`, `language: vi` |
| 4 | Thời gian nhận tiền hoàn và cách kiểm tra | [Shopee #189473](https://help.shopee.vn/portal/4/article/189473) | 2026-09-19 / `not-stated` | 3.632 | `audience: buyer`, `category: refund-timeline`, `language: vi` |
| 5 | Nghĩa vụ người bán Shopee Mall khi xử lý trả hàng | [Shopee #77262](https://help.shopee.vn/portal/4/article/77262) | 2026-09-19 / `effective-2026-05-08` | 4.101 | `audience: seller`, `category: seller-return-obligations`, `language: vi` |
| 6 | Quyền và nghĩa vụ người bán trên Shopee | [Shopee #77245](https://help.shopee.vn/portal/4/article/77245) | 2026-09-19 / `updated-2025-01-03` | 3.822 | `audience: seller`, `category: seller-rights-and-duties`, `language: vi` |

**Danh sách kiểm tra quản trị dữ liệu (Data governance checklist):**
- [x] Tập tài liệu (Corpus) chỉ chứa nguồn công khai/được phép dùng và không chứa dữ liệu cá nhân, thông tin đăng nhập hoặc tài liệu nội bộ.
- [x] Mỗi tài liệu có `source_url`, `retrieved_at`, `document_version` (hoặc ngày hiệu lực) trong metadata.

### Cấu trúc Metadata (Metadata Schema)

| Trường metadata | Kiểu | Ví dụ giá trị | Tại sao hữu ích cho truy xuất (retrieval)? |
|----------------|------|---------------|-------------------------------|
| `doc_id` | String duy nhất | `buyer-return-conditions` | Liên kết chunk với tài liệu nguồn và hỗ trợ xóa toàn bộ chunk của một tài liệu. |
| `title` | String | `Quy định chung về trả hàng và hoàn tiền` | Giúp nhận diện tài liệu và hiển thị nguồn dễ hiểu trong kết quả. |
| `source_url` | URL | `https://help.shopee.vn/portal/4/article/188931` | Cho phép truy vết và kiểm chứng nội dung tại nguồn chính thức. |
| `retrieved_at` | Ngày `YYYY-MM-DD` | `2026-09-19` | Cho biết thời điểm nhóm thu thập dữ liệu và hỗ trợ đánh giá độ mới. |
| `document_version` | String | `effective-2026-05-08` | Phân biệt phiên bản hoặc ngày hiệu lực của chính sách; dùng `not-stated` khi nguồn không nêu. |
| `audience` | Enum | `buyer`, `seller` | Lọc đúng tài liệu dành cho người mua hoặc người bán, tránh nhầm điều kiện và thời hạn. |
| `category` | String | `refund-timeline` | Thu hẹp tìm kiếm theo loại thông tin như điều kiện, quy trình, thời hạn hoặc nghĩa vụ. |
| `language` | String | `vi` | Hỗ trợ lọc theo ngôn ngữ khi corpus được mở rộng. |

---

## 2. Thiết kế chiến lược (Strategy Design) — Nhóm (15 điểm)

> Mỗi thành viên thử **một chiến lược khác nhau** trên cùng bộ tài liệu; nhóm tổng hợp và so sánh ở đây.

### Phân tích đường cơ sở (Baseline Analysis)

Chạy `ChunkingStrategyComparator().compare()` trên 2-3 tài liệu:

| Tài liệu | Chiến lược (Strategy) | Số lượng Chunk | Độ dài trung bình | Giữ được ngữ cảnh không? |
|-----------|----------|-------------|------------|-------------------|
| | FixedSizeChunker (`fixed_size`) | | | |
| | SentenceChunker (`by_sentences`) | | | |
| | RecursiveChunker (`recursive`) | | | |

### Chiến lược của từng thành viên

> Mỗi thành viên điền một khối dưới đây (copy thêm nếu nhóm có nhiều hơn 3 người).

**Thành viên 1 — [Tên]**
- **Loại chiến lược:** [FixedSize / Sentence / Recursive / custom]
- **Mô tả & lý do chọn cho chủ đề này:** *(2-3 câu)*
- **Code snippet (nếu custom):**
```python
# Dán mã nguồn (implementation) vào đây
```

**Thành viên 2 — [Tên]**
- **Loại chiến lược:**
- **Mô tả & lý do chọn:**
- **Code snippet (nếu custom):**

**Thành viên 3 — [Tên]**
- **Loại chiến lược:**
- **Mô tả & lý do chọn:**
- **Code snippet (nếu custom):**

### So Sánh Giữa Các Thành Viên

| Thành viên | Chiến lược (Strategy) | Điểm truy xuất (/10) | Điểm mạnh | Điểm yếu |
|-----------|----------|----------------------|-----------|----------|
| | | | | |
| | | | | |
| | | | | |

**Chiến lược nào tốt nhất cho chủ đề này? Tại sao?**
> *Viết 2-3 câu — đây là phần được đánh giá cao nhất (khả năng suy nghĩ & giải thích):*

---

## 3. Câu hỏi đánh giá & Chất lượng truy xuất (Retrieval Quality) — Nhóm (10 điểm)

### Câu hỏi đánh giá & Câu trả lời chuẩn (nhóm thống nhất)

> **Đúng 5 câu hỏi**, đa dạng, có thể kiểm chứng; **ít nhất 1 câu** cần lọc metadata mới trả lời tốt. Đây là bộ câu hỏi chung cho mọi thành viên chạy.

| # | Câu hỏi (Query) | Câu trả lời chuẩn (Gold Answer) | Chunk nào chứa thông tin? |
|---|-------|-------------------------------|--------------------------|
| 1 | | | |
| 2 | | | |
| 3 | | | |
| 4 | | | |
| 5 | | | |

### Tổng hợp chất lượng truy xuất của nhóm

> Cách chấm (theo `docs/SCORING.md`): **2 điểm/câu** — top-3 chứa chunk liên quan + agent trả lời đúng (2), có liên quan nhưng thiếu/không ở top-1 (1), không có trong top-3 (0).

| # | Câu hỏi | Chiến lược tốt nhất cho câu này | Có chunk liên quan trong top-3? | Ghi chú |
|---|---------|-------------------------------|-------------------------------|---------|
| 1 | | | | |
| 2 | | | | |
| 3 | | | | |
| 4 | | | | |
| 5 | | | | |

**Lọc bằng metadata có giúp ích không? Ở câu hỏi nào?**
> *Viết 2-3 câu:*

---

## 4. Thuyết trình (Demo) & Bài học nhóm — Nhóm (5 điểm)

**Những phân tích (insights) hay nhất nhóm sẽ trình bày:**
> *Liệt kê 2-3 ý:*

**Bài học rút ra khi so sánh trong nhóm:**
> *Viết 2-3 câu — cùng tài liệu nhưng chiến lược khác nhau dẫn tới khác biệt gì?*

**Nếu làm lại, nhóm sẽ thay đổi gì trong chiến lược dữ liệu (data strategy)?**
> *Viết 2-3 câu:*

---

## Tự Đánh Giá (Phần Nhóm)

| Tiêu chí | Điểm tự đánh giá |
|----------|-------------------|
| Lựa chọn tài liệu (Document Set Quality) | / 10 |
| Thiết kế chiến lược (Strategy Design) | / 15 |
| Chất lượng truy xuất (Retrieval Quality) | / 10 |
| Thuyết trình (Demo) | / 5 |
| **Tổng phần nhóm** | **/ 40** |
