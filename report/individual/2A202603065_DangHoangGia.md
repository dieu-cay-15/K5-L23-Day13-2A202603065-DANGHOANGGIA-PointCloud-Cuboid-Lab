# Báo cáo Day 13 — Đặng Hoàng Gia, 2A202603065

## 1. Kết luận trước

- **Thông tin bài lab:** MSSV `2A202603065`, task CVAT `D13-2A202603065`, bộ 10 frame được giao gồm `s01f18`, `s01f41`, `s02f09`, `s02f46`, `s03f22`, `s03f52`, `s04f24`, `s04f56`, `s05f01`, `s05f17`. Môi trường làm việc: Windows 10 (build 19045), trình duyệt Chrome/Chromium, CVAT `v2.83.0`. MSSV bạn ngồi cạnh: tự làm cá nhân (chưa ngồi ghép đôi).
- **Tổng quan kết quả:** Đã vẽ và lưu tổng cộng **73 cuboid** dạng `Shape` (`62 Car`, `4 Bus`, `5 Cyclist`, `2 Truck`, `0 Track`) trên cả 10 frame. Các frame ít cuboid nhất là `s02f46`, `s03f52` và `s04f56` (mỗi frame **6 cuboid**) vì đường hẹp có rào chắn/tường quây sát hai bên và ít phương tiện rõ trong vòng `30 m`. Phiếu `selfcheck` chỉ cờ **2/73 cuboid** (`s02f46` `#1` và `#3` do dải phân cách kim loại sát hông xe). Phần tự tin nhất là **hướng mũi tên yaw (`+X`), phân loại class và độ cao đáy chạm mặt đường** (`ground_gap_m` phổ biến `0,02–0,03 m`); phần còn yếu nhất là **ước lượng chính xác chiều dài các vật ở xa `30–40 m`** và các xe bị che khuất một phần phía sau bụi cây do cụm điểm thưa.
- **Cách dùng CVAT 3D:**
  1. **Đổi góc nhìn:** Kéo chuột trái để xoay khung 3D (xoay thẳng đứng từ trên xuống để quét 360° quanh xe), kéo chuột phải để dời khung, lăn con lăn để phóng to/thu nhỏ, nháy đúp chỗ trống để về góc mặc định.
  2. **Vẽ và chỉnh cuboid:** Chọn công cụ Cuboid ở thanh trái → chọn class → bấm **Shape** (không bấm Track) → nháy đúp lên cụm điểm để đặt; chọn hộp rồi chỉnh tâm, hướng yaw ở khung **Top**, chỉnh độ cao đáy chạm đường ở **Side/Front** (tuyệt đối không xoay ở `Side`/`Front`), và điều chỉnh kích thước `L/W/H` ở mục `DETAILS`.
  3. **Bật hướng và đối chiếu camera:** Bật `Appearance → Cuboid orientation` để căn vạch đỏ `+X` hướng thẳng về đầu xe/hướng mặt người; bấm `Save` (`Ctrl+S`) sau mỗi frame và chạy `python scripts/lab.py preview` để chiếu hộp lên đủ 7 ảnh camera (`cam01`–`cam09`) trong `work/xem/`.

## 2. Đọc frame (Bước 2)

Hai frame đọc kỹ (ảnh chụp: `t1-s01f18-3d.png`, `t1-s01f18-top.png`, `t1-s01f41-3d.png`, `t1-s01f41-top.png`):

