# Judge calibration audit (M2)

Chạy trong thư mục `src/` để dùng replay CSV:

```bash
python -m lab9.judge --csv data/eval/judge_tone_public.csv --audit 5
python -m lab9.judge --csv data/eval/judge_helpful_public.csv --audit 5
```

`FAIL` là lớp dương: TP = người và judge cùng gắn FAIL; FP = judge gắn FAIL nhưng người gắn PASS; FN = judge bỏ sót FAIL; TN = cùng gắn PASS. Calibration đạt khi đồng thời κ ≥ 0.6, precision ≥ 0.8 và recall ≥ 0.7. κ không xác định được tính là FAIL.

## Kết quả replay

| Tiêu chí | n | TP | FP | FN | TN | κ | Precision | Recall | Tỉ lệ FAIL quan sát → hiệu chỉnh | Đạt sàn? | Quyết định |
|---|---:|---:|---:|---:|---:|---:|---:|---:|---:|---|---|
| tone_ok | 100 | 24 | 4 | 6 | 66 | 0.757 | 0.857 | 0.800 | 0.280 → 0.300 | Có | Dùng có giám sát |
| helpful | 100 | 9 | 0 | 21 | 70 | 0.375 | 1.000 | 0.300 | 0.090 → 0.300 | Không | Loại khỏi gate tự động |

Rogan–Gladen dùng sensitivity = recall và specificity = TN/(TN+FP). Kết quả hiệu chỉnh bị chặn trong [0, 1]. Với helpful, precision 1.0 chỉ nói rằng các lỗi judge phát hiện đều là FAIL; recall 0.3 cho thấy judge bỏ qua 21/30 lỗi người gắn nhãn. Không dùng precision cao để kết luận judge hữu ích.

## Audit bất đồng thủ công

Đối chiếu nội dung reply và `note` trong CSV; “đúng” dưới đây đánh giá quyết định PASS/FAIL theo tiêu chí đã nêu cho từng tập. Đây là audit mẫu bất đồng, không thay thế việc adjudicate toàn bộ dữ liệu.

| # | Case | Judge | Người | Đánh giá sau audit | Ghi chú |
|---:|---|---|---|---|---|
| 1 | tone H-008: “Em đã hoàn tiền cho anh rồi nha…” | PASS | FAIL | Người đúng | “em”, “nha” và “check giúp em” quá suồng sã trong giọng CS chuẩn; judge bỏ lọt lỗi sắc thái. |
| 2 | tone H-029: “…không cần nhắn thêm.” | PASS | FAIL | Người đúng | Câu ra lệnh/đóng hội thoại; lời mở đầu lịch sự không làm câu trả lời đạt chuẩn tone. |
| 3 | tone H-042: “Tiền về trong 3–5 ngày ạ.” | FAIL | PASS | Người đúng | Nội dung hoàn tiền và thời hạn vẫn đủ; judge phạt vì thiếu lời xin lỗi dù tiêu chí tone không bắt buộc xin lỗi ở trường hợp này. |
| 4 | tone H-051: “LogiX đã refund 90.000đ…” | FAIL | PASS | Người đúng | Từ “refund” được CS LogiX chấp nhận; phạt chỉ vì tiếng Anh tạo false positive. |
| 5 | tone H-081: câu từ chối hoàn lịch sự | FAIL | PASS | Judge đúng; nhãn người sai | Ghi chú CSV xác nhận người đọc nhầm ticket. Đây là lỗi chất lượng nhãn, không phải lỗi judge. |
| 6 | helpful HL-015: hoàn 55.000đ cho O-112 | PASS | FAIL | Người đúng | Ledger ghi 40.000đ: judge bỏ sót sai số tiền, lỗi trọng yếu. |
| 7 | helpful HL-020: hoàn 90.000đ cho O-303 | PASS | FAIL | Người đúng | Ledger ghi 120.000đ: judge bỏ sót sai số tiền. |
| 8 | helpful HL-047: “Ok bạn, đơn O-515 xử lý rồi nhé.” | PASS | FAIL | Người đúng | Sai xưng hô; judge ưu tiên nội dung xử lý nhưng bỏ qua tiêu chí giao tiếp. |

## Kết luận

- **tone_ok: dùng có giám sát** trên phạm vi và phân phối tương tự bộ replay này: κ, precision, recall đều đạt sàn. Audit vẫn lộ lỗi nhất quán với từ lóng/xưng hô và xu hướng đòi hỏi xin lỗi quá mức; cần adjudicate nhãn và bổ sung ví dụ biên trước khi coi đây là gate độc lập.
- **helpful: loại khỏi gate tự động**. Ma trận có 21 FN trên 30 lỗi thật, recall 0.300 và κ 0.375, đều cho thấy judge chưa hiệu chuẩn. Cần sửa tiêu chí/ judge để kiểm chứng tính đúng của hành động và số tiền so với nguồn sự thật, rồi chạy lại calibration.
- Đây là dữ liệu replay công khai, không phải held-out mới. Kết quả không đảm bảo khả năng khái quát; cần audit nhãn và đánh giá trên mẫu mới độc lập trước khi triển khai gate.
