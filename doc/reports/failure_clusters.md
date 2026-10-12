# Phân tích lỗi Phase A

Nhóm: Không Tên

Người phụ trách: Trần Mạnh Hùng - 2A202602708

Nguồn: `artifacts/ragas_50q.json`; run timestamp: 2026-10-12T15:28:11.436310+00:00.
Generation mode/fallback và điều kiện chạy: Chạy pipeline thực tế với Hybrid Search (BM25 + Dense Qdrant) và LLM generation qua OpenAI API (gpt-4o-mini-2024-07-18).
Models generation/evaluator: Generation: gpt-4o-mini-2024-07-18; Evaluator: gpt-4o-mini (thông qua m4 RAGAS 4 metrics).

## Điểm theo distribution

| Metric | factual (20) | multi_hop (20) | adversarial (10) |
|---|---|---|---|
| faithfulness | 0.9333 | 0.5500 | 0.8000 |
| answer_relevancy | 0.7622 | 0.7513 | 0.7125 |
| context_precision | 0.9583 | 0.9417 | 0.9417 |
| context_recall | 0.8500 | 0.7833 | 0.6500 |
| avg_score | 0.8760 | 0.7566 | 0.7761 |

## Bottom 10 và nguyên nhân

| Rank | ID | Question | Distribution | Avg Score | Worst Metric | Diagnosis | Suggested Fix |
|---|---|---|---|---|---|---|---|
| 1 | 50 | Nhân viên Manager có thể dùng VPN cá nhân (như NordVPN) khi WFH để tăng bảo mật thêm không? | adversarial | 0.3333 | faithfulness | LLM hallucinating | Tighten system prompt, lower temperature |
| 2 | 6 | Thông tin lương thuộc cấp độ phân loại dữ liệu nào? | factual | 0.3750 | faithfulness | LLM hallucinating | Tighten system prompt, lower temperature |
| 3 | 44 | Bao lâu phải đổi mật khẩu một lần? | adversarial | 0.3750 | faithfulness | LLM hallucinating | Tighten system prompt, lower temperature |
| 4 | 30 | So sánh quyền lợi bảo hiểm giữa nhân viên thử việc và nhân viên chính thức. | multi_hop | 0.3750 | faithfulness | LLM hallucinating | Tighten system prompt, lower temperature |
| 5 | 33 | Nhân viên Manager có thâm niên 12 năm: tổng phụ cấp hàng tháng và số ngày phép năm theo v2024 là bao nhiêu? | multi_hop | 0.3750 | faithfulness | LLM hallucinating | Tighten system prompt, lower temperature |
| 6 | 5 | Muốn mua thiết bị trị giá 55 triệu cần ai phê duyệt? | factual | 0.5000 | answer_relevancy | Answer doesn't match question | Improve prompt template |
| 7 | 39 | So sánh yêu cầu mật khẩu giữa policy v1.0 và v2.0 về độ dài tối thiểu, thời hạn đổi và MFA. | multi_hop | 0.5450 | context_precision | Too many irrelevant chunks | Add reranking or metadata filter |
| 8 | 27 | Nhân viên có thâm niên 7 năm theo v2024 được nghỉ bao nhiêu ngày phép? Nếu bị trừ thêm 4 ngày ốm không có giấy y tế thì còn lại bao nhiêu? | multi_hop | 0.6738 | faithfulness | LLM hallucinating | Tighten system prompt, lower temperature |
| 9 | 38 | Nhân viên đi công tác nước ngoài 4 ngày tại thành phố lớn, ở khách sạn 200 USD/đêm (3 đêm). Công ty thanh toán bao nhiêu tiền khách sạn? | multi_hop | 0.7057 | faithfulness | LLM hallucinating | Tighten system prompt, lower temperature |
| 10 | 35 | Nhân viên Junior P1 có lương cơ bản 12 triệu vừa vào thử việc: nhận lương bao nhiêu và phụ cấp gì tháng đầu tiên? | multi_hop | 0.7144 | faithfulness | LLM hallucinating | Tighten system prompt, lower temperature |

