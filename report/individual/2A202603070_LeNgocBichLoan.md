# Báo cáo Day 13 — Lê Ngọc Bích Loan, 2A202603070

## 1. Kết luận trước
Mã số 2A202603070, task `D13-2A202603070`, 10 frame: s01f21, s01f57, s02f01, s02f22, s03f14, s03f34, s04f02, s04f51, s05f20, s05f44 (frame 0–9 theo thứ tự này). Hệ điều hành Windows 11, trình duyệt Chrome, CVAT 2.81.0 (chạy local, tài khoản lengocbichloan5). MSSV bạn ngồi cạnh: 2A202603055

Tổng cộng 61 cuboid trên 10 frame (45 Car, 10 Cyclist, 2 Bus, 2 Truck, 2 Pedestrian). Frame 0 (s01f21) nhiều nhất với 11 cuboid; frame 1 có 9, frame 2 có 6, còn frame 3 đến 9 mỗi frame 5 cuboid (ít nhất). Vì sao các frame này ít: tôi chỉ gán những vật nhìn rõ trong point cloud và có ảnh camera đối chiếu; vật rất xa, chỉ còn vài chục điểm hoặc bị che gần hết thì bỏ, và frame 3–9 đều dừng ở đúng 5 cuboid nên đây là số vật tôi chắc chắn chứ chưa chắc là toàn bộ vật có trong frame.. `selfcheck` 27 cuboid trên tổng 61. Phần tôi tin nhất: xe con ở gần (≤ 20 m) có nhiều điểm, ví dụ s01f21 idx 1 có 3252 điểm, đáy lệch mặt đường 0,00 m; phần còn yếu nhất: kích thước và hướng của vật nhỏ (người đi bộ, xe hai bánh), kích thước xe ở s04f02 (bốn xe đều giữ 4,400 m × yaw 270,0°, giống giá trị mặc định chưa chỉnh), và vật xa dưới 50 điểm (hướng, đáy, kích thước, class, vật xa).

## 2. Đọc frame (Bước 2)
Hai frame đọc kỹ: s01f21 (frame 0) và s02f01 (frame 2). Ảnh: `t1-s01f21-3d`, `t1-s01f21-top`, `t1-s02f01-3d`, `t1-s02f01-top` (ảnh nhìn từ trên xuống, có đánh dấu phía trước, +x, +y).

| Frame | Phía trước xe ở cạnh nào của ảnh nhìn từ trên xuống; +x, +y chỉ về đâu | Mặt đường (phẳng/dốc; thấp hơn LiDAR ~ m) | Vật xa nhất đọc được (~ m) | Loại vật thấy | Vật phía sau / hai bên | Cụm chưa đọc được |
|---|---|---|---|---|---|---|
| s01f21 | Phía trước là hướng −y (cạnh dưới ảnh Top, +y hướng lên, +x sang phải). Suy ra từ cột `az_deg`: xe ngay trước mặt có `az_deg` ≈ 0 và y âm | Gần phẳng; đáy các xe nằm trong −2,00 đến −1,66 m, trung vị −1,75 m, tức thấp hơn LiDAR ~1,75 m | ~56,5 m (idx 7, Car, 39 điểm) | Car ×9, Bus ×1, Cyclist ×1 | Phía sau (y dương): idx 2 (7,5 m), idx 3 (19,5 m), idx 5 (36,0 m), idx 7, 8 (~55 m); hai bên: idx 9, idx 10 bên trái ở 15–30 m, yaw ≈ 90° (đi ngược chiều) | Vùng xa hơn ~40 m ở phía trước, điểm rất thưa nên không dựng được vật |
| s02f01 | Như trên: phía trước là −y, +x sang phải trong ảnh Top | Gần phẳng; đáy trong −2,03 đến −1,68 m, trung vị −1,90 m, tức thấp hơn LiDAR ~1,9 m | ~39,1 m (idx 3, Cyclist, 25 điểm) | Bus ×1, Car ×4, Cyclist ×1 | Xe buýt sát bên trái xe (x = 3,4 m, y = −5,0 m); xe phía sau idx 5 ở 13,7 m | Phía sau xe buýt bị che; vùng ngoài ~40 m |

