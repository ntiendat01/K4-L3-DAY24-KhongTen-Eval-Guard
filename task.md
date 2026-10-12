# Kế Hoạch Làm Việc Song Song (Zero-Dependency Parallel Plan)
## K4 Track 3A — Production Eval + Guardrail Stack

> **Mục tiêu cốt lõi:** Phân chia thành **3 Track độc lập hoàn toàn** từ đầu đến cuối. Cả 3 thành viên bắt tay vào làm ngay cùng một lúc (từ phút thứ 0), không ai phải chờ ai, không sửa chung file, không xung đột Git (Merge Conflict-Free).

---

## 1. Ma Trận Độc Lập Tuyệt Đối (Zero-Dependency Matrix)

| Tiêu chí | Track 1 (Thành viên 1) | Track 2 (Thành viên 2) | Track 3 (Thành viên 3) |
|---|---|---|---|
| **Chủ đề** | **Phase A — RAGAS & Phân tích lỗi RAG** | **Phase B — LLM Judge & Đo lường Bias** | **Phase C — Guardrails, Latency & Blueprint** |
| **File Code duy nhất** | `src/phase_a_ragas.py` | `src/phase_b_judge.py` | `src/phase_c_guard.py` |
| **File Unit Test riêng** | `tests/test_phase_a.py` | `tests/test_phase_b.py` | `tests/test_phase_c.py` |
| **Lệnh Test độc lập** | `pytest tests/test_phase_a.py -q` | `pytest tests/test_phase_b.py -q` | `pytest tests/test_phase_c.py -q` |
| **Lệnh Live Runner riêng** | `python scripts/run_experiments.py --phase a --live` | `python scripts/run_experiments.py --phase b --live` | `python scripts/run_experiments.py --phase c --live --latency-samples 20` |
| **Nguồn dữ liệu đầu vào** | `artifacts/answers_50q.json` | `fixtures/human_labels_10q.json`<br>`fixtures/test_set_50q.json` | `fixtures/adversarial_20.json`<br>`fixtures/clean_set_8.json`<br>`fixtures/output_set_4.json` |
| **Artifact JSON đầu ra** | `artifacts/ragas_50q.json` | `artifacts/judge_results.json` | `artifacts/guard_results.json` |
| **Báo cáo Markdown riêng** | `doc/reports/failure_clusters.md` | `doc/reports/bias_report.md` | `doc/reports/blueprint.md` |
| **Điểm mục tiêu** | **30đ** *(+4đ bonus)* | **35đ** *(+3đ bonus)* | **35đ** *(+3đ bonus)* |

---

## 2. Chi Tiết Nhiệm Vụ Của Từng Track (Chạy Song Song 100%)

### 🔵 TRACK 1: Thành viên 1 — RAG Evaluation & Failure Clusters (Phase A)
*Bắt đầu ngay: Code hoàn toàn trong `src/phase_a_ragas.py`, test độc lập bằng `test_phase_a.py`.*

1. **Chuẩn bị dữ liệu answers:**
   - Đảm bảo 5 module `m1`–`m5` có trong `src/` và chạy lệnh tạo answers:
     ```bash
     python scripts/setup_answers.py --live
     ```
   - Xác nhận có `artifacts/answers_50q.json` (50 câu: 20 factual, 20 multi-hop, 10 adversarial).
2. **Implement Tasks 1–4 trong `src/phase_a_ragas.py`:**
   - `group_by_distribution(test_set)`: Chia đúng 20 factual, 20 multi-hop, 10 adversarial.
   - `run_ragas_50q(answers, m4_eval)`: Gọi `m4.evaluate_ragas`, trả về `RagasEvaluation` chứa 50 results với 4 metrics [0,1] và `evaluator_models`.
   - `bottom_10(results)`: Lấy 10 câu có `avg_score` thấp nhất, viết `diagnosis` và `suggested_fix` bám sát nội dung lỗi (retrieval thiếu/sai, hallucination, bẫy version).
   - `cluster_analysis(results)`: Tạo ma trận phân bố lỗi; xác định nhóm yếu nhất bằng **mean `avg_score` thấp nhất**.
