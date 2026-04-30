# Đánh giá: SD 1.5 cho Terrain Tile + 2D vs 3D góc 45° cho game kiểu Don't Starve (Unity 2022.3 URP)

> Repo hiện tại (`qquang01/projectL3`) là Unity **2022.3.62f3** với **URP 14.0.12** đã được cấu hình sẵn (URP-Performant / Balanced / HighFidelity). Chưa có code/asset nào — đây là thời điểm tốt để chốt kiến trúc trước khi đổ asset vào.

---

## TL;DR (kết luận trước, lý giải sau)

| Câu hỏi | Khuyến nghị |
|---|---|
| SD 1.5 có dùng được để sinh tile terrain kiểu Don't Starve không? | **CÓ, rất khả thi** — nhưng phải dùng đúng kỹ thuật seamless tiling + 1 LoRA style chung. Không phải "prompt rồi xài". |
| 2D thuần hay 3D camera 45°? | **Đi 3D với camera perspective ~45° (đúng cách Don't Starve làm), nhân vật/object là billboard sprite.** Đây là mô hình **2.5D** hybrid. |
| Có nên dùng URP đang setup sẵn? | **Giữ nguyên URP 3D**. URP hỗ trợ tốt cả mesh terrain lẫn sprite/billboard, day-night lighting, post-processing — phù hợp với 2.5D. |

---

## Phần 1 — Đánh giá SD 1.5 cho terrain tile

### 1.1 SD 1.5 phù hợp như thế nào với style Don't Starve?

Style Don't Starve là **hand-painted / painterly** với:
- Nét outline đen dày, hơi bẩn (ink wash)
- Palette ám tối, saturation thấp, nhiều nâu/xám/xanh rêu
- Brush stroke lộ rõ, không photoreal
- Hình khối đơn giản, có chút Tim Burton

SD 1.5 mạnh ở mảng painterly/illustration vì:
- Hệ sinh thái **LoRA** trên Civitai cực lớn (gấp nhiều lần SDXL) cho style hand-painted, gothic, storybook, ink
- Nhẹ — chạy được trên GPU **6–8GB VRAM**, generate 512×512 trong vài giây
- ControlNet 1.1 ổn định, đặc biệt là `tile`, `canny`, `seg`, `depth` rất hữu ích cho tile asset

**Hạn chế của SD 1.5 so với SDXL/Flux:**
- Resolution gốc 512×512 (hi-res fix lên 768–1024 OK, nhưng yếu hơn SDXL ở 1024+)
- Composition phức tạp kém hơn → nhưng **với tile texture đơn giản thì không phải vấn đề**
- Hands/face yếu → **không quan trọng** với terrain, chỉ ảnh hưởng nếu sinh full character

→ **Kết luận**: SD 1.5 **đủ tốt và tối ưu cost/perf** cho việc sinh ground tile. Không cần SDXL.

### 1.2 Vấn đề lớn nhất: Seamless Tiling

SD vanilla **KHÔNG** sinh ra texture liền mạch (tileable). Bạn cần một trong các cách sau:

1. **Asymmetric Tiling extension** (A1111 / Forge / ComfyUI có node `Tiling`):
   - Patch các Conv2d layer sang `circular padding`
   - Bật cả X + Y → tile seamless 4 hướng
   - Đây là cách tốt nhất, output hầu như không cần fix mép

2. **Offset → Inpaint** (truyền thống):
   - Sinh tile thường → Photoshop/Krita filter `Offset` (256, 256) → thấy đường nối → SD inpaint mép → done
   - Tốn 1 bước thủ công nhưng chất lượng cao

3. **ComfyUI workflow** với `Seamless` custom node — dễ batch hàng loạt biome

### 1.3 Consistency giữa các biome (cỏ, cát, đá, tuyết, đầm lầy…)

Đây là điểm dễ vỡ nhất của SD: cùng style chứ không phải cùng "look". Phải kiểm soát:

- **1 LoRA style chung** train từ 20–50 reference (concept tự vẽ hoặc art public domain — **không train trực tiếp từ ảnh Don't Starve vì lý do IP của Klei**)
- **Prompt skeleton cố định**, chỉ thay danh từ biome:
  ```
  top-down view of {biome} terrain, hand-painted, painterly,
  dark ink outline, muted earth palette, brush texture,
  <lora:my_dontstarve_style:0.8>, seamless tileable
  ```
- **Cùng sampler / steps / CFG / seed family** (ví dụ chỉ thay seed +1 mỗi lần)
- **ControlNet tile** từ 1 reference noise pattern → giữ frequency của brush stroke giống nhau giữa biome

### 1.4 Variants & Auto-tiling

Đừng cố nhờ SD sinh từng **edge/corner/transition tile** — sẽ rất tốn công và khó consistent. Cách đúng:

- SD sinh **base texture** cho mỗi biome (4–8 variant để random rotate/shuffle)
- Việc transition giữa các biome xử lý ở:
  - **Unity 2D Tilemap + Rule Tile** (nếu đi 2D thuần) — auto chọn tile dựa neighbor
  - **Splatmap shader** (nếu đi 3D mesh terrain) — blend bằng alpha mask, không cần edge tile
  - **MicroSplat / Polybrush** (asset store) cho 3D — production-grade

### 1.5 Bản quyền (cảnh báo)

- **KHÔNG** train LoRA trên screenshot/ảnh asset của Don't Starve. Klei sở hữu IP, dùng để thương mại sẽ rủi ro pháp lý.
- Cách an toàn: dùng style **"hand-painted dark fantasy storybook"**, "Tim Burton illustration", "ink wash painterly" — cảm giác giống nhưng không sao chép trực tiếp.
- Hoặc thuê 1 artist vẽ 30 ảnh concept → train LoRA của riêng bạn → sạch IP.

---

## Phần 2 — 2D vs 3D camera 45°

### Thực tế kỹ thuật của Don't Starve
Don't Starve **không phải 2D**. Đây là **3D world, render dạng 2.5D**:
- Sàn (ground) là **mesh 3D phẳng** với texture tileable
- Cây, nhân vật, quái, item đều là **sprite phẳng billboard yaw-only** (chỉ xoay quanh trục Y, đứng thẳng với camera)
- Camera **perspective**, FOV thấp (~25–35°), tilt ~45°, **xoay được quanh nhân vật** (Q/E)
- Lighting động (đèn ngày/đêm, ánh lửa point light)

### 2.1 Option A — 2D thuần (Tilemap top-down)

| | Đánh giá |
|---|---|
| Pipeline | Đơn giản nhất. Sprite + 2D Tilemap + Rule Tile |
| Texture từ SD | Plug-and-play với Sprite |
| Camera xoay | **KHÔNG** — nếu top-down thì xoay sẽ lộ vì sprite không có depth |
| Layering | Phải tự code Y-sort, custom axis. Dễ bug khi nhiều object overlap |
| Lighting | URP 2D Light được, nhưng không có normal map đẹp như 3D |
| Cảm giác chiều sâu | Yếu, parallax không thay được perspective thật |
| Mở rộng (đào hố, dốc, multi-floor) | **Rất khó** |
| Performance | Cực tốt (thậm chí WebGL/mobile dễ) |
| Phù hợp | Game 2D top-down kiểu Stardew, Pokémon — **KHÔNG đúng feel Don't Starve** |

### 2.2 Option B — 3D true camera 45°

| | Đánh giá |
|---|---|
| Pipeline | Phức tạp nhất. Mesh, material, lighting, shader |
| Texture từ SD | Áp lên mesh material — tốt với splatmap |
| Camera xoay | **CÓ**, mượt, đúng feel Don't Starve |
| Z-sorting | Free từ depth buffer, không cần custom |
| Lighting động | Mạnh — point light cho lửa, directional cho mặt trời |
| Mở rộng | Dễ (height, dốc, hang động đa tầng) |
| Performance | Nặng hơn 2D nhưng URP + low-poly chạy tốt trên mobile |
| Vấn đề | Nếu làm character bằng 3D mesh thì phí công và mất feel painterly |

### 2.3 Option C — **2.5D Hybrid (KHUYẾN NGHỊ)**

Đúng công thức Don't Starve:
- **Ground**: mesh 3D (chunk-based mesh hoặc Unity Terrain) + tileable texture từ SD + splatmap blending
- **Object/Character/Tree/Mob**: **sprite billboard** (Quad + script yaw-only billboard) với art hand-painted (cũng sinh từ SD + rembg cho transparent)
- **Camera**: Perspective, FOV ~30°, tilt ~45°, follow + xoay quanh player
- **Lighting**: Directional (ngày/đêm), Point light (lửa trại), Post-process Volume cho color grading theo time-of-day
- **Shader**: URP Lit cho mesh, URP Unlit + custom alpha-cutout cho sprite billboard (giữ outline sắc nét)

| Tiêu chí | Điểm |
|---|---|
| Đúng feel Don't Starve | ★★★★★ |
| Tận dụng URP đã setup | ★★★★★ |
| Pipeline asset | ★★★★ (SD sinh được cả tile lẫn sprite) |
| Performance | ★★★★ |
| Mở rộng tương lai | ★★★★★ |
| Solo-dev friendly | ★★★★ |

### 2.4 Vì sao KHÔNG nên đi 2D thuần với project này?

1. Bạn đã setup **URP 3D** từ đầu (`com.unity.render-pipelines.universal` 14.0.12, 3 quality preset). Đi 2D thuần phải đổi sang URP 2D Renderer → mất công, và mất nhiều tính năng 3D.
2. Don't Starve có camera xoay — **2D top-down không làm được** mà giữ feel.
3. Khi gameplay phình ra (đào hố, hang, ban đêm + đèn lửa, fog of war), 3D xử lý dễ hơn nhiều.
4. SD 1.5 sinh tileable texture **đẹp hơn** khi áp lên mesh (có thể blend, có normal map fake) so với áp lên sprite tile cứng.

---

## Phần 3 — Pipeline đề xuất chi tiết

### 3.1 Asset pipeline

```
[Style Reference 3-5 ảnh]
         ↓
[Train LoRA (kohya_ss, ~30-50 imgs, ~1500 steps)]
         ↓
   ┌─────┴─────┐
   ↓           ↓
[Ground Tiles]   [Billboard Sprites]
SD 1.5            SD 1.5
+ Asymmetric      + Transparent BG
  Tiling          + rembg/SAM cleanup
+ ControlNet      + Hand-touch trong Krita
  tile/seg
   ↓               ↓
512x512 PNG       PNG with alpha
seamless          (cây, đá, mob…)
   ↓               ↓
Unity Material    Unity Sprite
+ Splatmap        + Quad + Billboard.cs
```

### 3.2 Stable Diffusion settings gợi ý

- **WebUI**: A1111 hoặc ComfyUI (ComfyUI nếu cần batch tự động)
- **Base model**: SD 1.5 — chọn checkpoint painterly (vd: `revAnimated`, `dreamshaper`, `meinamix` — sau đó train LoRA riêng đè lên)
- **Sampler**: `DPM++ 2M Karras`, **steps 28–35**, **CFG 6–7.5**
- **Resolution**: 512×512, hi-res fix 1.5x → 768 nếu cần chi tiết
- **Extension bắt buộc**: `Asymmetric Tiling` (hoặc tương đương ở Comfy)
- **ControlNet**: `tile_resample` để giữ frequency, `seg` để chia khu vực
- **Negative prompt**: `realistic, photo, 3d render, low contrast, blurry, jpeg artifacts, watermark, text`

### 3.3 Unity setup gợi ý (giữ nguyên URP hiện tại)

- **Terrain**: chunk-based mesh 32×32 quad, mỗi quad mang 4 splat layer (cỏ/đất/đá/cát) với mask alpha
  - Hoặc dùng **Unity Terrain** built-in (đã có `com.unity.modules.terrain`) — nhanh setup
  - Asset store: **MicroSplat** (free tier) cho splat blending production-grade
- **Billboard**: Quad + script đơn giản xoay theo yaw camera, freeze pitch/roll. Shader URP Unlit + Alpha Clip + bật `Receive Shadows`
- **Camera**: `Cinemachine` (cần thêm package `com.unity.cinemachine`) Free Look hoặc Virtual Camera, tilt 45°, FOV 30°
- **Lighting**:
  - 1 Directional Light (rotate theo TimeOfDay system tự code)
  - URP Lit cho ground, nhận shadow
  - Point Light cho lửa trại (Soft Shadow tắt cho perf)
- **Post-process**: Volume với Color Grading + Vignette + Bloom nhẹ

### 3.4 Roadmap đề xuất (solo dev, ước tính)

| Milestone | Thời gian | Output |
|---|---|---|
| 1. Style bible + LoRA | 1–2 tuần | LoRA painterly riêng, 5 ảnh chuẩn |
| 2. SD pipeline tile | 3–5 ngày | 4 biome × 6 variant tile seamless |
| 3. Unity terrain + camera | 1 tuần | Walkable map có 4 biome blend |
| 4. Billboard sprite system | 3–5 ngày | Cây/đá/mob sprite render đúng |
| 5. Day-night + lighting | 1 tuần | Vòng quay 24h, lửa trại |
| 6. Gameplay loop cơ bản (gather/craft) | 2–3 tuần | Có thể chơi thử |

---

## Phần 4 — Rủi ro & lưu ý

1. **IP Klei**: Tránh train trên ảnh Don't Starve. Style "hand-painted dark fantasy" là an toàn.
2. **Consistency drift**: SD vẫn drift dù cùng LoRA + prompt. Phải QA bằng mắt, không nhận hết output.
3. **Animation**: SD KHÔNG sinh được anim. Sprite character phải vẽ frame-by-frame hoặc dùng **Unity 2D Animation** (skeletal) — package này KHÔNG có trong manifest hiện tại, sẽ phải thêm.
4. **Sprite billboard hi-res**: nếu sprite 1024×1024 mà có 200 mob trên màn → VRAM căng. Atlas + LOD sprite cho xa.
5. **URP shader graph**: nếu cần custom shader (vd: dither alpha cho fog of war), nên thêm `com.unity.shadergraph` (chưa có trong manifest).

---

## Phần 5 — Khuyến nghị cuối cùng

> **Đi hướng 2.5D Hybrid: 3D mesh ground + sprite billboard. SD 1.5 + LoRA tự train là pipeline asset hợp lý nhất cho solo/small team. Giữ nguyên URP 3D đã setup.**

Câu hỏi mở để bạn chốt:
1. Bạn dự định **solo** hay **có team art**? (ảnh hưởng đến chuyện train LoRA hay outsource)
2. Target platform: **PC/Steam** hay **mobile**? (mobile thì giới hạn tile size & sprite count chặt hơn)
3. Đã có style reference của riêng bạn chưa, hay cần tôi đề xuất 1 moodboard?

Nếu bạn muốn, tôi có thể:
- Viết một **PoC Unity scene** với mesh terrain chunk + sprite billboard + camera 45° xoay được
- Hoặc viết **ComfyUI workflow JSON** sẵn để batch tile seamless
- Hoặc cả hai — chỉ cần bạn nói tiếp.
