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
#### Message Delivery
- Queue manager sắp xếp việc gửi tin nhắn bằng cách gọi tác nhân chuyển phát thích hợp. Postfix cẩn thận để không làm quá tải hệ thống đích và cung cấp một số tham số để kiểm soát tài nguyên dành cho thư gửi đi. Đối với hầu hết các trường hợp, cài đặt mặc định là chính xác, nhưng nếu bạn đang gặp vấn đề về tài nguyên hoặc bạn đang cố gắng tối ưu hóa việc phân phối, bạn có thể thử nghiệm với cấu hình queue manager.
- Outgoing message có thể được gửi qua bất kỳ phương tiện truyền tải nào có sẵn trong tệp master.cf. Mỗi lần vận chuyển có thể có giới hạn về tổng số quy trình của nó, được chỉ định trong cột `maxproc`. Nếu một giá trị không được chỉ định ở đó, Postfix sẽ sử dụng `default_ process_limit` cho giới hạn của nó.
- Tham số `initial_destination_concurrency` giới hạn số lượng message được gửi ban đầu (mặc định là 5). Bạn có thể tăng giá trị, nhưng giá trị này không thể cao hơn giá trị `maxproc` hoặc `default_ process_limit` cho phương tiện truyền tải được sử dụng. Sau lần gửi thư đầu tiên, nếu có nhiều thư hơn trong queue cho một đích cụ thể, Postfix sẽ tăng số lần thử gửi đồng thời, miễn là nó không phát hiện bất kỳ sự cố nào từ hệ thống đích ở tải hiện tại. Postfix tiếp tục tăng số lượng phân phối đồng thời lên đến con số được chỉ định trong tham số `default_destination_concurrency_limit`, theo mặc định là 20. Nói chung, bạn không muốn tăng giới hạn đồng thời hoặc bạn có nguy cơ áp đảo hệ thống nhận.
- Bạn có thể ghi đè giá trị `default_destination_concurrency_limit` cho bất kỳ phương tiện vận chuyển nào bằng cách đặt tham số có dạng `transport_destination_concurrency_limit` . Ví dụ: bạn có thể giới hạn kết nối đồng thời với các hệ thống bên ngoài bằng tham số `smtp_destination_concurrency_limit` hoặc giới hạn phân phối cục bộ bằng `local_destination_concurrency_limit`.
- Ngoài ra các tham số `transport_destination_recipient_limit` kiểm soát số lượng người nhận Postfix chỉ định cho một bản sao của một email message. Nếu một transport-specific parameter không config, nó lấy giá trị mặc định từ `default_destination_recipient_limit`. nếu số lượng người nhận cho một message vượt quá giới hạn, Postfix sẽ chia nhỏ danh sách người nhận thành các nhóm địa chỉ nhỏ hơn và gửi các bản sao riêng biệt của message tới từng nhóm địa chỉ
#### Corrup Message 
- Corrupt queue đơn giản được sử dụng để lưu trữ các message bị hỏng hoặc không thể đọc được. Nếu một message cũng hỏng để làm mọi thứ với nó, Postfix để nó ở đây. Nếu bạn muốn điều tra một vấn đề, vấn đề message là biến trong queue này ở nơi bạn có thể xem thủ công nếu cần. Corrupt message rất hiếm. Nếu bạn có chúng, chúng có thể là triệu chứng của một hệ điều hành tiềm ẩn hoặc vấn đề phần cứng
#### Error Notifications
- Postfix có thể report chắc chắn lỗi bằng gửi lỗi message tới một admin. Postfix phân loại lỗi để thông báo, như được show trong bảng dưới. `notify_classes` parameter trong `main.cf` chứa danh sách các lớp lỗi rằng nên tạo thông báo lỗi. Theo mặc định parameter gồm `resource` và `software` error
- Mỗi lớp của lỗi có thể config để gửi thông báo tới một địa chỉ email cụ thể, sử dụng parameter của form `class_notice_recipient`. Theo mặc định chúng tất cả đi tới postmaster. Bảng dưới đây cung cấp danh sách các loại lỗi có thể xảy ra, cùng với các tham số cho biết ai sẽ nhận thông báo lỗi