3. **Kiểm thử & Báo cáo:**
   - Chạy test offline: `pytest tests/test_phase_a.py -q`
   - Chạy live batch: `python scripts/run_experiments.py --phase a --live`
   - Điền báo cáo [doc/reports/failure_clusters.md](file:///v:/K4-Track-3A-Day24-Eval-Guard/doc/reports/failure_clusters.md).
   - *Bonus (+4đ):* Phân tích rõ vì sao adversarial mean < factual mean.

---

### 🟢 TRACK 2: Thành viên 2 — LLM Judge, Swap Agreement & Bias (Phase B)
*Bắt đầu ngay: Code hoàn toàn trong `src/phase_b_judge.py`, test độc lập bằng `test_phase_b.py`. Không phụ thuộc code Phase A hay Phase C.*

1. **Implement Tasks 5–8 trong `src/phase_b_judge.py`:**
   - `pairwise_judge(q, A, B, reference)`: So sánh 2 answers, trả về JSON với `winner` (A/B/tie), `reasoning`, `scores` [0,1].
   - `swap_and_average(q, A, B, reference)`: Gọi 2 lượt (A/B và B/A), ánh xạ lượt 2 về danh tính ban đầu. Nếu 2 lượt đồng thuận thì giữ, nếu bất đồng thì `tie`.
   - `cohen_kappa(judge_labels, human_labels)`: Tính hệ số Cohen's $\kappa = \frac{p_o - p_e}{1 - p_e} \in [-1, 1]$ trên 10 nhãn nhị phân.
   - `bias_report(pairwise_results)`: Tính `position_bias_rate` và `verbosity_bias` (chỉ tính trên decisive cases, bỏ qua `tie`).
2. **Đảm bảo tính độc lập dữ liệu:**
   - Unit test `test_phase_b.py` dùng mock judge, pass 100% không cần mạng hay dữ liệu từ Phase khác.
   - Chạy live: Đọc 5 cặp ID `1, 5, 12, 21, 41` và 10 binary judgments với `fixtures/human_labels_10q.json` (không leak nhãn người vào prompt).
3. **Kiểm thử & Báo cáo:**
   - Chạy test offline: `pytest tests/test_phase_b.py -q`
   - Chạy live batch: `python scripts/run_experiments.py --phase b --live`
   - Điền báo cáo [doc/reports/bias_report.md](file:///v:/K4-Track-3A-Day24-Eval-Guard/doc/reports/bias_report.md).
   - *Bonus (+3đ):* Đạt $\kappa > 0.6$ trên 10 câu nhãn người.

---

### 🟡 TRACK 3: Thành viên 3 — Guardrails, Benchmark P95 & Blueprint (Phase C + Task 13)
*Bắt đầu ngay: Dữ liệu của Phase C nằm sẵn trong `fixtures/` (20 attacks, 8 clean controls, 4 outputs), hoàn toàn không cần RAG pipeline hay answers!*

1. **Implement Tasks 9a–12 trong `src/phase_c_guard.py`:**
   - `pii_scan(text)`: Quét và ẩn danh PII (CCCD/CMND, SĐT VN, email) dùng Presidio Analyzer & Anonymizer.
   - `check_input_rail(text)`: Gọi NeMo input rail async theo policy `guardrails/config.yml`.
   - `run_adversarial_suite(inputs, ...)`: Chạy batch 28 inputs (20 attacks + 8 clean inputs). Giữ nguyên `expected` để đối chiếu, chặn $\ge 15/20$ attacks và cho qua $8/8$ clean controls.
   - `check_output_rail(answer)`: Kiểm tra 4 outputs có sẵn: giữ nguyên output an toàn, redact output vi phạm.
   - `measure_p95_latency(samples, ...)`: Đo latency tuần tự 2 tầng input (Presidio $\rightarrow$ NeMo input): warm-up thật 2 lượt, đo $\ge 20$ timed samples bằng `perf_counter()`, tính P50/P95/P99 bằng nội suy tuyến tính.
2. **Kiểm thử & Báo cáo:**
   - Chạy test offline: `pytest tests/test_phase_c.py -q`
   - Chạy live batch: `python scripts/run_experiments.py --phase c --live --latency-samples 20`
   - *Bonus (+3đ):* Chặn $\ge 18/20$ attacks và vẫn cho qua 8/8 clean inputs.
3. **Task 13 — Blueprint & Quản lý nộp bài:**
   - Điền kiến trúc, CI/CD gates, monitoring vào [doc/reports/blueprint.md](file:///v:/K4-Track-3A-Day24-Eval-Guard/doc/reports/blueprint.md).
   - Phối hợp lấy nội dung của TV1, TV2 điền mục `INDIVIDUAL` vào [doc/TEAM.md](file:///v:/K4-Track-3A-Day24-Eval-Guard/doc/TEAM.md).

---

## 3. Quy Trình Làm Việc Song Song 4 Bước (Không Ai Chờ Ai)

```
Thời gian     Track 1 (TV 1)               Track 2 (TV 2)               Track 3 (TV 3)
   │
Phút 00-30    [Code phase_a_ragas.py]      [Code phase_b_judge.py]      [Code phase_c_guard.py]
   │          Test: pytest test_phase_a    Test: pytest test_phase_b    Test: pytest test_phase_c
   │          (Dùng mock của m4)           (Dùng mock judge)            (Dùng mock rails)
   │
Phút 30-60    [Sinh answers & Run Live A]  [Run Live B]                 [Run Live C & Đo Latency]
   │          scripts/run_experiments.py   scripts/run_experiments.py   scripts/run_experiments.py
   │          --phase a --live             --phase b --live             --phase c --live
   │
Phút 60-80    [Viết Báo Cáo A]             [Viết Báo Cáo B]             [Viết Blueprint C]
   │          doc/reports/                 doc/reports/                 doc/reports/
   │          failure_clusters.md          bias_report.md               blueprint.md
   │
Phút 80-90    [Điền TEAM.md cá nhân]       [Điền TEAM.md cá nhân]       [Tổng hợp & Chạy Check Lab]
   ▼          Commit & Push branch         Commit & Push branch         python scripts/check_lab.py --with-tests
```

### Điểm mấu chốt để không nghẽn ở bước Live Run:
- **Track 3 (Phase C):** Hoàn toàn tự chủ từ phút 0 đến phút 90 vì dùng fixtures độc lập (`adversarial_20.json`, `clean_set_8.json`).
- **Track 2 (Phase B):** 10 câu Binary Calibration chạy trên `fixtures/human_labels_10q.json` có sẵn. Chỉ 5 cặp Pairwise cần lấy Candidate A từ `artifacts/answers_50q.json` (do Track 1 cung cấp sau phút 30). Trong lúc chờ, TV 2 đã hoàn thành code, pass test và chuẩn bị xong template báo cáo.
- **Track 1 (Phase A):** Code xong Task 1-4, chạy sinh answers rồi đánh giá RAGAS trực tiếp.

---

## 4. Chiến Lược Git Không Xung Đột (Merge-Conflict Free)

Mỗi thành viên làm việc trên một branch riêng hoặc commit trên tập file tách biệt:

- **Branch `feature/phase-a` (TV 1):**
  - Chỉ commit: `src/phase_a_ragas.py`, `artifacts/answers_50q.json`, `artifacts/ragas_50q.json`, `doc/reports/failure_clusters.md`.
- **Branch `feature/phase-b` (TV 2):**
  - Chỉ commit: `src/phase_b_judge.py`, `artifacts/judge_results.json`, `doc/reports/bias_report.md`.
- **Branch `feature/phase-c` (TV 3):**
  - Chỉ commit: `src/phase_c_guard.py`, `artifacts/guard_results.json`, `doc/reports/blueprint.md`.
- **Branch `main` (Merge & Submit):**
  - Merge cả 3 branches vào `main`. Vì không ai sửa chung bất kỳ file nào, **tỷ lệ conflict là 0%**!
  - Cả 3 cùng mở [doc/TEAM.md](file:///v:/K4-Track-3A-Day24-Eval-Guard/doc/TEAM.md) để điền phần `INDIVIDUAL` của mình.
  - Chạy lệnh kiểm tra tổng thể cuối cùng:
    ```bash
    python scripts/check_lab.py --with-tests
    ```
  - Trưởng nhóm push commit cuối cùng và nộp link repo chung.
