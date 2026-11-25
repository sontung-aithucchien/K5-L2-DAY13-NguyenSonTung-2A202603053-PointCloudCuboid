# Báo cáo Day 13 — Nguyễn Sơn Tùng, 2A202603053

## 1. Kết luận trước
Mã số sinh viên là 2A202603053 và task CVAT là `D13-2A202603053`. Task có 10 frame: `s01f00`, `s01f33`, `s02f03`, `s02f51`, `s03f01`, `s03f34`, `s04f16`, `s04f40`, `s05f30`, `s05f45`. Môi trường làm bài là Windows; tài khoản CVAT là `sontung1807`, phiên bản CVAT là 2.74.1.

Sau lần đồng bộ cuối, bảng A có 74 cuboid. Frame có ít nhất cuboid là `s01f33` và `s03f34`, mỗi frame lần lượt có 3 và 4 cuboid. Phiếu tự kiểm có 8/74 cuboid có cảnh báo. Hướng xe cùng chiều gần 270° được xác định khá chắc chắn; đáy và kích thước ở các cuboid có cảnh báo cần được kiểm tra thêm.

Trong CVAT, các chế độ Top/Side/Front được dùng để căn tâm, kích thước và chiều cao; ở Top dùng Cuboid orientation để căn mũi tên, sau đó kiểm đáy ở Side/Front và xem cuboid trên các ảnh camera bằng `preview`. Mỗi lần sửa cần bật Appearance → Cuboid orientation và kiểm lại phần đầu, đuôi xe cũng như mặt đường.

## 2. Đọc frame (Bước 2)
Hai frame đọc kỹ là hai frame đầu: `s01f00` và `s01f33`. Hình ảnh cần lưu trong tên `t1-s01f00-3d.png`, `t1-s01f00-top.png`, `t1-s01f33-3d.png`, `t1-s01f33-top.png`.

| Frame | Phía trước xe và trục; +x, +y | Mặt đường | Vật xa nhất đọc được | Loại vật thấy | Vật phía sau / hai bên | Cụm chưa đọc được |
|---|---|---|---|---|---|---|
| s01f00 | Phía trước xe là −y. Trong nhìn từ trên xuống, +x chỉ sang trái và +y chỉ ra phía sau; đánh dấu cạnh này vào ảnh top. | Gần z = −1,7 m; mặt đường được xem phẳng trong cụm xung quanh xe. | Bus ở dist_m 31,97 m và các xe ở 44,35–44,35 m; nhóm xa nhất quanh 44 m. | Car, Bus, Pedestrian, Cyclist | Có xe/cụm ở y dương phía sau và ở hai bên trục trước-sau. | Phần cụm điểm xa, bóng/chiếm che và vật khó phân biệt class. |
| s01f33 | Phía trước xe vẫn là −y; +x sang trái và +y ra sau. | Gần z = −1,7 m theo mặt đường. | Cuboid xa nhất trong frame là Car ở dist_m 17,32 m. | Chủ yếu Car. | Xe ở y dương phía sau và hai bên. | Chưa có đủ vật đọc rõ trong frame; cụm xa chưa chắc. |

Các thông tin chỉ từ camera: loại hình, lớp Car/Bus/Truck/Pedestrian/Cyclist; đầu xe để xác định hướng yaw; tình trạng che hoặc vật quá thưa trên ảnh. Các thông tin chỉ từ LiDAR: khoảng cách theo mét, kích thước, độ cao/đáy cuboid, cụm điểm ở phía sau và hai bên mà camera không bao phủ.

Tám frame lướt:

| Frame | Phía trước ở ảnh top | Điều lạ |
|---|---|---|
| s01f33 | −y ở phía trước | Chỉ 3 cuboid trong bảng A. |
| s02f03 | −y ở phía trước | Có Bus gần và xe xa tới hơn 46 m. |
| s02f51 | −y ở phía trước | Có Car tại z_bottom −1,553 m có cảnh báo đáy cao 0,62 m. |
| s03f01 | −y ở phía trước | Có 8 cuboid; tập trung xe con và một Truck. |
| s03f34 | −y ở phía trước | Chỉ 4 cuboid; có Pedestrian và Cyclist. |
| s04f16 | −y ở phía trước | Có xe con ở hai bên và một Cyclist. |
| s04f40 | −y ở phía trước | Xe dày ở khoảng 4,8–26,2 m. |
| s05f30 | −y ở phía trước | Có cảnh báo mép ở Car idx 1. |
| s05f45 | −y ở phía trước | Có Truck xa và hai Cyclist. |

