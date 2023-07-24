- Trình quản lý queue qmgr theo nhiều cách là trái tim của hệ thống Postfix của bạn
- Tất cả message, cả outbound và inbound phải thông qua queue
- Bạn nên hiểu queue và Postfix sử dụng nó như thế nào trong trường hợp bạn phải xử lý sự cố một vấn đề
- Queue manager chứa 5 five queue khác nhau: incoming, active, deffered, hold, corrupt. Postfix sử dụng một thư mục riêng biệt cho mỗi queue dưới đường dẫn cụ thể trong `queue_directory` parameter. Theo mặc định đường dẫn là `/var/spool/postfix`, cung cấp cho bạn một cấu trúc thư mục:
```
/var/spool/postfix/active
/var/spool/postfix/bounce
/var/spool/postfix/corrupt
/var/spool/postfix/deferred
/var/spool/postfix/hold
```
- qmgr daemon chạy trong background xử lý hầu hết các tác vụ queue management. Các lệnh postsuper và postqueue được sử dụng bởi admin để sử dụng cho các tác vụ queue management. Chap này xem xét cách qmgr và command-line tools làm việc như thế nào, cũng như Postfix parameter ảnh hưởng đến queue
### How qmgr Works
![image](https://github.com/DinhHa1011/Postfix/assets/119484840/afb8860b-1750-4772-9b7d-5ed7ba8e1b2d)
- Ảnh trên minh họa cách chuyển message qua queue. Incoming queue là nơi đầu tiên của message khi enter Postfix. Queue manager cung câp sự bảo vệ cho hệ thống file queue qua `queue_minfree` parameter. Mặc định giá trị là 0. Bạn có thể đảm bảo đĩa lưu trữ queue của bạn không bị hết dung lượng bằng cách setting một giới hạn
- Từ incoming queue, queue manager chuyển message tới active queue và gọi delivery agent thích hợp để xử lý chúng. Phần lớn, nếu không có vấn đề gì với việc gửi, thì việc di chuyển qua queue sẽ nhanh đến mức bạn sẽ không thấy message trong queue. Nếu Postfix đang cố gửi đến một máy chủ SMTP chậm hoặc không khả dụng, bạn có thể thấy các message trong queue active. Postfix đợi 30 giây để quyết định xem có thể truy cập hệ thống từ xa hay không.
- Một message không thể gửi được sẽ được đặt trong deferred queue. Messsage chỉ bị hoãn lại khi chúng gặp sự cố tạm thời trong quá trình gửi, chẳng hạn như sự cố DNS tạm thời hoặc khi máy chủ thư đích báo cáo sự cố tạm thời. Các thư bị từ chối hoặc gặp phải lỗi vĩnh viễn sẽ ngay lập tức được trả lại cho người gửi trong một báo cáo lỗi và không nằm trong queue.
#### Deferred Mail
- Message trong queue hoãn lại ở đó cho đến khi chúng được gửi thành công hoặc hết hạn và được trả lại cho người gửi. Tham số `bounce_size_limit` xác định số lượng thư không thể gửi được bị trả lại cho người gửi trong báo cáo lỗi. Mặc định là 50.000 byte.
- Khi một message gửi không thành công, Postfix sẽ đánh dấu thư đó bằng timestamp để cho biết khi nào lần gửi tiếp theo sẽ diễn ra. Postfix giữ một danh sách ngắn hạn các hệ thống ngừng hoạt động để tránh các nỗ lực phân phối không cần thiết. Nếu có các message deferred được lên lịch cho một nỗ lực gửi lại và có sẵn dung lượng trong active queue, thì queue manager sẽ luân phiên giữa việc nhận message từ deferred queue và incoming queue, để các thư mới không bị buộc phải chờ sau một lượng lớn message deferred.
#### Queue Scheduling
- Postfix định kỳ quét queue để xem liệu có message bị deferred nào có timestamp cho biết chúng đã sẵn sàng cho lần gửi khác hay không. Những nỗ lực chuyển phát không thành công sau đó gây ra sự chậm trễ theo lịch trình tăng gấp đôi, vì vậy Postfix phải đợi lâu hơn mỗi lần trước khi cố gắng gửi thư. Bạn có thể định cấu hình độ trễ tối đa bằng tham số `maximal_queue_lifetime`. Khi hết thời gian, Postfix ngừng cố gắng gửi thư và trả lại thư cho người gửi. Theo mặc định, khoảng thời gian là năm ngày (5d). Bạn có thể đặt nó ở bất kỳ khoảng thời gian nào hoặc bằng 0 để trả lại thư không gửi được ngay lập tức.
- Quá trình quét queue diễn ra tại một khoảng thời gian được chỉ định bởi tham số `queue_run_delay`. Theo mặc định, tham số được đặt thành 1.000 giây (1000 giây). Với cài đặt này, cứ sau 1.000 giây, Postfix sẽ kiểm tra hàng đợi bị trì hoãn để xem liệu có bất kỳ thư nào đến hạn cho một lần gửi khác hay không.
- Các tham số `minimum_backoff_time` và `maximal_backoff_time` đặt giới hạn thời gian tối thiểu và tối đa về tần suất Postfix cố gắng gửi lại message bị deferred. Mỗi khi một message bị hoãn lại, queue manager sẽ tăng lượng thời gian chờ để gửi lại tin nhắn đó. Mức tăng thời gian được tính toán không bao giờ được phép vượt quá `maximal_backoff_time` (4.000 giây mặc định) và không bao giờ nhỏ hơn `minimum_backoff_time` (1.000 giây mặc định). Nếu bạn thấy rằng bạn có một lượng lớn thư bị hoãn tồn đọng, bạn có thể muốn tăng `maximal_backoff_time` để Postfix không sử dụng tài nguyên hệ thống trong việc cố gắng gửi thư đến các máy chủ không khả dụng.
