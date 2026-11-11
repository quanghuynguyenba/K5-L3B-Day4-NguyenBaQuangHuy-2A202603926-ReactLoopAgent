# Ghi chú phân tích · Day 04

Họ và tên: Nguyễn Bá Quang Huy
MSSV: 2A202603926

## 1. Case được chọn

- ID case: A5
- Câu trả lời của agent: Danh sách sự cố (incident) liên quan đến GitHub Actions trong năm 2025 không có kết quả nào khả thi. Dữ liệu cho thấy hiện tại tất cả hệ thống đều hoạt động bình thường và không có sự cố nào được ghi nhận trong năm 2025. Kết quả chỉ gồm các sự cố đã giải quyết tháng 10–11/2026; có nhắc “chỉ diễn ra trong 50 sự cố gần nhất” nhưng vẫn suy ra theo hướng không có sự cố năm 2025.
- Điểm ở lần chạy 1: 40

## 2. Bằng chứng trong trace

Dán đoạn trace chứng minh lỗi và nêu nguyên nhân có thể xảy ra.

```text
Tool get_status(provider=github, component="GitHub Actions") trả:
- matching_incidents_in_last_50: 11
- note: "Chỉ tìm theo chuỗi trong 50 incident gần nhất; không tìm thấy không chứng minh không có sự cố."
- incidents chỉ có updated_at năm 2026 (ví dụ t89g2q1800fr, xr3myzj7ms51, ...)

Final answer (eval_before) lại viết:
"không có sự cố nào được ghi nhận trong năm 2025"
và "Do không có chứng cứ nào cho thấy có sự cố trong năm 2025, tôi không thể khẳng định có sự cố nào trong năm đó."

Nguyên nhân có thể: system prompt chưa siết đủ về giới hạn coverage; model lẫn “không thấy trong 50 incident gần nhất / không thấy năm 2025 trong snapshot” với “không tồn tại / không có sự cố năm 2025”, trong khi expected yêu cầu thừa nhận không thể chứng minh danh sách đầy đủ cho 2025.
```

## 3. Thay đổi đã thực hiện

- Sửa gì, ở đâu: Siết `SYSTEM_PROMPT` trong `agent.py`: bắt buộc nêu giới hạn tối đa 50 incident gần nhất; kết quả rỗng hoặc thiếu năm yêu cầu không được coi là chứng minh không tồn tại; không bịa danh sách đầy đủ theo năm ngoài phạm vi tool.
- Vì sao chọn thay đổi này: Lỗi A5 nằm ở cách diễn giải giới hạn dữ liệu, không phải lỗi tool call. Sửa prompt đúng chỗ gây lỗi, khớp expected của A5 và vẫn giữ contract loop/tests offline.

## 4. So sánh hai lần chạy

| Chỉ số | `eval_before.json` | `eval_after.json` |
|---|---|---|
| Điểm tổng | 77.09 | 82.91 |
| Điểm case đã chọn | 40 | 100 |
| Số model calls | 50 (toàn bộ); A5: 2 | 47 (toàn bộ); A5: 2 |
| Token, chi phí nếu có |  |  |

- Lỗi đã giải quyết: A5 không còn kết luận “không có sự cố năm 2025”; nêu rõ chỉ xem được tối đa 50 incident gần nhất và không đủ để khẳng định danh sách đầy đủ (40 → 100).
- Lỗi còn lại hoặc mới xuất hiện: Một số case giảm điểm so với lần 1 (E1 75→65, E5 75→45, H8 90→80, A3 100→95). H1/H2 vẫn thấp (60/50). Điểm judge là ước lượng, có thể dao động giữa các lần chạy dù cùng model.

## 5. Giới hạn

Nêu giới hạn của điểm judge, của snapshot và của việc chỉ chạy một lần mỗi phiên bản.

Điểm LLM judge cùng model với agent nên có thể thiên lệch hoặc không ổn định giữa các lần. Snapshot cố định (GitHub/Cloudflare lưu ngày 09/11/2026) chỉ phản ánh cửa sổ 50 incident gần nhất, không phải lịch sử đầy đủ theo năm. Chỉ chạy một lần mỗi phiên bản agent nên chưa đo được độ ổn định; cải thiện A5 không bảo đảm mọi case khác đều tăng.
