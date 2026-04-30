# Danh mục công cụ AnkleBreaker Unity MCP

AnkleBreaker cung cấp hơn 280 công cụ chia thành các nhóm chính. Dưới đây là chi tiết các khả năng mà AI có thể thực hiện trực tiếp trong Unity Editor của bạn.

## 1. Nhóm Quản lý Scene & Hierarchy
Đây là nhóm công cụ cơ bản để xây dựng thế giới game.
- **Create/Delete GameObject:** Tạo mới hoặc xóa các đối tượng.
- **Transform Control:** Thay đổi vị trí (Position), xoay (Rotation) và kích thước (Scale).
- **Hierarchy Management:** Thay đổi cha-con (Parenting) của các Object.
- **Scene Navigation:** Tìm kiếm Object theo tên, tag hoặc layer.

## 2. Nhóm Quản lý Component
Cho phép AI can thiệp vào logic của từng Object.
- **Add/Remove Component:** Thêm các thành phần như Rigidbody, BoxCollider, MeshRenderer, hoặc các Script tùy chỉnh.
- **Get/Set Properties:** Đọc và ghi các giá trị trong Inspector (ví dụ: thay đổi độ kéo của trọng lực, đổi màu vật liệu).
- **Method Invocation:** Kích hoạt các hàm (Function) có sẵn trong các Component.

## 3. Nhóm Quản lý Assets & Files
AI có thể làm việc với Project Window.
- **Asset Creation:** Tạo Material, Prefab, Animator Controller, và C# Scripts.
- **Folder Management:** Tạo và sắp xếp cấu trúc thư mục Assets.
- **Resource Import:** Kiểm tra trạng thái import của các asset.

## 4. Nhóm Đồ họa & Shader (Nâng cao)
Đây là điểm mạnh nhất của AnkleBreaker so với các MCP khác.
- **Shader Graph:** AI có thể tạo node, kết nối các node trong Shader Graph để tạo hiệu ứng hình ảnh.
- **Amplify Shader Editor:** Hỗ trợ đầy đủ cho người dùng ASE.
- **Material Editing:** Thay đổi Texture, thông số thuộc tính của Shader mà không cần mở Inspector.

## 5. Nhóm Địa hình & Môi trường (Terrain)
- **Terrain Sculpting:** Nâng, hạ, làm phẳng địa hình.
- **Texture Painting:** Vẽ các lớp cỏ, đá, cát lên địa hình dựa trên yêu cầu.
- **Tree & Detail Planting:** Rải cây cối và cỏ tự động theo mật độ.

## 6. Nhóm Hệ thống Vật lý & AI
- **NavMesh:** Tính toán và nướng (Bake) dữ liệu đường đi cho AI.
- **Collider Setup:** Tự động bao quanh vật thể bằng các Collider phù hợp.

## 7. Nhóm Debug & Profiling
- **Console Monitoring:** Đọc các lỗi (Error) và cảnh báo (Warning) từ Unity Console để tìm cách sửa.
- **Performance Profiling:** Kiểm tra xem Object nào đang ngốn nhiều tài nguyên nhất.

## 8. Nhóm Build Automation
- **Build Settings:** Cấu hình danh sách các Scene được Build.
- **Platform Switching:** Chuyển đổi giữa Android, iOS, Windows trực tiếp từ AI.
- **Trigger Build:** Thực hiện lệnh Build ra file thực thi (.exe, .apk).

---
**Lưu ý:** Để AI sử dụng được các công cụ này, bạn cần đảm bảo cửa sổ **Bridge** trong Unity (`Window > AnkleBreaker > Open Bridge`) luôn ở trạng thái **Connected**.

*Tài liệu được biên soạn bởi Gemini CLI.*