### Đối chiếu đại diện các ca lỗi thực tế:

- **Lỗi Fallback do Retrieval không đủ ngưỡng (ID 50, ID 6, ID 44, ID 30, ID 33):**
  - **ID 50 (adversarial):** Câu hỏi về việc Manager dùng VPN cá nhân khi WFH. Reference nêu rõ: *KHÔNG, Chính sách VPN cấm dùng VPN cá nhân, bắt buộc dùng WireGuard công ty*. Tuy nhiên trong bước sinh answer, do retrieval không vượt ngưỡng tự tin nên pipeline trả về `"Không tìm thấy."`. Khi đánh giá RAGAS, do answer không giải quyết được câu hỏi và không có thông tin khớp ground truth, cả faithfulness và answer_relevancy đều bị chấm 0.0, context_recall chỉ đạt 0.3333.
  - **ID 6 (factual):** Câu hỏi *"Thông tin lương thuộc cấp độ phân loại dữ liệu nào?"*. Reference là *Dữ liệu Bí mật (cấp 3)*. Pipeline trả về `"Không tìm thấy."` do retrieval không lấy đúng chunk chứa quy chế bảo mật lương từ `phan_loai_du_lieu.md`, dẫn đến avg_score tụt xuống 0.3750.
  - **ID 30 & ID 33 (multi_hop):** Cả hai câu đều đòi hỏi kết hợp thông tin từ nhiều văn bản (ID 30 cần kết hợp `thu_viec.md` và `bao_hiem_suc_khoe.md`; ID 33 cần kết hợp `phu_cap.md` và `nghi_phep_nam_v2024.md`). Retrieval đơn lẻ không gom đủ tất cả các chunks của 2 tài liệu nên pipeline trả về `"Không tìm thấy."`, kéo faithfulness xuống 0.0.

- **Lỗi Generation / Trả lời sai thẩm quyền (ID 5 - factual):**
  - Câu hỏi: Mua thiết bị trị giá 55 triệu cần ai duyệt?
  - Context có đoạn nói về thẩm quyền phê duyệt mua sắm. Nhưng pipeline sinh câu trả lời: *"Cần thêm phê duyệt của Kế toán trưởng"*, trong khi Reference chính xác là: *Đơn hàng trên 50 triệu cần Tổng Giám đốc (CEO) phê duyệt*. Faithfulness đạt 1.0 (do câu trả lời bám vào ngữ cảnh kế toán có trong chunk), nhưng Answer Relevancy tụt về 0.0 vì trả lời sai trọng tâm câu hỏi về người có thẩm quyền tối hậu.

- **Lỗi Context Precision & Xung đột phiên bản (ID 39 - multi_hop):**
  - Câu hỏi so sánh mật khẩu v1.0 và v2.0. Pipeline lấy được chunk nhưng các chunk rác hoặc chunk v2.0 chiếm vị trí xếp hạng cao hơn chunk v1.0, khiến Context Precision giảm xuống 0.0. Answer trả lời đúng được ý v2.0 nhưng thiếu ý đổi mật khẩu mỗi 90 ngày của v1.0.

## Matrix và nhóm yếu nhất

| Metric | factual | multi_hop | adversarial | Tổng |
|---|---|---|---|---|
| faithfulness | 2 | 15 | 2 | 19 |
| answer_relevancy | 12 | 0 | 1 | 13 |
| context_precision | 3 | 1 | 1 | 5 |
| context_recall | 3 | 4 | 6 | 13 |
| Tổng số câu | 20 | 20 | 10 | 50 |

Nhóm yếu theo mean thấp nhất: multi_hop; mean: 0.7566.
Worst metric xuất hiện nhiều nhất: faithfulness; số câu: 19.

