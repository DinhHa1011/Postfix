- Postfix sử dụng nội dung của address classes để xác định điểm đến để chấp nhận gửi và cách thức gửi diễn ra.
- address classes chính là local, virtual alias, virtual mailbox và relay. Địa chỉ điểm đến không thuộc một trong các lớp được gửi qua mạng bởi SMTP client (giả sử nó được nhận bởi một client được ủy quyền)
- Tùy thuộc vào address class, queue manager gọi delivery agent thích hợp để xử lý message
### Local Delivery
- Local delivery agent xử lý mail cho user với một shell account trên hệ thống nơi mà Postfix chạy. Domain names cho local delivery được liệt kê trong tham số mydestination
- 
