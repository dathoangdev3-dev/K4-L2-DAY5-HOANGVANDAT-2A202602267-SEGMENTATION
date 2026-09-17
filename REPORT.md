# Báo cáo Day 5 — điền trực tiếp trong fork của bạn

- Mã học viên theo lớp: hoangdat / 2A202602267
- Ngày / CVAT local: 17/09/2026 / CVAT local lớp
- Công cụ đã dùng: Brush, Polygon

## 1. Bài đã nộp

| Task | File ZIP đúng tên | Hoàn thành mấy ảnh | Điểm tối đa (coach chấm sau) |
| --- | --- | ---: | ---: |
| easy_semantic | easy_semantic.zip | 3 / 3 | 20 |
| medium_instance | medium_instance.zip | 3 / 3 | 32 |
| hard_panoptic | hard_panoptic.zip | 2 / 2 | 30 |
| cp1_holes | cp1_holes.zip | 1 / 1 | 3 |
| cp2_slice | cp2_slice.zip | 1 / 1 | 3 |
| cp5_occlusion | cp5_occlusion.zip | 1 / 1 | 3 |
| cp3_thin | cp3_thin.zip | 1 / 1 | 3 |
| cp4_curb | cp4_curb.zip | 1 / 1 | 3 |
| cp6_coverage | cp6_coverage.zip | 1 / 1 | 3 |
| **Tổng tối đa** | | | **100** |

Đã chạy scorer ba tier trên máy sau khi nhận ground truth: easy_semantic 16.9/20, medium_instance 16.3/32, hard_panoptic 13.8/30, tổng **47.0/82**. Checkpoint không có ground truth trong repo học viên nên chưa tính được điểm tự động — chờ coach chấm. Không tự điền điểm bonus.

## 2. Một quyết định trước khi dùng gợi ý

- Ảnh, vị trí và object Medium đầu tiên tự vẽ: `000000181542.jpg` — người đứng góc trái phía trước, gần vỉa hè. Vẽ hoàn toàn bằng Brush trước khi dùng bất kỳ gợi ý nào.
- Class và quy tắc tôi dùng để chọn biên: Class `person`. Tôi chỉ vẽ phần thân người nhìn thấy từ đầu đến chân; biên dừng ngay tại đường viền quần áo tiếp giáp nền đường. Không vẽ phần bị xe hoặc người khác che vì quy tắc chỉ tô phần nhìn thấy. Bóng dưới chân không đưa vào mask vì không thuộc thân người.
- Không dùng gợi ý: toàn bộ mask vẽ thủ công bằng Brush và Polygon. Quyết định quan trọng nhất là với người đứng sát xe — tôi dừng biên tại chỗ thân người tiếp xúc xe, không kéo sang phần xe dù hai vùng gần nhau.

## 3. Một lỗi tôi tìm thấy và sửa

- Task/ảnh/vùng: `medium_instance` — ảnh `000000373353.jpg` và `000000458325.jpg`, class `car`
- Lỗi thuộc loại: thừa vật — tách một xe thành nhiều instance riêng lẻ
- Bằng chứng tôi nhìn thấy: Sau khi chạy scorer lần đầu, submitted=82 nhưng GT=71 (thừa +11, FP=27). Phóng to ảnh `000000373353` trong CVAT thấy hai mask `car` sát nhau thực ra là hai phần nhìn thấy của cùng một xe bị cột điện che giữa — tôi đã vẽ thành hai instance riêng thay vì một.
- Quy tắc và hành động sửa: Theo quy tắc instance, một vật bị che thành hai mảng nhìn thấy rời nhau vẫn là **một instance**. Tôi xác định từng cặp mask `car` thuộc cùng một xe rồi xóa mask thừa. Ảnh `000000373353` đưa `car` từ 20 → 13 khớp đúng GT.
- Sau sửa đã Save và export lại chưa? Đã Save và export lại thành `medium_instance.zip`.
- Kết quả sau sửa: `medium_instance` metric tăng từ 0.602 → 0.629, điểm từ 14.4 → **16.3/32**. FP giảm từ 27 → 16, Recall tăng từ 0.775 → 0.820. Kết quả ba tier tổng **47.0/82**, không tự điền PASS hay bonus.

## 4. Ba ca chưa chắc hoặc đã cân nhắc

| Ảnh/vị trí | Hai cách hiểu có thể | Quy tắc/chứng cứ | Quyết định hoặc câu hỏi cho coach |
| --- | --- | --- | --- |
| `medium_instance` — `000000181542.jpg`, xe máy đứng sát người góc trái | `motorcycle` riêng một instance, hoặc phần xe bị người che quá nhiều nên bỏ qua | Quy tắc: motorcycle và person là hai class khác nhau, phải tách dù đứng sát; vật nhìn thấy một phần vẫn tô | Tôi tách thành hai instance riêng. Scorer cho nộp=5, GT=4. Hỏi coach: chiếc motorcycle nào không được tính — cái bị che hơn 50% thân không? |
| `hard_panoptic` — `000000460147.jpg`, vùng `sidewalk` bị nhiều xe che phần lớn | Tô 1 segment sidewalk bao toàn bộ vùng vỉa hè kể cả phần ước lượng bị xe che, hoặc chỉ tô phần pixel nhìn thấy rõ | Quy tắc: chỉ tô phần nhìn thấy; không tự đoán vùng bị che hoàn toàn | Tôi tô phần nhìn thấy. Kết quả sidewalk PQ=0 — biên không khớp GT. Hỏi coach: với stuff bị xe che nhiều, có tô liền xuyên qua vật che không? |
| `hard_panoptic` — `000000350023.jpg`, vùng thực vật gồm cây to, hoa nhỏ, bụi cỏ xen kẽ nhau | Gộp tất cả vào 1 segment `vegetation` duy nhất, hoặc tách riêng từng vùng liên tục vì hình dạng khác nhau | Quy tắc panoptic: vegetation là stuff — toàn bộ vùng thực vật gộp thành 1 segment, không tách theo loại | Tôi gộp thành 1 segment. Kết quả vegetation PQ=0 — biên không khớp GT. Hỏi coach: cây và bụi hoa nhỏ ở xa nhau trong ảnh có phải cùng 1 segment không? |
