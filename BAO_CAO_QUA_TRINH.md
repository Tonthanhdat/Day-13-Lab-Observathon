# Báo Cáo Quá Trình Làm Observathon

## Mục Tiêu

Sửa agent thương mại điện tử dạng hộp đen để trả lời đơn hàng chính xác, dùng tool hợp lý, giảm chi phí/độ trễ, không lộ PII và chống prompt injection.

## Các Bước Đã Làm

1. Đọc yêu cầu trong `README.md`, `RULES.md`, `docs/SUBMIT.md`, `docs/WRAPPER_API.md`, `docs/PROMPT_OPTIMIZATION.md`.

2. Sửa `solution/config.json`:
   - Giảm `temperature` xuống `0.2`.
   - Bật `retry`, `cache`, `loop_guard`, `normalize_unicode`, `redact_pii`, `verify`.
   - Xóa `catalog_override` sai.
   - Đặt `tool_budget = 4`, `self_consistency = 2`, `max_steps = 8`.

3. Viết lại `solution/prompt.txt`:
   - Bắt agent gọi tool trước khi trả lời.
   - Chỉ lấy giá, tồn kho, giảm giá và phí ship từ tool.
   - Tính tiền bằng công thức rõ ràng.
   - Từ chối nếu hết hàng, không tìm thấy sản phẩm hoặc không ship được.
   - Không lặp lại email/số điện thoại khách hàng.
   - Chống instruction trong note, quote hoặc `GHI CHU`.
   - Phân biệt câu hỏi “còn hàng/giá bao nhiêu” với câu hỏi cần tính tổng thanh toán.

4. Sửa `solution/wrapper.py`:
   - Sanitize input để giảm prompt injection.
   - Thêm retry và cache thread-safe.
   - Redact PII trong câu trả lời/log.
   - Ghi telemetry: latency, token, cost, tool usage, status.
   - Chuẩn hóa output: đơn hợp lệ kết thúc bằng `Tong cong: <integer> VND`; câu hỏi chỉ hỏi giá/tồn kho thì không có `Tong cong`.

5. Chạy `selfcheck`:

```powershell
python harness\selfcheck.py
```

6. Chạy simulator bằng Docker vì binary Windows bị lỗi load DLL:

```powershell
docker run --rm --env-file .env -v "${PWD}:/lab" -w /lab python:3.12-slim bash -lc "chmod +x bin/practice/observathon-sim && ./bin/practice/observathon-sim --config solution/config.json --wrapper solution/wrapper.py --out run_output.json --concurrency 8"
```

7. Chạy scorer:

```powershell
docker run --rm -v "${PWD}:/lab" -w /lab python:3.12-slim bash -lc "chmod +x bin/practice/observathon-score && ./bin/practice/observathon-score --run run_output.json --out score_output.json"
```

## Kết Quả Hiện Tại

- Simulator chạy `120` request.
- `status ok = 120`.
- Scorer public: `120/120 correct`.
- Headline score: `100.0 / 100`.

- score private: `100.0 / 100`.