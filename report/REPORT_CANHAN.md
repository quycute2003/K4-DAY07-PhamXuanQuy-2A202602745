# Báo Cáo Cá Nhân — Lab 7: Embedding & Vector Store

**Họ tên:** Phạm Xuân Quý — 2A202602745
**Nhóm:** Shopee Return & Refund
**Ngày:** 2026-09-19

> **Nộp 1 bản / sinh viên.** Phần nhóm (lựa chọn tài liệu, thiết kế chiến lược, bộ câu hỏi đánh giá, demo) nộp chung 1 bản trong `REPORT_NHOM.md`. Chi tiết thang điểm: `docs/SCORING.md`.

**Tổng điểm phần cá nhân: 60** = Khởi động (5) + Hướng tiếp cận (10) + Hoàn thiện code (30) + Dự đoán độ tương tự (5) + Kết quả truy xuất của tôi (10).

---

## 1. Khởi động (Warm-up) — Cá nhân (5 điểm)

### Độ tương tự Cosine (Cosine Similarity) (Bài tập 1.1)

**Độ tương tự cosine cao (High cosine similarity) nghĩa là gì?**
> Độ tương tự cosine cao nghĩa là hai vector embedding có hướng gần nhau, cho thấy hai đoạn văn bản có ý nghĩa hoặc ngữ cảnh tương tự. Chúng không nhất thiết phải dùng cùng từ vựng.

**Ví dụ có độ tương tự CAO:**
- Câu A: Shopee cho phép khách hàng gửi yêu cầu hoàn trả trong 15 ngày.
- Câu B: Người mua có hai tuần và một ngày để đề nghị trả lại sản phẩm.
- Tại sao tương đồng: Hai câu dùng từ khác nhau nhưng cùng nói về quyền của người mua và cùng mốc thời hạn 15 ngày.

**Ví dụ có độ tương tự THẤP:**
- Câu A: Tiền hoàn về thẻ tín dụng có thể mất từ 7 đến 14 ngày làm việc.
- Câu B: Người bán phải đóng gói sản phẩm đúng quy cách trước khi giao hàng.
- Tại sao khác: Một câu nói về thời gian hoàn tiền cho người mua, còn câu kia nói về nghĩa vụ đóng gói của người bán.

**Tại sao độ tương tự cosine (cosine similarity) được ưu tiên hơn khoảng cách Euclid (Euclidean distance) cho text embeddings?**
> Cosine tập trung vào hướng của vector, tức mẫu đặc trưng ngữ nghĩa, và ít bị ảnh hưởng bởi độ lớn vector hoặc độ dài văn bản. Khoảng cách Euclid còn phụ thuộc vào độ lớn nên hai embedding cùng hướng nhưng khác chuẩn có thể bị xem là xa nhau.

### Bài toán tính toán Chunking (Bài tập 1.2)

**Tài liệu 10,000 ký tự, chunk_size=500, overlap=50. Bao nhiêu chunks?**
> Phép tính: `ceil((10.000 - 50) / (500 - 50)) = ceil(9.950 / 450) = ceil(22,11...) = 23`.
> Đáp án: **23 chunks**. Kiểm tra bằng `FixedSizeChunker` trong repo cũng trả về 23.

**Nếu độ chồng chéo (overlap) tăng lên 100, số lượng chunk thay đổi thế nào? Tại sao muốn độ chồng chéo nhiều hơn?**
> Khi overlap tăng lên 100, số chunk là `ceil((10.000 - 100) / (500 - 100)) = ceil(9.900 / 400) = 25`; kết quả chạy thực tế cũng là 25. Overlap lớn giúp giữ ngữ cảnh nằm ở ranh giới giữa hai chunk, nhưng tạo thêm dữ liệu trùng lặp, tăng số embedding và chi phí lưu trữ/tìm kiếm.

---

## 2. Hướng tiếp cận của tôi (My Approach) — Cá nhân (10 điểm)

Giải thích cách tiếp cận của bạn khi lập trình (implement) các phần chính trong gói `src`.

### Các hàm chia nhỏ (Chunking Functions)

**`SentenceChunker.chunk`** — hướng tiếp cận:
> Tôi dùng regex `(?<=[.!?])(?:[ \t]+|\r?\n+)` để tách tại khoảng trắng hoặc xuống dòng nằm sau dấu kết thúc câu, nhờ đó dấu câu vẫn được giữ lại. Các câu được strip và gom tối đa theo `max_sentences_per_chunk`; text rỗng trả về `[]`. Cách đơn giản này chưa xử lý hoàn hảo chữ viết tắt như `TS.`, `v.v.` hoặc một số biểu diễn số thập phân có khoảng trắng bất thường.

