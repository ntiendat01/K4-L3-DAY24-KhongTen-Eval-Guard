# Blueprint CI/CD cho RAG và guardrails

Nhóm: Không Tên

Người phụ trách: Nguyễn Thu Trang - 2A202602435

Đây là **đề xuất CI/CD**, không phải workflow đã được triển khai/chạy.
Nguồn kết quả: artifacts/ragas_50q.json, judge_results.json, guard_results.json.
Run timestamp: 2026-10-12T10:13:30.859933+00:00; guard models: ['gpt-4o-mini'].
Models generation/evaluator/judge từ A/B: Generation: gpt-4o-mini-2024-07-18; Judge: gpt-4o-mini; Guard: gpt-4o-mini.

## Kiến trúc và failure action

Input → Presidio → NeMo input → RAG → NeMo output → response.
Request thực dừng sớm khi bị chặn; khác phép đo input layers chạy đủ hai tầng.

Data flow tuần tự hoạt động như sau:
1. Người dùng gửi câu hỏi tới cổng API của hệ thống.
2. Tầng 1 (Presidio PII Analyzer & Anonymizer): Quét và phát hiện các thực thể nhạy cảm như CCCD/CMND, số điện thoại Việt Nam, địa chỉ email. Nếu phát hiện PII xâm nhập hoặc cố ý trích xuất, request bị chặn ngay lập tức mà không chuyển tiếp vào mô hình ngôn ngữ lớn (short-circuit).
3. Tầng 2 (NeMo Input Guardrail): Đánh giá nội dung câu hỏi về mặt an toàn nội dung, phát hiện prompt injection, jailbreak, giả mạo danh tính, hoặc các câu hỏi nằm ngoài phạm vi chính sách nội bộ. Nếu vi phạm, bot lập tức trả về thông điệp từ chối chuẩn mực từ rails.co.
4. Tầng 3 (RAG Pipeline): Khi vượt qua 2 lớp bảo vệ đầu vào, câu hỏi được chuyển tới bộ tìm kiếm Hybrid Search và Cross-Encoder Reranker để truy xuất ngữ cảnh chính sách phù hợp, sau đó LLM sinh câu trả lời.
5. Tầng 4 (NeMo Output Guardrail): Kiểm duyệt câu trả lời trước khi gửi tới người dùng để đảm bảo không bị rò rỉ dữ liệu mật hoặc PII.
Về bảo mật và nhật ký: Tất cả API keys và tokens được quản lý an toàn qua biến môi trường (.env / HashiCorp Vault), không lưu trữ trong mã nguồn. Dữ liệu ghi log được redact hoàn toàn các trường PII để tuân thủ quyền riêng tư. Khi hệ thống LLM gặp sự cố mạng hoặc quá tải, cơ chế fallback kích hoạt chế độ trích xuất tài liệu trực tiếp (extractive fallback) hoặc thông báo hệ thống đang bận.

## Phạm vi và số đo latency

Bắt buộc Task 12: **input_layers_sequential**, Presidio + NeMo input, không
short-circuit. Engines tạo trước đo; có hai warm-up thật không tính vào mẫu.
Percentile nội suy tuyến tính từ raw samples; tổng từ tổng từng sample.

Số mẫu timed: 20; warm-up: 2;
phương pháp: linear.

| Layer / scope | P50 ms | P95 ms | P99 ms | Tình trạng |
|---|---|---|---|---|
| Presidio | 11.55 | 14.04 | 14.33 | Đạt ngân sách (xử lý regex nội bộ cực nhanh) |
| NeMo input | 798.14 | 1338.06 | 1543.48 | Vượt ngân sách (phụ thuộc độ trễ mạng OpenAI API) |
| Input guard sequential | 808.90 | 1350.77 | 1556.43 | Vượt ngân sách 500ms (bottleneck tại NeMo input) |
| RAG | chưa đo | chưa đo | chưa đo | Ngoài phạm vi bắt buộc |
| NeMo output | chưa đo | chưa đo | chưa đo | Ngoài phạm vi bắt buộc |
| Full guard / short-circuit request | chưa đo | chưa đo | chưa đo | Không suy từ input_guard_ms |

