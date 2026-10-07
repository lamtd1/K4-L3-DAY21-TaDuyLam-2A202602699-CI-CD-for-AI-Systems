# Báo Cáo Lab Day 21 - CI/CD cho AI Systems

| | |
|---|---|
| Họ và tên | Tạ Duy Lâm |
| MSSV | 2A202602699 |
| Lớp / Khóa | K4 |
| Repo GitHub | https://github.com/lamtd1/K4-L3-DAY21-TaDuyLam-2A202602699-CI-CD-for-AI-Systems |
| Ngày nộp | 07/10/2026 |

---

## 1. Bộ Siêu Tham Số Đã Chọn và Lý Do

| Lần chạy | n_estimators | learning_rate | max_depth | f1_score | accuracy |
|---|---|---|---|---|---|
| 1 | 50 | 0.05 | 2 | 0.6051 | 0.846 |
| 2 | 100 | 0.1 | 3 | 0.7109 | 0.878 |
| 3 | 200 | 0.1 | 5 | 0.7149 | 0.874 |

**Bộ siêu tham số đã chọn:** `n_estimators=100`, `learning_rate=0.1`, `max_depth=3`.

**Lý do:** Bộ 100/0,1/3 vượt ngưỡng 0,65 rõ ràng, còn hai bộ nhỏ thì không. Bộ 200/0,1/5 chỉ hơn 0,004 f1_score, nhiễu vì holdout chỉ có 500 mẫu, mà mô hình nặng gấp đôi. Lần có accuracy cao nhất (lần 3) không trùng lần có f1_score cao nhất (lần 4), nên accuracy không đủ đánh giá lớp thu nhập cao. Với cùng 50 cây, learning_rate thấp hơn cho f1_score thấp hơn, tức cần nhiều cây hơn để bù.

---

## 2. Vì Sao Ngưỡng Chất Lượng Đặt Trên F1 Chứ Không Phải Accuracy

Lớp thu nhập trên 50K chỉ chiếm khoảng 25% (holdout: 24,8%), nên mô hình luôn trả lời "thu nhập thấp" vẫn đạt accuracy khoảng 0,75 mà vô dụng. F1 của lớp dương bằng 0 với mô hình như vậy, nên phản ánh đúng khả năng tìm ra nhóm thu nhập cao. Tôi dùng `pos_label=1`, không dùng `weighted` hay `macro` vì chúng trộn F1 rất cao của lớp đa số vào, che mất chất lượng thật trên lớp thiểu số.

---

## 3. Khó Khăn Gặp Phải và Cách Giải Quyết

| Khó khăn | Nguyên nhân | Cách giải quyết |
|---|---|---|
| Train lỗi `MissingConfigException`. | Lỡ commit `mlruns/` không đầy đủ. | Gỡ khỏi git, thêm `.gitignore`. |
| Release lỗi, `income-api` crash khi `joblib.load`. | scikit-learn trên EC2 khác bản 1.4.2. | Cài đúng `scikit-learn==1.4.2`. |
| Workflow mẫu cho GCP, tôi dùng S3. | Lệch thư viện và profile `dvc-lab`. | Đổi sang `boto3`, `dvc[s3]`, AWS secrets. |

---

## 4. So Sánh Bước 2 và Bước 3 (bắt buộc, 2 - 3 câu)

| | f1_score | accuracy |
|---|---|---|
| Bước 2 (chỉ `train_batch1`) | 0.7109 | 0.878 |
| Bước 3 (thêm `train_batch2`) | 0.7014 | 0.874 |

**Nhận xét:** Thêm dữ liệu làm f1_score giảm 0,0095 vì hai nửa dữ liệu cùng phân phối, không thêm thông tin mới, và chênh lệch nằm trong dao động của holdout 500 mẫu. Điều Bước 3 kiểm chứng là pipeline tự chạy trọn vòng, cả bốn job đều xanh.


