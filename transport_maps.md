- Postfix có thể config để relay tới bất kỳ host nào khác, bất kể bản ghi DNS MX được set up như thế nào.
- Section này thảo luận về tham số transport_maps nói chung
- Section sau và chapter khác trình bày config cụ thể để sử dụng nó
- Về mặt khái niệm, transport maps ghi đè các loại default transport để gửi message. Mục dưới đây thiết lập /etc/postfix/transport như một bảng tra cứu transport map:
```
transport_maps = hash:/etc/postfix/transport
```
- key trong transport lookup table là địa chỉ email hoặc domain đầy đủ và subdomains. (Email address as lookup key for transport maps require Postfix 2.0 or later)
- Khi một destination address hoặc domain matches 1 lefthand key nó sử dụng righthand value để xác định delivery mothod và destination
- Ví dụ:
```
example.com                smtp:[192.168.23.56]:20025
oreilly.com                relay:[gateway.oreilly.com]
oreillynet.com             smtp
ora.com                    maildrop
kdent@ora.com              error:no mail accepted for kdent
```
- Format của righthand values có thể khác nhau tùy theo transport type, nhưng có form là transport:nexthop, nơi mà nexthop thường chỉ ra 1 host và port for delivery. Mỗi possible portion của righthand value thì described ở đây:
  - transport:
    - Refers to an entry from master.cf. If you are adding a new transport type, first create an entry for it in master.cf
  - host:
    - The destination host for delivery of message. The host is used only with inet transport such as SMTP and LMTP. Postfix treats the hostname like any destination domain.
    - If performs an MX lookup to determine where to deliver message
    - Nếu không có MX records, Postfix gửi tới A record IP address
    - Nếu bạn biết rằng Postfix sẽ phân phối trực tiếp tới địa chỉ IP trong A record cho specified host, bạn có thể yêu cầu Postfix skip check MX records bằng cách đặt name trong brackets.
    - Nếu bạn sử dụng một địa chỉ IP, dấu ngoặc là bắt buộc
  - port:
    - Destination port cho message deliver
    - The port được sử dụng chỉ với inet transport như SMTP và LMTP
    - Port có thể được chỉ định sử dụng số thực hoặc tên tượng trưng của từ file /etc/services
- Mỗi sample entries từ ví dụ sử dụng format khác trong righthand values của họ, giải thích dưới đây:
  ```
  example.com smtp:[192.168.23.56]:20025
  ```
    - Tất cả message cho http://example.com đươc relay suer dụng smtp transport tới host tại IP address 192.168.25.56. Message được gửi trên port 20025 thay vì SMTP mặc định port 25
    - Lưu ý rằng, địa chỉ IP nằm trong ngoặc, như yêu cầu cho địa chỉ IP
  ```
  oreilly.com relay:[gateway.oreilly.com]
  ```
    
