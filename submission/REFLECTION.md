# Reflection — Top 5 Lakehouse Anti-Patterns

Trong “Top 5 Lakehouse Anti-Patterns”, tôi cảm thấy có nguy cơ cao nhất với **small files**. Log LLM, agent trajectory và sự kiện observability cần được ghi gần thời gian thực. Nhiều service cùng ghi micro-batch, retry và chạy các phiên bản agent sẽ khiến số file tăng nhanh dù dung lượng chưa lớn.

Lab cho tôi thấy rõ tác động. Ở NB2, sau `OPTIMIZE` và Z-ORDER, số file giảm từ 200 xuống 55; truy vấn nhanh hơn 5,6 lần và chỉ 1/55 file cần đọc, tương ứng pruning 55 lần. Ở NB6, compaction giảm 200 file xuống 11 file, tức khoảng 18 lần. Nút thắt vì thế không chỉ nằm ở số byte mà còn ở chi phí liệt kê, mở file, xử lý metadata và lập kế hoạch truy vấn.

Vì vậy, tôi sẽ đặt kích thước file mục tiêu từ ingestion, giám sát số file và kích thước trung bình, chạy compaction định kỳ, rồi Z-ORDER theo cột truy vấn phổ biến. Tôi cũng sẽ chạy orphan removal riêng, vì NB6 chứng minh `VACUUM` không xóa được file do writer lỗi trước khi commit.
