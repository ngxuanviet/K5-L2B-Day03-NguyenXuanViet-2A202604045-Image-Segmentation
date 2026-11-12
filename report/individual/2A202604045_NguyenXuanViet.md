# Báo cáo thực hành Day 3 — Image Segmentation

- Mã học viên theo lớp: 2A202604045 (Nguyễn Xuân Việt)
- Ngày / CVAT local: 12/11/2026 · CVAT local v2.74.1 tại `http://localhost:8080`
- Công cụ đã dùng:
  - CVAT local: tạo task, xem và sửa mask bằng Brush/Polygon, Save, export.
  - SAM 3 qua Roboflow API (prompt chữ theo tên lớp) để sinh mask đề xuất cho cả 9 task.
  - Trợ lý AI lập trình (Claude Code) để gọi API Roboflow, ghép và lọc các mask đề xuất, soát sơ bộ từng ảnh rồi đẩy mask lên CVAT qua REST API.
  - Không dùng auto-annotation có sẵn của CVAT. Script và khóa API nằm ngoài repo.

Cách làm: mask đề xuất của SAM 3 được lọc trùng, bỏ vùng ngoài schema rồi đưa lên CVAT. Sau đó tôi mở từng job trong CVAT, kiểm lớp, số vật, biên và sửa tay những chỗ sai (liệt kê ở mục 3). ZIP nộp là bản export sau khi đã sửa và Save.

## 1. Bài đã nộp

| Task | File ZIP đúng tên | Hoàn thành mấy ảnh | Điểm tối đa (coach chấm sau) |
| --- | --- | ---: | ---: |
| easy_semantic | `easy_semantic.zip` | 3 / 3 | 20 |
| medium_instance | `medium_instance.zip` | 3 / 3 | 32 |
| hard_panoptic | `hard_panoptic.zip` | 2 / 2 | 30 |
| cp1_holes | `cp1_holes.zip` | 1 / 1 | 3 |
| cp2_slice | `cp2_slice.zip` | 1 / 1 | 3 |
| cp5_occlusion | `cp5_occlusion.zip` | 1 / 1 | 3 |
| cp3_thin | `cp3_thin.zip` | 1 / 1 | 3 |
| cp4_curb | `cp4_curb.zip` | 1 / 1 | 3 |
| cp6_coverage | `cp6_coverage.zip` | 1 / 1 | 3 |
| **Tổng tối đa** | | | **100** |

Không có lỗi export. Semantic xuất `Segmentation mask 1.1`, instance và panoptic xuất `COCO 1.0`. `scripts/inspect_submissions.py` báo `OK` cho cả 9 ZIP (chỉ là kiểm cấu trúc, không phải điểm). Trên CVAT các task có hậu tố `_v2` vì tôi còn giữ task nháp cùng tên; tên ZIP vẫn theo đúng mã task.

Mức độ tự kiểm không đều: tôi đã mở và sửa trực tiếp `easy_semantic`, `medium_instance`, `hard_panoptic`, `cp1_holes`, `cp6_coverage`. Bốn trạm `cp2_slice`, `cp5_occlusion`, `cp3_thin`, `cp4_curb` giữ nguyên mask đề xuất sau lượt soát sơ bộ, chưa chỉnh tay.

## 2. Một quyết định trước khi dùng gợi ý

- Ảnh, vị trí và object Medium đầu tiên tự vẽ: **không có object Medium nào được vẽ tay trước khi xem gợi ý trong bản nộp này.** Toàn bộ mask Medium bắt đầu từ đề xuất của SAM 3. Trước đó tôi có một lượt vẽ tay nháp trên CVAT (task nháp riêng, không nộp), nhưng bản nộp không lấy mask từ lượt đó. Tôi ghi rõ để coach biết đúng quy trình thực tế.
- Class và quy tắc tôi dùng để chọn biên (áp cho mọi object Medium khi kiểm): mỗi vật đếm được là một mask riêng; biên chỉ theo phần nhìn thấy, không vẽ xuyên qua vật che; hai vật cùng lớp sát nhau vẫn là hai mask; chi tiết gắn liền với xe (gương, giá nóc) thuộc mask của xe đó.
- Nếu dùng gợi ý sau đó — vùng gợi ý sai/đúng, hành động sửa/giữ và lý do:
  - **Sửa:** ảnh `000000458325.jpg`, hai xe (object 74 và 99 trên CVAT): gợi ý sai ở phần gương chiếu hậu. Tôi sửa lại để gương thuộc đúng xe của nó, vì gương là bộ phận của xe và hai xe cùng lớp vẫn là hai instance.
  - **Sửa:** cùng ảnh, một xe (object 77) bị thiếu phần giá nóc; tôi tô thêm vì giá nóc là phần nhìn thấy của xe.
  - **Bỏ:** ảnh `000000181542.jpg`, gợi ý đánh dấu hình người trên tấm poster cửa hàng là `person`. Bỏ vì đó là ảnh in, không phải người thật trong cảnh.
  - **Đổi lớp:** ảnh `000000373353.jpg`, vật tối ở mép trái được gợi ý là `motorcycle`; đổi thành `car` vì hình dạng là đầu một ô tô bị cắt bởi mép ảnh.
  - **Giữ:** xe bán tải ở ảnh `000000181542.jpg` giữ lớp `truck` (gợi ý `truck` tin cậy cao hơn `car`), taxi giữ là `car`.