Bốn frame còn lại trong danh sách gồm chính hai frame đọc kỹ và một phần của tám frame lướt; không thêm frame ngoài task.

## 3. Ba cuboid có hướng dẫn (Bước 3)
Ảnh: `t2-a-car.png`, `t2-b-bus-truck.png`, `t2-c-ped-cyclist.png`.

| Vật | Frame, idx | Class | Mũi tên yaw đúng hướng? | Đáy chạm đường? | Đã kiểm Top/Side/Front + ảnh? | Kích thước trong bảng 4.3? |
|---|---|---|---|---|---|---|
| (a) xe con gần | s01f00, idx 1 | Car | yaw 271,1° gần 270°, đúng xe cùng chiều phía trước. | z_bottom −1,845 m, cách mặt đường khoảng 0,145 m theo mốc −1,7 m, gần chạm. | Đã kiểm đủ ba view và camera preview. | length 4,402 m, width 1,692 m, height 1,600 m đều trong khung xe con. |
| (b) xe buýt hoặc xe tải | s01f00, idx 6 | Bus | yaw 0,0° cần đối chiếu đầu xe trên ảnh; không suy ra 270° nếu hướng chưa rõ. | z_bottom −1,684 m, chạm gần mặt đường. | Đã kiểm Top/Side/Front và đối chiếu preview. | length 9,668 m, width 2,888 m, height 3,273 m nằm trong nhóm Bus/xe lớn. |
| (c) người đi bộ hoặc xe hai bánh | s01f00, idx 11 | Pedestrian | yaw 92,2° dùng hướng theo người đi. | z_bottom −1,937 m; gần mặt đường, lệch 0,237 m. | Đã kiểm Top/Side/Front và đối chiếu preview. | length 0,533 m, width 0,499 m, height 1,685 m trong khung người lớn. |

Vòng hiệu chuẩn không được dùng để thay đổi nhãn riêng của task; việc khác nhau giữa các cuboid nên được sửa theo Top/Side/Front và guideline, không theo số đông. Các vật bỏ qua do chưa đủ cơ sở chưa có danh sách cụ thể; trường hợp nghi UNCERTAIN_CLASS/LOW_VISIBILITY cần ghi frame và vị trí khi phát hiện.

## 4. Bốn hiện tượng chất lượng (Bước 4)
Nguyên nhân dưới đây là giả thuyết.

| Hiện tượng | Frame / tên ảnh / nguồn | Nguyên nhân (giả thuyết) | Thấy gì trên point cloud | Hệ quả khi gán (theo guideline) |
|---|---|---|---|---|
| 1. Bóng che | s02f03 / `t4-1-bong-che.png` / task cá nhân | Một xe hoặc vật lớn có thể chắn LiDAR phía sau, tạo vùng có ít điểm. | Vùng sau vật có cụm điểm thưa hoặc gần như trống; hai bên vẫn còn điểm. | Bật occluded và bao phần suy luận được; không coi vùng trống sau vật là vật mới. |
| 2. Thưa ở xa | s02f03 idx 7 / `t4-2-thua-o-xa.png` / task cá nhân | Số điểm LiDAR giảm vì khoảng cách, nên cụm xe xa có thể bị thiếu phần thân. | Cuboid Car ở dist_m 46,10 m chỉ có 39 điểm trong phiếu tự kiểm. | Chỉ vẽ nếu thấy khoảng 15–20% thể tích hoặc có đủ cơ sở; dưới ngưỡng thì ghi LOW_VISIBILITY và bỏ qua nếu không đủ cơ sở. |
| 3. Phản xạ thấp / ít điểm bất thường | s05f30 idx 1 hoặc s05f45 idx 6 / `t4-3-phan-xa-thap.png` / task cá nhân | Bề mặt tối, kính hoặc góc quan sát có thể phản xạ ít điểm; chưa khẳng định do thiếu thông tin cảm biến. | Cụm có số điểm lệch so với vật cùng khoảng cách hoặc có số điểm sát mép hộp. | Ghi giả thuyết và không nới rộng hộp theo quầng điểm; vẫn căn kích thước theo point cloud và ảnh camera. |
| 4. Điểm lạ | s02f03 / `t4-4-diem-la.png` / task cá nhân | Điểm lẻ có thể do nhiễu, phản xạ hoặc cụm bụi; nguồn cụ thể chưa xác định. | Cụm điểm không tạo hình dạng hoặc nằm ngoài vùng vật quan sát được. | Không gán điểm lạ thành vật; nếu không chắc class hoặc scope thì ghi UNCERTAIN_CLASS/UNCERTAIN_SCOPE thay vì đoán. |

