# Reflection — Top 5 Lakehouse Anti-Patterns

Trong danh sách “Top 5 Lakehouse Anti-Patterns”, vấn đề rủi ro lớn nhất theo tôi là hiện tượng file nhỏ (small files). Việc ghi log của LLM, hành trình agent và sự kiện để giám sát thường cần ghi gần như thời gian thực; khi nhiều service ghi theo micro-batch, có retry hoặc nhiều phiên bản agent chạy cùng lúc, số lượng file sẽ tăng nhanh mặc dù mỗi file có dung lượng nhỏ.

Lab minh họa rõ điều này: ở NB2, sau khi chạy OPTIMIZE và Z‑ORDER số file giảm từ 200 còn 55; truy vấn nhanh hơn ~5.6 lần và chỉ cần đọc 1/55 file nhờ pruning. Ở NB6, compaction gom khoảng 200 file xuống còn 11 (khoảng 18x giảm). Do đó, nút thắt không chỉ là số byte lưu trữ mà còn là chi phí liệt kê file, mở file, xử lý metadata và lập kế hoạch truy vấn.

Vì vậy kế hoạch của tôi là: đặt mục tiêu kích thước file ngay từ bước ingestion; theo dõi số file và kích thước trung bình; thực hiện compaction định kỳ và Z‑ORDER theo các cột truy vấn phổ biến; đồng thời chạy orphan removal riêng, vì NB6 cho thấy VACUUM có thể không xóa được file khi writer lỗi trước commit.


