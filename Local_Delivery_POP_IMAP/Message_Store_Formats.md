### Message Store Formats
- Khi Postfix làm local delivery nó chuyển nội dung của message tới local message store. Loại phổ biến nhất của message store là định dạng mbox truyền thống và loại maildir mới hơn
- Cả hai đều sử dụng các tệp thông thường để lưu trữ tin nhắn, nhưng chúng được cấu trúc theo những cách khác nhau
- Trong Postfix, bạn chỉ định kiểu maildir bằng cách bao gồm dấu gạch chéo khi bạn config mail file hoặc director parameter nào
#### The Mbox Format
- Lịch sử, hệ thống Unix đã sử dụng một file duy nhất để lưu trữ mỗi email message của người dùng
- Loại định dạng message này thường được gọi là mbox
- Mỗi message trong file bắt đầu bằng dòng bắt đầu bằng từ From
- Nó thì quan trọng để một chuỗi bắt đầu trên ký tự đầu tiên của dòng, và có một khoảng trắng sau khi kết thúc từ.
- Dòng From thường được gọi là From_ với ký tự gạch dưới _ biểu tượng cho khoảng trắng sau từ
- Đừng nhầm lẫn dòng From_ được sử dụng để phân tách thư trong tệp mbox với dòng From : có trong tiêu đề thư email. Dòng cuối cùng của một tin nhắn luôn là một dòng trống.
- Một dòng From hoàn chỉnh có dạng như sau:
```
From jmbrown@exemple.com Sun Feb 3 16:54:01 2023
```
- Như mô tả, dòng trên bắt đầu với từ From theo sau bởi khoảng trắng. Theo sau khoảng trắng là một địa chỉ email thường là địa chỉ envelope của message. Sau địa chỉ envelope là date giao hàng ở định dạng date Unix phổ biến chiếm 24 ký tự. mbox format cho phép một chuối comment tùy chọn sau date, nhưng nó thường không được sử dụng
- Khi postfix chuyển một message tới một file mbox, đầu tiên nó tạo một dòng From_ sử dụng envelope người gửi và date hiện tại. Postfix sau đó copy nội dung của delivery message trên mbox file. Nếu Postfix bắt gặp bất kỳ dfng nào bắt đầu với From và theo sau bởi khoảng trắng, Postfix phải trích xuất chúng bằng cách thêm dấu > để không bị nhầm lẫn với phần đầu của message tiếp theo
- Khi một server POP/IMAP đọc message từ file mbox, nó scans file, thấy dòng From_, nơi mà đánh dấu bắt đầu của mỗi message. Nó có thể đọc phần tiếp theo của dòng From (hoặc kết thúc của file) để biết khi một message kết thúc. POP/IMAP server có thể bỏ trích dẫn bất kỳ dòng trích dẫn ">From" nào hoặc chúng vẫn ở dạng trích dẫn.
- Từ cả Postfix và POP/IMAP server truy cập mailbox file, họ phải sử dụng file locking. Postfix phải có được một khóa trên file khi nó chuyển một message, vì vậy nó có thể viết message tới file. Postfix cung cấp nhiều cơ chế khóa khác nhau, tùy thuộc vào nền tảng. Bạn có thể sử dụng dòng lệnh postconf -l để xem Postfix có thể sử dụng cơ chế nào trên hệ thống của bạn:
![image](https://github.com/DinhHa1011/Postfix/assets/119484840/ab8aff76-f0b8-41a0-9688-a263309670b2)
- Nếu bạn muốn nhiều thông tin hơn về các loại khóa được liệt kê bởi Postfix trên hệ thống của bạn, hãy kiểm tra các trang hướng dẫn của hệ thống để biết tên khóa cụ thể:
```
man flock
```
- Loại dotlock, có sẵn trên tất cả các hệ thống, có thể không được ghi lại trên hệ thống của bạn, vì nó không phải là một chức năng của hệ điều hành hoặc các thư viện hỗ trợ như flock và fcntl.
- Dotlock chỉ đơn giản là một file. The lock file được tạo thành từ tên của tệp sẽ bị khóa với phần mở rộng. lock được thêm vào nó. Nếu một tệp khóa như vậy tồn tại, thì Postfix biết rằng một quy trình khác đang sử dụng file. Nếu tệp không tồn tại, Postfix sẽ tạo nó để báo hiệu cho các quy trình khác rằng nó đang sử dụng mail file. Khi Postfix hoàn tất, nó sẽ xóa lock file, làm cho mail file khả dụng trở lại. Hạn chế của khóa dotlock là dễ bị khóa cũ và không hiệu quả lắm.
- Phần lớn, bạn không cần phải lo lắng về việc khóa và các loại khóa có sẵn, bởi vì Postfix thực hiện tốt công việc tìm ra tùy chọn tốt nhất.
#### The maildir format
- Định dạng maildir mailbox khác với mbox ở chỗ nó sử dụng cấu trúc thư mục để lưu trữ email message
- Nó được thiết kế để giải quyết một vài 
