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
| 3 | Nguyễn Thu Trang | 2A202602435 | Phụ trách Phase C (Presidio PII, NeMo Guardrails, Benchmark Latency P95 & Blueprint CI/CD) | branch phase_c, artifacts/guard_results.json, doc/reports/blueprint.md |

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

## Cập nhật sau khi nộp

Bản nộp chính thức đúng hạn ngày 12/10/2026; không có cập nhật chỉnh sửa sau deadline.
