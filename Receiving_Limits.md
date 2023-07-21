- smtpd daemon có thể thực thi một số giới hạn với incoming mail
- Các giới hạn có thể định cấu hình thông qua một số tham số trong file main.cf
- Bạn có thể giới hạn kích thước của message, số lượng người nhận cho một lần gửi và độ dài của các dòng trong một tin nhắn
- Bạn cũng có thể giới hạn số lỗi cho phép từ một client trước khi ngắt giao tiếp
- Để giới hạn số lượng người nhận cho sigle message, sử dụng `smtpd_recipient_limit` parameter. Mặc định là 1000 người nhận, và nó nên đủ cho hoạt động bình thường
- `message_size_limit` parameter giới hạn size của message nào trên hệ thống của bạn sẽ được chấp nhận. Mặc định là 10MB. Nếu bạn có dung lượng đĩa hoặc bộ nhớ hạn chế, bạn có thể muốn hạ thấp giá trị. Mặt khác, nếu người dùng của bạn thường nhận các têp đính kèm lơn, bạn có thể tăng nó
- Các lỗi ngày càng tăng và từ một client có thể là một sự cố hoặc một cuộc tấn công. Postfix giữ bộ đếm lỗi và xử lý các máy khách có vấn đề tiềm ẩn bằng cách đưa ra độ trễ với mỗi lỗi. Delay có thể giúp bảo vệ hệ thống của bạn khỏi client ác tính hoặc bị cấu hình sai.
- Khi số lượng lỗi tăng lên thì độ dài của mỗi độ trễ cũng tăng theo. Độ dài của độ trễ ban đầu được chỉ định bởi `smtpd_error_sleep_time` với giá trị mặc định là một giây. Sau khi số lượng lỗi vượt quá giá trị được đặt cho smtpd_soft_error_limit , Postfix sẽ tăng độ trễ thêm một giây cho mỗi lỗi, sao cho với mỗi lỗi, có độ trễ lâu hơn một chút. Cuối cùng, khi số lỗi đạt đến giá trị được đặt trong `smtpd_hard_error_limit`, Postfix từ bỏ ứng dụng khách và ngắt kết nối.
- Nếu một chương trình độc hại kết nối với máy chủ thư của bạn và gửi các lệnh rác, cố gắng đánh sập máy chủ của bạn, thì các lệnh không có thật sẽ xuất hiện với Postfix dưới dạng lỗi từ một máy khách hoạt động sai. Giả sử các giá trị sau cho các tham số độ trễ:
    + smtpd_error_sleep_time = 1s
    + smtpd_soft_error_limit = 10
    + smtpd_hard_error_limit = 20
- Với cài đặt trên, Postfix đợi ít nhất 1s (smtpd_error_sleep_time) sau mỗi lỗi trước phản hồi từ client
- Sau 10 (smtpd_soft_error_limit) thăm dò như vậy, Postfix bắt đầu tăng độ lớn mỗi delay
- Sau 11 lỗi, postfix đợi 11s, sau 12 lỗi, postfix đợi 12s
- Khi số lượng đạt 20 (smtpd_hard_error_limit), Postfix ngắt kết nối, nó chỉ đơn giản là được xử lý như cũ mỗi khi nó bắt đầu tạo vấn đề