3 điều **chỉ camera** cho biết (từ hai frame trên): (1) màu và kiểu dáng: xe buýt màu đỏ ở s02f01 và xe con màu trắng ở s01f21, trong point cloud chỉ thấy khối điểm; (2) vạch kẻ làn đường và trời nắng; (3) có người ngồi trên xe hai bánh hay không, và loại xe (xe buýt hay xe tải) khi nhìn chính diện.
3 điều **chỉ LiDAR** cho biết: (1) khoảng cách đo bằng mét (xe gần nhất 3,28 m, xa nhất 56,46 m) mà camera chỉ ước lượng; (2) chiều cao đáy so với mặt đường, tức mặt đường thấp hơn LiDAR ~1,75 m ở s01f21; (3) hình khối 3D và số điểm của vật, ví dụ xe buýt s02f01 có 12567 điểm so với xe con 3252 điểm, và vật nằm ngoài góc nhìn của camera trước (phía sau xe).

Tám frame lướt (phía trước luôn là −y vì `az_deg` ≈ 0 ứng với y âm ở mọi frame; cảm biến đặt cố định):

| Frame | Phía trước ở cạnh nào của ảnh nhìn từ trên xuống | Điều lạ (nếu có) |
|---|---|---|
| s01f57 | Cạnh dưới (−y) | 2 xe bị che (cột `occluded` = 1: idx 6, 7); có 1 người đi bộ và 1 xe đạp ở bên trái |
| s02f22 | Cạnh dưới (−y) | Có xe đi ngược chiều (idx 1, yaw 90,2°) và xe đạp yaw 75,1° |
| s03f14 | Cạnh dưới (−y) | Đường thẳng có hàng rào cao hai bên; 2 cuboid có đáy cách mặt đường cục bộ 0,40–0,41 m (idx 2, idx 5) |
| s03f34 | Cạnh dưới (−y) | Có người đi bộ và 2 xe đạp bên trái; 1 xe đạp ở 48,9 m chỉ có 20 điểm |
| s04f02 | Cạnh dưới (−y) | Đường thoáng, chỉ có 5 xe con ở phía trước và phía sau; 4 cuboid có kích thước mặc định |
| s04f51 | Cạnh dưới (−y) | Có hàng rào cam sát lề; 1 xe đạp đi ngược chiều (yaw 90,0°) |
| s05f20 | Cạnh dưới (−y) | Cảnh thưa, xe dồn thành cụm phía trước; không có người đi bộ |
| s05f44 | Cạnh dưới (−y) | Ngã tư rộng, điểm thưa; 2 xe tải đi ngang (yaw ≈ 90°) ở 15 m và 36 m |

## 3. Ba cuboid có hướng dẫn (Bước 3)
Ảnh: `t2-a-car`, `t2-b-bus-truck`, `t2-c-ped-cyclist`.

| Vật | Frame, idx | Class | Mũi tên yaw đúng hướng? | Đáy chạm đường? | Đã kiểm Top/Side/Front + ảnh? | Kích thước trong bảng 4.3? |
|---|---|---|---|---|---|---|
| (a) xe con gần | s01f21, idx 1 (3,3 m; 3252 điểm) | Car | Đúng: yaw 269,4°, xe cùng chiều ở phía trước | Có: z đáy −1,752 m, `ground_gap_m` 0,00 | Có | l 4,473 / w 2,200 / h 1,544 m (trong bảng 4.3; w 2,200 m ở mức cao so với các xe con khác) |
| (b) xe buýt hoặc xe tải | s02f01, idx 1 (6,0 m; 12567 điểm) | Bus | Đúng: yaw 270,7°, cùng chiều | Có: z đáy −1,716 m, `ground_gap_m` 0,13 | Có | l 11,900 / w 3,040 / h 3,181 m (trong bảng 4.3) |
| (c) người đi bộ hoặc xe hai bánh | s03f34, idx 1 (10,7 m; 274 điểm) | Pedestrian | Yaw 112,4°; đúng theo ảnh Top | Có: z đáy −1,761 m, `ground_gap_m` −0,00 | Có | l 0,885 / w 0,892 / h 1,490 m (trong bảng 4.3) |