| Frame | Phía trước xe ở cạnh nào của ảnh nhìn từ trên xuống; +x, +y chỉ về đâu | Mặt đường (phẳng/dốc; thấp hơn LiDAR ~ m) | Vật xa nhất đọc được (~ m) | Loại vật thấy | Vật phía sau / hai bên | Cụm chưa đọc được |
|---|---|---|---|---|---|---|
| `s01f18` | Phía trước (`−y`, khớp `cam01`) ở cạnh trên-trái (~11h30); `+x` (trái) chỉ sang trái-hơi xuống (~8h30); `+y` (sau) chỉ xuống dưới-phải (~5h30) | Tương đối phẳng; thấp hơn LiDAR khoảng `1,8 m` (`z ≈ −1,79 m`) | Khoảng `45–50 m` dọc theo trục đường trước và sau | `Car`, `Bus` | Phía sau có 3 `Car` (`cam05`/`cam08`/`cam09`: xanh đậm sau-trái `11,28 m`, bạc sau-phải `11,97 m`, trắng ngay sau `25,71 m`) và 1 `Car` xa hơn (`38,03 m`); bên trái qua dải phân cách có 2 `Car` ngược chiều (`cam05`/`cam06`, `18,39 m` và `27,32 m`) | Vài cụm điểm nhỏ rời rạc sát hàng cây và bụi rậm lề phải (`x ≈ −9 m` đến `−12 m`, cách `13–24 m`), lẫn tán cây nên chưa xác định được là vật thể giao thông hay cọc/chậu cây |
| `s01f41` | Phía trước (`−y`, khớp `cam01`) ở cạnh trên-trái (~11h30); `+x` (trái) chỉ sang trái-hơi xuống (~8h30); `+y` (sau) chỉ xuống dưới-phải (~5h30) | Tương đối phẳng; thấp hơn LiDAR khoảng `1,8 m` (`z ≈ −1,81 m`) | Khoảng `40–45 m` dọc trục đường phía trước | `Car`, `Cyclist` | Phía sau có 2 `Car` (`cam05`/`cam09`: xanh đậm sau-trái `11,62 m`, trắng ngay sau `26,54 m`); bên phải trên làn xe đạp ngoài bụi cây có nhiều `Cyclist` (`cam01`/`cam07`/`cam08`, cách `10–25 m`) | Cụm điểm thưa phía sau dải cây bên phải cách trên `27 m` bị tán bụi che một phần và cụm thấp lẫn lề dải phân cách trái ở tầm `19 m` |

**Ba điều chỉ camera cho biết và ba điều chỉ LiDAR cho biết (từ `s01f18` và `s01f41`):**
- **Chỉ camera cho biết:** (1) Màu sắc và chi tiết phân biệt class chính xác (nhận ra xe buýt đỏ-trắng ở `s01f18` thay vì nhầm sang xe tải, nhận ra người mặc áo đỏ đang đạp xe ở `s01f41`); (2) Vạch kẻ làn, mũi tên mặt đường, đèn pha/đèn hậu để khẳng định chắc chắn đâu là đầu xe và đuôi xe; (3) Nhận biết người đi xe đạp nằm sát bụi cây ở `cam07`/`cam08` trong `s01f41` nơi điểm LiDAR hòa lẫn với tán lá.
- **Chỉ LiDAR cho biết:** (1) Tọa độ 3D và khoảng cách hình học chính xác theo mét từ cảm biến tới từng vật (ví dụ xe trắng gần nhất ở `s01f18` cách `6,59 m`, mặt đường thấp hơn LiDAR `~1,80 m`); (2) Kích thước thật theo mét (dài, rộng, cao) của vật thể ngay cả khi ở góc mù hoặc bị che một phần; (3) Toàn cảnh 360° liền mạch trong cùng hệ trục, cho thấy rõ độ phẳng mặt đường và vùng bóng che hình học phía sau xe lớn.

Tám frame lướt:

| Frame | Phía trước ở cạnh nào của ảnh nhìn từ trên xuống | Điều lạ (nếu có) |
|---|---|---|
| `s02f09` | Cạnh dưới-trái / góc trên-trái tùy góc xoay ban đầu, trùng hướng `cam01` (`−y`) | Ngã tư lớn có 2 xe buýt lớn bên trái tạo vùng bóng che hình quạt rất rộng; có nhiều hướng xe cắt ngang |
| `s02f46` | Hướng dọc trục `−y` trùng `cam01` | Dải phân cách kim loại dài chạy sát bên trái (`x ≈ +5 m`), mặt đường hơi dốc xuống (`z ≈ −2,3 m`) và có cụm điểm nhiễu dưới mặt đường |
| `s03f22` | Hướng dọc trục `−y` trùng `cam01` | Giao lộ có nhiều xe đi ngược chiều bên trái và xe/xe máy cắt ngang phía sau phải |
| `s03f52` | Hướng dọc trục `−y` trùng `cam01` | Xe sedan tối màu ngay phía trước (`9,74 m`) có vùng kính hậu ít điểm; xe buýt ngược chiều chạy sau dải cây bên trái |
| `s04f24` | Hướng dọc trục `−y` trùng `cam01` | Ngã tư rất rộng, có xe máy phía sau (`cam09`), xe ngược chiều đi sát bên trái (`7,61 m`) và hàng xe chờ bên phải |
| `s04f56` | Hướng dọc trục `−y` trùng `cam01` | Rào vàng bên trái và tường quây công trình dài bên phải chắn hoàn toàn tia LiDAR sang vùng ngoài tường |
| `s05f01` | Hướng dọc trục `−y` trùng `cam01` | Xe tải ben lớn đi sát ngay phía sau (`11,55 m`, thấy rõ trên `cam09`) và xe đa dụng trắng sát sau-phải (`5,17 m`) tạo bóng che lớn |
| `s05f17` | Hướng dọc trục `−y` trùng `cam01` | Xe tải ben lớn ngược chiều bên trái qua dải cây (`12,60 m`, `6192` điểm) chắn một vùng trống rộng phía sau nó |

## 3. Ba cuboid có hướng dẫn (Bước 3)

Ảnh chụp mẫu: `t2-a-car.png`, `t2-b-bus-truck.png`, `t2-c-ped-cyclist.png`.

| Vật | Frame, idx | Class | Mũi tên yaw đúng hướng? | Đáy chạm đường? | Đã kiểm Top/Side/Front + ảnh? | Kích thước trong bảng 4.3? |
|---|---|---|---|---|---|---|
| (a) xe con gần | `s01f18`, `idx = 1` | `Car` | Đúng (`yaw_deg = 268,0°`, vạch đỏ `+X` chỉ thẳng đầu xe theo `−y`) | Chạm đường (`z_bottom = −1,804 m`, `ground_gap_m = 0,03 m`) | Đã kiểm đủ Top/Side/Front và `cam01`, `cam06`, `preview` | Trong khoảng Sedan/hatchback (`4,550 × 1,850 × 1,585 m` so với `3,8–5,0 × 1,70–1,90 × 1,40–1,60 m`) |
| (b) xe buýt hoặc xe tải | `s01f18`, `idx = 2` | `Bus` | Đúng (`yaw_deg = 270,0°`, vạch đỏ `+X` chỉ thẳng đầu xe buýt theo `−y`) | Chạm đường (`z_bottom = −2,041 m`, `ground_gap_m = 0,03 m`) | Đã kiểm đủ Top/Side/Front và `cam01`, `cam03`, `preview` | Dài và cao trong khoảng Xe buýt thường (`11,489 × 2,620 × 3,286 m` so với `10,0–12,5 × 2,50–2,60 × 3,00–3,50 m`; rộng vượt `0,02 m` trong sai số ±5%) |
| (c) người đi bộ hoặc xe hai bánh | `s01f41`, `idx = 7` *(lấy ở frame 2 vì frame 1 không có cụm gần rõ)* | `Cyclist` | Đúng (`yaw_deg = 270,0°`, vạch đỏ `+X` chỉ theo hướng di chuyển `−y`) | Chạm nền làn xe đạp (`z_bottom = −1,306 m`, `ground_gap_m = 0,03 m`) | Đã kiểm đủ Top/Side/Front và `cam01`, `cam03`, `cam07`, `preview` | Dài và cao trong khoảng Xe đạp có người (`1,800 × 0,720 × 1,641 m` so với `1,6–1,9 × 0,50–0,70 × 1,60–1,90 m`; rộng nhỉnh `0,02 m` trong ±10%) |