Input-layer budget target: 500ms; kết quả thực tế/budget_ok: 1350.77ms và False.
Phân tích bottleneck: Điểm nghẽn độ trễ lớn nhất tập trung hoàn toàn tại NeMo Input Guardrail (P95 đạt 1338.06ms) do phải thực hiện lời gọi mạng tới API OpenAI (mô hình gpt-4o-mini). Ngược lại, tầng Presidio chạy cục bộ có độ trễ cực thấp (P95 chỉ 14.04ms). Do đó tổng độ trễ tuần tự hai tầng đầu vào (Input guard sequential P95 = 1350.77ms) chưa đạt mục tiêu 500ms.
Các phạm vi RAG, NeMo output và Full guard được ghi nhận "chưa đo" vì nằm ngoài yêu cầu bắt buộc của Task 12. Kế hoạch đo lường mở rộng trong tương lai sẽ thực hiện bằng cách đo end-to-end với ít nhất 50 mẫu sau 2 vòng warm-up, ghi nhận timestamp chi tiết từng chặng.

## CI gates đề xuất

- RAGAS faithfulness ≥0.75, avg_score ≥0.65 trên 50 câu.
- Attacks ≥18/20 cho CI đề xuất; clean 8/8 allowed; output cases đúng.
- Full guard P95 <500ms chỉ áp dụng sau khi thực sự đo full guard.

Mốc bắt buộc lab vẫn attacks ≥15/20; bonus ≥18/20 kèm clean controls đạt.
Đề xuất tích hợp kiểm thử tự động trong CI/CD pipeline:
1. Lệnh chạy kiểm thử đơn vị offline: `pytest tests/ -q` (bắt buộc pass 100% không dùng mạng).
2. Lệnh kiểm tra chất lượng nộp bài: `python scripts/check_lab.py --with-tests`.
Nếu bất kỳ chỉ số nào vi phạm ngưỡng (ví dụ attacks bị lọt > 2 câu, clean controls bị chặn nhầm > 0 câu, hoặc code còn chứa TODO), script sẽ trả về mã lỗi nonzero (exit code 1) khiến quy trình CI/CD fail ngay lập tức, ngăn chặn việc merge code lỗi vào nhánh main. Hiện tại đây là kiến trúc đề xuất và nhóm đã kiểm chứng tính tương thích qua bộ test contract có sẵn.

## Monitoring và cải tiến

Hệ thống giám sát vận hành cần theo dõi liên tục các chỉ số quan trọng:
1. Tỷ lệ chặn đầu vào (blocked rate) theo từng danh mục tấn công.
2. Tỷ lệ chặn nhầm (false positive rate) dựa trên phản hồi của người dùng thực tế.
3. Độ trễ P50, P95, P99 theo từng tầng guardrail và toàn bộ pipeline RAG.
4. Độ trôi dạt ngữ nghĩa (data drift / prompt drift) đối với các câu hỏi mới.
Ngưỡng cảnh báo: Kích hoạt cảnh báo PagerDuty / Slack tới đội ngũ MLOps/SRE nếu P95 latency vượt quá 2000ms trong 5 phút liên tiếp hoặc tỷ lệ chặn nhầm tăng quá 1%. Toàn bộ dữ liệu ghi log được ẩn danh và mã hóa theo tiêu chuẩn bảo mật doanh nghiệp.

RAGAS mean/worst distribution: Đang theo dõi và phân tích (dự kiến factual mean 0.85, adversarial mean 0.60).
Cohen’s κ: 1.0 (hoàn toàn đồng thuận giữa LLM Judge và nhãn con người theo artifacts/judge_results.json).
Attacks passed: 20/20; clean false_positive_rate: 0.0 (đạt 8/8 câu sạch được cho qua).

Các hành động cải tiến hệ thống dựa trên bằng chứng thực nghiệm:
1. Thay thế lời gọi OpenAI API trong NeMo input guard bằng mô hình phân loại nội bộ nhỏ gọn (như DistilBERT hoặc ModernBERT ONNX) để đưa P95 latency từ 1350.77ms xuống dưới 80ms.
2. Triển khai bộ nhớ đệm ngữ nghĩa (Semantic Cache với Redis/Qdrant) cho các câu hỏi phổ biến, cho phép bỏ qua bước kiểm tra lặp lại đối với các truy vấn an toàn đã biết.
3. Tối ưu hóa tập regex cho Presidio và bổ sung quy tắc nhận diện mã số thuế doanh nghiệp và số thẻ bảo hiểm y tế Việt Nam.
4. Tinh chỉnh ngưỡng từ chối của NeMo input guard đối với các câu hỏi về IT chính sách mật khẩu để đảm bảo không bao giờ chặn nhầm câu hỏi chính sách hợp lệ.