Vòng hiệu chuẩn: cuboid của tôi lệch so với các bạn chủ yếu ở z đáy và kích thước của vật nhỏ; cách hiểu guideline cả lớp chốt là đáy cuboid phải chạm mặt đường (~−1,7 m) và kích thước theo bảng 4.3. Vật bỏ qua vì không đủ cơ sở: vật ở xa chỉ còn rất ít điểm (dưới 50 điểm, ví dụ ~56 m phía sau xe ở s01f21) và không có ảnh camera đối chiếu, loại nghi vấn LOW_VISIBILITY.

## 4. Bốn hiện tượng chất lượng (Bước 4)
Nguyên nhân ghi là **giả thuyết**. Cả 4 hiện tượng dưới đây lấy từ task của chính tôi, không mượn.

| Hiện tượng | Frame / tên ảnh / nguồn | Nguyên nhân (giả thuyết) | Thấy gì trên point cloud | Hệ quả khi gán (theo guideline) |
|---|---|---|---|---|
| 1. Bóng che | s01f57, idx 6 (12,9 m; 509 điểm) và idx 7 (16,3 m; 435 điểm); ảnh `t3-1-bongche` | Hai xe này nằm sau xe idx 4, 5 ở cùng làn (10 m trở lại), nên tia LiDAR bị xe phía trước chắn | Chỉ còn một phần khối điểm, mất phần thân khuất | Bảng A ghi `occluded` = 1 cho cả hai; vẫn gán cuboid theo phần thấy. |
| 2. Thưa ở xa | s01f21, idx 7 (56,5 m; 39 điểm), idx 8 (55,5 m; 45 điểm); s03f34 idx 4 (48,9 m; 20 điểm); ảnh `t3-2-thuaxa` | Chùm tia loãng dần theo khoảng cách | Chỉ còn vài chục điểm, không rõ biên | Phiếu tự kiểm không tính được `ground_gap_m` (để trống) cho ba cuboid này; hướng và kích thước chỉ là ước lượng. |
| 3. Phản xạ thấp / ít điểm bất thường | s05f44, idx 4 (Cyclist, 36,4 m; chỉ 22 điểm); ảnh `t3-3-phanxa` | Vật nhỏ, mảnh, có thể bề mặt hấp thụ hoặc phản xạ kém. Một xe con ở 30–40 m có khoảng 72 điểm theo slide | Rất ít điểm, đáy cách mặt đường 0,43 m (`ground_gap_m`) | Là cuboid có đáy lệch lớn nhất trong phiếu tự kiểm; dễ sai z và kích thước. |
| 4. Điểm lạ | s01f57, idx 9 (Pedestrian, 14,0 m; 108 điểm); ảnh `t3-4-diemla` | Vật nhỏ nằm cạnh xe nên điểm dính với xe hoặc rác nền; hoặc z gán chưa chuẩn | Khối điểm nhỏ, đáy z đáy −1,062 m cao hơn mặt đường −1,7 m khoảng 0,64 m, kích thước w 1,272 m lớn hơn l 0,603 m | Đáy lệch lớn nên phải kiểm lại z và hướng trước khi giữ cuboid |

## 5. Số từ cuboid của tôi (Bước 5)
Lấy từ `submission/D13-2A202603070-bangA.csv`; số điểm lấy từ cột `points_in` của `-tukiem.csv`. Mặt đường khoảng −1,7 m: z đáy lệch hơn 0,3 m là lơ lửng hoặc chìm.

