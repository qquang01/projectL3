# Hướng dẫn sử dụng AnkleBreaker Unity MCP

AnkleBreaker là một công cụ mạnh mẽ giúp kết nối các AI Assistant (Claude, Cursor, Gemini) trực tiếp với Unity Editor thông qua giao thức MCP (Model Context Protocol).

## Thông tin hệ thống hiện tại
- **Vùng lưu trữ:** Ổ F (để bảo vệ ổ C không bị đầy).
- **Cấu hình NPM Prefix:** `F:\npm_global`
- **Cấu hình NPM Cache:** `F:\npm_cache`
- **Cổng kết nối (Bridge Port):** 7890

## Cách sử dụng
1. **Mở Unity:** Đảm bảo dự án `ProjectL3` đang mở.
2. **Kích hoạt Bridge:** Trong Unity, vào menu `Window > AnkleBreaker > Open Bridge`.
3. **Kết nối AI:** Ra lệnh cho AI thực hiện các tác vụ trong Unity.

## Các lệnh phổ biến AI có thể thực hiện
- "Tạo một GameObject Cube tại vị trí (0,0,0)"
- "Liệt kê tất cả các Script đang có trong dự án"
- "Tìm tất cả các Object bị thiếu (Missing) Component"
- "Thay đổi cường độ của Directional Light thành 1.5"
- "Nướng (Bake) lại NavMesh cho Scene hiện tại"

---
*Tài liệu này được tạo tự động bởi Gemini CLI.*