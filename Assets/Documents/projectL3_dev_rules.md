# Gợi ý các "rule" dev game thường đặt ra (cho projectL3 — Unity 2022 + URP, kiểu Don't Starve)

Phân thành 7 nhóm. Bạn KHÔNG cần áp dụng tất cả ngay — pick những cái phù hợp với scope solo/small team rồi viết vào `AGENTS.md` hoặc `CONTRIBUTING.md` để giữ kỷ luật.

---

## 1. Rule về Game Design / Scope

> Đây là rule QUAN TRỌNG NHẤT cho solo dev. 80% project chết vì scope creep, không phải vì code.

- **MVP rule**: chỉ release khi có 1 vertical slice 5–10 phút **chơi được trọn vẹn** (gather → craft → đêm xuống → sống sót → lặp lại). Mọi feature ngoài đó là backlog.
- **One-pager design**: mỗi feature mới phải có 1 file `.md` ngắn (≤1 trang) trả lời 4 câu: *Player cảm thấy gì? Verb chính là gì? Loop ngắn nhất? Win/Loss condition?* Không có doc → không code.
- **Pillar rule**: định 3–5 design pillar (vd: "Survival luôn là áp lực", "Mỗi đêm đều chết người", "Crafting là quyết định, không phải grind"). Mọi feature mới phải qua bài test pillar — không pass thì cắt.
- **Cut list từ ngày 1**: viết sẵn list "feature SẼ KHÔNG có ở v1" (vd: multiplayer, mod support, controller hỗ trợ đầy đủ). Đặt ngay trong README để chống cám dỗ.
- **Magic number rule**: mọi giá trị gameplay (HP, damage, hunger rate, spawn rate) phải nằm trong **ScriptableObject** hoặc config asset, KHÔNG hardcode trong script. Designer chỉnh không cần dev.
- **No "we'll fix in polish"**: bug đã thấy mà bỏ qua "để polish sau" 99% sẽ không bao giờ được fix. Ghi vào issue ngay.

---

## 2. Rule về Code / Kiến trúc Unity

### 2.1 Tổ chức folder
- Cấu trúc `Assets/_Project/{Scripts, Art, Audio, Prefabs, ScriptableObjects, Scenes}` — gạch dưới `_` để folder của bạn nổi lên top, tách khỏi asset 3rd-party (`Assets/ThirdParty/`)
- Mỗi feature 1 namespace + 1 folder: `Assets/_Project/Scripts/Inventory/`, namespace `ProjectL3.Inventory`
- Không bao giờ để asset 3rd-party trộn vào folder của bạn — khi update package sẽ vỡ

### 2.2 Naming
- Class: `PascalCase`, file = class name
- Private field: `_camelCase` (gạch dưới đầu)
- Public field: hạn chế tối đa — dùng property `{ get; private set; }`
- Const: `UPPER_SNAKE_CASE`
- Prefab: `PF_*`, ScriptableObject: `SO_*`, Material: `M_*`, Texture: `T_*`, Animation Clip: `A_*`
- Scene: `S_Main`, `S_Forest_01`

### 2.3 Architecture rules
- **Không** truy cập trực tiếp Singleton lung tung. Dùng **dependency injection nhẹ** hoặc **Service Locator** có giới hạn. Hoặc Zenject/VContainer nếu team quen.
- **Event-driven cho cross-system**: dùng C# event hoặc ScriptableObject Event Channel (Unity Open Project pattern). Không gọi hàm chéo giữa Inventory ↔ Combat ↔ UI trực tiếp.
- **MonoBehaviour mỏng**: logic phức tạp → POCO class hoặc service. MonoBehaviour chỉ là adapter Unity.
- **Update() là kẻ thù**: gom logic vào 1 `TickManager` chia frame, hoặc dùng `UniTask`/coroutine. KHÔNG để 200 GameObject chạy `Update()` riêng.
- **Cấm Find/SendMessage**: `GameObject.Find`, `FindObjectOfType` chỉ dùng trong Editor script hoặc init một lần. SendMessage là banned.
- **No `static` mutable state** trừ khi có lý do rõ ràng. Static field là test killer.
- **Scene additive**: chia scene theo trách nhiệm (Boot, Persistent, World_X, UI) và load additive. KHÔNG có 1 mega-scene.