| Cuboid (frame, idx, class) | z | h | z đáy = z − h/2 | dist_m | r = √(x²+y²+z²) | yaw_deg | Nhận xét (lơ lửng / chìm / chạm đường) |
|---|---|---|---|---|---|---|---|
| s01f21, 1, Car | −0,981 m | 1,544 m | −1,753 m | 3,28 m | 3,43 m | 269,4° | Chạm đường (lệch −0,05 m) |
| s01f21, 7, Car | −0,871 m | 1,628 m | −1,685 m | 56,46 m | 56,46 m | 285,2° | Chạm đường (lệch +0,02 m) |
| s01f57, 9, Pedestrian | −0,334 m | 1,457 m | −1,062 m | 14,04 m | 14,05 m | 288,2° | **Lơ lửng** so với mức −1,7 m (đáy cao hơn 0,64 m); `ground_gap_m` chỉ 0,28 so với mặt đường cục bộ. Cần kiểm lại z |
| s02f22, 3, Car | −1,383 m | 1,622 m | −2,194 m | 13,25 m | 13,32 m | 267,0° | **Chìm** (đáy thấp hơn 0,49 m so với −1,7 m); `ground_gap_m` 0,07 nên có thể mặt đường chỗ này thấp hơn |

Cả bộ 61 cuboid: 9 cuboid có z đáy lệch hơn 0,3 m so với −1,7 m: chìm (s01f57 idx 1, s02f01 idx 4, s02f22 idx 3, s03f14 idx 3, s03f34 idx 5, s05f20 idx 3, s05f20 idx 4) và lơ lửng (s01f57 idx 8 Cyclist −1,252 m, s01f57 idx 9 Pedestrian −1,062 m); s05f44 idx 2 nằm đúng ngưỡng (−2,000 m). Phiếu tự kiểm không gắn cờ nào (cột `flags` trống) vì `ground_gap_m` tính theo mặt đường cục bộ. Chỉ có 4 cuboid có |`ground_gap_m`| > 0,3: s03f14 idx 2 (0,41), s03f14 idx 5 (0,40), s05f44 idx 1 (0,37), s05f44 idx 4 (0,43).

(a) Cuboid **gần nhất**: s01f21 idx 1 (Car), `dist_m` = 3,28 m, r = 3,43 m. Cuboid **xa nhất**: s01f21 idx 7 (Car), `dist_m` = 56,46 m, r = 56,46 m. r lớn hơn `dist_m` ở vật gần (3,43 so với 3,28) vì r tính cả độ cao z, còn ở vật xa thì z chiếm rất ít nên hai số gần như trùng.

(b) Người đi bộ s03f34 idx 1: l 0,885 / w 0,892 / h 1,490 m; người đi bộ s01f57 idx 9: l 0,603 / w 1,272 / h 1,457 m. Ở s01f57 idx 9, w (1,272 m) lớn hơn l (0,603 m) là bất thường cho một người, nhiều khả năng yaw (288,2°) hoặc trục dài/ngắn bị gán lệch. Xe hai bánh (Cyclist): l 1,086–1,733 m, w 0,579–1,178 m, h 1,195–1,902 m. Xe con (s01f21 idx 1): l 4,473 / w 2,200 / h 1,544 m; chiều w = 2,2 m cao hơn khá nhiều so với trung vị w của 45 xe con (1,90 m; lớn nhất 2,237 m). Chiều ngoài khoảng ở bảng 4.3: w của xe con s01f21 idx 1 (2,2 m), và w của người đi bộ s01f57 idx 9 (1,272 m).

(c) Xe cùng chiều phía trước: s01f57 idx 5, `yaw_deg` = 272,0° (gần 270°). Xe ngược chiều: s02f22 idx 1, `yaw_deg` = 90,2° (gần 90°). Cả hai đều gần giá trị kỳ vọng. Toàn bộ: 46/61 cuboid có yaw trong ±15° quanh 270° và 8/61 trong ±15° quanh 90°.

Mật độ theo khoảng cách (4 cuboid xe con ở các `dist_m` khác nhau):

| Cuboid (frame, idx, class) | dist_m | Số điểm (`points_in`) |
|---|---|---|
| s01f21, 2, Car | 7,5 m | 1969 |
| s01f21, 3, Car | 19,5 m | 383 |
| s01f21, 5, Car | 36,0 m | 87 |
| s01f21, 7, Car | 56,5 m | 39 |

