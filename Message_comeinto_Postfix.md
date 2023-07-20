- Message come into Postfix bằng 4 cách:
  + Message có thể được chấp nhận vào Postfix locally (gửi từ một user trên cùng máy)
  + Message có thể được chấp nhận vào Postfix qua network
  + Message có thể được chấp nhận vào Postfix bằng phương thức khác là chuyển tiếp từ địa chỉ khác
  + Postfix tự tạo thông báo về khi nó gửi thông báo bị trả về hoặc bị trì hoãn
    => luôn có khả năng message bị reject trước khi nó vào postfix hoặc một vài message defer để gửi sau
### Local Email Submission
- Các thành phần postfix khác hoạt động cùng nhau bằng cách viết message và đọc message từ queue. Trình quản lý queue có trách nhiệm quản lý message trong queue và cảnh báo đúng component khi nó có công việc phải làm
![image](https://github.com/DinhHa1011/Postfix/assets/119484840/836157da-f7d6-453a-89a2-8332861a2654)
- Local message được gửi vào thư mục maildrop của postfix queue bằng lệnh postdrop, thường thông qua trình thông dịch senmail. Pickup daemon đọc message từ queue và đưa nó đến cleanup daemon. Một vài message đến mà không có tất cả thông tin cho một email message hợp lệ. Vì vậy ngoài kiểm tra sự chính xác trên message, cleanup daemon kết hợp với trivial-rewite daemon thường chèn các message headers bị thiếu, chuyển đổi địa chỉ thành dạng user@domain.tld mà các chương trình Postfix khác mong muốn, và có thể dịch các địa chỉ dựa trên bảng tra cứu canonical or virtual
- Cleanup daemon xử lý tất cả mail đến và thông báo cho trình quản lý queue sau khi nó đã đặt cleaned-up message đến incoming queue. Trình quản lý queue sau đó gọi tác nhân phân phối thích hợp để gửi thông báo tới next hop hoặc điểm đến cuối cùng
### Email from the Network
![image](https://github.com/DinhHa1011/Postfix/assets/119484840/896b07ca-55d6-4945-827a-5aacd985710c)

- Message được nhận qua network được chấp nhận bởi Postfix smtpd daemon. Daemon này thực hiện kiểm tra độ chính xác và có thể config để cho phép client relay mail trên hệ thống hoặc từ chối chúng làm như vậy. smtpd daemon chuyể message tới cleanup daemon, thực hiện kiểm tra riêng của mình sau đó gửi message vào queue. Trình quản lý queue sau đó gọi tác nhân phân phối thích hợp để gửi message tới next hop hoặc điểm cuối cùng của nó
### Postfix Email Notifications
- Khi một message của người dùng defer hoặc không thể gửi, Postfix sử dụng defer hoặc bounce daemons để tạo một error message mới. Error message được chuyển giao cho cleanup daemon. Nó thực hiện kiểm tra bình thường trước khi gửi thông báo lỗi vào queue, nơi nó được chọn bởi queue manager
### Email forwarding
- Thỉnh thoảng, sau khi xử lý một email message, Postfix xác định địa chỉ đích thực sự trỏ đến một địa chỉ khác trên hệ thống khác. Bắt tay đơn giản của message tới SMTP client cho việc gửi ngay tức khắc, nhưng để đảm bảo rằng mọi người nhận đều được xử lý và đăng nhập chính xác, Postfix gửi lại nó như một message mới nơi nó được xử lý giống như bất kỳ message gửi local nào khác