![image](https://github.com/DinhHa1011/Postfix/assets/119484840/3ac339d2-4512-41ff-aa50-537e785d9ffe)
- Nếu bạn muốn nhận thông báo tất cả vấn đề, set parameter như dưới đây:
```
notify_classes = bounce, 2bounce, delay, policy, protocol, resource, software
```
### Queue Tools
- Postfix cung cấp command-line tools để hiển thị và quản lý message trong queue của bạn. Dòng lệnh primary là postsuper và postqueue. Bạn có thể thực hiện các tác vụ dưới đây trên message trong queue:
  - Listing messages
  - Deleting messages
  - Holding messages
  - Requeuing messages
  - Displaying messages
  - Flushing messages
- Mỗi nhiệm vụ và các lệnh để hoàn thành chúng được giải thích trong các phần tiếp theo
#### Listing the Queue
- queue display chứa một mục nhập cho mỗi message để show message ID, size, arrival time, sender, và recipient addresses. Deferred message cũng gồm lý do chúng không thể gửi được. Message trong active queue được đánh dấu bằng dấu hoa thị sau Queue ID. Message trong hold queue được đánh dấu bằng một dấu chấm than. Deferred message không được đánh dấu
- Bạn có thể list tất cả message trong queue với dòng lệnh `postqueue -p`. Postfix cũng cung cấp dòng lệnh mailq cho khả năng tương thích với Sendmail. Postfix thay thế cho mailq tạo ra cùng một output như `postqueue -p`. Một typical queue entry trông như sau:
![image](https://github.com/DinhHa1011/Postfix/assets/119484840/f5dcefa5-555d-4fd6-81e1-3ccc48ce6501)
#### Deleting Messages
- Dòng lệnh postsuper cho phép bạn remove message từ queue
- Để xóa message trong mục nhập mẫu được hiển thị ở trên, hãy thực thi postsuper với tùy chọn -d:
  ``` 
  # postsuper -d DBA3F1A9
  postsuper: DBA3F1A9: removed
  postsuper: Deleted: 1 message
  ```
- Nếu bạn có nhiều message cần xóa, bạn có thể clear toàn bộ queue với ALL argument:
  ```
  # postsuper -d ALL
  postsuper: Deleted: 23 messages
  ```
- Đối số ALL phải được viết hoa. Hãy thật cẩn thận khi sử dụng lệnh, vì nó sẽ xóa tất cả các tin nhắn đã xếp hàng đợi mà không hỏi bất kỳ câu hỏi nào.
- Thay vì xóa tất cả queue message hoặc chỉ xóa một, bạn thường muốn xóa message với một địa chỉ email cụ thể. Ví dụ dưới đây là một Perl script để cung cấp một cách thuận tiện để chỉ định địa chỉ email để xóa các thư cụ thể khỏi queue
```
#!/usr/bin/perl -w
#
# pfdel - deletes message containing specified address from
# Postfix queue. Matches either sender or recipient address.
#
# Usage: pfdel <email_address>
#
use strict;
# Change these paths if necessary.
my $LISTQ = "/usr/sbin/postqueue -p";
my $POSTSUPER = "/usr/sbin/postsuper";
my $email_addr = "";
my $qid = "";
my $euid = $>;
if ( @ARGV != 1 ) {
die "Usage: pfdel <email_address>\n";
} else {
$email_addr = $ARGV[0];
}
if ( $euid != 0 ) {
die "You must be root to delete queue files.\n";
}
open(QUEUE, "$LISTQ |") ||
die "Can't get pipe to $LISTQ: $!\n";
my $entry = <QUEUE>;
$/ = "";
# skip single header line
# Rest of queue entries print on
# multiple lines.
while ( $entry = <QUEUE> ) {
if ( $entry =~ / $email_addr$/m ) {
($qid) = split(/\s+/, $entry, 2);
$qid =~ s/[\*\!]//;
next unless ($qid);
}
}
close(QUEUE);
#
# Execute postsuper -d with the queue id.
# postsuper provides feedback when it deletes
# messages. Let its output go through.
#
if ( system($POSTSUPER, "-d", $qid) != 0 ) {
# If postsuper has a problem, bail.
die "Error executing $POSTSUPER: error " .
"code " . ($?/256) . "\n";
}
if (! $qid ) {
die "No messages with the address <$email_addr> " .
"found in queue.\n";
}
exit 0;
```
#### Holding Messages
- Hold queue thì có sẵn cho message bạn muốn giữ trong queue của bạn vô thời hạn
- Hình đưới đây show hold queue và bạn có thể chuyển message tới hold queue ở đâu, chúng sẽ không được gửi cho đến khi bạn xóa chúng một cách cụ thể hoặc di chuyển chúng trở lại để xử lý queue bình thường. Để đặt thông báo ví dụ vào hold queue, sử dụng dòng lệnh postsuper với -h option:
```
postsuper -h DBA3F1A9
```
- The queue entry chứa dấu chấm than để cho biết rằng message trong hold:
```
-Queue ID- --Size-- ----Arrival Time---- -Sender/Recipient-------
DBA3F1A9 !  553 Mon May 5 14:42:15 kdent@example.com
        (connect to mail.ora.com[192.168.155.63]: Connection refused)
                                          kdent@ora.com
```
![image](https://github.com/DinhHa1011/Postfix/assets/119484840/82fbbfab-10af-45b3-a2f5-d4bf5fc903dd)
- Để di chuyển thư trở lại queue thông thường để xử lý thông thường, thay vào đó hãy thực hiện lệnh với tùy chọn -H viết hoa:
```
postsuper -H DBA3F1A9
```
- Sau khi message chuyển trở lại, queue manager đánh dấu nó để phân phối lại theo lịch trình thông thường, hoặc bạn có thể xóa message để có nó được gửi đi ngay lập tức
#### Requeuing Message
- Nếu bạn có message bị deferred do vấn đề config đã được khắc phục, bạn có thể phải xếp queue để chúng được gửi thành công. Nếu cấu hình sai khiến Postfix lưu trữ không đúng thông tin về bước nhảy hoặc trường chuyển, hoặc để viết lại đúng địa chỉ, yêu cầu Postfix cập nhật thông tin không chính xác dựa trên config mới của bạn. Dòng lệnh postsuper sử dụng option -r để yêu cầu message. Bạn có thể chỉ định một queue ID cho một message hoặc từ ALL bằng chữ in hoa để yêu cầu mọi thứ
```
# postsuper -r ALL
```
- Các message được yêu cầu nhận queue ID mới và một tiêu đề bổ sung Received
#### Display Messages