### 2.4 Asset reference rules
- KHÔNG hardcode đường dẫn `Resources.Load("...")`. Dùng **Addressables** hoặc inspector reference.
- Prefab reference qua inspector hoặc `AssetReference`. Không string-based lookup.

---

## 3. Rule về Asset Pipeline (đặc biệt khi dùng SD-generated)

### 3.1 Source vs Final
- Mọi asset có **2 trạng thái**: `_Source/` (PSD, .kra, SD output gốc 1024+) và `Assets/_Project/Art/` (PNG export sized cho game). Source KHÔNG đi vào Unity (gitignore).
- Naming version: `T_grass_base_v01.png`, `v02`… không bao giờ overwrite không có version.

### 3.2 Texture import preset
- Bắt buộc dùng **Preset Manager** + **Asset Postprocessor** để tự động set: max size, compression, filter mode, sprite mode khi import. Không ai tự click setting tay → không consistent.
- Sprite cho billboard: **Point filter, no compression, no mipmap** nếu pixel art; **Bilinear, BC7, mipmap** nếu painterly hi-res.
- Ground tile: **Repeat wrap mode bắt buộc**, mipmap on, BC7.

### 3.3 Style consistency rule (đặc biệt quan trọng với SD)
- **Style bible** — 1 file `STYLE.md` ghi: palette (hex codes), brush size, outline weight, lighting direction. Mọi prompt SD bắt nguồn từ đây.
- **Approved-only**: SD output phải qua review (bạn hoặc art lead) trước khi vào `Assets/`. Tránh "for now" art tích lũy thành nợ.
- **LoRA + seed lock**: mỗi batch sinh phải log `prompt + LoRA + seed + sampler + steps` vào file `_Source/sd_log.csv`. Reproducible.
- **No mixed styles**: không trộn output từ 2 LoRA khác nhau trong cùng 1 region. Drift là nguyên nhân #1 game indie nhìn "rẻ tiền".

### 3.4 Audio
- Loudness chuẩn: **-23 LUFS** cho music, **-18 LUFS** cho SFX (hoặc theo platform spec)
- Format: **OGG Vorbis** cho music dài, **WAV** cho SFX ngắn → Unity tự reimport theo platform

---

## 4. Rule về Git / Workflow

### 4.1 Branching
- `main` — release-ready, luôn build được
- `develop` — integration
- `feature/<short-desc>`, `fix/<issue>`, `art/<asset-batch>`
- KHÔNG commit trực tiếp lên `main` hoặc `develop` — chỉ qua PR

### 4.2 Commit rule
- **Conventional Commits**: `feat:`, `fix:`, `art:`, `chore:`, `refactor:`. Vd: `feat(inventory): add stack split via shift-click`
- 1 commit = 1 ý. KHÔNG commit "WIP" hay "stuff". Squash trước khi merge.
- Không commit Library/, Temp/, *.csproj, *.sln (đã có Unity gitignore chuẩn rồi — kiểm tra `.gitignore` của bạn)

### 4.3 LFS bắt buộc
- Bật **Git LFS** cho `*.psd, *.kra, *.png > 1MB, *.fbx, *.wav, *.ogg, *.mp4`
- KHÔNG commit binary lớn vào git thường — repo sẽ phình không cứu được
- Kiểm tra: `git lfs track "*.psd"` etc. → commit `.gitattributes` (bạn đã có file này, mở ra check)

### 4.4 Scene merge
- Scene và Prefab Unity là YAML — merge xung đột rất đau
- Bật **Smart Merge** (Unity → Edit → Project Settings → Editor → Asset Serialization = Force Text + cài UnityYAMLMerge)
- Quy ước: **1 scene = 1 người chỉnh tại 1 thời điểm**. Nếu có conflict → người sau phải redo, không cố merge tay.

### 4.5 PR rule
- PR ≤ 400 dòng diff (không tính generated/asset). Lớn hơn → chia
- Mọi PR cần: tựa đề rõ, link issue, screenshot/gif nếu UI/visual change, ghi rõ test đã chạy
- Không merge PR có CI fail (sau này dựng CI)

---

## 5. Rule về Performance

### 5.1 Budget cứng (đặt từ đầu, không đợi tối ưu cuối project)
- **Frame budget**: 16.6ms cho 60fps — chia: gameplay 6ms, render 6ms, UI 2ms, misc 2ms
- **Draw call**: ≤ 200 trên PC mid, ≤ 100 trên mobile
- **Triangle**: ≤ 200k onscreen mobile, ≤ 1M PC
- **Texture VRAM**: ≤ 512MB mobile, ≤ 2GB PC
- **GC Allocation per frame trong gameplay**: 0 byte (không tạo garbage). Dùng pooling cho mọi runtime spawn.

