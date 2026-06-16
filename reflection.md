# Day 14 — Reflection
## Evaluation Report & Failure Analysis

---

## 1. Benchmark Results Summary

Paste results từ Exercise 3.2 và tóm tắt:

**Overall pass rate:** 10.0%

**Average scores:**

| Metric | Average | Min | Max | Std Dev |
|--------|---------|-----|-----|---------|
| Faithfulness | 0.44 | 0.00 | 1.00 | 0.26 |
| Relevance | 0.27 | 0.00 | 0.89 | 0.21 |
| Completeness | 0.71 | 0.00 | 1.00 | 0.39 |
| Overall Score | 0.47 | 0.00 | 0.96 | 0.25 |

**Score interpretation (theo bài giảng):**
- Bao nhiêu metrics ở Good (0.8–1.0)? 0
- Bao nhiêu metrics ở Needs Work (0.6–0.8)? 1 (Completeness: 0.71)
- Bao nhiêu metrics ở Significant Issues (<0.6)? 2 (Faithfulness: 0.44, Relevance: 0.27)

**Failure type distribution:**

| Failure Type | Count | Percentage |
|--------------|-------|------------|
| hallucination | 6 | 30.0% |
| irrelevant | 5 | 25.0% |
| incomplete | 0 | 0.0% |
| off_topic | 7 | 35.0% |
| refusal | 0 | 0.0% |

---

## 2. Top 3 Worst Failures — 5 Whys Analysis

### Failure 1

**Question:** *Hạn chót nộp báo cáo giữa kỳ là ngày nào?* (ID: E04)

**Agent Answer:** *Hạn chót nộp báo cáo giữa kỳ đồ án là ngày 30 tháng 12 năm 2026.*

**Scores:** Faithfulness: 0.25 | Relevance: 0.00 | Completeness: 0.00 | Overall: 0.08

**5 Whys Analysis:**
| Level | Question | Answer |
|-------|----------|--------|
| Symptom | Vấn đề là gì? | Agent trả lời sai hạn chót nộp báo cáo giữa kỳ (trả lời 30/12/2026 thay vì 25/6/2026). |
| Why 1 | Tại sao xảy ra? | Agent tự bịa ra ngày 30/12/2026 mà không sử dụng thông tin trong tài liệu hướng dẫn đồ án. |
| Why 2 | Tại sao Why 1 xảy ra? | Prompt của Agent không yêu cầu Agent bắt buộc phải từ chối trả lời nếu thông tin không có trong context. |
| Why 3 | Tại sao Why 2 xảy ra? | Hệ thống không có validator/guardrail kiểm tra độ tin cậy (grounding) của câu trả lời so với tài liệu lấy ra trước khi trả về. |
| Why 4 | Root cause là gì? | Thiếu bộ lọc Hallucination hoặc cơ chế kiểm soát chất lượng đầu ra (Grounding Guardrail). |

**Root cause (from `find_root_cause()`):**
> *Multiple issues detected — review full pipeline*

**Bạn có đồng ý với root cause suggestion không? Tại sao?**
> *Đồng ý. Vì cả Relevance và Completeness đều bằng 0, chứng tỏ Agent không chỉ bịa thông tin mà còn không trả lời đúng trọng tâm câu hỏi, cho thấy sự sụp đổ của toàn bộ prompt/pipeline trong trường hợp này.*

**Proposed fix (cụ thể, actionable):**
> *1. Cập nhật prompt cấm LLM tự phỏng đoán số liệu/ngày tháng nếu không có trong context; 2. Tích hợp bộ lọc Hallucination Guardrail ở đầu ra.*

---

### Failure 2

**Question:** *Những milestone nào bị ảnh hưởng trực tiếp bởi việc trễ tiến độ deploy Docker ở tuần 3?* (ID: M05)

**Agent Answer:** *Việc này ảnh hưởng đến việc phân chia công việc CSDL PostgreSQL.*

**Scores:** Faithfulness: 0.09 | Relevance: 0.28 | Completeness: 0.17 | Overall: 0.18

**5 Whys Analysis:**
| Level | Question | Answer |
|-------|----------|--------|
| Symptom | Vấn đề là gì? | Agent trả lời lạc đề, nói về phân chia công việc CSDL thay vì chỉ ra các milestone bị ảnh hưởng. |
| Why 1 | Tại sao xảy ra? | Retriever lấy ra các chunk tài liệu không liên quan đến milestone mà lại chứa thông tin phân công của tuần 2. |
| Why 2 | Tại sao Why 1 xảy ra? | Từ khóa tìm kiếm "deploy Docker" bị nhiễu do database PostgreSQL cũng được deploy trong tuần đó. |
| Why 3 | Tại sao Why 2 xảy ra? | Thuật toán tìm kiếm ngữ nghĩa chưa đủ nhạy hoặc không có reranking để đẩy các tài liệu về milestone lên đầu. |
| Why 4 | Root cause là gì? | Chất lượng của bước Retrieval kém (không lấy đúng context liên quan). |

