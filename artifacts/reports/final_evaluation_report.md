# Enterprise Customer Support RAG - Final Evaluation Report

## 1. Corpus Summary

Hệ thống sử dụng hai nguồn dữ liệu chính: handbook nội bộ công ty và corpus pháp luật/tài liệu Việt Nam.

| source_type      |   num_chunks |   num_parent_docs |   avg_chunk_chars |   min_chunk_chars |   max_chunk_chars | languages   |
|:-----------------|-------------:|------------------:|------------------:|------------------:|------------------:|:------------|
| company_handbook |           95 |                16 |           1038.95 |               128 |              1199 | English     |
| legal            |        91353 |             68663 |            746.39 |                 4 |              1200 | Vietnamese  |


## 2. System Pipeline


Pipeline cuối cùng của hệ thống gồm các bước:

1. **Chunking & Corpus Building**: chuẩn hóa tài liệu thành các chunks.
2. **BM25 Retrieval**: tìm kiếm theo từ khóa.
3. **Dense Retrieval**: tìm kiếm theo ngữ nghĩa bằng multilingual embedding.
4. **Weighted Hybrid RRF**: kết hợp BM25 và Dense.
5. **Reranker**: đọc lại candidate chunks và xếp hạng lại.
6. **Corrective RAG**: quyết định trả lời, hỏi lại, cảnh báo hoặc từ chối.
7. **Generation**: tạo câu trả lời cuối cùng có nguồn và evidence.

## 3. Retrieval Methods Comparison

Bảng dưới đây so sánh các phương pháp retrieval trên validation set.

| metric    |     BM25 |   Dense_E5 |   Weighted_Hybrid_RRF |   Reranker | best_method   |   best_value |   reranker_minus_BM25 |   reranker_vs_BM25_percent |   reranker_minus_Dense_E5 |   reranker_vs_Dense_E5_percent |   reranker_minus_Weighted_Hybrid_RRF |   reranker_vs_Weighted_Hybrid_RRF_percent |
|:----------|---------:|-----------:|----------------------:|-----------:|:--------------|-------------:|----------------------:|---------------------------:|--------------------------:|-------------------------------:|-------------------------------------:|------------------------------------------:|
| recall@1  | 0.343754 |   0.421615 |              0.422313 |   0.539686 | Reranker      |     0.539686 |              0.195932 |                    56.9977 |                  0.118071 |                       28.0045  |                             0.117373 |                                  27.7929  |
| recall@3  | 0.528995 |   0.624869 |              0.625102 |   0.728646 | Reranker      |     0.728646 |              0.199651 |                    37.7416 |                  0.103777 |                       16.6078  |                             0.103544 |                                  16.5643  |
| recall@5  | 0.605113 |   0.700407 |              0.708077 |   0.784195 | Reranker      |     0.784195 |              0.179082 |                    29.5948 |                  0.083788 |                       11.9628  |                             0.076118 |                                  10.75    |
| recall@10 | 0.687391 |   0.783614 |              0.785125 |   0.83498  | Reranker      |     0.83498  |              0.147589 |                    21.4709 |                  0.051366 |                        6.55501 |                             0.049855 |                                   6.34994 |
| mrr       | 0.453844 |   0.540115 |              0.541863 |   0.644133 | Reranker      |     0.644133 |              0.190289 |                    41.9283 |                  0.104018 |                       19.2585  |                             0.10227  |                                  18.8738  |
| ndcg@10   | 0.497519 |   0.585338 |              0.586727 |   0.676292 | Reranker      |     0.676292 |              0.178773 |                    35.9329 |                  0.090954 |                       15.5387  |                             0.089565 |                                  15.2652  |


## 4. Key Retrieval Findings


- Phương pháp tốt nhất tổng thể: **Reranker**
- Metric chọn chính: **recall@5**
- BM25 Recall@5: **0.605113**
- Dense_E5 Recall@5: **0.700407**
- Weighted Hybrid RRF Recall@5: **0.708077**
- Reranker Recall@5: **0.784195**

Kết luận: Reranker đạt kết quả tốt nhất trên toàn bộ metric retrieval, đặc biệt cải thiện Recall@5 so với Weighted Hybrid.

## 5. Corrective RAG Behavior

Corrective RAG giúp hệ thống quyết định khi nào nên trả lời, khi nào nên hỏi lại hoặc từ chối.