- **Vòng hiệu chuẩn và chốt guideline:** Đã đối chiếu quy ước trên task chung `D13-HC485087` và rà lại các lỗi dễ lệch: (1) phân biệt đúng `Bus` với `Truck` trên `s01f18` (lúc đầu từng gán nhầm xe buýt thành `Truck` trước khi sửa về `Bus`); (2) luôn bật `Cuboid orientation` để kiểm tra mũi tên đỏ `+X` dọc thân và chỉ đúng đầu xe theo ảnh camera; (3) với xe chỉ thấy mặt đuôi hoặc đầu, vẫn kéo đủ chiều dài chuẩn theo bảng 4.3 thay vì bó hẹp vào mảng điểm đơn mặt.
- **Các vật bỏ qua vì không đủ cơ sở:**
  - `s01f18`, phía trước trên `45 m` (`y < −45 m`): hai xe rất xa phía trước xe buýt, chỉ còn vài điểm rời rạc không đủ 15–20% thể tích (`LOW_VISIBILITY`, `UNCERTAIN_SCOPE`).
  - `s01f41`, làn xe đạp bên phải gần bụi cây (`x ≈ −9 m`, vùng `y` quanh `0 m` đến `+8 m`): người đi xe đạp bị hàng rào bụi cây che phần lớn cụm điểm LiDAR, điểm lẫn vào bụi (`LOW_VISIBILITY`).
  - `s04f56`, xe hai bánh đỗ sát vỉa hè không có người lái (không thuộc class `Cyclist`) và xe ngược chiều xa phía sau hàng rào vàng bị che khuất mạnh (`LOW_VISIBILITY`).

## 4. Bốn hiện tượng chất lượng (Bước 4)

| Hiện tượng | Frame / tên ảnh / nguồn | Nguyên nhân (giả thuyết) | Thấy gì trên point cloud | Hệ quả khi gán (theo guideline) |
|---|---|---|---|---|
| 1. Bóng che | `s02f09` / `t4-1-bong-che.png` / task cá nhân `D13-2A202603065` | **Giả thuyết:** Thân xe buýt đô thị cao lớn (`idx = 2`, dài `10,82 m`, cao `2,86 m`, cách `10,24 m` bên trái) chắn hoàn toàn các tia laser phát ra theo hướng phía sau nó | Phía sau thân xe buýt xuất hiện một vùng trống đen hình rẻ quạt rộng không có điểm mặt đường hay vật thể, trong khi hai bên góc quạt vẫn dày đặc các vòng tia LiDAR | Vùng trống là vùng chưa quan sát được, không suy luận là đường trống; nếu có vật bị che một phần thì bật cờ `occluded` và vẽ cuboid bao trọn phần thể tích suy luận được |
| 2. Thưa ở xa | `s04f56` / `t4-2-thua-o-xa.png` / task cá nhân `D13-2A202603065` | **Giả thuyết:** Các tia của LiDAR 40 kênh phân kỳ theo góc đứng và góc ngang khi khoảng cách tăng lên `40,62 m`, làm khoảng hở giữa các tia lớn hơn nên chỉ rất ít tia trúng vào xe trắng phía trước phải (`idx = 6`) | Tại khoảng cách `40,62 m`, chiếc xe chỉ còn `34` điểm thưa rời rạc ở phần đuôi/nóc, không còn đường nét bánh xe hay hai bên thân rõ như xe ở `6–10 m` | Áp dụng quy tắc `15–20%` thể tích: chiếc này còn nhìn rõ trên `cam03` và đủ cơ sở suy luận kích thước theo bảng 4.3 nên vẽ; các vật xa hơn quá ít điểm và không đủ cơ sở phải bỏ qua, ghi `LOW_VISIBILITY` |
| 3. Phản xạ thấp / ít điểm bất thường | `s03f52` / `t4-3-phan-xa-thap.png` / task cá nhân `D13-2A202603065` | **Giả thuyết:** Lớp sơn xe tối màu hấp thụ tia laser mạnh hơn và kính hậu/kính thân gây phản xạ gương hoặc truyền qua trên chiếc sedan tối màu phía trước (`idx = 2`, `dist_m = 9,74 m`) | Dù ở rất gần (`9,74 m`, tổng `1148` điểm tập trung ở cản sau và biển số sáng), vùng kính hậu và thân trên tối màu bị thưa/hụt điểm và có mức xám (intensity) thấp rõ rệt so với vùng phản xạ mạnh | Ghi nhận nguyên nhân là giả thuyết, không khẳng định; không thu nhỏ cuboid theo vùng bị mất điểm kính/sơn tối mà vẫn bao đủ thân xe dựa vào ảnh camera và bảng 4.3 |
| 4. Điểm lạ | `s02f35` / `t4-4-diem-la.png` / mượn từ task chung `D13-HC485087` | **Giả thuyết:** Phản xạ đa đường (multipath) giữa bề mặt phản xạ mạnh và mặt đường hoặc nhiễu đo thời gian bay của tia laser tạo ra tọa độ ảo sâu bên dưới mặt đất | Khi xoay nghiêng khung 3D nhìn ngang mặt đường, hiện rõ cụm điểm trắng rời rạc nằm sâu dưới mặt đường (thấp hơn mặt đường phẳng khoảng `11 m`, quanh `z ≈ −13,5 m`, vùng sau-trái cách `~29 m`) | Đây là sai hình học chắc chắn; theo guideline **tuyệt đối không gán nhãn thành vật thể**, bỏ qua và ghi nhận là điểm nhiễu/điểm lạ |