**Root cause:**
> *Context is missing or irrelevant — improve retrieval*

**Proposed fix:**
> *1. Cải tiến Retriever (áp dụng Hybrid Search + Reranker); 2. Tăng kích thước chunk và bổ sung overlap.*

---

### Failure 3

**Question:** *So sánh tiến độ thực tế tuần này với milestone đã lên kế hoạch trong tài liệu kiến trúc.* (ID: M01)

**Agent Answer:** *Thực tế nhóm đã hoàn thành thiết kế Figma và API đăng nhập đúng tiến độ. Việc deploy Docker hoàn thành xuất sắc và đúng hạn.*

**Scores:** Faithfulness: 0.59 | Relevance: 0.32 | Completeness: 0.49 | Overall: 0.46

**5 Whys Analysis:**
| Level | Question | Answer |
|-------|----------|--------|
| Symptom | Vấn đề là gì? | Báo cáo của Agent không trung thực, khẳng định deploy Docker đúng hạn trong khi thực tế bị chậm 2 ngày. |
| Why 1 | Tại sao xảy ra? | Agent ưu tiên tạo ra câu trả lời có tính tích cực/xu nịnh thay vì phản ánh trung thực lỗi cấu hình trong context. |
| Why 2 | Tại sao Why 1 xảy ra? | Prompt của Agent chưa có cấu trúc quy định cách viết khi có trễ hạn hoặc blockers. |
| Why 3 | Tại sao Why 2 xảy ra? | Thiếu few-shot examples hướng dẫn cách đối chiếu và so sánh giữa kế hoạch và thực tế. |
| Why 4 | Root cause là gì? | Thiếu prompt clarity và few-shot examples hướng dẫn. |

**Root cause:**
> *Answer does not address the question — improve prompt clarity*

**Proposed fix:**
> *1. Tinh chỉnh system prompt yêu cầu báo cáo trung thực các lỗi và sự trễ hạn; 2. Bổ sung few-shot examples về cách so sánh tiến độ thực tế vs kế hoạch.*

---

## 3. Failure Clustering

**Cluster Analysis:**

| Cluster | Root Cause | Failures in cluster | Priority |
|---------|-----------|--------------------:|----------|
| 1 | Hallucination do thiếu Grounding Guardrail hoặc prompt lỏng lẻo | E04, M01, H03, H05, A01, A02 | High |
| 2 | Sai lệch Retrieval dẫn đến lấy thiếu/sai context | M02, M05 | High |
| 3 | Trả lời lan man/lạc đề do Prompt Clarity chưa tốt | E01, E02, E03, M03, M04, M06, H01, H02, H04, A03 | Medium |

**Nếu chỉ fix 1 cluster, bạn chọn cluster nào? Tại sao?**
> *Chọn Cluster 1 (Hallucination/Grounding). Lý do: Trong báo cáo tiến độ học tập và đồ án tốt nghiệp, tính trung thực và chính xác của dữ liệu là quan trọng nhất. Sự sai lệch số liệu có thể gây mất uy tín hoặc làm GVHD đánh giá sai thực tế dự án.*

---

## 4. Improvement Log (from `generate_improvement_log`)

Paste output của `generate_improvement_log()`:

```
| Failure ID | Type | Root Cause | Suggested Fix | Status |
|------------|------|------------|---------------|--------|
| F001 | irrelevant | Answer does not address the question — improve prompt clarity | Refine routing or classification step to handle out-of-domain queries | Open |
| F002 | off_topic | Answer does not address the question — improve prompt clarity | Implement hallucination checker to filter unsupported claims | Open |
| F003 | off_topic | Answer does not address the question — improve prompt clarity | Improve prompt clarity to prevent irrelevant responses | Open |
| F004 | hallucination | Multiple issues detected — review full pipeline |  | Open |
| F005 | off_topic | Answer does not address the question — improve prompt clarity |  | Open |
| F006 | off_topic | Answer does not address the question — improve prompt clarity |  | Open |
| F007 | irrelevant | Answer does not address the question — improve prompt clarity |  | Open |
| F008 | off_topic | Answer does not address the question — improve prompt clarity |  | Open |
| F009 | hallucination | Context is missing or irrelevant — improve retrieval |  | Open |
| F010 | off_topic | Answer does not address the question — improve prompt clarity |  | Open |
| F011 | irrelevant | Answer does not address the question — improve prompt clarity |  | Open |
| F012 | irrelevant | Answer does not address the question — improve prompt clarity |  | Open |
| F013 | hallucination | Answer does not address the question — improve prompt clarity |  | Open |
| F014 | off_topic | Answer does not address the question — improve prompt clarity |  | Open |
| F015 | hallucination | Answer does not address the question — improve prompt clarity |  | Open |
| F016 | hallucination | Multiple issues detected — review full pipeline |  | Open |
| F017 | hallucination | Multiple issues detected — review full pipeline |  | Open |
| F018 | irrelevant | Answer does not address the question — improve prompt clarity |  | Open |
```

