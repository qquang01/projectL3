# Game Design Document — LSS-78

## 1. Tổng quan

| Thuộc tính | Giá trị |
|---|---|
| Thể loại | 2.5D Survival / Base Building / Territory Conquest |
| Phong cách đồ họa | Don't Starve — hand-drawn 2D, isometric/2.5D camera |
| Engine | Unity URP |
| Nền tảng đầu tiên | PC offline (single-player) |
| Multiplayer | Listen Server (2–4 người), sau khi offline ổn định |
| Ngôn ngữ | Tiếng Việt |

---

## 2. Cốt truyện & Bối cảnh

**Bối cảnh**: Người chơi là thành viên đội xâm lăng đến hành tinh LSS-78. Phi thuyền bị phá hủy, buồng hồi sinh (Respawn Chamber) là công trình duy nhất còn sót lại.

**Mục tiêu**: Xây dựng căn cứ, khai thác tài nguyên, tiêu diệt sinh vật bản địa, và cuối cùng chiếm lĩnh toàn bộ hành tinh.

**Cơ chế hồi sinh**: Khi chết, người chơi hồi sinh tại buồng hồi sinh gần nhất. Tài nguyên trong túi đồ rơi lại tại chỗ chết (có thể nhặt lại).

---

## 3. Core Gameplay Loop

```
[Khám phá] → [Thu thập] → [Chế tạo/Xây dựng] → [Sinh tồn qua đêm]
    ↑                                              ↓
[Chiến đấu/Khám phá sâu hơn] ←── [Mở rộng lãnh thổ]
```

1. **Ngày**: Khám phá, thu thập, xây dựng
2. **Hoàng hôn**: Chuẩn bị vũ khí, đốt lửa, về căn cứ
3. **Đêm**: Sinh vật hung dữ xuất hiện, người chơi phải phòng thủ hoặc trốn tránh
4. **Sáng**: Đánh giá thiệt hại, lên kế hoạch mở rộng

---

## 4. Hệ thống Sinh tồn (Survival Systems)

### 4.1 Thanh chỉ số chính

| Chỉ số | Mô tả | Hồi phục | Hậu quả khi = 0 |
|---|---|---|---|
| **Máu (Health)** | Chịu sát thương từ quái, đói, lạnh | Thuốc, băng bó, hồi sinh | **Chết** → hồi sinh tại buồng |
| **Đói (Hunger)** | Giảm theo thời gian | Ăn thức ăn | Giảm máu từ từ |
| **Tinh thần (Sanity)** | Giảm khi đêm, thấy quái, ở một mình | Ăn đồ ngon, ở gần lửa, ngủ | Quái vật bóng tối tấn công |
| **Thể lực (Stamina)** | Dùng khi chạy, đánh, chặt cây | Nghỉ ngơi, ngủ | Không thể chạy/đánh |

### 4.2 Thời gian & Chu kỳ ngày đêm

