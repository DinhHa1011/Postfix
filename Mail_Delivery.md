- Postfix sử dụng nội dung của address classes để xác định điểm đến để chấp nhận gửi và cách thức gửi diễn ra.
- address classes chính là local, virtual alias, virtual mailbox và relay. Địa chỉ điểm đến không thuộc một trong các lớp được gửi qua mạng bởi SMTP client (giả sử nó được nhận bởi một client được ủy quyền)
- Tùy thuộc vào address class, queue manager gọi delivery agent thích hợp để xử lý message
### Local Delivery
- Local delivery agent xử lý mail cho user với một shell account trên hệ thống nơi mà Postfix chạy. Domain names cho local delivery được liệt kê trong tham số `mydestination`
- Message gửi tới một user bất kì tại domain mydestination được gửi đến shell account cho user. Trong trường hợp đơn giản, local delivery agent gửi một email vào local message store. Nó cũng kiểm tra aliases và file .forward của user để xem liệu local có nên gửi tới nơi khác không.
- Khi một message được forward tới nơi khác, nó sẽ được gửi tới Postfix để chuyển tới địa chỉ mới. Nếu có vấn đề tạm thời chuyển message, delivery agent thông báo cho trình quản lý queue để đánh dấu thư cho lần gửi tiếp trong tương lai và lưu trữ ó trong defered queue. Các sự cố thường trực khiến trình quản lý queue trả lại message cho người gửi ban đầu
### Virtual Alias Message
- Địa chỉ virtual alias thì forward tất cả tới địa chỉ khác. Tên domain cho virtual aliasing được liệt kê trong `virtual_alias_domains` parameter
- Mỗi domain có một nhóm người dùng riêng không nhất thiết phải là duy nhất trên các domain
- User và địa chỉ thực của họ được liệt kê trong bảng tra cứu được chỉ định trong `virtual_alias_maps` parameter.
- Message nhận được cho địa chỉ virtual alias được gửi lại để gửi đến địa chỉ real
### Virtual Mailbox Message
- virtual delivery agent xử lý mail cho virtual mailbox address. Có mailbox thì không liên quan với shell account cụ thể trên hệ thống. Domain name cho virtual mailbox được list trong `virtual_mailbox_domain` parameter.
- Mỗi domain có nhóm người dùng riêng không nhất thiết là duy nhất trên các domain
- File user và mailbox của họ được list trong bảng tra cứu được chỉ định trong `virtual_mailbox_maps` parameter
### Relay message
- smtp delivery agent xử lý mail cho relay domain. Địa chỉ email trong relay domain được tổ chức trong hệ thống khác, nhưng Postfix chấp nhận message cho domain và relay chúng tới đúng hệ thống
- Relay config thì phổ bến khi Postfix chấp nhận mail qua Internet và vượt qua nó tới hệ thống trên một mạng internal
- Domain name cho relay domain được liệt kê trong `relay_domains` parameter
### Other Messsages
- Message không phù hợp với các lớp địa chỉ thường được dành cho các domain khác được lưu trữ ở nơi khác trên mạng.
- Postfix chỉ chấp nhận các message như vậy từ client đươc ủy quyền, như hệ thống chạy trên local network.
- Khi một message phải được gửi qua mạng, người quản lý gọi smtp delivery agent. smtp agent xác định máy chủ hoặc máy chủ có thể nhận message và tạo kết nối tới từng máy chủ lần lượt cho đến khi có người chấp nhận nó
- Nếu có sự cố tạm thời khi gửi message, smtp delivery agent sẽ thông báo cho người quản lý message để đánh dấu message cho một nỗ lực gửi trong tương lai và lưu trữ nó trong deferred queue. Sự cố vĩnh viễn khiến trình quản lý queue trả lại message gửi lại người gửi ban đầu.
- Khi một hệ thống đích không khả dụng online trở lại, Postfix cẩn thận để không làm quá tải nó với tất cả các message đang chờ xử lý. Cho dù gửi thư bị defer trước đó hay message mới, Postfix, lúc đầu chỉ tạo một một số lượng kết nối giới hạn tới một hệ thống nhận
- Sau khi Postfix xác định gửi thành công tới một trang web cụ thể, nó tăng từ từ (lên đến giới hạn có thể định cấu hình) các kết nối đồng thời với nó
- Nếu Postfix phát hiện rắc rối nào từ nơi tiếp nhận, nó bắt đầu gửi lại ngay lập tức
### Other Delivery Agents
- CÓ Postfix delivery agent khác có thể config để xử lý message cho class hoặc destination cụ thể
- Delivery agent khác phải config trong file master.cf
- Chúng được gọi thông qua `class_transport` hoặc thông qua một mục trong một bảng transport, được liệt kê trong `transport_maps` parameter.
- Hai tác nhân phân phối phổ biến là lmtp và pipe agent
#### Delivery via LMTP
#### Pipe delivery
- Postfix cung cấp tùy chọn gửi message tới program khác thông qua pipe daemon. Pipe daemon gửi message tới các command bên ngoài
- Một cách sử dụng phổ biến cho pipe daemon là có email được gửi tới một filter nội dung bên ngoài hoặc phương tiện liên lạc khác, pipe daemon thông báo đến người quản lý queue để đánh dấu message cho một future delivery attempt và lưu trữ nó trong defered queue
