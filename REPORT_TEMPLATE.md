# CSC4005 Lab 6 Report – Export ONNX + Consistency Test + Benchmark

## 1. Thông tin

* Họ tên: Đặng Quốc An
* Mã sinh viên: 1675020001
* Lớp: KHMT-1701 - Nhóm 12
* Link GitHub repo: https://github.com/FIT-DNU-CS-16-01/csc4005-lab6-onnx-starter-kit-khmt_1701_nhom12
* Link checkpoint hoặc mô tả checkpoint sử dụng:

  * Checkpoint được sử dụng từ Lab 5: `best_model.pt`
* Link file ONNX nếu không commit trực tiếp:

  * `outputs/vit_smartcampus.onnx`

---

## 2. Mô tả mô hình đầu vào

| Nội dung                | Giá trị                                            |
| ----------------------- | -------------------------------------------------- |
| Bài toán                | Smart Campus Scene Classification                  |
| Dataset                 | MIT Indoor Scenes 67 subset                        |
| Số lớp                  | 5                                                  |
| Classes                 | classroom, computerroom, library, corridor, office |
| Model PyTorch           | vit_b_16                                           |
| Checkpoint              | best_model.pt                                      |
| Image size              | 224                                                |
| Train mode từ lab trước | head_only                                          |

---

## 3. Export ONNX

| Thông số      | Giá trị                      |
| ------------- | ---------------------------- |
| ONNX path     | outputs/vit_smartcampus.onnx |
| Opset         | 17                           |
| Dynamic batch | no                           |
| Input name    | input                        |
| Output name   | logits                       |
| Model size    | 0.102 MB                     |

Lệnh đã chạy:

```bash
python -m src.export_onnx \
--checkpoint checkpoints/best_model.pt \
--onnx_path outputs/vit_smartcampus.onnx \
--model_name vit_b_16 \
--img_size 224 \
--opset 17
```

Kết quả export:

```json
{
  "status": "exported_and_checked"
}
```

Mô hình đã được export thành công sang định dạng ONNX và vượt qua bước ONNX checker.

---

## 4. Consistency Test

| Metric          |               Giá trị |
| --------------- | --------------------: |
| passed          |                  True |
| num_samples     |                    32 |
| batch_size      |                     1 |
| max_abs_diff    | 3.147125244140625e-05 |
| mean_abs_diff   | 7.019052294054973e-06 |
| pred_match_rate |                   1.0 |
| atol            |                  1e-4 |
| rtol            |                  1e-3 |

Nhận xét:

* PyTorch và ONNX có tính nhất quán rất cao.
* Sai số tuyệt đối giữa hai runtime rất nhỏ.
* Prediction match rate đạt 100%.
* Sai khác số học không làm thay đổi nhãn dự đoán.
* Consistency test đã pass hoàn toàn.

---

## 5. Benchmark

| Runtime     | Batch size | Mean latency (ms) | Median latency (ms) | P95 latency (ms) | Throughput (img/s) | Model size (MB) |
| ----------- | ---------: | ----------------: | ------------------: | ---------------: | -----------------: | --------------: |
| PyTorch     |          1 |            249.30 |              247.77 |           262.47 |               4.01 |          327.37 |
| ONNXRuntime |          1 |            225.95 |              227.27 |           247.77 |               4.43 |            0.10 |

Kết quả benchmark cho thấy ONNX Runtime có tốc độ inference nhanh hơn PyTorch trên CPU.

---

## 6. Phân tích kết quả

### 1. ONNXRuntime có nhanh hơn PyTorch không?

Có. ONNXRuntime có latency thấp hơn và throughput cao hơn so với PyTorch khi chạy trên CPU.

### 2. Batch size ảnh hưởng thế nào đến latency và throughput?

Batch size lớn hơn thường giúp tăng throughput do xử lý nhiều ảnh cùng lúc, tuy nhiên latency của từng batch có thể tăng lên. Batch size nhỏ phù hợp hơn cho inference thời gian thực.

### 3. Vì sao cần warm-up trước khi đo benchmark?

Warm-up giúp loại bỏ ảnh hưởng của quá trình khởi tạo runtime, cache hoặc tối ưu graph ở lần chạy đầu tiên. Điều này giúp kết quả benchmark ổn định và chính xác hơn.

### 4. Vì sao không nên chỉ đo một lần rồi kết luận?

Một lần đo có thể bị ảnh hưởng bởi nhiều yếu tố ngẫu nhiên như CPU scheduling hoặc cache system. Chạy nhiều lần giúp lấy trung bình chính xác hơn.

### 5. Nếu triển khai lên CPU/edge device, bạn chọn batch size nào? Vì sao?

Batch size = 1 phù hợp hơn cho hệ thống Smart Campus thời gian thực vì giúp giảm độ trễ và phản hồi nhanh hơn.

---

## 7. Liên hệ triển khai thực tế

ONNX giúp chuẩn hóa mô hình và cho phép triển khai trên nhiều nền tảng khác nhau mà không phụ thuộc trực tiếp vào PyTorch. Điều này rất hữu ích cho các hệ thống Smart Campus hoặc edge devices có tài nguyên hạn chế.

Consistency test giúp phát hiện các lỗi sai lệch output giữa PyTorch và ONNX Runtime sau khi export mô hình. Điều này đảm bảo mô hình ONNX hoạt động chính xác trước khi triển khai thực tế.

Benchmark giúp đánh giá hiệu năng inference để lựa chọn runtime phù hợp cho hệ thống triển khai. Các chỉ số như latency và throughput hỗ trợ đưa ra quyết định kỹ thuật về tối ưu hóa hệ thống.

Nếu triển khai thực tế trong Smart Campus, cần kiểm thử thêm:

* hiệu năng trên thiết bị thật,
* khả năng xử lý nhiều camera đồng thời,
* độ ổn định lâu dài,
* mức sử dụng CPU/RAM,
* và độ chính xác trên dữ liệu thực tế ngoài môi trường huấn luyện.

---

## 8. Kết luận

Bài lab đã export thành công mô hình Vision Transformer sang định dạng ONNX.

Consistency test đã pass hoàn toàn với prediction match rate đạt 100%, cho thấy ONNX Runtime hoạt động gần như giống hệt PyTorch.

Benchmark cho thấy ONNXRuntime nhanh hơn PyTorch trên CPU với latency thấp hơn và throughput cao hơn.

Qua bài lab này, có thể thấy ONNX là giải pháp phù hợp cho triển khai mô hình AI thực tế nhờ khả năng portable, lightweight và tối ưu inference trên nhiều nền tảng khác nhau.