### Phân tích nguyên nhân và đề xuất cải tiến:
- **Vì sao multi_hop là nhóm yếu nhất:** Điểm trung bình của nhóm multi_hop là 0.7566 (thấp nhất trong 3 nhóm, so với factual là 0.8760 và adversarial là 0.7761). Nguyên nhân trực tiếp là **faithfulness** của nhóm multi_hop rất thấp (chỉ đạt 0.5500), và có tới 15/20 câu có worst_metric là faithfulness. Khi một câu hỏi yêu cầu liên kết thông tin giữa 2 tài liệu (ví dụ thâm niên tính phép và phụ cấp chức vụ), việc retrieval chỉ lấy top-k đoạn tương đồng đơn lẻ thường bỏ sót văn bản thứ hai, khiến pipeline rơi vào tình trạng thiếu dữ kiện (trả về fallback "Không tìm thấy." hoặc LLM tự suy diễn sai).
- **Đề xuất khắc phục:**
  1. Triển khai **Query Decomposition / Sub-query Generation**: Tách câu hỏi phức hợp đa chặng thành các câu hỏi con độc lập trước khi truy vấn vector/BM25.
  2. Nâng số lượng candidate retrieval $K$ từ 5 lên 10 kết hợp với Cross-Encoder Reranker (`m3_rerank`) để giữ lại các chunk từ cả hai nguồn tài liệu.
  3. Cải tiến prompt template với few-shot ví dụ suy luận từng bước (Chain-of-Thought) để giảm thiểu hallucination ở bước tổng hợp.
  4. Chạy lại benchmark: Thực thi lại lệnh `python scripts/run_experiments.py --phase a --live` và so sánh điểm số `multi_hop avg_score` để kiểm chứng cải tiến.

## Bẫy adversarial và giới hạn

### Phân tích bẫy phiên bản và phủ định:
- Điểm trung bình của nhóm `adversarial` là 0.7761, thấp hơn đáng kể so với nhóm `factual` (0.8760). Điểm yếu tập trung lớn nhất ở **context_recall (0.6500)** và **answer_relevancy (0.7125)**.
- **Bẫy mâu thuẫn phiên bản (Version Conflict Trap):** Điển hình ở câu ID 44 (chính sách đổi mật khẩu) và ID 41 (ngày phép năm v2023 vs v2024). Bộ dữ liệu chứa cả hai phiên bản chính sách cũ và mới. Khi người dùng hỏi câu hỏi trung lập không ghi rõ phiên bản, hệ thống dễ truy xuất nhầm chunk phiên bản cũ (v1.0 hoặc v2023) hoặc lấy lẫn lộn cả hai, dẫn đến context recall thấp và câu trả lời không phản ánh chính sách hiện hành.
- **Bẫy phủ định và tính hợp lý giả tạo (Policy Contradiction Trap):** Ở câu ID 50, câu hỏi nghe rất hợp lý ("Manager dùng VPN cá nhân khi WFH để tăng bảo mật thêm có được không?"). Do câu hỏi chứa từ khóa "bảo mật", retriever tìm được tài liệu về VPN nhưng mô hình không nhận diện được điều cấm ngặt nghèo trong chính sách CNTT, dẫn đến việc không cung cấp được câu trả lời kiên quyết từ chối.
- **Giới hạn thực nghiệm:**
  - Cỡ mẫu 50 câu (10 câu adversarial) đủ để chỉ ra các điểm nghẽn kiến trúc nhưng chưa bao quát toàn bộ các vector tấn công tinh vi.
  - Mô hình evaluator là `gpt-4o-mini`, dù tiết kiệm chi phí nhưng đôi khi đánh giá khắt khe ở các câu trả lời ngắn dạng fallback.
  - Cần bổ sung tầng Guardrails (như triển khai ở Phase C) để chặn đứng các câu hỏi mang bẫy vi phạm trước khi chuyển tiếp vào RAG pipeline.
