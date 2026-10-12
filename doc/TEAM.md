# Nhóm thực hiện K4 Track 3A

Tên nhóm: Không Tên

Repo chung: <https://github.com/ntiendat01/K4-Track-3A-Day24-Eval-Guard>

Trưởng nhóm/người nộp: Nguyễn Tiến Đạt - 2A202602606

Ngày lab/deadline áp dụng: 2026-10-12 23:59 — Asia/Ho\_Chi\_Minh (UTC+07:00)

Commit dùng để chấm: {{Commit SHA}}

Mỗi nhóm nộp một repo. Điền đủ thành viên và một mục INDIVIDUAL cho từng người.
Thêm/xóa rows để khớp nhóm thật; mẫu ba rows không ép nhóm phải có ba người.

## Thành viên và phân công

| STT | Họ và tên        | MSSV        | Vai trò / công việc thật                                                                   | Bằng chứng                                                                                        |
| --- | ---------------- | ----------- | ------------------------------------------------------------------------------------------ | ------------------------------------------------------------------------------------------------- |
| 1   | {{Họ tên}}       | {{MSSV}}    | {{Phase/file/hàm}}                                                                         | {{Commit/PR/report}}                                                                              |
| 2   | Nguyễn Tiến Đạt  | 2A202602606 | Phụ trách Phase B (LLM-as-Judge, Pairwise Swap-and-Average, Cohen's Kappa & Bias Analysis) | PR-1 (branch ntiendat01), artifacts/judge\_results.json, doc/reports/NguyenTienDat-2A202602606.md |
| 3   | Nguyễn Thu Trang | 2A202602435 | Phụ trách Phase C (Presidio PII, NeMo Guardrails, Benchmark Latency P95 & Blueprint CI/CD) | PR-3 (branch phase\_c), artifacts/guard\_results.json, doc/reports/blueprint.md                   |

Người tích hợp/chạy batch/nộp link: Nguyễn Tiến Đạt - 2A202602606 (tích hợp các nhánh vào main, chạy kiểm thử và chuẩn bị nộp bài).

## Môi trường và chuẩn bị trước lớp

OS/Python/compiler: macOS / Windows 11, Python 3.11.16, Apple Clang / MSVC C++ Build Tools.
Virtualenv/dependencies/imports: Tạo .venv với Python 3.11, cài đặt pyarrow<18 tương thích numpy 1.26.4, đã kiểm tra lệnh python -c "import ragas, nemoguardrails, presidio\_analyzer, spacy; print('imports OK')" thành công.
Models/Qdrant/50 answers đã chuẩn bị: Tải mô hình spaCy en\_core\_web\_lg, tải all-MiniLM-L6-v2 cho NeMo Guardrails, chạy Qdrant Docker v1.9.7 và sinh đầy đủ 50 câu trả lời RAG trong artifacts/answers\_50q.json.
Nếu cài full NeMo chưa thành công, ghi lỗi thực tế; không dùng unit test pass
hoặc pip resolve để khẳng định runtime đã chạy.

## INDIVIDUAL

Sao chép mẫu cho mỗi thành viên; tiêu đề họ tên không dấu-MSSV.

### Nguyen Tien Dat-2A202602606

- Phần được giao và phần thực tế đã làm: Phụ trách toàn diện Phase B (LLM-as-Judge & Đo lường Bias): Lập trình bộ đánh giá cặp pairwise\_judge so sánh 2 câu trả lời theo 3 tiêu chí (chính xác, đầy đủ, súc tích); xây dựng cơ chế swap\_and\_average hoán đổi thứ tự và chuyển đổi không gian nhãn để triệt tiêu position bias, đồng thuận giữ winner, bất đồng quy về tie; lập trình tính toán hệ số Cohen's κ (cohen\_kappa) đo lường độ đồng thuận giữa LLM Judge và chuyên gia con người trên 10 câu nhãn chuẩn (human\_labels\_10q.json) mà không làm lộ nhãn con người; lập trình đo lường position bias và verbosity bias (bias\_report); thực thi đánh giá thực nghiệm live trên 5 cặp câu hỏi và 10 câu calibration; phân tích chuyên sâu các case không nhất quán và hoàn thiện báo cáo doc/reports/NguyenTienDat-2A202602606.md.
- Bằng chứng: Commit 15b38ec ("phase B"), Commit eac5548 ("phase B"), Pull Request PR-1 (nhánh ntiendat01 -> main), file kết quả thực nghiệm live artifacts/judge\_results.json (5 cặp pairwise: Position bias rate 20.0%, Verbosity win rate 75.0%; Calibration 10 câu đạt Cohen's κ = 1.0, 10/10 đồng thuận với human labels), code sạch không TODO trong src/phase\_b\_judge.py và báo cáo phân tích doc/reports/NguyenTienDat-2A202602606.md.
- Cách kiểm tra: Chạy pytest tests/test\_phase\_b.py -v (pass 100% với mock judge offline) và chạy thực nghiệm trực tiếp python scripts/run\_experiments.py --phase b --live sinh đầy đủ artifacts/judge\_results.json với model judge gpt-4o-mini; kiểm tra tính hợp lệ qua python scripts/check\_lab.py.
- Điều đã học: Nắm vững phương pháp luận LLM-as-Judge và các dạng thiên lệch đặc trưng trong đánh giá RAG: Position Bias (thiên vị vị trí thứ tự xuất hiện) và Verbosity Bias (thiên vị câu trả lời dài dòng). Thành thạo kỹ thuật Swap-and-Average để triệt tiêu position bias, cách tính toán và diễn giải hệ số Cohen's Kappa theo thang đo Landis & Koch (đạt mức Perfect Agreement κ = 1.0, vượt ngưỡng bonus +3 điểm), và phương pháp thiết kế prompt phân định rạch ròi 3 tiêu chí cùng nguyên tắc bảo mật không làm lộ dữ liệu chuẩn (label leakage).
- Giới hạn/cách xử lý: Ở cặp ID 5 (thiết bị 55 triệu), LLM judge ở lượt 1 chọn sai đáp án B do ngộ nhận thẩm quyền phòng ban, nhưng sau khi swap ở lượt 2 đã nhận diện cả hai đáp án đều sai; cơ chế swap-and-average đã xử lý chuẩn xác bằng cách đưa kết quả về hòa (tie). Ở hiện tượng verbosity bias, phân tích định tính case ID 21 chứng minh judge không thiên vị độ dài máy móc mà vẫn chọn đáp án ngắn hơn nếu súc tích và đúng trọng tâm. Cỡ mẫu đánh giá (5 cặp, 10 câu) còn khiêm tốn, do đó trong môi trường sản xuất thực tế cần thiết lập cơ chế Human-in-the-loop đối với các ca bất đồng thuận hoặc có mức độ rủi ro cao.
- Nguồn/AI hỗ trợ nếu có: Sử dụng Antigravity hỗ trợ phân tích định dạng schema prompt JSON, công thức thống kê Cohen's Kappa và đối chiếu hợp đồng kiểm thử test\_report\_contracts.py; toàn bộ code thực thi, chạy batch OpenAI API và phân tích kết quả thực nghiệm đều được tự thực hiện và kiểm chứng 100% qua pytest và check\_lab.py.

### Nguyễn Thu Trang-2A202602435

- Phần được giao và phần thực tế đã làm: Phụ trách toàn diện Phase C: Lập trình phát hiện và ẩn danh PII tiếng Việt với Presidio (pii\_scan), tích hợp NeMo Guardrails kiểm duyệt đầu vào và đầu ra bất đồng bộ (check\_input\_rail, check\_output\_rail), thực thi bộ kiểm thử 28 câu hỏi (run\_adversarial\_suite), đo đạc độ trễ P95 tuần tự 2 tầng với warm-up thật (measure\_p95\_latency), và hoàn thiện báo cáo kiến trúc Blueprint CI/CD (blueprint.md).
- Bằng chứng: Commit aef06bc ("feat(phase-c): implement guardrails stack, p95 benchmark and blueprint report (2A202602435)"), Commit c49a42e ("hoàn thiện src/phase\_c\_guard.py"), Pull Request PR-3 (nhánh phase\_c -> main), file kết quả thực nghiệm live artifacts/guard\_results.json (20/20 attacks blocked, 8/8 clean allowed, 4/4 outputs đúng), code sạch không TODO trong src/phase\_c\_guard.py, guardrails/config.yml và báo cáo doc/reports/blueprint.md.
- Cách kiểm tra: Chạy pytest tests/test\_phase\_c.py -v (12/12 pass) và chạy thực nghiệm trực tiếp python scripts/run\_experiments.py --phase c --live --latency-samples 20 đạt 28/28 passed (100%).
- Điều đã học: Nắm vững kiến trúc bảo vệ đa tầng trong hệ thống RAG doanh nghiệp, cách thiết lập Colang 1.0 và cơ chế tự kiểm tra của NeMo Guardrails, phương pháp đo độ trễ P95/P99 theo nội suy tuyến tính, và cách thiết lập các quality gates trong CI/CD.
- Giới hạn/cách xử lý: Ban đầu mô hình spaCy tiếng Anh nhận diện nhầm văn bản tiếng Việt thành PERSON gây chặn nhầm câu sạch (false positive); đã giải quyết triệt để bằng cách lọc đúng tập thực thể mục tiêu TARGET\_PII\_ENTITIES, đưa tỷ lệ false positive về 0.0%. Xử lý prompt trong NeMo Guardrails để phân biệt câu hỏi chính sách mật khẩu hợp lệ với yêu cầu lấy mật khẩu admin.
- Nguồn/AI hỗ trợ nếu có: Sử dụng Antigravity hỗ trợ phân tích traceback môi trường, gỡ lỗi nhận diện thực thể PII và tối ưu prompt; toàn bộ code và kết quả đo đạc đều được tự chạy và kiểm chứng 100% qua pytest và check\_lab.py.

## Cập nhật sau khi nộp

Bản nộp chính thức đúng hạn ngày 12/10/2026; không có cập nhật chỉnh sửa sau deadline.