### 5.2 Rule cụ thể
- **Object Pooling bắt buộc** cho: bullet, vfx, particle, dropped item, damage number, mob nhỏ
- **Static batching** cho mọi GameObject không di chuyển (terrain props)
- **GPU Instancing** cho vegetation/billboard cùng material
- **Profiling weekly**: mở Unity Profiler 1 lần/tuần, ghi lại baseline. Không đợi feel lag mới mở.
- **Atlas sprite**: tất cả UI sprite + tất cả billboard cùng 1 biome → Sprite Atlas. KHÔNG để 100 sprite riêng lẻ.
- **No `Camera.main` trong Update**: cache reference. `Camera.main` là FindObjectWithTag bên dưới.

---

## 6. Rule về Testing / QA

- **Edit Mode test** cho mọi pure logic (inventory math, save/load, stat formula). Unity Test Framework đã có trong manifest của bạn.
- **Play Mode test** cho integration cơ bản (spawn player, pickup item, save/load round-trip)
- **Smoke test trước mỗi commit lên `develop`**: mở Boot scene, chơi 60s, không crash, không error log đỏ
- **Bug template**: title / steps to reproduce / expected / actual / screenshot/log / build version
- **No silent catch**: `catch (Exception)` mà không log là banned. Ít nhất `Debug.LogException`.

---

## 7. Rule về Build / Release

- **Build định kỳ**: tối thiểu 1 build/tuần kể cả khi không release. Build vỡ phát hiện sớm hơn vỡ ngay deadline.
- **Versioning**: SemVer `0.X.Y` cho pre-release, increment Y mỗi build. Hiện ngay trong main menu góc dưới phải.
- **Crash log**: tích hợp **Unity Cloud Diagnostics** hoặc **Sentry** từ build đầu tiên public. Đừng đợi user kêu mới biết crash.
- **Save migration rule**: mỗi khi đổi schema save → version bump + migration code. KHÔNG bao giờ break save cũ của tester/playtester.
- **Feature flag**: feature đang phát triển bật/tắt qua flag (ScriptableObject), build prod có flag = off. Không xé code mỗi lần test.

---

## 8. Bonus — Rule cho AI/SD-augmented dev (vì project bạn dùng SD)

- **Reproducibility**: mỗi asset SD-generated có metadata: prompt, neg prompt, model hash, LoRA hash, seed, sampler, steps. Embed trong PNG hoặc lưu CSV bên cạnh.
- **No final art at first try**: SD output luôn cần 5–15% hand-touch (Krita/Photoshop) để fix artifact, sharpen edge. KHÔNG đẩy thẳng SD output vào game.
- **License audit**: mỗi base model + LoRA dùng → log license trong `LICENSES_AI.md`. Một số model cấm thương mại — phát hiện sớm.
- **Style review weekly**: 1 lần/tuần xếp hết asset SD vừa sinh lên 1 board → check drift. Cắt cái nào lệch.
- **Don't-train list**: ghi rõ những gì KHÔNG được train LoRA (vd: "No Don't Starve official screenshots", "No Klei IP", "No copyrighted artist names in prompt").

---

## Cách áp dụng cho projectL3

Đề xuất thứ tự setup ngay tuần này:
1. Tạo `AGENTS.md` chứa: design pillars, cut list, naming convention rút gọn (tóm tắt mục 1, 2.1, 2.2)
2. Tạo `STYLE.md` cho art bible
3. Bật Git LFS cho asset binary (mục 4.3)
4. Cấu hình Unity: Asset Serialization = Force Text, smart merge (mục 4.4)
5. Setup Preset Manager cho texture import (mục 3.2)
6. Tạo folder structure `Assets/_Project/...` (mục 2.1)
7. Viết `CONTRIBUTING.md` rút gọn cho mục 4 + 6

Phần còn lại (testing, performance budget, addressables) bật khi có vertical slice chạy được.

---

> **Quy tắc của các quy tắc**: rule chỉ có giá trị nếu *enforce*. Không enforce → trở thành noise. Bắt đầu với 5–10 rule quan trọng nhất, viết vào AGENTS.md, đọc lại mỗi 2 tuần.
