# Đánh giá LLM Judge Phase B

Nhóm: K4-Track-3A-Day24

Người phụ trách: Nguyễn Tiến Đạt - 2A202602606

Nguồn: `artifacts/judge_results.json`; run timestamp: 2026-10-12T08:22:51.522873+00:00.
Models generation/judge: Generation: gpt-4o-mini-2024-07-18; Judge: gpt-4o-mini.

## Năm cặp A/B và hai lượt

Nguồn cố định: A từ artifacts/answers\_50q.json:answer; B từ
fixtures/human\_labels\_10q.json:model\_answer; reference từ fixtures/test\_set\_50q.json:ground\_truth.
Ids: 1, 5, 12, 21, 41. Không đổi nguồn sau khi xem kết quả judge.

| ID | Candidate A (Pipeline)                                                                | Candidate B (Supplied)                                                             | Pass 1 | Pass 2 (quy về gốc) | Final Winner | Consistent | Scores Pass 1 (A/B) | Scores Pass 2 (A/B) |
| -- | ------------------------------------------------------------------------------------- | ---------------------------------------------------------------------------------- | ------ | ------------------- | ------------ | ---------- | ------------------- | ------------------- |
| 1  | Nhân viên được nghỉ 3 ngày làm việc khi kết hôn.                                      | Nhân viên được nghỉ 3 ngày làm việc có lương khi kết hôn.                          | B      | B                   | B            | True       | 0.8 / 1.0           | 0.8 / 1.0           |
| 5  | Cần thêm phê duyệt của Kế toán trưởng.                                                | Cần Giám đốc phòng ban phê duyệt.                                                  | B      | tie                 | tie          | False      | 0.2 / 0.8           | 0.0 / 0.0           |
| 12 | Thưởng Tết tối thiểu cho nhân viên chính thức có từ 6 tháng trở lên là 1 tháng lương. | Nhân viên được thưởng Tết tối thiểu 1 tháng lương.                                 | A      | A                   | A            | True       | 1.0 / 0.7           | 1.0 / 0.8           |
| 21 | Nhân viên 9 năm thâm niên được 18 ngày phép (15 + 3)... Không tìm thấy.               | Được nghỉ 18 ngày phép (15 cơ bản + 3 thâm niên). Lương Senior từ 20 đến 35 triệu. | B      | B                   | B            | True       | 0.7 / 1.0           | 0.7 / 1.0           |
| 41 | Nhân viên chính thức được hưởng 15 ngày phép năm.                                     | Nhân viên được nghỉ 12 ngày phép năm.                                              | A      | A                   | A            | True       | 1.0 / 0.0           | 1.0 / 0.3           |

Position bias count/rate: 1 / 5 (20.0%, rate: 0.2).
Verbosity win rate: 75.0% (0.75); decisive cases: 4 (trên tổng số 5 cặp).

### Giải thích các case không nhất quán và các winner dài hơn

- **Case không nhất quán (ID 5):** Ở lượt 1, LLM chọn B vì nghĩ Giám đốc phòng ban phê duyệt là đủ; nhưng ở lượt 2 (sau khi swap), LLM nhận diện cả hai câu đều sai vì thiết bị 55 triệu vượt ngưỡng 50 triệu phải do Tổng Giám đốc duyệt, dẫn đến kết quả hòa (`tie`). Cơ chế swap-and-average đã phát huy tác dụng tuyệt đối khi loại bỏ phán quyết sai lệch ở lượt 1 và đưa kết quả về `tie`.
- **Phân tích Verbosity Bias:** Trong 4 trường hợp phân định rõ thắng thua, có 3 trường hợp câu trả lời dài hơn giành chiến thắng (ID 1, ID 12, ID 41) và 1 trường hợp câu ngắn hơn thắng (ID 21). Đáng chú ý ở ID 21, câu A dài hơn (193 ký tự) do trích xuất thừa thông tin không liên quan, còn câu B ngắn hơn (86 ký tự) nhưng đủ cả 2 ý (phép và lương), và LLM Judge đã chấm B thắng. Điều này cho thấy LLM Judge không chỉ thiên vị độ dài một cách máy móc mà thực sự ưu tiên độ chính xác và tính súc tích khi thông tin dài bị lan man.

## Calibration đúng model\_answer

Helper đánh giá đúng question/model\_answer đang có trong fixtures/human\_labels\_10q.json;
không lấy answer mới của pipeline. Prompt chỉ nhận question, answer, reference.
Không gửi human\_label, human\_note hoặc phân bố nhãn human vào prompt.