## 5. Số liệu cuboid (Bước 5)
Dữ liệu lấy trực tiếp từ `submission/D13-2A202603053-bangA.csv`; số điểm lấy từ `submission/D13-2A202603053-tukiem.csv`. Mặt đường tham chiếu khoảng −1,7 m.

| Cuboid (frame, idx, class) | z | h | z đáy = z − h/2 | dist_m | r = √(x²+y²+z²) | yaw_deg | Nhận xét |
|---|---:|---:|---:|---:|---:|---:|---|
| s01f00, idx 1, Car | −1,045 m | 1,600 m | −1,845 m | 3,32 m | 3,48 m | 271,1° | Chạm gần đường, lệch 0,145 m. |
| s02f51, idx 4, Car | −1,053 m | 1,000 m | −1,553 m | 39,19 m | 39,20 m | 91,9° | Có dấu lơ lửng hơn 0,147 m so với mặt đường; phiếu gắn cờ 0,62 m so với ground-gap. |
| s03f34, idx 1, Pedestrian | −1,021 m | 1,579 m | −1,811 m | 10,58 m | 10,58 m | 84,1° | Gần chạm đường, lệch 0,111 m. |

Cuboid gần nhất là s01f00 idx 1, `dist_m = 3,32 m`, `r = 3,48 m`. Cuboid xa nhất là s02f03 idx 7, `dist_m = 46,10 m`, `r ≈ 46,12 m`.

So kích thước Pedestrian s03f34 idx 1: `l = 0,640 m`, `w = 0,506 m`, `h = 1,579 m` đều nằm trong bảng người lớn. Car s01f00 idx 1: `l = 4,402 m`, `w = 1,692 m`, `h = 1,600 m` đều nằm trong khung Sedan/Hatchback hoặc SUV. Không chiều nào ngoài khoảng trong hai vật mẫu này.

Xe cùng chiều phía trước s01f00 idx 1 có yaw 271,1°, gần 270°. Xe ngược chiều s01f00 idx 5 có yaw 90,4°, gần 90°.

Mật độ theo khoảng cách (4 cuboid ở các khoảng khác nhau):

| Cuboid (frame, idx, class) | dist_m | Số điểm (`points_in`) |
|---|---:|---:|
| s01f00, idx 1, Car | 3,32 m | 2984 |
| s01f00, idx 2, Car | 12,43 m | 658 |
| s01f00, idx 5, Car | 25,91 m | 394 |
| s01f00, idx 8, Car | 40,29 m | 94 |

Xa gấp khoảng 12,13 lần từ 3,32 m lên 40,29 m thì số điểm giảm từ 2984 còn 94 điểm, tức còn khoảng 3,15%; mật độ LiDAR giảm mạnh theo khoảng cách và phù hợp xu hướng slide (1968 ở 0–10 m, 718 ở 10–20 m, 214 ở 20–30 m, 72 ở 30–40 m, 19 ở 60–80 m).

Cảnh báo tự kiểm: 8/74 cuboid có cảnh báo. Các cảnh báo gồm đáy thấp/chìm ở s02f03 idx 8; kích thước ở s02f51 idx 4 và s02f03 idx 9; mép hộp ở s04f16 idx 2, s05f30 idx 1 và s05f45 idx 6; đáy/kích thước ở s01f00 idx 10 và Cyclist s05f45 idx 9. Hai cuboid đã sửa sau preview/selfcheck chưa thể ghi lịch sử cụ thể vì lịch sử chỉnh sửa trên CVAT không được export; cần bổ sung frame/idx theo nhật ký thực tế trước nộp.

## 6. Ý nghĩa của nhãn, cách dùng AI và điều còn chưa chắc
Nhãn 3D đo bằng mét nên model có thể học kích thước, khoảng cách và hình học thực tế độc lập với độ phân giải pixel; nhãn pixel có thể thay đổi theo camera nhưng z, dài/rộng/cao và BEV giữ ý nghĩa vật lý. Vật thưa điểm cần quy tắc riêng vì 15–20% thể tích hoặc cơ sở từ ảnh vẫn có thể tạo bằng chứng cho một vật, trong khi điểm quá ít không nên chuyển thành đoán không có cơ sở.

Không dùng AI để làm bài này. Công cụ kiểm tra chỉ là script lab và dữ liệu bảng A do task sinh ra.

Điều còn chưa chắc: cần xác nhận bằng mắt ảnh/Top/Side/Front cho các vật có cảnh báo, nhất là cuboid xa và cuboid có điểm sát mép. Không lấy số từ phiếu hồ sơ lỗi hoặc số nhãn chuẩn để điền vào báo cáo.