## 5. Số từ cuboid của tôi (Bước 5)

Ba cuboid đối chiếu độ cao đáy `z đáy = z − h/2` từ `submission/D13-2A202603065-bangA.csv`:

| Cuboid (frame, idx, class) | z | h | z đáy = z − h/2 | dist_m | r = √(x²+y²+z²) | yaw_deg | Nhận xét (lơ lửng / chìm / chạm đường) |
|---|---|---|---|---|---|---|---|
| `s01f18`, `idx = 1`, `Car` | `−1,012 m` | `1,585 m` | `−1,804 m` | `6,59 m` | `6,66 m` | `268,0°` | Chạm đường (`ground_gap_m = 0,03 m`, lệch mặt đường `−1,7 m` chỉ `0,10 m`) |
| `s01f41`, `idx = 1`, `Car` | `−0,994 m` | `1,580 m` | `−1,784 m` | `5,13 m` | `5,22 m` | `268,0°` | Chạm đường (`ground_gap_m = 0,03 m`, sát mặt đường quanh `−1,8 m`) |
| `s05f01`, `idx = 2`, `Truck` | `−0,189 m` | `2,992 m` | `−1,685 m` | `11,55 m` | `11,56 m` | `270,0°` | Chạm đường (`ground_gap_m = 0,03 m`, sát mặt đường `−1,7 m`) |

- **(a) Cuboid gần nhất và xa nhất:**
  - **Gần nhất:** `s01f41`, `idx = 1` (`Car`) với `dist_m = 5,13 m` và khoảng cách 3D `r = 5,22 m`.
  - **Xa nhất:** `s04f56`, `idx = 6` (`Car`) với `dist_m = 40,62 m` và khoảng cách 3D `r = 40,64 m`.
