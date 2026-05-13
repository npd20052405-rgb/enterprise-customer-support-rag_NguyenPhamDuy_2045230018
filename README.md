\# Enterprise Customer Support RAG



Dự án xây dựng hệ thống End-to-End Corrective RAG cho bài toán hỏi đáp nội bộ doanh nghiệp, kết hợp handbook công ty và tài liệu pháp luật/tài liệu Việt Nam.



\## Mục tiêu



Hệ thống hỗ trợ trả lời các câu hỏi liên quan đến:



\- Chính sách nội bộ công ty

\- Quản lý thiết bị làm việc

\- Code, secrets, VPN và thiết bị cá nhân

\- Quy định pháp luật/tài liệu Việt Nam liên quan đến lao động

\- Các câu hỏi cần đối chiếu giữa chính sách nội bộ và bối cảnh Việt Nam



\## Pipeline



Question  

→ BM25 Retrieval  

→ Dense Retrieval  

→ Weighted Hybrid RRF  

→ Reranker  

→ Corrective Gate  

→ Gemini LLM Generation  

→ Answer + Sources + Evidence  



\## Thành phần chính



| Thành phần | Vai trò |

|---|---|

| BM25 | Truy xuất theo từ khóa |

| Dense Retrieval | Truy xuất theo ngữ nghĩa |

| Weighted Hybrid RRF | Kết hợp BM25 và Dense |

| Reranker | Xếp hạng lại context |

| Corrective RAG | Quyết định trả lời, hỏi lại hoặc từ chối |

| Gemini LLM | Sinh câu trả lời cuối cùng |

| Streamlit | Giao diện demo |



\## Kết quả chính



| Metric | Reranker |

|---|---:|

| Recall@1 | 0.539686 |

| Recall@3 | 0.728646 |

| Recall@5 | 0.784195 |

| Recall@10 | 0.834980 |

| MRR | 0.644133 |

| nDCG@10 | 0.676292 |



Best Retriever: Reranker  

Selected Metric: Recall@5  

Reranker Recall@5: 0.784195  

Final LLM Quality: 5/5 PASS  



\## Corrective RAG



| Route | Decision |

|---|---|

| cross\_policy | answer\_with\_legal\_review\_notice |

| company\_only | answer |

| legal\_only | answer |

| out\_of\_scope | refuse |

| need\_clarification | ask\_clarification |



\## Gemini LLM Generation



Notebook 13 đã tích hợp Gemini API để sinh câu trả lời thật.



LLM provider: Gemini API  

Model: gemini-2.5-flash  

Final LLM Quality: 5/5 PASS  



\## Streamlit App



Chạy app:



```bash

streamlit run app/streamlit\_app.py

