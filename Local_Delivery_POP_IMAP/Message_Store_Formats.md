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
- Nó được thiết kế để giải quyết một vài vấn đề về độ tin cậy và khóa của định dạng mbox. Ví dụ, nếu một hệ thống gặp sự cố ngay lập tức khi một email đang được gửi đến file mbox, có thể message sẽ bị cắt bớt tại điểm gửi bị gián đoạn. Khi một hệ thống online trở lại, MTA sẽ cố gắng delivery message sau đó. Thông điệp được viết một phần tại botton của mbox file có thể gây ra vấn đề khi next message append to file
- Vấn đề khác có thể xảy ra nếu một POP/IMAP server cố gắng để truy cập mbox file tại cùng thời gian như SMTP. Nếu chương trình không sử dụng cùng cơ chế lock, mail file rất có thể sẽ bị hỏng. Có một số cơ chế lock file có thể (xem ở trên), không nhất thiết phải được sử dụng bởi tất cả mail program. Với định dạng maildir, không cần khóa vì mỗi message có file riêng. Các quy trình mail khác nhau không cần truy cập vào cùng một tệp tại cùng một thời điểm.
- Thư mục maildir-style có 3 subdirectories, must tất cả trên filesystem như: tmp, new, cur. Subdirectories thì luôn dưới một mail directory trong thư mục home của người dùng:
![image](https://github.com/DinhHa1011/Postfix/assets/119484840/4e25fbc3-cd08-44a2-a07c-90d2cabcffc9)
- File trong thư mục new là message đã được gửi nhưng chưa được đọc.
- Thời gian sửa đổi của file là ngày gửi tin nhắn. Tệp này thường chứa thông báo ở định dạng RFC 2822 và không cần dòng From_.
- Message đã được xem được chuyển vào thư mục cur. Thư mục tmp được sử dụng trong quá trình gửi tin nhắn để lưu trữ nội dung của file trước khi nó có thể được xác nhận là đã được ghi vào thư mục mới.
#### Mbox Versus Maildir
- Không có câu trả lời đơn giản nào giúp bạn quyết định loại định dạng mailbox nào là tốt nhất cho mình. Định dạng mbox có lợi thế là hầu như được hỗ trợ phổ biến, nhưng có vấn đề về khóa tệp đã thúc đẩy sự phát triển của định dạng maildir.
- Mặt khác, có những lo ngại về khả năng mở rộng quy mô của định dạng maildir để xử lý số lượng lớn thư trên một số hệ thống tệp. Có các đối số hiệu suất để hỗ trợ cả hai định dạng: định vị và truy cập hoặc xóa một thư cụ thể có thể nhanh hơn với maildir, nhưng việc gửi bằng cách chỉ cần thêm văn bản của thư vào cuối một tệp có thể nhanh hơn ở định dạng mbox.
- Sự lựa chọn của bạn rất có thể sẽ được thúc đẩy bởi việc bạn chọn máy chủ POP/IMAP. Nếu bạn chọn một máy chủ POP/IMAP yêu cầu định dạng maildir, thì sự lựa chọn sẽ dành cho bạn. Postfix dễ dàng hỗ trợ một trong hai định dạng, vì vậy bạn có thể yên tâm cho phép các cân nhắc khác đưa ra quyết định của mình. Nếu bạn nghĩ rằng nó sẽ có ý nghĩa trong môi trường của bạn, bạn nên chạy thử nghiệm cả hai định dạng, mô phỏng các tác vụ thư của riêng bạn càng sát càng tốt.