**Thêm 3 improvement suggestions từ `generate_improvement_suggestions()`:**
1. *Refine routing or classification step to handle out-of-domain queries*
2. *Implement hallucination checker to filter unsupported claims*
3. *Improve prompt clarity to prevent irrelevant responses*

---

## 5. Regression Testing Strategy

### CI/CD Integration

**Câu 1: Khi nào chạy `run_regression()` trong production system?**
> *Chạy tự động trong CI/CD pipeline bất cứ khi nào có sự thay đổi trong: mã nguồn của agent/evaluator, prompt templates, tham số cấu hình retriever, hoặc khi cập nhật bộ dữ liệu vàng (Golden Dataset).*

**Câu 2: Threshold regression 0.05 có phù hợp domain của bạn không?**
> *Rất phù hợp. Vì hệ thống đánh giá tiến độ đồ án tốt nghiệp yêu cầu độ chính xác cao. Sự sụt giảm lớn hơn 5% trong điểm số trung bình biểu thị một sự suy giảm nghiêm trọng về chất lượng câu trả lời hoặc khả năng tìm kiếm.*

**Câu 3: Khi phát hiện regression — block deployment hay chỉ alert?**
> *Block deployment đối với các lỗi suy giảm Faithfulness (hallucination) để tránh đưa thông tin sai lệch lên hệ thống thực. Đối với các suy giảm nhỏ về Relevance hoặc Completeness ở các câu hỏi khó, có thể gửi Alert để nhà phát triển xem xét thủ công.*

**Câu 4: Eval pipeline nên chạy ở đâu trong CI/CD flow?**

```
Code change → [1. Chạy Unit Tests (pytest)] → [2. Chạy Offline Eval trên Golden Dataset] → [3. Chạy Regression Check (run_regression)] → Deploy
```
> *Điền 3 bước eval vào flow trên:*
> - Bước 1: Chạy Unit Tests (pytest) để đảm bảo cú pháp và chức năng cơ bản không lỗi.
> - Bước 2: Chạy Offline Eval trên Golden Dataset để thu thập điểm số mới.
> - Bước 3: Chạy Regression Check (run_regression) so sánh với baseline để quyết định deploy hay block.

---

## 6. Continuous Improvement Loop

**Sau lab hôm nay, 3 actions tiếp theo bạn sẽ làm để improve agent:**

| Priority | Action | Metric sẽ improve | Expected impact |
|----------|--------|-------------------|-----------------|
| 1 | Tích hợp bộ Reranking (bge-reranker-large) | Context Precision | Đưa các tài liệu liên quan nhất lên đầu, giảm nhiễu. |
| 2 | Cải tiến system prompt và few-shot examples | Relevance & Completeness | Giúp Agent trả lời đúng trọng tâm và cấu trúc đầy đủ. |
| 3 | Thêm bộ lọc Hallucination Guardrail ở đầu ra | Faithfulness | Ngăn chặn việc Agent trả về các thông tin tự bịa đặt. |

**Bạn sẽ thêm failure cases nào vào benchmark cho sprint tiếp theo?**
> *1. SV viết tắt nhiều thuật ngữ chuyên ngành (ví dụ: FE, BE, DB) xem retriever có nhận diện được không; 2. Câu hỏi yêu cầu tổng hợp tiến độ của 5 tuần liên tiếp để xem context window có bị tràn không.*

---

## 7. Framework Reflection

**Framework bạn đã dùng trong lab:** *RAGAS-inspired heuristic (Simplified word-overlap)*

**Nếu dùng trong production, bạn sẽ chọn framework nào? Tại sao?**
> *Chọn DeepEval hoặc RAGAS.*

| Tiêu chí | Lý do chọn |
|----------|------------|
| Focus phù hợp vì... | DeepEval hỗ trợ unit testing cực tốt (pytest-native), có nhiều metrics đa dạng bao gồm cả safety và hallucination. |
| CI/CD integration vì... | Cả hai framework đều có công cụ CLI chạy tự động dễ dàng tích hợp vào GitHub Actions để block deployment. |
| Team workflow vì... | Giao diện trực quan (DeepEval Confident AI platform hoặc Ragas dashboard) giúp cả team theo dõi trực quan chất lượng của các phiên bản agent. |