- **(b) So kích thước `l, w, h` với bảng 4.3:**
  - Xe hai bánh có người (`s01f41`, `idx = 7`, `Cyclist`): `l = 1,800 m`, `w = 0,720 m`, `h = 1,641 m`. So với *Xe đạp có người* trong bảng 4.3 (`l = 1,6–1,9 m`, `w = 0,50–0,70 m`, `h = 1,60–1,90 m`), chiều dài và chiều cao nằm trong khoảng, còn chiều rộng `w = 0,720 m` vượt nhẹ `0,02 m` (~2,9%, trong sai số ±10%) do bao trọn phần tay lái/khuỷu tay người lái ở khoảng cách `21,75 m`.
  - Xe con (`s01f18`, `idx = 1`, `Car`, sedan): `l = 4,550 m`, `w = 1,850 m`, `h = 1,585 m` — cả ba chiều đều nằm gọn trong khoảng *Sedan, hatchback* bảng 4.3 (`3,8–5,0 m`, `1,70–1,90 m`, `1,40–1,60 m`). (Một xe con xa bị che một phần như `s01f41`, `idx = 9` có `w = 2,127 m` vượt nhẹ ngưỡng `2,10 m` của SUV `0,027 m` do cụm điểm ở `32,92 m` hơi nở ngang).
- **(c) Góc `yaw_deg` của xe cùng chiều và ngược chiều:**
  - Xe cùng chiều phía trước (`s01f41`, `idx = 3`, `Car`): `yaw_deg = 270,0°` — đúng `270°` (hướng `−y`).
  - Xe ngược chiều (`s01f18`, `idx = 7`, `Car`): `yaw_deg = 90,0°` (và `s01f18`, `idx = 6`, `Car` là `92,0°`) — rất sát `90°` (hướng `+y`).

Mật độ theo khoảng cách (4 cuboid `Car` ở các `dist_m` khác nhau, `points_in` lấy từ `submission/D13-2A202603065-tukiem.csv`):

| Cuboid (frame, idx, class) | dist_m | Số điểm (`points_in`) |
|---|---|---|
| `s01f41`, `idx = 1`, `Car` | `5,13 m` | `2631` điểm |
| `s04f56`, `idx = 2`, `Car` | `10,54 m` | `1082` điểm |
| `s01f41`, `idx = 5`, `Car` | `26,46 m` | `199` điểm |
| `s04f56`, `idx = 6`, `Car` | `40,62 m` | `34` điểm |

- **Quy luật mật độ:** Khi khoảng cách tăng gần gấp **2 lần** (từ `5,13 m` lên `10,54 m`), số điểm còn khoảng **41%** (`2631` xuống `1082` điểm); xa gấp **5 lần** (`26,46 m`) chỉ còn khoảng **7,6%** (`199` điểm), và xa gấp **8 lần** (`40,62 m`) chỉ còn **1,3%** (`34` điểm), giảm nhanh xấp xỉ theo nghịch đảo bình phương khoảng cách (`~1/r²`). Kết quả này rất khớp với đường cong trung vị xe con trên slide bài giảng (`1968` điểm ở `0–10 m`, `718` ở `10–20 m`, `214` ở `20–30 m`, `72` ở `30–40 m`, `19` ở `60–80 m`).
- **Tự kiểm và chỉnh sửa sau `preview` / `selfcheck`:**
  - Các cuboid đã sửa sau đợt `sync` kiểm tra đầu: (1) `s01f41`, `idx = 7` (`Cyclist`) trước đó bị lơ lửng `+1,23 m` và lệch sang trái còn `2` điểm → đã hạ sát nền làn xe đạp và dời đúng tâm cụm điểm, tăng lên `39` điểm với `ground_gap_m = 0,03 m`; (2) `s01f41`, `idx = 5` (`Car`) trước đó xoay ngược `yaw = 62,4°` → đã xoay 180° về đúng `270,0°`; (3) `s02f09`, `idx = 1` (`Car`) trước đó xoay ngang `164,0°` → đã xoay dọc làn về `272,0°` (`928` điểm); (4) `s02f46`, `idx = 2` (`Car`) trước đó đặt cao lơ lửng `+8,32 m` → đã hạ chạm mặt đường (`ground_gap_m = 0,03 m`, `34` điểm).
  - Các cảnh báo giữ nguyên sau lần `sync` cuối: `s02f46`, `idx = 1` (`123 điểm sát ngay ngoài mép`) và `s02f46`, `idx = 3` (`11 điểm sát ngay ngoài mép`). Lý do giữ nguyên: sau khi đối chiếu ảnh `cam01`, `cam03`, `cam06` và point cloud, các điểm sát mép đó thuộc hàng rào dải phân cách kim loại chạy ngay bên cạnh hai xe ngược chiều chứ không phải thân xe nên không nới hộp.
