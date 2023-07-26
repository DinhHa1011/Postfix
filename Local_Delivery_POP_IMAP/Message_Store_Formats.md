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
- Như mô tả, dòng trên bắt đầu với từ From theo sau bởi khoảng trắng. Theo sau khoảng trắng là một địa chỉ email thường là địa chỉ envelope của message. Theo sau 
