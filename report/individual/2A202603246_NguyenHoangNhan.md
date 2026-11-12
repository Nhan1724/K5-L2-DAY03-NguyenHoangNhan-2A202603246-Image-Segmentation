# Báo cáo thực hành Day 3 — điền trực tiếp trong fork của bạn

**Cách dùng:** Thay mọi dấu `…` bằng bài làm thật của bạn trước khi nộp link fork trên [LMS](https://aithucchien.vinuni.edu.vn). Giữ nguyên bốn mục và bảng để coach đọc nhanh. Viết ngắn, cụ thể theo ảnh/vùng; không cần thuật ngữ chuyên sâu. Ví dụ trong [hướng dẫn mẫu](../../reports/REPORT_TEMPLATE.md) chỉ giúp hiểu cách điền, không phải câu trả lời để chép lại.

- Mã học viên theo lớp: 2A202603246
- Ngày / CVAT local: 12/11/2026
- Công cụ đã dùng: CVAT

Mã học viên là mã lớp cấp; không cần ghi họ tên trong report nếu kênh [LMS](https://aithucchien.vinuni.edu.vn) đã nhận diện bạn. Chỉ ghi công cụ thật sự đã dùng; không có SAM vẫn làm bài bình thường.

## 1. Bài đã nộp
Task	File ZIP đúng tên	Hoàn thành mấy ảnh	Điểm tối đa (coach chấm sau)
easy_semantic	easy_semantic.zip	3 / 3	20
medium_instance	medium_instance.zip	3 / 3	32
hard_panoptic	hard_panoptic.zip	2 / 2	30
cp1_holes	chưa có	0 / 1	3
cp2_slice	chưa có	0 / 1	3
cp5_occlusion	chưa có	0 / 1	3
cp3_thin	chưa có	0 / 1	3
cp4_curb	chưa có	0 / 1	3
cp6_coverage	chưa có	0 / 1	3
Tổng tối đa			100

Chi tiết từng ZIP:

- easy_semantic (Segmentation mask 1.1): 3 ảnh, mỗi ảnh có mask trong SegmentationClass/ và SegmentationObject/. Năm lớp đã dùng: building, road, sidewalk, sky, vegetation. Phần chưa gán nhãn (nền đen) chiếm 16,9% / 6,7% / 12,4% diện tích ba ảnh (7ee6d192-89e2408b, 817bca71-00000000, 81ae7cbb-6bc63a4a). Ảnh 7ee6d192-89e2408b chỉ có sky, road, vegetation (không có building/sidewalk).
- medium_instance (COCO 1.0): 99 annotation — ảnh 000000181542 có 32 (22 person, 5 motorcycle, 4 car, 1 bus), ảnh 000000373353 có 36 (20 car, 15 person, 1 bus), ảnh 000000458325 có 31 (20 car, 11 person). Gồm 97 mask RLE và 2 polygon.
- hard_panoptic (COCO 1.0): 103 annotation — ảnh 000000350023 có 33, ảnh 000000460147 có 70 (trong đó 16 annotation không có polygon/RLE, xem mục 3). Gồm 75 mask RLE và 12 polygon.
Checkpoint cp1–cp6: chưa có ZIP, không tạo ZIP rỗng.

Không gặp lỗi export: cả ba ZIP đều mở được và có dữ liệu.

## 2. Một quyết định trước khi dùng gợi ý
- Ảnh, vị trí và object Medium đầu tiên tự vẽ: ảnh 000000181542.jpg (ảnh đầu của medium_instance), object id 1 trong file export — person, người lái xe máy ở giữa-trái ảnh, bbox [111, 113, 97, 274]. Xác nhận đây đúng là object vẽ đầu tiên (thứ tự id trong export phản ánh thứ tự tạo, nhưng bạn nên đối chiếu lại).
- Class và quy tắc tôi dùng để chọn biên: class person; mask chỉ bám phần nhìn thấy của người lái. Người lái và xe máy là hai instance riêng (person #1 và motorcycle #24, hai mask không chồng đáng kể), nên biên giữa chân/thân người và thân xe được dừng ở ranh nhìn thấy giữa hai vật, không để mask xe máy phủ lên người.
- Nếu dùng gợi ý sau đó:  điền — có hay không dùng gợi ý tự động của CVAT (SAM/OpenCV…); nếu có, ghi vùng gợi ý sai/đúng, sửa hay giữ và vì sao. Nếu không dùng, ghi “không dùng”.

## 3. Một lỗi tôi tìm thấy và sửa
Task/ảnh/vùng: medium_instance, ảnh 000000181542.jpg, annotation id 30 (class car), phần giữa-trên ảnh và mép trái ảnh.
Lỗi thuộc loại: gộp-tách (hai xe khác nhau bị gộp thành một instance).
Bằng chứng tôi nhìn thấy: mask #30 có bbox [0, 113, 501, 131], diện tích 22.540 px, trong khi các xe còn lại trong ảnh chỉ rộng 24–57 px. Mask gồm 3 mảnh rời nhau: xe bán tải ở giữa (≈ 15.980 px, x từ 269 đến 473), taxi ở mép trái (≈ 6.024 px, x từ 0 đến 122) và một mảnh nhỏ ở x 488–500 (≈ 536 px). Taxi và xe bán tải là hai xe khác nhau, cách nhau khoảng 150 px, không phải một vật bị che chia cắt.
Quy tắc và hành động sửa: “cùng class không có nghĩa cùng instance” (cp2_slice); chỉ gộp các mảnh khi chúng là một vật duy nhất bị vật khác che (cp5_occlusion). Cần tách #30 thành hai instance car (taxi và xe bán tải), và xem lại mảnh nhỏ x 488–500: gán cho xe phía sau nếu đó là xe, hoặc xóa.
Sau sửa đã Save và export lại chưa? Chưa — lỗi này vẫn còn trong medium_instance.zip hiện tại. Cập nhật dòng này sau khi sửa trong CVAT, Save và export COCO 1.0 lại.

Kết quả chạy python3 scripts/inspect_submissions.py --dir submissions (chỉ kiểm cấu trúc, chưa có điểm vì chưa có reference):

medium_instance: OK.
easy_semantic: LỖI — labelmap.txt dùng tên viết hoa (Building, Road, Sidewalk, Sky, Vegetation) trong khi classes.json dùng chữ thường (building, road, sidewalk, sky, vegetation). Scorer so khớp tên phân biệt hoa/thường (lab_utils.py, hàm load_semantic_submission), nên nếu giữ nguyên, các pixel đã vẽ sẽ không được tính vào lớp nào. Cách sửa: đổi tên 5 label sang chữ thường trong CVAT, Save và export lại Segmentation mask 1.1.
hard_panoptic: LỖI — 16 annotation (id 34–49, tất cả ở ảnh 000000460147) không có polygon/RLE. 15/16 trong số đó trùng bbox (IoU > 0,8) và cùng class với một mask hợp lệ khác (ví dụ #43 truck trùng #64 truck, #35 car trùng #75 car), nên nhiều khả năng là shape dạng khung còn sót, chưa xóa sau khi vẽ lại thành mask. Cách sửa: xóa 16 shape này trong CVAT, Save và export lại.

Scorecard ba tier tối đa 82 điểm, không phải điểm cuối trên 100; chưa có điểm.

## 4. Ba ca chưa chắc hoặc đã cân nhắc
Ảnh/vị trí	Hai cách hiểu có thể	Quy tắc/chứng cứ	Quyết định hoặc câu hỏi cho coach
1. medium_instance / 000000181542.jpg — xe buýt #28 và hành khách ở cửa sổ (#2, #5, #18, #20, #22, #32)	(a) Mask xe buýt phủ kín cả vùng cửa sổ, hành khách vẽ riêng chồng lên. (b) Khoét vùng cửa sổ có người ra khỏi mask xe buýt.	cp1_holes: chi tiết/kính bên trong vật vẫn nằm trong mask, không khoét. Số đo: 4 mask người nằm 100% trong mask xe buýt (338, 312, 358, 348 px), 2 mask nằm 90% và 73%.	Tôi chọn (a): giữ xe buýt nguyên khối và mỗi hành khách là một instance riêng. Câu hỏi: phần chồng lấn giữa xe buýt và hành khách có bị tính là sai khi chấm instance không?
2. hard_panoptic / 000000460147.jpg — xe chở xe: truck #64 và car #102	(a) Xe trên xe chở là vật riêng (car), vẽ chồng lên xe tải. (b) Xe trên xe chở là một phần của xe tải, không tách riêng.	Panoptic: mỗi pixel chỉ nên thuộc một segment. Số đo: car #102 nằm 100% trong mask truck #64 (2.673 px); toàn ảnh có 1,84% diện tích bị phủ bởi từ 2 mask trở lên.	Tôi chọn (a) vì thấy rõ từng xe trên xe chở. Câu hỏi: panoptic nên cắt phần xe ra khỏi mask xe tải để không chồng, hay chồng lấn là chấp nhận được?
3. hard_panoptic / 000000350023.jpg — phương tiện ở góc dưới phải (khoảng x 540–640, y 425–480)	(a) Gán bus. (b) Để trống vì chỉ thấy một phần và schema không có lớp xe điện/tàu.	cp6_coverage: không tô ép vùng chưa chắc; schema 12 lớp chỉ có bus, truck, car… Số đo: vùng này chỉ được phủ 2,5%.	Hiện để trống (chọn b). Câu hỏi: nếu là xe buýt/xe điện bị cắt bởi mép ảnh thì có nên gán bus không?