- **Đổi kiểm:** Chưa thực hiện đổi kiểm trực tiếp với bạn ngồi cạnh (làm cá nhân từ xa); đã tự đối chiếu chéo lại toàn bộ các con số `z`, `height`, `z_bottom`, `dist_m`, `yaw_deg` và `points_in` giữa báo cáo với `submission/D13-2A202603065-bangA.csv` và `submission/D13-2A202603065-tukiem.csv`, mọi số liệu đều khớp chính xác.

## 6. Ý nghĩa của nhãn, cách dùng AI và điều còn chưa chắc

- **(a) Từ góc độ mô hình học máy trên nhãn 3D:**
  - Nhãn 3D phải đo bằng **mét** trong hệ tọa độ thực quanh xe thay vì pixel 2D vì pixel bị biến dạng theo phối cảnh và tiêu cự camera, còn mô hình nhận thức tự lái cần học chính xác khoảng cách, kích thước hình học thật và hướng di chuyển trong không gian 3D để lập kế hoạch quỹ đạo và tính thời gian va chạm an toàn.
  - Vật thưa điểm cần quy tắc riêng (ngưỡng `15–20%` thể tích hoặc đủ cơ sở suy luận; không chắc thì bỏ qua) vì nếu người gán đoán bừa vị trí, kích thước hay hướng trên vài điểm nhiễu sẽ đưa nhãn sai (label noise) vào tập huấn luyện, khiến mô hình học sai đặc trưng hình học và dễ sinh dự đoán ảo hoặc sai hướng khi vận hành.
- **(b) Cách dùng AI:**
  - **Công cụ:** Trợ lý AI Google Antigravity (tích hợp điều khiển trình duyệt và kiểm tra script trong môi trường lab).
  - **Câu hỏi chính:** Hỗ trợ thực hiện các bước đọc frame, đối chiếu hệ tọa độ VF (`−y` trước, `+x` trái, `+y` sau), căn chỉnh cuboid 3D trên CVAT, đối chiếu ảnh 7 camera và kiểm tra chéo số liệu Bảng A / phiếu tự kiểm để điền báo cáo đúng chuẩn.
  - **Cách dùng kết quả:** Trực tiếp kiểm chứng lại trên giao diện CVAT 3D (`Top`/`Side`/`Front` + `Cuboid orientation`), tự chạy lệnh `lab.py sync`, `pack`, `check_submission.py`, xem ảnh chiếu `work/xem/` và kiểm tra lại toàn bộ con số thực từ `D13-2A202603065-bangA.csv` và `-tukiem.csv` trước khi chốt bài nộp.
- **(c) Điều còn chưa chắc:** Hai xe con đi ngược chiều sau dải bụi cây rậm (`s01f18` `idx = 6` và `s05f01` `idx = 6`) bị tán cây che vùng gầm nên `selfcheck` không đủ điểm mặt đường cục bộ để tính `ground_gap_m`, do đó độ cao đáy chỉ ước lượng tương đồng với mặt đường kế cận. Ngoài ra, một vài xe ở khoảng cách xa `35–40 m` (`s01f18` `idx = 8`, `s02f46` `idx = 3`, `s04f56` `idx = 6`) chỉ còn `14–34` điểm nên phần chiều dài thân xe suy luận theo bảng 4.3 có thể còn chênh nhẹ so với kích thước thực tế.
