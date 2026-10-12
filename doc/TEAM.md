# Nhóm thực hiện K4 Track 3A

Tên nhóm: Không Tên

Repo chung: <https://github.com/ntiendat01/K4-L3-DAY24-KhongTen-Eval-Guard)>

Trưởng nhóm/người nộp: Nguyễn Tiến Đạt - 2A202602606

Ngày lab/deadline áp dụng: 2026-10-12 23:59 — Asia/Ho_Chi_Minh (UTC+07:00)

Commit dùng để chấm: 3b8f012cfff6f12cb05257a06b54d7dcf11351b7

Mỗi nhóm nộp một repo. Điền đủ thành viên và một mục INDIVIDUAL cho từng người.
Thêm/xóa rows để khớp nhóm thật; mẫu ba rows không ép nhóm phải có ba người.

## Thành viên và phân công

| STT | Họ và tên | MSSV | Vai trò / công việc thật | Bằng chứng |
| --- | --- | --- | --- | --- |
| 1 | Trần Mạnh Hùng | 2A202602708 | Phụ trách Phase A (Chuẩn bị Pipeline CP0, RAGAS 50q, Bottom 10 & Failure Clusters Analysis) | Commit 3b8f012 ("phase_a: eval"), artifacts/ragas_50q.json, doc/reports/failure_clusters.md |
| 2 | Nguyễn Tiến Đạt | 2A202602606 | Phụ trách Phase B (LLM-as-Judge, Pairwise Swap-and-Average, Cohen's Kappa & Bias Analysis) | PR-1 (branch ntiendat01), artifacts/judge_results.json, doc/reports/bias_report.md |
| 3 | Nguyễn Thu Trang | 2A202602435 | Phụ trách Phase C (Presidio PII, NeMo Guardrails, Benchmark Latency P95 & Blueprint CI/CD) | PR-3 (branch phase_c), artifacts/guard_results.json, doc/reports/blueprint.md |

Người tích hợp/chạy batch/nộp link: Nguyễn Tiến Đạt - 2A202602606 (tích hợp các nhánh vào main, chạy kiểm thử và chuẩn bị nộp bài).

## Môi trường và chuẩn bị trước lớp

OS/Python/compiler: macOS / Windows 11, Python 3.11.16, Apple Clang / MSVC C++ Build Tools.
Virtualenv/dependencies/imports: Tạo .venv với Python 3.11, cài đặt pyarrow<18 tương thích numpy 1.26.4, đã kiểm tra lệnh python -c "import ragas, nemoguardrails, presidio_analyzer, spacy; print('imports OK')" thành công.
Models/Qdrant/50 answers đã chuẩn bị: Tải mô hình spaCy en_core_web_lg, tải all-MiniLM-L6-v2 cho NeMo Guardrails, chạy Qdrant Docker v1.9.7 và sinh đầy đủ 50 câu trả lời RAG trong artifacts/answers_50q.json.
Nếu cài full NeMo chưa thành công, ghi lỗi thực tế; không dùng unit test pass
hoặc pip resolve để khẳng định runtime đã chạy.

## INDIVIDUAL

Sao chép mẫu cho mỗi thành viên; tiêu đề họ tên không dấu-MSSV.

### Tran Manh Hung-2A202602708

- Phần được giao và phần thực tế đã làm: Phụ trách toàn diện Phase A và giai đoạn chuẩn bị CP0: Tích hợp và kiểm tra các module RAG m1-m5 (chunking, hybrid search BM25 + Qdrant dense, reranker, evaluation m4, enrichment m5); sinh đầy đủ 50 câu trả lời từ pipeline thực tế trong setup_answers.py; hoàn thiện 4 tasks trong src/phase_a_ragas.py gồm phân nhóm 50 câu hỏi theo 3 distribution (group_by_distribution), thực thi đánh giá 4 metrics RAGAS (run_ragas_50q), phân tích trích xuất 10 câu có điểm trung bình thấp nhất (bottom_10) cùng chẩn đoán nguyên nhân và giải pháp, xây dựng ma trận phân bố lỗi theo worst-metric và xác định nhóm yếu nhất theo mean avg_score (cluster_analysis); chạy live thực nghiệm và hoàn thiện báo cáo doc/reports/failure_clusters.md.
- Bằng chứng: Commit 65cda49 ("add m1-m5"), Commit de27c88 ("feat(1): upload file 50 artifacts"), Commit 3b8f012 ("phase_a: eval"), file kết quả thực nghiệm live artifacts/answers_50q.json và artifacts/ragas_50q.json (50 câu: factual 0.8760, multi_hop 0.7566, adversarial 0.7761), code sạch không TODO trong src/phase_a_ragas.py và báo cáo phân tích lỗi doc/reports/failure_clusters.md.
- Cách kiểm tra: Chạy pytest tests/test_phase_a.py -v (6/6 pass với mock m4) và chạy thực nghiệm trực tiếp python scripts/run_experiments.py --phase a --live sinh đầy đủ artifacts/ragas_50q.json với mô hình evaluator gpt-4o-mini; kiểm tra toàn bộ pipeline và báo cáo qua python scripts/check_lab.py --with-tests.
- Điều đã học: Nắm vững phương pháp luận đánh giá hệ thống RAG doanh nghiệp theo 4 metrics cốt lõi của RAGAS (Faithfulness, Answer Relevancy, Context Precision, Context Recall). Hiểu rõ sự khác biệt về bản chất giữa các phân bố câu hỏi (factual tra cứu trực tiếp, multi-hop suy luận đa chặng, adversarial bẫy phiên bản và mâu thuẫn chính sách). Nắm vững cách xây dựng Diagnostic Tree để chẩn đoán điểm nghẽn của hệ thống (do retrieval thiếu context hay LLM hallucination) và phương pháp xác định nhóm yếu nhất dựa trên mean avg_score thay vì đếm số câu thuần túy.
- Giới hạn/cách xử lý: Nhóm multi-hop là nhóm có điểm trung bình thấp nhất (0.7566) do đặc thù câu hỏi đòi hỏi thông tin từ nhiều tài liệu khác nhau trong khi retrieval đơn chặng dễ bị thiếu ngữ cảnh. Đã phân tích chi tiết các ca thất bại tiêu biểu (ID 50, ID 6, ID 30, ID 33) và đề xuất giải pháp kiến trúc Query Decomposition kết hợp mở rộng top-k candidates và Cross-Encoder reranking để khắc phục trong thực tế.
- Nguồn/AI hỗ trợ nếu có: Sử dụng Antigravity hỗ trợ kiểm tra đối chiếu hợp đồng interface m1-m5, cấu trúc schema phase_a và chuẩn hóa line-ending JSON cross-platform; toàn bộ code thực thi, chạy batch OpenAI API và phân tích kết quả thực nghiệm đều được tự thực hiện và kiểm chứng 100% qua pytest và check_lab.py.

### Nguyen Tien Dat-2A202602606

- Phần được giao và phần thực tế đã làm: Phụ trách toàn diện Phase B (LLM-as-Judge & Đo lường Bias): Lập trình bộ đánh giá cặp pairwise_judge so sánh 2 câu trả lời theo 3 tiêu chí (chính xác, đầy đủ, súc tích); xây dựng cơ chế swap_and_average hoán đổi thứ tự và chuyển đổi không gian nhãn để triệt tiêu position bias, đồng thuận giữ winner, bất đồng quy về tie; lập trình tính toán hệ số Cohen's κ (cohen_kappa) đo lường độ đồng thuận giữa LLM Judge và chuyên gia con người trên 10 câu nhãn chuẩn (human_labels_10q.json) mà không làm lộ nhãn con người; lập trình đo lường position bias và verbosity bias (bias_report); thực thi đánh giá thực nghiệm live trên 5 cặp câu hỏi và 10 câu calibration; phân tích chuyên sâu các case không nhất quán và hoàn thiện báo cáo doc/reports/bias_report.md.
- Bằng chứng: Commit 15b38ec ("phase B"), Commit eac5548 ("phase B"), Pull Request PR-1 (nhánh ntiendat01 -> main), file kết quả thực nghiệm live artifacts/judge_results.json (5 cặp pairwise: Position bias rate 20.0%, Verbosity win rate 75.0%; Calibration 10 câu đạt Cohen's κ = 1.0, 10/10 đồng thuận với human labels), code sạch không TODO trong src/phase_b_judge.py và báo cáo phân tích doc/reports/bias_report.md.
- Cách kiểm tra: Chạy pytest tests/test_phase_b.py -v (pass 100% với mock judge offline) và chạy thực nghiệm trực tiếp python scripts/run_experiments.py --phase b --live sinh đầy đủ artifacts/judge_results.json với model judge gpt-4o-mini; kiểm tra tính hợp lệ qua python scripts/check_lab.py.
- Điều đã học: Nắm vững phương pháp luận LLM-as-Judge và các dạng thiên lệch đặc trưng trong đánh giá RAG: Position Bias (thiên vị vị trí thứ tự xuất hiện) và Verbosity Bias (thiên vị câu trả lời dài dòng). Thành thạo kỹ thuật Swap-and-Average để triệt tiêu position bias, cách tính toán và diễn giải hệ số Cohen's Kappa theo thang đo Landis & Koch (đạt mức Perfect Agreement κ = 1.0, vượt ngưỡng bonus +3 điểm), và phương pháp thiết kế prompt phân định rạch ròi 3 tiêu chí cùng nguyên tắc bảo mật không làm lộ dữ liệu chuẩn (label leakage).
- Giới hạn/cách xử lý: Ở cặp ID 5 (thiết bị 55 triệu), LLM judge ở lượt 1 chọn sai đáp án B do ngộ nhận thẩm quyền phòng ban, nhưng sau khi swap ở lượt 2 đã nhận diện cả hai đáp án đều sai; cơ chế swap-and-average đã xử lý chuẩn xác bằng cách đưa kết quả về hòa (tie). Ở hiện tượng verbosity bias, phân tích định tính case ID 21 chứng minh judge không thiên vị độ dài máy móc mà vẫn chọn đáp án ngắn hơn nếu súc tích và đúng trọng tâm. Cỡ mẫu đánh giá (5 cặp, 10 câu) còn khiêm tốn, do đó trong môi trường sản xuất thực tế cần thiết lập cơ chế Human-in-the-loop đối với các ca bất đồng thuận hoặc có mức độ rủi ro cao.
- Nguồn/AI hỗ trợ nếu có: Sử dụng Antigravity hỗ trợ phân tích định dạng schema prompt JSON, công thức thống kê Cohen's Kappa và đối chiếu hợp đồng kiểm thử test_report_contracts.py; toàn bộ code thực thi, chạy batch OpenAI API và phân tích kết quả thực nghiệm đều được tự thực hiện và kiểm chứng 100% qua pytest và check_lab.py.

### Nguyễn Thu Trang-2A202602435

- Phần được giao và phần thực tế đã làm: Phụ trách toàn diện Phase C: Lập trình phát hiện và ẩn danh PII tiếng Việt với Presidio (pii_scan), tích hợp NeMo Guardrails kiểm duyệt đầu vào và đầu ra bất đồng bộ (check_input_rail, check_output_rail), thực thi bộ kiểm thử 28 câu hỏi (run_adversarial_suite), đo đạc độ trễ P95 tuần tự 2 tầng với warm-up thật (measure_p95_latency), và hoàn thiện báo cáo kiến trúc Blueprint CI/CD (blueprint.md).
- Bằng chứng: Commit aef06bc ("feat(phase-c): implement guardrails stack, p95 benchmark and blueprint report (2A202602435)"), Commit c49a42e ("hoàn thiện src/phase_c_guard.py"), Pull Request PR-3 (nhánh phase_c -> main), file kết quả thực nghiệm live artifacts/guard_results.json (20/20 attacks blocked, 8/8 clean allowed, 4/4 outputs đúng), code sạch không TODO trong src/phase_c_guard.py, guardrails/config.yml và báo cáo doc/reports/blueprint.md.
- Cách kiểm tra: Chạy pytest tests/test_phase_c.py -v (12/12 pass) và chạy thực nghiệm trực tiếp python scripts/run_experiments.py --phase c --live --latency-samples 20 đạt 28/28 passed (100%).
- Điều đã học: Nắm vững kiến trúc bảo vệ đa tầng trong hệ thống RAG doanh nghiệp, cách thiết lập Colang 1.0 và cơ chế tự kiểm tra của NeMo Guardrails, phương pháp đo độ trễ P95/P99 theo nội suy tuyến tính, và cách thiết lập các quality gates trong CI/CD.
- Giới hạn/cách xử lý: Ban đầu mô hình spaCy tiếng Anh nhận diện nhầm văn bản tiếng Việt thành PERSON gây chặn nhầm câu sạch (false positive); đã giải quyết triệt để bằng cách lọc đúng tập thực thể mục tiêu TARGET_PII_ENTITIES, đưa tỷ lệ false positive về 0.0%. Xử lý prompt trong NeMo Guardrails để phân biệt câu hỏi chính sách mật khẩu hợp lệ với yêu cầu lấy mật khẩu admin.
- Nguồn/AI hỗ trợ nếu có: Sử dụng Antigravity hỗ trợ phân tích traceback môi trường, gỡ lỗi nhận diện thực thể PII và tối ưu prompt; toàn bộ code và kết quả đo đạc đều được tự chạy và kiểm chứng 100% qua pytest và check_lab.py.

## Cập nhật sau khi nộp

Bản nộp chính thức đúng hạn ngày 12/10/2026; không có cập nhật chỉnh sửa sau deadline.