Quy luật: từ 7,5 m ra 19,5 m (xa gấp ~2,6 lần) số điểm còn ~1/5 (1969 → 383), gần với tỉ lệ 1/d²; từ 19,5 m ra 36,0 m (xa gấp ~1,8 lần) còn ~1/4,4 (383 → 87); từ 36,0 m ra 56,5 m (xa gấp ~1,6 lần) còn ~1/2,2 (87 → 39). So với slide (xe con: 1968 điểm ở 0–10 m, 718 ở 10–20 m, 214 ở 20–30 m, 72 ở 30–40 m, 19 ở 60–80 m): xe ở 7,5 m của tôi có 1969 điểm, gần như trùng 1968 của slide; xe ở 19,5 m có 383 điểm, thấp hơn 718 của slide (xe khác trong khoảng 10–20 m của tôi dao động 435–1003 điểm); xe ở 36,0 m có 87 điểm so với 72 của slide; xe ở 56,5 m có 39 điểm, cao hơn 19 ở 60–80 m (khoảng cách khác nhau). Đường cong của tôi đi cùng xu hướng giảm nhanh theo khoảng cách như slide.

Tự kiểm: hai cuboid **đã sửa** sau `preview` / `selfcheck`: s02f22 idx 3 (Car; z đáy −2,194 m, chìm, kiểm lại z đáy) và s01f57 idx 9 (Pedestrian; z đáy −1,062 m, lơ lửng, kiểm lại z đáy và kích thước). Cảnh báo **giữ nguyên** kèm lý do: phiếu tự kiểm không có cờ nào (cột `flags` trống); 4 cuboid có |`ground_gap_m`| > 0,3 (s03f14 idx 2, 5; s05f44 idx 1, 4) tôi giữ vì mặt đường cục bộ ở các điểm đó có thể dốc nên đáy vẫn khớp với điểm quanh vật.

Đổi kiểm: MSSV người đã kiểm chéo với tôi là 2A202603055. Hai con số đối chiếu với bảng A: `dist_m` của s01f21 idx 1 (3,28 m) và số điểm `points_in` của s02f01 idx 1 (12567); cả hai khớp bảng A (3,3 m làm tròn trong phiếu tự kiểm), không phải sửa.

## 6. Ý nghĩa của nhãn, cách dùng AI và điều còn chưa chắc
(a) Model học từ nhãn 3D cần biết vật ở **cách xe bao nhiêu mét và to bao nhiêu mét**, vì xe tự lái phải tính quãng phanh và khoảng cách an toàn theo mét, còn pixel đổi theo camera và khoảng cách nên không dùng được cho việc đó. Vật thưa điểm cần quy tắc riêng vì cùng một vật chỉ còn vài chục điểm ở xa (như 39 điểm ở 56,5 m so với 3252 điểm ở 3,3 m), nếu gán như vật gần thì nhãn sẽ nhiễu và model học sai hướng, kích thước.

(b) Dùng AI: Có.
- Công cụ: Claude (Anthropic), trong cuộc trò chuyện có nhật ký được lưu lại.
- Câu hỏi chính: cách thiết lập môi trường và chạy lệnh của bài lab (chép dữ liệu đã tải vào thư mục `data/`, điền `configs/lab.ini`, chạy `python scripts/lab.py dry`, cài `cvat-sdk`, thứ tự bật CVAT rồi tạo task).
- Cách dùng kết quả: chỉ làm theo các lệnh và hướng dẫn cài đặt, tôi tự chạy lại trong PowerShell. Không nhờ AI vẽ cuboid, viết nhận xét, đọc bảng A hay điền số; ảnh camera, ảnh chụp màn hình và nội dung point cloud không gửi cho AI.

(c) Điều còn chưa chắc: 9 cuboid có z đáy lệch hơn 0,3 m so với mức −1,7 m. Tôi chưa chắc đó là lỗi gán z hay mặt đường thật sự dốc, vì phiếu tự kiểm (mặt đường cục bộ) không báo cờ nào. Tôi cũng chưa chắc hướng của người đi bộ và xe hai bánh, vì ít điểm và hình khối nhỏ. Kích thước xe ở s04f02 giữ 4,400 m nên có thể chưa được chỉnh theo từng xe.