**`RecursiveChunker.chunk` / `_split`** — hướng tiếp cận:
> Thuật toán thử separator theo thứ tự từ ranh giới lớn đến nhỏ; mảnh còn quá dài tiếp tục được đệ quy với các separator còn lại, sau đó các mảnh nhỏ liền kề được gom ngược lên tới gần `chunk_size`. Ba trường hợp dừng là text rỗng, text đã không vượt `chunk_size`, và không còn separator hoặc gặp separator `""`; trường hợp cuối dùng fixed-size slicing để luôn kết thúc an toàn.

### Lớp EmbeddingStore

**`add_documents` + `search`** — hướng tiếp cận:
> `add_documents` coi mỗi `Document` là một record đã được chunk ở tầng ngoài, copy metadata, bảo đảm có `doc_id`, tạo embedding cho `content` rồi lưu in-memory. `search` tạo embedding cho query và chuyển toàn bộ ứng viên qua `_search_records`, dùng dot product để xếp hạng giảm dần; vì embedding đã chuẩn hóa nên dot product tương đương cosine similarity. Kết quả chỉ trả `id`, `content`, metadata và score, không trả vector embedding.

**`search_with_filter` + `delete_document`** — hướng tiếp cận:
> `search_with_filter` lọc metadata trên toàn bộ store trước, sau đó mới xếp hạng top-k bằng cùng helper với `search`; cách này tránh để tài liệu sai đối tượng chiếm các vị trí top-k. `delete_document` loại tất cả record có `metadata["doc_id"]` khớp tài liệu gốc, vì vậy xóa được đồng thời mọi chunk như `file#0`, `file#1`, và trả `True` chỉ khi kích thước store thực sự giảm.

### Tác tử KnowledgeBaseAgent

**`answer`** — hướng tiếp cận:
> `answer` truy xuất top-k, đánh số từng chunk `[1]`, `[2]`, `[3]` và đính kèm `title` cùng `source_url` hoặc `doc_id` trước nội dung. Prompt yêu cầu chỉ sử dụng ngữ cảnh, trích dẫn theo số nguồn và nói rõ khi tài liệu không đủ thông tin để hạn chế bịa đặt. Nếu store không trả kết quả, agent trả thông báo trực tiếp và không gọi LLM.

---

## 3. Hoàn thiện code (Core Implementation) — Cá nhân (30 điểm)

Vượt qua bộ kiểm thử là điều kiện tính điểm phần này.

### Kết Quả Kiểm Thử (Test Results)

```
============================= test session starts =============================
platform win32 -- Python 3.11.4, pytest-9.0.3
collected 42 items

TestProjectStructure                                  2 passed
TestClassBasedInterfaces                              2 passed
TestFixedSizeChunker                                  7 passed
TestSentenceChunker                                   4 passed
TestRecursiveChunker                                  4 passed
TestEmbeddingStore                                    8 passed
TestKnowledgeBaseAgent                                2 passed
TestComputeSimilarity                                 4 passed
TestCompareChunkingStrategies                         3 passed
TestEmbeddingStoreSearchWithFilter                    3 passed
TestEmbeddingStoreDeleteDocument                      3 passed

============================= 42 passed in 0.24s ==============================
```

**Số lượng bài test vượt qua (pass):** 42 / 42

---

## 4. Dự đoán độ tương tự (Similarity Predictions) — Cá nhân (5 điểm)

| Cặp | Câu A | Câu B | Dự đoán | Điểm thực tế | Đúng? |
|------|-----------|-----------|---------|--------------|-------|
| 1 | Shopee hiện chưa hỗ trợ yêu cầu đổi hàng. | Nền tảng hiện không cung cấp hình thức đổi sản phẩm. | Cao | 0,5511 | Có |
| 2 | Tiền hoàn về thẻ tín dụng mất từ 7 đến 14 ngày làm việc. | Khoản hoàn trả vào thẻ cần khoảng một đến hai tuần tùy ngân hàng. | Cao | 0,6880 | Có |
| 3 | Người bán có 2 ngày để phản hồi yêu cầu Hoàn Tiền Ngay. | Nhà bán hàng phải trả lời yêu cầu hoàn tiền tức thời trong vòng 48 giờ. | Cao | 0,6301 | Có |
| 4 | Người mua có thể gửi yêu cầu trả hàng và hoàn tiền. | Người bán phải đóng gói sản phẩm đúng quy cách trước khi giao hàng. | Thấp | 0,6170 | Không |
| 5 | Thực phẩm tươi sống phải được yêu cầu trả hàng trong 24 giờ. | Thời tiết Hà Nội hôm nay có nắng và ít mây. | Thấp | 0,0816 | Có |

