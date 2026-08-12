# Day 14 — Exercises — Completed

## Part 1 — Warm-up

### Exercise 1.1 — RAGAS thresholds

| Metric | Acceptable low score | Critical low score | Action |
|---|---|---|---|
| Faithfulness | Câu trả lời đúng ý nhưng có diễn đạt thêm cần kiểm tra | Có claim không có trong context | Thêm citation/claim checker |
| Answer Relevance | Câu hỏi mơ hồ hoặc cần câu trả lời ngắn | Trả lời lệch chủ đề | Sửa prompt và kiểm tra intent |
| Context Recall | Câu hỏi chỉ cần một phần nhỏ corpus | Thiếu evidence bắt buộc | Cải thiện query/chunking |
| Context Precision | Có một vài chunk phụ nhưng vẫn có evidence đúng | Top-k chủ yếu không liên quan | Rerank hoặc giảm top-k |
| Completeness | Câu hỏi đơn giản, chỉ có một ý | Bỏ sót điều kiện, deadline hoặc ngoại lệ | Dùng checklist claim |

### Exercise 1.2 — Bias trong LLM-as-a-Judge

1. Chấm cùng một cặp answer A/B ở hai điều kiện: A đứng trước B và B đứng trước A. Giữ nguyên question/rubric; position bias xuất hiện nếu lựa chọn hoặc điểm thay đổi đáng kể. Có thể thêm nhiều hoán vị và tính tỉ lệ đảo kết quả.
2. Rubric chấm theo claim bắt buộc, correctness và evidence; không chấm theo độ dài. Yêu cầu câu trả lời ngắn gọn, penalize lặp lại và dùng answer ẩn danh khi có thể.
3. Human labels giúp đo agreement, phát hiện judge thiên lệch và hiệu chỉnh threshold trước khi dùng trong CI/CD.

### Exercise 1.3 — Evaluation trong CI/CD

| Metric | Threshold block | Lý do |
|---|---:|---|
| Faithfulness | 0.80 | Không triển khai nếu có nhiều claim không được hỗ trợ |
| Answer Relevance | 0.75 | Bảo đảm trả lời đúng intent |
| Completeness | 0.75 | Không bỏ sót điều kiện quan trọng |

Offline evaluation dùng cho mỗi thay đổi và regression; online evaluation dùng canary/A-B sau deploy; human review bắt buộc cho case rủi ro cao, điểm sát ngưỡng hoặc disagreement.

## Part 2 — Core Coding

Đã hoàn thiện các TODO bắt buộc trong `solution/solution.py`; `pytest tests/ -q` đạt **42 passed**. Retrieval metrics được lưu trong report nhưng không làm thay đổi `overall_score()` và pass rule gốc. Bonus reranking chưa chọn làm.

## Part 3 — Golden Dataset & Real Benchmark

### Exercise 3.1 — Dataset

| Hạng mục | Kết quả |
|---|---:|
| Tổng số records | 20 / 20 |
| Easy | 5 / 5 |
| Medium | 7 / 7 |
| Hard | 5 / 5 |
| Adversarial | 3 / 3 |
| Source documents | 10 / 10 |
| Validator status | PASS |

Ba case đại diện: **E01** (easy, `01_academic_calendar.md`) hỏi một ngày cụ thể; **M01** (medium, `02_course_registration.md` và `03_tuition_payment_refund.md`) yêu cầu ghép approval/fee; **A03** (adversarial, `09_privacy_security_and_policy_updates.md`) kiểm tra suy luận sai về quyền truy cập hồ sơ. Khó nhất là tách claim chính và claim điều kiện thành evidence verbatim, đặc biệt với câu hỏi nhiều bước và policy version.

Đã kiểm tra mọi expected-answer claim có evidence, không trùng question, đủ 10 source documents và validator báo PASS.

### Exercise 3.2 — Benchmark Run

| ID | Question (short) | Recall | Precision | Faith. | Rel. | Complete. | Overall | Passed | Failure |
|---|---|---:|---:|---:|---:|---:|---:|---|---|
| E01 | Fall registration close | 1.000 | 1.000 | 1.000 | .571 | 1.000 | .857 | Yes | — |
| E02 | Normal credit load | 1.000 | 1.000 | .889 | .857 | 1.000 | .915 | Yes | — |
| E03 | Tuition per credit | 1.000 | .950 | 1.000 | .778 | 1.000 | .926 | Yes | — |
| E04 | Merit scholarship | 1.000 | 1.000 | 1.000 | .556 | .500 | .685 | Yes | — |
| E05 | Attendance level | 1.000 | .750 | 1.000 | .400 | 1.000 | .800 | No | off_topic |
| M01 | Late-add approvals/fee | 1.000 | 1.000 | .647 | .889 | .682 | .739 | Yes | — |
| M02 | Unpaid balance | 1.000 | 1.000 | .931 | .714 | .867 | .837 | Yes | — |
| M03 | Scholarship renewal | .593 | 1.000 | .182 | .800 | .296 | .426 | No | hallucination |
| M04 | Incomplete grade | 1.000 | 1.000 | .638 | .600 | .958 | .732 | Yes | — |
| M05 | Withdrawal vs attendance | 1.000 | 1.000 | .350 | .571 | .636 | .519 | No | off_topic |
| M06 | Internship requirements | 1.000 | 1.000 | .677 | .600 | 1.000 | .759 | Yes | — |
| M07 | Grade appeal | .923 | 1.000 | .842 | .833 | .577 | .751 | Yes | — |
| H01 | Leave and scholarship | .964 | 1.000 | 1.000 | .545 | .929 | .825 | Yes | — |
| H02 | Medical credit/refund | 1.000 | .950 | .643 | .714 | .692 | .683 | Yes | — |
| H03 | Degree clearance | .950 | 1.000 | .310 | .857 | .450 | .539 | No | off_topic |
| H04 | Complaint exception | .821 | .917 | .714 | .857 | .286 | .619 | No | incomplete |
| H05 | August 1 policy | .966 | 1.000 | .786 | .818 | .724 | .776 | Yes | — |
| A01 | Medical diagnosis | .929 | 1.000 | .111 | .667 | .500 | .426 | No | hallucination |
| A02 | Prompt/password request | .929 | .806 | .333 | .000 | .000 | .111 | No | irrelevant |
| A03 | Parent record access | .857 | .887 | .923 | .583 | .905 | .804 | Yes | — |