| Question ID | Model Answer Gốc                                                                   | Nhãn Judge | Nhãn Human | Đồng Thuận (Agree) | Lý Do Của Judge                                                                            |
| ----------- | ---------------------------------------------------------------------------------- | ---------- | ---------- | ------------------ | ------------------------------------------------------------------------------------------ |
| 1           | Nhân viên được nghỉ 3 ngày làm việc có lương khi kết hôn.                          | 1          | 1          | True               | Câu trả lời chính xác theo reference, nêu rõ số ngày nghỉ và điều kiện có lương.           |
| 5           | Cần Giám đốc phòng ban phê duyệt.                                                  | 0          | 0          | True               | Sai vì đơn hàng trên 50 triệu cần Tổng Giám đốc phê duyệt, không phải Giám đốc phòng ban.  |
| 12          | Nhân viên được thưởng Tết tối thiểu 1 tháng lương.                                 | 1          | 1          | True               | Câu trả lời chính xác theo reference cho nhân viên chính thức từ 6 tháng trở lên.          |
| 21          | Được nghỉ 18 ngày phép (15 cơ bản + 3 thâm niên). Lương Senior từ 20 đến 35 triệu. | 1          | 1          | True               | Chính xác cả về số ngày phép và khoảng lương cho nhân viên Senior 9 năm thâm niên.         |
| 23          | Hoàn trả 25 triệu vì chưa đủ 1 năm cam kết.                                        | 1          | 1          | True               | Chính xác, nêu rõ số tiền hoàn trả 100% vì nghỉ việc trước 12 tháng cam kết.               |
| 29          | Trưởng phòng phê duyệt. Phạt 2% tháng trên 8 triệu.                                | 0          | 0          | True               | Sai vì thiếu Kế toán trưởng phê duyệt và cách tính phạt phải pro-rata cho 15 ngày quá hạn. |
| 33          | Phép: 19 ngày. Phụ cấp: 1.500.000 VNĐ/tháng.                                       | 1          | 1          | True               | Cung cấp đúng cả số ngày phép và phụ cấp hàng tháng cho Manager 12 năm thâm niên.          |
| 41          | Nhân viên được nghỉ 12 ngày phép năm.                                              | 0          | 0          | True               | Sai vì sử dụng chính sách cũ v2023 thay vì chính sách mới hiệu lực v2024 (15 ngày).        |
| 46          | Nhân viên thử việc không được nghỉ phép năm và phải xin nghỉ không lương nếu cần.  | 1          | 1          | True               | Chính xác theo reference, nêu đúng quy định cho nhân viên thử việc.                        |
| 50          | Được, miễn là đảm bảo kết nối an toàn.                                             | 0          | 0          | True               | Sai vì chính sách CNTT nghiêm cấm sử dụng VPN cá nhân khi WFH.                             |

Cohen’s κ: 1.0 (Hoàn hảo — Perfect Agreement).
Observed/chance agreement và diễn giải:

- Tỷ lệ đồng thuận quan sát được ($p\_o$): 10/10 = 1.0.
- Tỷ lệ đồng thuận ngẫu nhiên ($p\_e$): $(6/10 \times 6/10) + (4/10 \times 4/10) = 0.36 + 0.16 = 0.52$.
- Hệ số $\kappa = \frac{p\_o - p\_e}{1 - p\_e} = \frac{1.0 - 0.52}{1 - 0.52} = 1.0$.
- Đánh giá theo thang đo Landis & Koch: $\kappa = 1.0$ đạt mức "Almost Perfect / Perfect Agreement", vượt xa ngưỡng yêu cầu 0.6 để đạt điểm thưởng (+3 điểm bonus).

## Giới hạn và cách dùng judge

- **Về mô hình và prompt:** Mô hình `gpt-4o-mini` thể hiện khả năng phán đoán logic xuất sắc trên tập 10 câu nhãn người, bắt được cả các bẫy phiên bản chính sách cũ/mới (ID 41) và các điều kiện phức tạp (ngưỡng phê duyệt tài chính, quy định an toàn thông tin). Prompt phân định rõ ràng tiêu chí chính xác, đầy đủ và súc tích.
- **Giới hạn cỡ mẫu:** Tập kiểm thử pairwise (5 cặp) và calibration (10 câu) có quy mô nhỏ, đủ làm cơ sở xác nhận độ tin cậy ban đầu nhưng chưa phản ánh toàn bộ các corner cases trong môi trường production lớn.
- **Chiến lược áp dụng thực tế:** Không nên dùng LLM Judge đơn lẻ mà cần kết hợp kỹ thuật hoán đổi vị trí (Swap-and-Average) để loại bỏ position bias. Đối với các trường hợp critical (liên quan đến tuân thủ pháp lý, an toàn thông tin hoặc tài chính lớn), cần thiết lập threshold tự động đẩy sang Human-in-the-loop xem xét lại khi LLM Judge có độ tự tin thấp hoặc phán quyết không nhất quán giữa hai lượt.