**Kết quả nào bất ngờ nhất? Điều này nói gì về cách embeddings biểu diễn ý nghĩa?**
> Cặp 4 bất ngờ nhất vì hai câu nói về hai hành động khác nhau nhưng vẫn đạt 0,6170. Điều này cho thấy embedding không chỉ biểu diễn ý chính cần trả lời mà còn giữ mạnh ngữ cảnh chung như người mua, người bán, sản phẩm và giao dịch; vì vậy cosine cao chưa bảo đảm đoạn văn chứa đúng bằng chứng cần thiết.

---

## 5. Kết quả truy xuất của tôi (Competition Results) — Cá nhân (10 điểm)

Chạy **5 câu hỏi đánh giá của nhóm** trên mã nguồn cá nhân của bạn trong gói `src`. **5 câu hỏi này phải trùng với các thành viên cùng nhóm** (xem `REPORT_NHOM.md`).

| # | Câu hỏi (Query) | Top-1 Chunk truy xuất được (tóm tắt) | Điểm Score | Có liên quan không? (Relevant) | Câu trả lời của Agent (tóm tắt) |
|---|-------|--------------------------------|-------|-----------|------------------------|
| 1 | Thời hạn yêu cầu với thực phẩm tươi sống/đông lạnh? | `buyer-return-conditions#1`: nêu đúng mốc 24 giờ sau “Giao hàng thành công”. | 0,6827 | Có — đúng ngay top-1 | 24 giờ, trừ lý do chưa nhận được hàng; có căn cứ trong chunk [1]. |
| 2 | Shopee có đổi hàng không và người mua xử lý hàng có vấn đề thế nào? | `buyer-return-processing#9`: lưu ý xử lý sai sót và liên hệ CSKH, không trả lời trực tiếp việc đổi hàng. | 0,6367 | Top-1 không; chunk đúng ở top-2 | Không hỗ trợ đổi hàng; có thể từ chối khi đồng kiểm hoặc gửi yêu cầu trả hàng/hoàn tiền, dựa trên chunk top-2. |
| 3 | Có những cách nào để gửi yêu cầu trả hàng/hoàn tiền? | `buyer-return-processing#10`: nói về phản hồi đề xuất và khiếu nại, không liệt kê hai cách gửi. | 0,6444 | Không — bằng chứng vắng top-3 | Không đủ ngữ cảnh để liệt kê chính xác hai cách gửi yêu cầu. |
| 4 | Hoàn tiền về thẻ tín dụng/ghi nợ mất bao lâu? | `buyer-refund-timeline#0`: bảng phương thức và thời gian hoàn tiền, có dòng 7–14 ngày. | 0,7422 | Có — đúng ngay top-1 | 7–14 ngày làm việc, tùy ngân hàng; có căn cứ trong chunk [1]. |
| 5 | Một yêu cầu hoàn tiền cần được phản hồi trong bao lâu? | `seller-mall-return-obligations#3`: nghĩa vụ nhận sản phẩm hoàn trả trong 7 ngày, không phải phản hồi “Hoàn Tiền Ngay”. | 0,5290 | Không — đúng tài liệu nhưng sai đoạn | Không đủ ngữ cảnh để kết luận mốc 02 ngày cho “Hoàn Tiền Ngay”; các chunk top-3 chứa những mốc khác. |

**Bao nhiêu câu hỏi trả về chunk có liên quan trong top-3?** 3 / 5. Điểm theo rubric: **5/10** (câu 1: 2đ, câu 2: 1đ, câu 3: 0đ, câu 4: 2đ, câu 5: 0đ).

> Cấu hình cá nhân: `FixedSizeChunker(chunk_size=1000, overlap=500)`, 53 chunk, embedding `sentence-transformers/paraphrase-multilingual-MiniLM-L12-v2`. Câu trả lời được đánh giá theo bằng chứng thật có trong ngữ cảnh top-3; khi thiếu bằng chứng, agent phải báo tài liệu chưa đủ thay vì suy đoán.

**Điều hay nhất tôi học được từ thành viên khác / nhóm khác (qua demo):**
> Qua so sánh bốn chiến lược, tôi học được rằng HeadingChunker phù hợp nhất với tài liệu chính sách có cấu trúc mục rõ ràng. Việc giữ tiêu đề cùng điều khoản giúp truy xuất đúng ngữ cảnh hơn so với chỉ dựa vào kích thước, ranh giới câu hoặc separator chung.

---

## Tự Đánh Giá (Phần Cá Nhân)

| Tiêu chí | Điểm tự đánh giá |
|----------|-------------------|
| Khởi động (Warm-up) | 5 / 5 |
| Hướng tiếp cận của tôi (My Approach) | 10 / 10 |
| Hoàn thiện code (Core Implementation — tests) | 30 / 30 |
| Dự đoán độ tương tự (Similarity Predictions) | 5 / 5 |
| Kết quả truy xuất của tôi (Competition Results) | 5 / 10 |
| **Tổng phần cá nhân** | **55 / 60** |