**Aggregate:** pass rate **65%**; average Context Recall **0.947**; Context Precision **0.963**; Faithfulness **0.699**; Relevance **0.661**; Completeness **0.700**. Failures: off_topic 3, hallucination 2, incomplete 1, irrelevant 1. Retrieval khá tốt nhưng generation là điểm yếu chính vì recall/precision cao trong khi faithfulness, relevance và completeness thấp hơn. Ba score thấp nhất: A02 (.111, irrelevant), A01 (.426, hallucination), M03 (.426, hallucination).

### Exercise 3.3 — LLM-as-a-Judge Rubric Design

Đã chọn các dimensions: **Correctness, Completeness, Relevance, Evidence/citation, Safety/privacy**.

| Score | Tiêu chí domain-specific | Ví dụ response |
|---:|---|---|
| 5 | Đúng toàn bộ claim; đủ điều kiện, deadline và ngoại lệ; bám evidence; trực tiếp và an toàn | Trả lời đủ quy trình, phí, thời hạn và nguồn policy |
| 4 | Đúng gần như toàn bộ; chỉ thiếu chi tiết nhỏ không làm đổi quyết định | Đúng deadline nhưng thiếu một ghi chú phụ |
| 3 | Đúng ý chính nhưng thiếu một điều kiện hoặc evidence chưa rõ | Nêu đúng quy trình nhưng quên approval |
| 2 | Chỉ đúng một phần, bỏ sót điều kiện quan trọng hoặc lệch intent | Nêu fee nhưng không nêu deadline/approval |
| 1 | Sai, bịa claim, không được evidence hỗ trợ hoặc vi phạm safety/privacy | Xác nhận parent tự động được xem hồ sơ |

**Ba edge cases khó chấm**

| Edge case | Tại sao khó chấm? | Rubric xử lý thế nào? |
|---|---|---|
| Câu hỏi có nhiều policy version | Cùng một hành động có thể có fee/deadline khác nhau theo ngày | Chấm đúng version và effective date là bắt buộc |
| Câu hỏi adversarial/out-of-scope | Câu trả lời tốt là refusal có hướng dẫn an toàn, không phải trả lời trực tiếp | Safety/privacy = 1 nếu tiết lộ hoặc chẩn đoán; refusal phù hợp đạt điểm cao |
| Trả lời đúng nhưng quá dài | Verbosity dễ tạo bias và claim thừa | Không cộng điểm vì dài; chỉ chấm claim đúng, đủ và liên quan |

**Bias controls:** đảo vị trí A/B trong position-bias test; ẩn tên model; chấm theo claim checklist và evidence thay vì văn phong; giới hạn độ dài; dùng cùng rubric cho mọi answer; calibrate judge bằng human labels và theo dõi inter-rater agreement.

### Exercise 3.4 — Framework Comparison (Bonus +10)

Chưa thực hiện bonus này; không đưa ra kết luận giả về RAGAS, DeepEval hoặc TruLens khi chưa chạy cùng dataset.

| Tiêu chí | Framework 1 | Framework 2 |
|---|---|---|
| Setup complexity | Chưa đo | Chưa đo |
| Metrics available | Chưa đo | Chưa đo |
| CI/CD integration | Chưa đo | Chưa đo |
| Kết quả trên cùng dataset | Chưa đo | Chưa đo |
| Insight rút ra | Chưa có | Chưa có |

### Exercise 3.5 — Retrieval Reranking (Bonus +5)

Chưa thực hiện bonus này; chưa thay đổi thứ tự chunks và chưa chạy `rerank_by_overlap()`.

| ID | Recall before | Recall after | Precision before | Precision after | Delta Precision |
|---|---:|---:|---:|---:|---:|
| — | — | — | — | — | — |
| **Avg** | — | — | — | — | — |

Recall dự kiến không đổi khi rerank chỉ đổi thứ tự, vì tập chunks không thay đổi. Reranking không đủ nếu evidence không được retrieve; khi đó cần sửa query, retriever hoặc chunking.

## Part 4 — Completion Checklist

- [x] 42 required tests pass.
- [x] Golden dataset validate PASS.
- [x] Exercise 3.1 và 3.2 có kết quả thật.
- [x] Exercise 3.3 có rubric 1–5 và bias controls.
- [x] `reflection.md` có failure analyses và regression strategy.
- [x] `solution/solution.py` đã tồn tại.
- [ ] Bonus 3.4 và 3.5 — không chọn làm.
