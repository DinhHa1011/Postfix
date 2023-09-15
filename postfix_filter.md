# Postfix filter
# Table of contents

- [Postfix filter](#postfix-filter)
  - [Postfix Before-Queue Content Filter](#postfix-before-queue-content-filter)
    - [Tổng quan](#tổng-quan)
    - [Ví dụ cấu hình](#ví-dụ-cấu-hình)
    - [Nhận xét](#nhận-xét)
  - [Postfix After-Queue Content Filter](#postfix-after-queue-content-filter)
    - [Tổng quan](#tổng-quan-1)
    - [Ví dụ cấu hình](#ví-dụ-cấu-hình-1)
      - [Content-filter đơn giản](#cấu-hình-content-filter-đơn-giản)
      - [Content filter nâng cao](#content-filter-nâng-cao)
    - [Nhận xét](#nhận-xét-1)
  - [References](#references)
## Postfix Before-Queue Content Filter 
### Tổng quan
- Xứ lí filter mail trước khi vào queue 
![](https://i.imgur.com/7zFquo6.png)
- Before-Queue Content Filter có thể
    + Đưa lại thư về Postfix qua SMTP, có thể sau khi thay đổi nội dung và/hoặc đích đến của thư.
    + Discard hoặc quarantine mail.
    + Reject mail với SMTP status code phù hợp gửi lại về Postfix để trả lại sender. Điều này giúp Postfix không phải gửi bounce mail 
- Before-Queue Content Filter hoạt động với SMTP pass-through proxy feature
### Ví dụ cấu hình
- Cấu hình before-filter Postfix nhận mail, chuyển sang cho  content filter chạy trên localport 10025. Content-filter xử lí xong thì gửi sang After-Queue Content Filter qua localport 10026 từ đó xử lí mail như bình thường.
- Edit file `/etc/postfix/master.cf`
    + ``` vim /etc/postfix/master.cf```
    + Trong phần `smtp      inet  n       -       n       -       -      smtpd` thêm dòng sau để chuyển mail sang content filter 
        ```
         smtp      inet  n       -       n       -       -      smtpd
                -o smtpd_proxy_filter=127.0.0.1:10025
                -o smtpd_client_connection_count_limit=10
                # -o smtpd_proxy_options=speed_adjust
        ```
         + Dòng 1: smtp mặc định nhận mail của Postfix
         + Dòng 2: Option chuyển mail sang cho content filter trên local port 10025 
         + Dòng 3: Giới hạn số lượng kết nối = 10 tránh quá tải ( có thể không set nếu không sợ)
         + Dòng 4: Tùy chọn cho phép Before-Queue Content Filter biết rằng nó sẽ nhận được toàn bộ thông báo email trước khi kết nối với content filter. Điều này làm giảm số lượng quá trình lọc đồng thời.
          NOTE 1: Nếu tính năng trên được bật => Content_filter không được từ chối chọn lọc recepient trong mail có nhiều người nhận. Chỉ có thể chấp nhận hết hoặc từ chối hết.
          NOTE 2: Tính năng này cần thêm  dung lượng để lưu mail vào một tệp tạm thời.
    + Thêm những dòng sau để xử lí nhận mail từ content filter sau khi lọc xong
        ```
        127.0.0.1:10026 inet n  -       n       -        -      smtpd
                -o smtpd_authorized_xforward_hosts=127.0.0.0/8
                -o smtpd_client_restrictions=
                -o smtpd_helo_restrictions=
                -o smtpd_sender_restrictions=
                # Postfix 2.10 and later: specify empty smtpd_relay_restrictions.
                -o smtpd_relay_restrictions=
                -o smtpd_recipient_restrictions=permit_mynetworks,reject
                -o smtpd_data_restrictions=
                -o mynetworks=127.0.0.0/8
                -o receive_override_options=no_unknown_recipient_checks
        ```
        + Dòng 1:**127.0.0.1:10026** phần này sẽ luôn phải để là local vì nó sẽ được xử lí trực tiếp vào hệ thống mail. KHÔNG được public 
        + Dòng 2: Cho phép **after-filter** SMTP server nhận thông tin client SMTP từ xa từ **before-filter** SMTP server, để các daemon Postfix của **after-filter** ghi lại thông tin máy khách SMTP từ xa thay vì ghi nhật ký máy chủ cục bộ [127.0.0.1]. do đi qua **content filter**
        + Những dòng sau đọc thêm ở chương 4 sách postfix 
### Nhận xét
- Before-Queue Content Filter nhận mail gửi sang content filter để lọc rồi gửi lại cho After-filter từ đó vào queue xử lí tiếp.
- After-filter ở đây khác với After-Queue Content Filter: 1 : đi vào queue xử lí tiếp 2. Xử lí sau khi đã được đưa vào queue
## Postfix After-Queue Content Filter
### Tổng quan
- Xứ lí filter mail sau khi vào queue 
![](https://i.imgur.com/AP8kGuI.png)
- After-Queue Content Filter có thể
    + Đưa lại thư về Postfix qua SMTP, có thể sau khi thay đổi nội dung và/hoặc đích đến của thư.
    + Discard hoặc quarantine mail.
    + Reject mail với SMTP status code phù hợp gửi lại về Postfix để trả lại sender. Điều này giúp Postfix không phải gửi bounce mail 

### Ví dụ cấu hình 

#### Content-filter đơn giản
- Postfix nhận mail tại **smtpd** server. Sau đó những mail chưa được lọc này được gửi sang content filter thông qua **pipe** delivery agent. Content Filter xử lí và gửi ngược lại cho Postfix thông qua **sendmail** command. 
![](https://i.imgur.com/NaQMlyK.jpg)
- Tạo content-filter script
    + Tạo file
    ``` vim /usr/bin/filter```
    + Thêm những dòng sau 
    ```
    #!/bin/bash
    # Localize these.
    SENDMAIL="/usr/sbin/sendmail -i"
    # Exit codes from <sysexits.h>
    EX_TEMPFAIL=75
    EX_UNAVAILABLE=69
    exit $EX_UNAVAILABLE;
    $SENDMAIL "$@" < /dev/null
    exit $?
    ```
    + Cấp quyền 
    ```
    chmod 755 /usr/bin/filter
    ```
    + Mail qua content-filter này sẽ bị trả về báo lỗi `service unavailable`
- Tạo user để chạy filter 
    ```
    useradd filter
    ```
- Cấu hình Postfix gửi mail sang **content-filter** thông qua **pipe**
    + Mở file cấu hình `master.cf` của Postfix 
    ```
    vim /etc/postfix/master.cf
    ```
    + Thêm những dòng sau 
        ```
        filter    unix  -       n       n       -       10      pipe
         flags=Rq user=filter
         argv=/usr/bin/filter -f ${sender} -- ${recipient}
        ```
        + Dòng 1: Các setting cơ bản để sử dụng pipe của Postfix
        + Dòng 2: Flags= Rq : dùng xử lí thông qua pipe các giá trị Return-Path: header và ${sender} , ${recipient}. User = user thực thi filter, user này nên tạo tách biệt không dùng root hay postfix 
        + Dòng 3: argv: lệnh thực thi content filter với các giá trị kèm theo để gửi mail 
    + Trong phần `smtp      inet  n       -       n       -       -      smtpd` thêm dòng sau để enable content filter
    ```
      -o content_filter=filter:dummy
    ```
- Reload Postfix để apply cấu hình 
```
postfix reload
```
- Test 
    + Từ bên ngoài gửi mail vào sẽ bị trả lại với lỗi service unavailable
     ![](https://i.imgur.com/qp6qcnu.png)

    + Log như sau: giá trị relay = filter => có tác dụng
    ```
    Aug 26 01:33:15 dinhha postfix/qmgr[3887703]: CAD2F813AE: from=<root@anthanh264.id.vn>, size=1080, nrcpt=1 (queue active)
    Aug 26 01:33:15 dinhha postfix/smtpd[3888562]: disconnect from unknown[103.97.126.18] ehlo=1 mail=1 rcpt=1 data=1 quit=1 commands=5
    Aug 26 01:33:15 dinhha postfix/pipe[3888811]: CAD2F813AE: to=<ah@dinhha.online>, relay=filter, delay=1.1, delays=1/0/0/0.01, dsn=5.3.0, status=bounced (service unavailable)
    Aug 26 01:33:15 dinhha postfix/cleanup[3888749]: BD84D813C4: message-id=<20230825183315.BD84D813C4@mail.dinhha.online>
    Aug 26 01:33:15 dinhha postfix/qmgr[3887703]: BD84D813C4: from=<>, size=3011, nrcpt=1 (queue active)
    Aug 26 01:33:15 dinhha postfix/bounce[3890602]: CAD2F813AE: sender non-delivery notification: BD84D813C4
    Aug 26 01:33:15 dinhha postfix/qmgr[3887703]: CAD2F813AE: removed
    Aug 26 01:33:16 dinhha postfix/smtp[3890604]: BD84D813C4: to=<root@anthanh264.id.vn>, relay=mail.anthanh264.id.vn[103.97.126.24]:25, delay=0.62, delays=0.01/0.01/0.33/0.27, dsn=2.0.0, status=sent (250 OK id=1qZbcS-00FfGy-0O)
    Aug 26 01:33:16 dinhha postfix/qmgr[3887703]: BD84D813C4: removed

    ```
- Cách làm content-filter trên chỉ để test thực tế không nên áp dụng theo vì vấn đề hiệu suất thấp
- Gỡ bỏ content-filter 
    + Mở file cấu hình `master.cf` của Postfix
        ``` vim /etc/postfix/master.cf```
        + Comment những dòng sau 
        ```
              -o content_filter=filter:dummy

                filter    unix  -       n       n       -       10      pipe
                 flags=Rq user=filter
                 argv=/usr/bin/filter -f ${sender} -- ${recipient}

        ```
    + Xóa hết queue hiện tại 
    ``` postsuper -r ALL ```
    + Reload Postfix
    ``` postfix reload ```
#### Content filter nâng cao
- Postfix nhận mail rồi chuyển sang **Content-filter** thông qua local port **10025**. **Content-filter** xử lí xong chuyển lại **Postfix** bằng localport **10026**
![](https://i.imgur.com/gbUHx1i.png)
- Cấu hình Postfix điều hướng mail về localport 10025
    + Sửa file `main.cf` của Postfix
    ``` vim /etc/postfix/main.cf```
    + Thêm những dòng sau
        ```
        content_filter = scan:localhost:10025
        receive_override_options = no_address_mappings
        ```
        + Dòng 2: No_address_mappings vô hiệu hóa các thao tác tác động vào địa chỉ trong header mail. Content filter sẽ đọc được header gốc không qua chỉnh sửa 
- Cấu hình Postfix smtp gửi mail chưa lọc sang content-filter
    + Sửa file `master.cf` của Postfix
    ``` vim /etc/postfix/master.cf```
    + Thêm dòng sau 
        ```
         scan      unix  -       -       n       -       10      smtp
            -o smtp_send_xforward_command=yes
            -o disable_mime_output_conversion=yes
            -o smtp_generic_maps=
        ```
        + Dòng 2: Chuyển thông tin header gốc tới after-filter smtpd 
        + Dòng 3: Ngăn chặn phá khóa tên miền và các chữ ký số 
        + Dòng 4: Ngăn chặn việc rewrite generic maps
- Khởi chạy content-filter
    + Sửa file `master.cf` của Postfix
    ``` vim /etc/postfix/master.cf```
    + Thêm dòng sau 
    ```
    localhost:10025 inet  n       n       n       -       10      spawn
        user=filter argv=/path/to/filter localhost 10026
    ```
- Cấu hình trả lại về Postfix thông qua port 10026 
    + Sửa file `master.cf` của Postfix
    ``` vim /etc/postfix/master.cf```
    + Thêm dòng sau 
    ```
     localhost:10026 inet  n       -       n       -       10      smtpd
        -o content_filter= 
        -o receive_override_options=no_unknown_recipient_checks,no_header_body_checks,no_milters
        -o smtpd_helo_restrictions=
        -o smtpd_client_restrictions=
        -o smtpd_sender_restrictions=
        # Postfix 2.10 and later: specify empty smtpd_relay_restrictions.
        -o smtpd_relay_restrictions=
        -o smtpd_recipient_restrictions=permit_mynetworks,reject
        -o mynetworks=127.0.0.0/8
        -o smtpd_authorized_xforward_hosts=127.0.0.0/8

    ```
- Gỡ bỏ content-filter 
    + Mở file cấu hình `main.cf` của Postfix
        ``` vim /etc/postfix/main.cf```
        + Comment những dòng sau 
        ```    
        content_filter = scan:localhost:10025
        receive_override_options = no_address_mappings
        ```
    + Xóa hết queue hiện tại 
    ``` postsuper -r ALL ```
    + Reload Postfix
    ``` postfix reload ```
- Phần này không có test như phần trước nhưng cậu có thể thấy nó ngay trong cách Amavis nó tương tác vs Postfix. 

### Nhận xét
- Triển khai Postfix After-Queue Content Filter thực chất các cách sử dụng Content-filter với Postfix
- Có 2 cách xử lí tích hợp Content-filter
    + Command-Based Filtering sử dụng pipe và sendmail
    + Daemon-Based Filtering sử dụng các localport 10025 và 10026
-> Cách 2 được đánh giá cao hơn cho hiệu năng tốt hơn.

## References
* [Postfix Before-Queue Content Filter](https://www.postfix.org/SMTPD_PROXY_README.html)
* [Postfix After-Queue Content Filter](https://www.postfix.org/FILTER_README.html#advanced_filter)
* [Kyle D. Dent - Postfix_ The Definitive Guide (2003, O'Reilly Media - Chapter 14.)](https://drive.google.com/file/d/1qgfeHMxqyThf6BI5T0blsc37QJV8BGzv/view?usp=sharing)
* [Images](https://imgur.com/a/osApLIU)
