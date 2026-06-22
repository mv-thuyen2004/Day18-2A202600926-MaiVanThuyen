# Reflection — Day 18

Trong các anti-pattern của Lakehouse, hệ thống giám sát LLM (LLM Observability) của chúng tôi dễ mắc phải **Vấn đề file nhỏ (Small File Problem)** nhất. 

**Lý do:**
Các lượt gọi API từ ứng dụng LLM được ghi nhận liên tục theo thời gian thực (streaming/real-time). Nếu ta liên tục append trực tiếp từng payload hoặc batch rất nhỏ vào tầng Bronze, hệ thống sẽ tạo ra hàng vạn file Parquet có kích thước chỉ vài KB trong thời gian ngắn. Điều này dẫn tới:
1. **Phình to transaction log (`_delta_log/`):** Tải đọc metadata tăng đột biến làm giảm hiệu năng của engine.
2. **Suy giảm hiệu năng truy vấn:** Đọc quá nhiều file nhỏ làm mất đi lợi thế đọc ghi tuần tự tốc độ cao của Parquet.

**Giải pháp khắc phục:**
Để giảm thiểu rủi ro này, chúng tôi cần triển khai định kỳ công việc `OPTIMIZE` kết hợp `Z-ORDER` (ví dụ: nhóm theo `model` hoặc `user_id` để tăng khả năng loại bỏ file khi lọc dữ liệu), đồng thời gom cụm dữ liệu thành các micro-batch trước khi ghi.
