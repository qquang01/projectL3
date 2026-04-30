Bắt đầu thiết kế tool tạo và quản lý terrain, ý tưởng của tôi là tự độn tối đa có thể, không cần can thiệp nhiều trong unity. đánh giá lại các lựa chọn hoặc đưa ra phương án phù hợp
1. Tạo tool trên menu/Tools/terrain
2. Nhập input gồm kích thước map, kích thước chunk (placeholder kích thước map 512x512, chunk 64x64)
3. Bao gồm chức năng tạo object terrain với các children object là các chunk
4. Sau khi được tham vấn thì mỗi chunk là một mesh