## 3. Một lỗi tôi tìm thấy và sửa

Lỗi chính chọn để báo cáo:

- Task/ảnh/vùng: `hard_panoptic`, ảnh `000000460147.jpg`, ba vật được gợi ý là `person` và chiếc xe tải chở ô tô ở giữa ảnh.
- Lỗi thuộc loại: sai lớp (ba vật) và biên (xe tải chở ô tô).
- Bằng chứng tôi nhìn thấy: ba mask được gợi ý là `person` nhưng khi kiểm thì là xe máy. Mask `truck` của xe chở ô tô bị sai so với phần nhìn thấy của xe.
- Quy tắc và hành động sửa: thing phải đúng lớp và mỗi vật một instance; đổi ba mask thành `motorcycle`, tô lại mask `truck` theo phần nhìn thấy của xe tải. Cùng ảnh, tô thêm `vegetation` và `road` còn thiếu.
- Sau sửa đã Save và export lại chưa? Đã Save và export lại; `hard_panoptic.zip` trong `submissions/` là bản sau sửa.

Các lỗi khác đã sửa trong cùng lượt kiểm:

| Task / ảnh | Lỗi quan sát | Hành động |
| --- | --- | --- |
| `easy_semantic` / `7ee6d192-89e2408b.jpg` | `road` thiếu đoạn đường ở xa, làn ngược chiều và phần sát lề; `vegetation` thiếu một ít | Tô thêm |
| `easy_semantic` / `817bca71-00000000.jpg` | `sidewalk` và `road` lẫn nhau ở một số chỗ; `vegetation` thiếu | Sửa ranh theo bó vỉa, tô thêm |
| `easy_semantic` / `81ae7cbb-6bc63a4a.jpg` | `sidewalk` sai một ít ở bên phải ảnh | Sửa lại |
| `medium_instance` / `000000458325.jpg` | Gương của hai xe bị gán sai; một xe thiếu giá nóc | Sửa biên, tô thêm |
| `hard_panoptic` / `000000350023.jpg` | `road` thiếu ở xa; một `person` xác định sai; một vật là `bicycle` chứ không phải `person` | Tô thêm, xóa mask sai, đổi lớp |
| `cp6_coverage` / `7daa6479-67988f3f.jpg` | Thiếu `vegetation`; `sidewalk` và `building` lẫn nhau | Tô thêm, sửa lại lớp |

Kết quả tự đánh giá: chưa có điểm. Tôi chưa chạy scorer với ground truth và chưa xem Summary của GitHub Actions có điểm.

## 4. Ba ca chưa chắc hoặc đã cân nhắc

| Ảnh/vị trí | Hai cách hiểu có thể | Quy tắc/chứng cứ | Quyết định hoặc câu hỏi cho coach |
| --- | --- | --- | --- |
| 1. `cp4_curb` / `7d83710e-4697c3b2.jpg`, nền bê tông bên phải nơi chiếc SUV trắng đang đỗ, sau lan can | (a) `sidewalk` vì cùng mặt bê tông, liền với vỉa hè; (b) để trống vì là chỗ đỗ xe/lối vào nhà, không phải vỉa hè cho người đi bộ | Phiếu quy tắc nói ranh theo chức năng, không theo màu. Schema trạm chỉ có `road` và `sidewalk`, không có lớp bãi đỗ | Giữ `sidewalk` (không phải lòng đường, cùng cao độ với vỉa hè). Bó vỉa cũng tính vào `sidewalk`. Hỏi coach: chỗ đỗ xe trên vỉa nên là `sidewalk` hay bỏ trống? |
| 2. `cp1_holes` / `000000144300.jpg`, xe van xanh và chiếc caravan trắng phía sau xe máy | Van: (a) `truck`, (b) `car`. Caravan: (a) `truck`, (b) không gán vì là rơ-moóc, không thuộc 6 lớp | `classes.json` của trạm chỉ có person/bicycle/car/motorcycle/bus/truck. Van là loại thùng kín cỡ lớn; caravan không tự chạy | Van gán `truck`, caravan không gán. Mask xe máy giữ kín, không khoét kính chắn gió và khe nan hoa theo yêu cầu của trạm |
| 3. `easy_semantic` / `7ee6d192-89e2408b.jpg` và `817bca71-00000000.jpg`: đồi cỏ khô, bãi cỏ trước nhà, tường chắn bê tông ven cao tốc, nắp capo xe ghi hình | (a) Tô cỏ là `vegetation`, tường là `building`, capo là `road` để phủ kín; (b) để trống vì không thuộc 5 lớp | Phiếu quy tắc: không ép đoán cho đủ coverage. `vegetation` tôi hiểu là cây và bụi; cỏ/đất, tường chắn, xe, cột, thùng rác là lớp khác | Để trống các vùng này; chỉ tô cây và bụi là `vegetation`. Hỏi coach: bãi cỏ và đồi cỏ có tính là `vegetation` trong bài này không? |
