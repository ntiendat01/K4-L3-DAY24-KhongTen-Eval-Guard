# Nhóm thực hiện K4 Track 3A

Tên nhóm: Không Tên

Repo chung: https://github.com/ntiendat01/K4-Track-3A-Day24-Eval-Guard

Trưởng nhóm/người nộp: Nguyễn Tiến Đạt - 2A202602606

Ngày lab/deadline áp dụng: 2026-10-12 23:59 — Asia/Ho_Chi_Minh (UTC+07:00)

Commit dùng để chấm: {{Commit SHA}}

Mỗi nhóm nộp một repo. Điền đủ thành viên và một mục INDIVIDUAL cho từng người.
Thêm/xóa rows để khớp nhóm thật; mẫu ba rows không ép nhóm phải có ba người.

## Thành viên và phân công

| STT | Họ và tên       | MSSV        | Vai trò / công việc thật                                                                     | Bằng chứng           |
| --- | --------------- | ----------- | -------------------------------------------------------------------------------------------- | -------------------- |
| 1   | {{Họ tên}}      | {{MSSV}}    | {{Phase/file/hàm}}                                                                           | {{Commit/PR/report}} |
| 2   | Nguyễn Tiến Đạt | 2A202602606 | *Phụ trách Phase B (LLM-as-Judge, Pairwise Swap-and-Average, Cohen's Kappa & Bias Analysis)* | PR-1                 |
| 3 | Nguyễn Thu Trang | 2A202602435 | Phụ trách Phase C (Presidio PII, NeMo Guardrails, Benchmark Latency P95 & Blueprint CI/CD) | Commit aef06bc, PR-3 (branch phase_c), artifacts/guard_results.json, doc/reports/blueprint.md |

Người tích hợp/chạy batch/nộp link: {{Họ tên, công việc và bằng chứng}}.

## Môi trường và chuẩn bị trước lớp

OS/Python/compiler: macOS / Windows 11, Python 3.11.16, Apple Clang / MSVC C++ Build Tools.
Virtualenv/dependencies/imports: Tạo .venv với Python 3.11, cài đặt pyarrow<18 tương thích numpy 1.26.4, đã kiểm tra lệnh python -c "import ragas, nemoguardrails, presidio_analyzer, spacy; print('imports OK')" thành công.
Models/Qdrant/50 answers đã chuẩn bị: Tải mô hình spaCy en_core_web_lg, tải all-MiniLM-L6-v2 cho NeMo Guardrails, chạy Qdrant Docker v1.9.7 và sinh đầy đủ 50 câu trả lời RAG trong artifacts/answers_50q.json.
Nếu cài full NeMo chưa thành công, ghi lỗi thực tế; không dùng unit test pass
hoặc pip resolve để khẳng định runtime đã chạy.

## INDIVIDUAL

Sao chép mẫu cho mỗi thành viên; tiêu đề họ tên không dấu-MSSV.

### {{HoVaTen}}-{{MSSV}}

- Phần được giao và phần thực tế đã làm: {{Nhiệm vụ, file/hàm, mục đích}}.
- Bằng chứng: {{Commit/PR và artifacts/reports liên quan}}.
- Cách kiểm tra: {{Unit mock hoặc live experiment, lệnh và kết quả}}.
- Điều đã học: {{Khái niệm và cách thể hiện trong bài}}.
- Giới hạn/cách xử lý: {{Vấn đề và kiểm chứng}}.
- Nguồn/AI hỗ trợ nếu có: {{Công cụ, phạm vi, cách tự kiểm chứng}}.

### Nguyễn Thu Trang-2A202602435

- Phần được giao và phần thực tế đã làm: Phụ trách toàn diện Phase C: Lập trình phát hiện và ẩn danh PII tiếng Việt với Presidio (pii_scan), tích hợp NeMo Guardrails kiểm duyệt đầu vào và đầu ra bất đồng bộ (check_input_rail, check_output_rail), thực thi bộ kiểm thử 28 câu hỏi (run_adversarial_suite), đo đạc độ trễ P95 tuần tự 2 tầng với warm-up thật (measure_p95_latency), và hoàn thiện báo cáo kiến trúc Blueprint CI/CD (blueprint.md).
- Bằng chứng: Commit aef06bc ("feat(phase-c): implement guardrails stack, p95 benchmark and blueprint report (2A202602435)"), Commit c49a42e ("hoàn thiện src/phase_c_guard.py"), Pull Request PR-3 (nhánh phase_c -> main), file kết quả thực nghiệm live artifacts/guard_results.json (20/20 attacks blocked, 8/8 clean allowed, 4/4 outputs đúng), code sạch không TODO trong src/phase_c_guard.py, guardrails/config.yml và báo cáo doc/reports/blueprint.md.
- Cách kiểm tra: Chạy pytest tests/test_phase_c.py -v (12/12 pass) và chạy thực nghiệm trực tiếp python scripts/run_experiments.py --phase c --live --latency-samples 20 đạt 28/28 passed (100%).
- Điều đã học: Nắm vững kiến trúc bảo vệ đa tầng trong hệ thống RAG doanh nghiệp, cách thiết lập Colang 1.0 và cơ chế tự kiểm tra của NeMo Guardrails, phương pháp đo độ trễ P95/P99 theo nội suy tuyến tính, và cách thiết lập các quality gates trong CI/CD.
- Giới hạn/cách xử lý: Ban đầu mô hình spaCy tiếng Anh nhận diện nhầm văn bản tiếng Việt thành PERSON gây chặn nhầm câu sạch (false positive); đã giải quyết triệt để bằng cách lọc đúng tập thực thể mục tiêu TARGET_PII_ENTITIES, đưa tỷ lệ false positive về 0.0%. Tinh chỉnh prompt trong guardrails/config.yml để phân biệt câu hỏi chính sách mật khẩu hợp lệ với yêu cầu lấy mật khẩu admin.
- Nguồn/AI hỗ trợ nếu có: Sử dụng Antigravity hỗ trợ phân tích traceback môi trường, gỡ lỗi nhận diện thực thể PII và tối ưu prompt; toàn bộ code và kết quả đo đạc đều được tự chạy và kiểm chứng 100% qua pytest và check_lab.py.

## Cập nhật sau khi nộp

Bản nộp chính thức đúng hạn ngày 12/10/2026; không có cập nhật chỉnh sửa sau deadline.
