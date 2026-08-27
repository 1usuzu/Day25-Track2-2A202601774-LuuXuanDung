# Báo Cáo Tổng Kết (Write-up) - GPU FinOps Lab

## 1. Baseline vs. Optimized
- Nhờ áp dụng các đòn bẩy tối ưu (Cascade, Caching, Batching), chi phí cho quá trình chạy mô hình (Inference) đã giảm mạnh từ **$6.48/1M-token** xuống chỉ còn **$1.12/1M-token**.
- Tổng chi phí hàng tháng của toàn bộ hạ tầng đã tiết kiệm được **46%** (giảm từ $27,133 xuống còn $14,626/tháng).

## 2. Đòn bẩy hiệu quả nhất
- **Purchasing (Mua sắm thông minh)** đóng góp nhiều nhất vào việc giảm chi phí. Bằng cách mua **Reserved instance** cho các dịch vụ thời gian thực (chạy >55% thời gian) và thuê **Spot instance** cho các tác vụ Training (có thể gián đoạn, checkpoint), hệ thống đã tiết kiệm được ngân sách khổng lồ nhất trong các phương pháp.

## 3. Vấn đề "GPU-Util Lie"
- Hệ thống phát hiện tình trạng báo cáo khống ở `gpu-h100-4`: Thông số GPU-Util lên tới 98.2% nhưng hiệu năng thực sự (MFU) chỉ đạt 19.4%. Điều này cho thấy máy chủ đang "bận rộn giả tạo" (có thể do nghẽn cổ chai bộ nhớ hoặc I/O) chứ không hề thực hiện tính toán hữu ích.

## 4. Các phần mở rộng (Extensions) đã thực hiện
- **Đánh giá lợi ích của Cache (Extension 3):** Đã xây dựng hàm `cache_is_worth_it` tính điểm hòa vốn cho Prompt Caching. Thực tế cho thấy nếu một prompt ghi vào cache nhưng chỉ được đọc lại 1 lần, chi phí tiết kiệm được chưa đủ để bù đắp chi phí ghi đắt đỏ ban đầu. Phải tính toán kỹ số lần `avg_cache_reads` trước khi dùng Cache.
- **Carbon-aware Scheduling (Extension 5):** Đã thêm logic dời các job training/finetune sang Na Uy (`europe-north1` - sử dụng năng lượng sạch). Kết quả là dù vẫn giữ nguyên chi phí rẻ của Spot tier, chúng ta đã giảm được **1,479.5 kg CO2/tháng**.

## 5. Khuyến nghị hành động cho NimbusAI
1. Quy hoạch lại toàn bộ job training/finetune sang dùng tier `spot` kết hợp lưu checkpoint tự động.
2. Điều tra kỹ nguyên nhân nghẽn I/O hoặc hạ cấp (Right-sizing) đối với những GPU đang mắc bệnh "GPU-Util Lie".
3. Thiết lập chính sách Chargeback (trừ ngân sách) bắt đầu từ tháng sau do hệ thống gắn Tag hiện tại đã đạt 92% độ phủ (đủ điều kiện tin cậy).