- 1 ngày trong game = 8 phút thực (4' ngày / 2' hoàng hôn / 2' đêm)
- Mùa: Xuân → Hè → Thu → Đông (mỗi mùa 10 ngày game)
- Mùa Đông: đói nhanh hơn, cần lửa/lều để sống sót

---

## 5. Hệ thống Xâm chiếm Lãnh thổ (Territory Conquest)

### 5.1 Ý tưởng cốt lõi

Thay vì "endless survival", game có **mục tiêu cuối cùng**: chiếm toàn bộ hành tinh.

### 5.2 Cấu trúc Map — Thế lực đối địch

Map chia thành các **Vùng (Zone)**. Mỗi vùng có một trong các trạng thái:

| Trạng thái | Mô tả |
|---|---|
| **Hoang dã (Wild)** | Chưa khám phá, quái mạnh, không thể xây dựng |
| **Đang khám phá (Explored)** | Đã đặt chân tới, quái yếu hơn, có thể đặt cờ hiệu |
| **Kiểm soát (Controlled)** | Đặt được căn cứ tiền đồn, quái không spawn gần |
| **Căn cứ (Base)** | Xây dựng tự do, có buồng hồi sinh, an toàn |
| **Đã thanh tẩy (Cleansed)** | Hoàn toàn an toàn, cung cấp tài nguyên passive |

### 5.3 Tiến trình xâm chiếm

```
Hoang dã ──khám phá──→ Đã khám phá ──đặt cờ──→ Kiểm soát
                                                        ↓
Đã thanh tẩy ←──tiêu diệt boss vùng─── Căn cứ tiền đồn ←──xây dựng──┘
```

- Mỗi vùng có **Boss địa phương** hoặc **Trụ cột kẻ thù**
- Tiêu diệt Boss → vùng chuyển sang Cleansed → không còn quái nguy hiểm
- Mục tiêu cuối: Tất cả vùng đều Cleansed = Chiếm hành tinh

---

## 6. Hệ thống Hồi sinh (Respawn System)

### 6.1 Buồng hồi sinh (Respawn Chamber)

- Công trình cố định, không thể di chuyển
- Có thể xây thêm buồng hồi sinh mới tại các căn cứ tiền đồn đã kiểm soát
- Khi chết:
  1. Mất 25% tài nguyên trong túi (rơi tại chỗ chết)
  2. Hồi sinh tại buồng gần nhất với đầy máu, 50% đói
  3. Có thể chạy về chỗ chết để nhặt lại đồ rơi

### 6.2 Ngục tối (Death Penalty)

- Mỗi lần chết tăng **Mức độ cảnh báo kẻ thù** → quái trong vùng gần đó mạnh hơn
- Nếu chết quá nhiều lần trong một vùng chưa kiểm soát: xuất hiện **Elite quái** trả thù

---

## 7. Hệ thống Xây dựng Căn cứ (Base Building)

### 7.1 Loại công trình

| Loại | Ví dụ | Chức năng |
|---|---|---|
| **Cơ bản** | Lều, lửa trại, rào chắn | Sinh tồn qua đêm |
| **Sản xuất** | Lò rèn, bàn chế tạo, máy nghiền | Chế đồ cao cấp |
| **Nông nghiệp** | Vườn trồng, chuồng nuôi | Nguồn thức ăn ổn định |
| **Quân sự** | Tháp canh, bẫy, tường thành | Phòng thủ |
| **Đặc biệt** | Buồng hồi sinh, Radar, Cổng dịch chuyển | Tiến trình cốt truyện |

### 7.2 Placement

- Công trình chỉ đặt được trong vùng **Kiểm soát** trở lên
- Một số công trình lớn (Radar, Cổng) yêu cầu **Cleansed** vùng
- Công trình có thể bị quái phá hủy nếu không được bảo vệ

---

## 8. Hệ thống Thế giới & Kẻ thù

### 8.1 Sinh vật bản địa

| Loại | Hành vi | Độ nguy hiểm |
|---|---|---|
| **Thụ động** | Chạy trốn khi bị tấn công | Thấp — nguồn thịt/lông |
| **Trung lập** | Tấn công nếu bị khiêu khích | Trung bình |
| **Hung dữ (đêm)** | Tấn công người chơi | Cao — spawn đêm |
| **Boss vùng** | Bảo vệ trụ cột | Rất cao — cần chuẩn bị |
| **Elite (trả thù)** | Xuất hiện khi chết nhiều | Cao — di chuyển nhanh |

### 8.2 Hệ thống sinh sản quái (Spawn)

- **Ngày**: Chỉ quái thụ động + trung lập
- **Hoàng hôn**: Quái hung dữ bắt đầu xuất hiện ở vùng Wild/Explored
- **Đêm**: Quái hung dữ spawn đại trà, ánh sáng là biện pháp phòng vệ duy nhất
- **Vùng Cleansed**: Không spawn quái hung dữ

---

## 9. Tiến trình người chơi (Player Progression)

### 9.1 Công nghệ (Tech Tree)

- Cây công nghệ mở khóa qua **Nghiên cứu** tại bàn chế tạo nâng cấp
- Ví dụ: Lửa trại → Lò rèn → Lò điện → Lò phản vật chất
- Công nghệ cao cấp yêu cầu tài nguyên từ vùng nguy hiểm

### 9.2 Trang bị & Vũ khí

| Tier | Chất liệu | Độ bền | Sát thương |
|---|---|---|---|
| 1 | Gỗ / Đá | Thấp | Thấp |
| 2 | Đồng / Sắt | Trung bình | Trung bình |
| 3 | Thép / Năng lượng | Cao | Cao |
| 4 | Alien Tech | Rất cao | Rất cao — cần tiêu diệt Boss |

---

## 10. Phạm vi MVP (Phiên bản Offline đầu tiên)

### 10.1 Map

- 1 vùng duy nhất: **Rừng hoang LSS-78** (kích thước 256×256)
- Không có hệ thống mùa (chỉ ngày/đêm)
- Không có Boss vùng (thay bằng quái mạnh hơn vào đêm)

### 10.2 Hệ thống

- ✅ Di chuyển, chặt cây, đào đá, đánh quái
- ✅ Thanh Máu + Đói + Thể lực (bỏ Sanity ở MVP)
- ✅ Chế tạo: lửa trại, lều, giáo gỗ, rìu đá
- ✅ Xây dựng: rào chắn, lều (placement tự do trong vùng an toàn)
- ✅ Ngày/đêm + spawn quái đêm
- ✅ Chết → hồi sinh tại buồng hồi sinh ban đầu
- ❌ Không có Tech Tree (chế tạo đơn giản)
- ❌ Không có Territory Conquest đầy đủ
- ❌ Không có multiplayer

### 10.3 Win/Lose MVP

- **Thua**: Chết 5 lần liên tiếp không nhặt được đồ rơi → Game Over, restart
- **Thắng MVP**: Sống sót 10 ngày và tiêu diệt **Quái đêm Alpha** (mini-boss)

---

## 11. Cấu trúc Code đề xuất (Folder)

```
Assets/Scripts/
├── Core/              # GameManager, TimeManager, SaveSystem
├── Player/            # PlayerController, Inventory, Stats
├── Survival/          # Hunger, Health, Stamina systems
├── World/             # DayNightCycle, Weather, ZoneManager
├── Enemy/             # AI, SpawnSystem, Boss
├── Building/          # BuildingManager, Placement, Crafting
├── Items/             # ItemData, Equipment, Consumable
├── UI/                # HUD, InventoryUI, CraftingUI
├── Editor/            # TerrainManager, LevelDesign tools
└── Data/              # ScriptableObjects (SO)
```

---

## 12. Thiết kế Multiplayer (Tương lai)

- **Mô hình**: Listen Server (1 host + 2–3 client)
- **Đồng bộ hóa**:
  - Host authority: thời gian, spawn quái, trạng thái vùng
  - Client prediction: di chuyển, đánh đập
  - Sync: máu, đói, vị trí, trạng thái công trình
- **Shared progress**: Tất cả người chơi trong cùng session chia sẻ công nghệ, vùng kiểm soát
- **Death**: Hồi sinh tại buồng chung, đồng đội có thể hồi sinh lẫn nhau tại chỗ (lâu hơn)
