# Tuning Log — RAG Pipeline (Day 08 Lab)

> Template: Ghi lại mỗi thay đổi và kết quả quan sát được.
> A/B Rule: Chỉ đổi MỘT biến mỗi lần.

---

## Baseline (Sprint 2)

**Ngày:** 13/04/2026  
**Config:**
```python
retrieval_mode = "dense"
chunk_size = 400 # tokens
overlap = 80 # tokens
top_k_search = 10
top_k_select = 3
use_rerank = False
llm_model = "gpt-4o-mini"
```

**Scorecard Baseline:**
| Metric | Average Score |
|--------|--------------|
| Faithfulness | 4.20 /5 |
| Answer Relevance | 4.20 /5 |
| Context Recall | 5.00 /5 |
| Completeness | 3.90 /5 |

**Câu hỏi yếu nhất (điểm thấp):**
> - q09 (ERR-403-AUTH) - Relevance = 1/5: Điểm thấp vì câu hỏi "ERR-403-AUTH là lỗi gì và cách xử lý?" thực tế không có trong dataset. Mô hình trả lời "Tôi không biết" là abstain chính xác nhưng bị chấm điểm thấp vì không đáp ứng được toàn bộ hướng dẫn trả lời hy vọng.
> - q10 (Hoàn tiền VIP) - Relevance = 1/5, Completeness = 2/5: Trả lời thiếu chi tiết về thời gian vì kỳ vọng có quy trình VIP (nhưng thực tế là standard).
> - q04 (Refund) - Completeness = 3/5: Bỏ sót thông tin đặc thù đối với sản phẩm số lượng và license.

**Giả thuyết nguyên nhân (Error Tree):**
- [ ] Indexing: Chunking cắt giữa điều khoản
- [ ] Indexing: Metadata thiếu effective_date
- [x] Retrieval: Dense bỏ lỡ exact keyword / alias
- [ ] Retrieval: Top-k quá ít → thiếu evidence
- [x] Generation: Prompt không đủ grounding
- [ ] Generation: Context quá dài → lost in the middle

---

## Variant 1 (Sprint 3)

**Ngày:** 13/04/2026  
**Biến thay đổi:** `retrieval_mode = "hybrid"` và `use_rerank = True`  
**Lý do chọn biến này:**
> Từ baseline, có thể thấy Dense gặp khó với các từ khóa khắt khe và các trường hợp ngầm định thông tin (như khách VIP áp dụng luật như thường). Việc bổ sung tìm kiếm Sparse (BM25) vào Hybrid Search giúp cải thiện khả năng chộp được keyword, trong khi Cross-encoder Reranking sẽ đánh giá lại mức độ liên quan ngữ cảnh chặt chẽ hơn trước khi chọn 3 chunk để đưa vào LLM Context.

**Config thay đổi:**
```python
retrieval_mode = "hybrid"
use_rerank = True
# Các tham số còn lại giữ nguyên như baseline
```

**Scorecard Variant 1:**
| Metric | Baseline | Variant 1 | Delta |
|--------|----------|-----------|-------|
| Faithfulness | 4.20/5 | 4.10/5 | -0.10 |
| Answer Relevance | 4.20/5 | 4.40/5 | +0.20 |
| Context Recall | 5.00/5 | 5.00/5 | 0.00 |
| Completeness | 3.90/5 | 3.90/5 | 0.00 |

**Nhận xét:**
> Variant 1 có sự cải thiện rõ ở Relevance (từ 4.20 lên 4.40) lớn nhất ghi nhận được ở q10 nhờ LLM nắm rõ hơn việc không có ngoại lệ và quy định quy trình hoàn tiền hiện tại ra sao. Tuy nhiên, Faithfulness có phần giảm xuống (4.20 xuống 4.10) do mô hình đưa ra suy luận lỏng lẻo thay vì bám sát thông tin Grounded (ví dụ ở q04).

**Kết luận:**
> Variant 1 có tốt hơn baseline chung về mặt ý nghĩa câu trả lời (Relevance), tuy nhiên việc Faithfulness giảm cho thấy khi chunking mix tạp từ hybrid, generation prompt vẫn còn dễ bị đánh lừa. Có thể áp dụng Variant 1, nhưng nên cải thiện thêm Generation Prompt.

---

## Variant 2 (nếu có thời gian)

**Biến thay đổi:** ___________  
**Config:**
```
# TODO
```

**Scorecard Variant 2:**
| Metric | Baseline | Variant 1 | Variant 2 | Best |
|--------|----------|-----------|-----------|------|
| Faithfulness | ? | ? | ? | ? |
| Answer Relevance | ? | ? | ? | ? |
| Context Recall | ? | ? | ? | ? |
| Completeness | ? | ? | ? | ? |

---

## Tóm tắt học được

> TODO (Sprint 4): Điền sau khi hoàn thành evaluation.

1. **Lỗi phổ biến nhất trong pipeline này là gì?**
   > Lỗi phổ biến xảy ra khi hệ thống đối mặt với truy vấn ngoài luồng (như câu "ERR-403-AUTH là lỗi gì và cách xử lý?" hoàn toàn không có trong dataset), lúc này mô hình sẽ đáp án "Tôi không biết" một cách cứng nhắc thay vì lót thêm lời khuyên. Hoặc khi context đủ, generation vẫn bỏ lỡ điểm cụ thể (ví dụ: license product exception) làm giảm Completeness và Faithfulness.

2. **Biến nào có tác động lớn nhất tới chất lượng?**
   > `retrieval_mode` kết hợp cùng module `use_rerank`. Cải thiện khâu retrieve chính là yếu tố làm đòn bẩy lớn nhất cho kết quả câu trả lời do mô hình gpt-4o-mini bị phụ thuộc rất lớn vào context chất lượng.

3. **Nếu có thêm 1 giờ, nhóm sẽ thử gì tiếp theo?**
   > Cải thiện Indexer Pipeline: extract keyword tốt hơn (như tên mã lỗi, alias văn bản) gắn vào metadata. Sau đó thử nghiệm tính năng Query Transform (ví dụ: HyDE) để xem LLM có thể định hướng tốt hơn cho câu hỏi nghèo nàn thông tin hay không.