| question                                                                                          | route              | decision                        | should_call_llm   |   num_sources |   num_evidence | overall_status   | answer_preview                                                                                                                                                                                                                                                                              |
|:--------------------------------------------------------------------------------------------------|:-------------------|:--------------------------------|:------------------|--------------:|---------------:|:-----------------|:--------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------------|
| Nếu công ty áp dụng chính sách quản lý thiết bị làm việc cho nhân viên Việt Nam thì cần lưu ý gì? | cross_policy       | answer_with_legal_review_notice | True              |             6 |              3 | PASS             | Theo tài liệu nội bộ, công ty có chính sách quản lý thiết bị làm việc, bao gồm cấu hình bảo mật thiết bị, kiểm soát truy cập vào hệ thống nội bộ, không lưu code hoặc secrets trên thiết bị cá nhân, và có thể remote wipe thiết bị khi bị mất hoặc khi nhân viên rời công ty. Khi áp dụ... |
| What is the company policy for managing work devices?                                             | company_only       | answer                          | True              |             5 |              3 | PASS             | Theo handbook nội bộ, công ty quản lý thiết bị làm việc bằng cách cấu hình và bảo mật thiết bị, bao gồm các thiết lập như mã hóa ổ đĩa, firewall, quy tắc mật khẩu và cập nhật bảo mật. Nhân viên nên dùng thiết bị làm việc đáng tin cậy để truy cập VPN, hệ thống nội bộ, code và secr... |
| Lương thử việc được quy định như thế nào?                                                         | legal_only         | answer                          | True              |             5 |              3 | PASS             | Theo nguồn pháp luật được truy xuất, tiền lương trong thời gian thử việc do hai bên thỏa thuận, nhưng ít nhất phải bằng 85% mức lương của công việc đó. Các nguồn liên quan cũng nhắc đến quy định về thử việc, thời gian thử việc và hành vi vi phạm quy định về thử việc....              |
| Cách nấu phở bò ngon tại nhà như thế nào?                                                         | out_of_scope       | refuse                          | False             |             0 |              0 | PASS             | Tôi chưa tìm thấy đủ thông tin phù hợp trong phạm vi tài liệu doanh nghiệp/pháp luật của hệ thống. Vì vậy tôi không nên trả lời để tránh suy đoán sai....                                                                                                                                   |
| Chính sách này áp dụng sao?                                                                       | need_clarification | ask_clarification               | False             |             0 |              0 | PASS             | Bạn vui lòng nói rõ hơn chính sách hoặc vấn đề cụ thể cần hỏi. Ví dụ: chính sách thiết bị làm việc, lương thử việc, bảo hiểm, hợp đồng lao động, hoặc quy định áp dụng cho nhân viên Việt Nam....                                                                                           |


### Route Summary

| route              |   count |
|:-------------------|--------:|
| cross_policy       |       1 |
| company_only       |       1 |
| legal_only         |       1 |
| out_of_scope       |       1 |
| need_clarification |       1 |


### Decision Summary

| decision                        |   count |
|:--------------------------------|--------:|
| answer                          |       2 |
| answer_with_legal_review_notice |       1 |
| refuse                          |       1 |
| ask_clarification               |       1 |


## 6. Generation Quality


Generation quality được kiểm tra bằng rule-based checks.

- Tổng số demo cases: **5**
- PASS: **5**
- FAIL: **0**
- Số case gọi LLM / RAG prompt: **3**
- Số case trả lời trực tiếp: **2**

| question                                                                                          | route              | decision                        | should_call_llm   |   num_sources |   num_evidence | overall_status   | failed_checks   | has_answer   | has_sources   | has_evidence   |   has_company_source |   has_legal_source |   has_legal_review_notice |   mentions_85_percent |   refuses_answer |   asks_clarification |
|:--------------------------------------------------------------------------------------------------|:-------------------|:--------------------------------|:------------------|--------------:|---------------:|:-----------------|:----------------|:-------------|:--------------|:---------------|---------------------:|-------------------:|--------------------------:|----------------------:|-----------------:|---------------------:|
| Nếu công ty áp dụng chính sách quản lý thiết bị làm việc cho nhân viên Việt Nam thì cần lưu ý gì? | cross_policy       | answer_with_legal_review_notice | True              |             6 |              3 | PASS             | []              | True         | True          | True           |                    1 |                  1 |                         1 |                   nan |              nan |                  nan |
| What is the company policy for managing work devices?                                             | company_only       | answer                          | True              |             5 |              3 | PASS             | []              | True         | True          | True           |                    1 |                nan |                       nan |                   nan |              nan |                  nan |
| Lương thử việc được quy định như thế nào?                                                         | legal_only         | answer                          | True              |             5 |              3 | PASS             | []              | True         | True          | True           |                  nan |                  1 |                       nan |                     1 |              nan |                  nan |
| Cách nấu phở bò ngon tại nhà như thế nào?                                                         | out_of_scope       | refuse                          | False             |             0 |              0 | PASS             | []              | True         | True          | True           |                  nan |                nan |                       nan |                   nan |                1 |                  nan |
| Chính sách này áp dụng sao?                                                                       | need_clarification | ask_clarification               | False             |             0 |              0 | PASS             | []              | True         | True          | True           |                  nan |                nan |                       nan |                   nan |              nan |                    1 |


## 7. Final Conclusion


Kết quả thực nghiệm cho thấy pipeline tốt nhất hiện tại là:

**Dense / BM25 Candidate Retrieval → Weighted Hybrid RRF → Reranker → Corrective RAG → Grounded Generation**

Trong đó, **Reranker** là retrieval method tốt nhất, đạt Recall@5 cao nhất trên validation set.  
Corrective RAG giúp hệ thống an toàn hơn bằng cách từ chối câu hỏi ngoài phạm vi, hỏi lại câu hỏi mơ hồ, và cảnh báo HR/pháp chế khi câu hỏi có yếu tố pháp lý.

Hệ thống cuối cùng có khả năng:
- Trả lời câu hỏi handbook nội bộ.
- Trả lời câu hỏi pháp luật/tài liệu Việt Nam.
- Đối chiếu chính sách nội bộ với bối cảnh Việt Nam.
- Hiển thị nguồn và evidence.
- Tránh trả lời khi câu hỏi ngoài phạm vi hoặc mơ hồ.
