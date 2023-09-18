# [Postfix]Transport Maps
# Table of contents

- [[Postfix]Transport Maps](#postfixtransport-maps)
  - [Config postfix relay email sang các host khác](#config-postfix-relay-email-sang-các-host-khác)
  - [Config Defer Mail](#config-defer-mail)
    - [Deferring Relay Mail](#deferring-relay-mail)
    - [Deferring Delivery Mail](#deferring-delivery-mail)
  - [References](#references)
## Config postfix relay email sang các host khác

- Thêm config trong main.cf
    + Mở file 
    ```
    vim /etc/postfix/main.cf
    ```
    + Thêm dòng
    ```
    transport_maps = hash:/etc/postfix/transport
    ```
- Tạo file map transport : chứa địa chỉ relay từng mail cụ thể
    + Tạo file 
    ```
    vim /etc/postfix/transport
    ```
    + Sửa file 
        ```
        example.com smtp:[192.168.23.56]:20025
        oreilly.com relay:[gateway.oreilly.com]
        oreillynet.com smtp
        ora.com maildrop
        kdent@ora.com error:no mail accepted for kdent
        ```
        + `example.com smtp:[192.168.23.56]:20025` mọi mail tới example.com sẽ được chuyển sang 192.168.23.56 ở cổng 20025
        + `oreilly.com relay:[gateway.oreilly.com]` mọi mail tới oreilly.com sẽ được chuyển sang gateway.oreilly.com ở cổng 25 <dòng này sẽ tự nhận là port smtp 25, còn với ví dụ trên là config cho port #25
        + ` oreillynet.com smtp ` mọi mail tới oreillynet.com sẽ được relay vs port 25 còn host thì do query DNS theo MX tương ứng của miền đó 
        + `ora.com maildrop` mọi mail tới ora.com sẽ được chuyển tới service maildrop. maildrop phải là 1 transport cấu hình trong master.cf 
        + `kdent@ora.com error:no mail accepted for kdent` từ chối theo email cụ thể
- Execute postmap file.
` postmap /etc/postfix/transport`
- Reload postfix để apply cấu hình 
```
postfix reload 
```

## Config Defer Mail 
### Deferring Relay Mail
- Tạo 1 service transport mới trong `master.cf` tên là ondemand 
    + Mở file
    `
    vim /etc/postfix/master.cf
    `
    + Thêm dòng 
    `ondemand unix - - n - - smtp`
- Cấu hình postfix để cho all mail qua transport ondemand đều bị defer
    + Mở file
    `
    vim /etc/postfix/main.cf
    `
    + Thêm dòng
        `defer_transports = ondemand`
       + **Note** Thêm dòng `transport_maps =  hash:/etc/postfix/transport` nếu chưa có, có rồi thì thôi.
- Sửa file transport_map: 
    + Mở file 
    ```
    vim /etc/postfix/transport
    ```
    + Thêm dòng
    ```
    example.com ondemand
     ```
- Execute postmap file.
` postmap /etc/postfix/transport`
- Reload postfix để apply cấu hình 
```
postfix reload 
```
Các mail tới example.com đều bị hoãn relay và chỉ được relay tiếp khi chạy lệnh 
`postqueue -f example.com`

### Deferring Delivery Mail
- Cấu hình postfix để cho all mail đều bị defer
    + Mở file
    `
    vim /etc/postfix/main.cf
    `
    + Thêm dòng
        `defer_transports = smtp`
- Reload postfix để apply cấu hình 
```
postfix reload 
```
Các mail tới hệ thống đều bị defer và chỉ được chuyển tiếp khi chạy lệnh:
`postqueue -f`
## References
* [Kyle D. Dent - Postfix_ The Definitive Guide (2003, O'Reilly Media - Chapter 9.)](https://drive.google.com/file/d/1qgfeHMxqyThf6BI5T0blsc37QJV8BGzv/view?usp=sharing)
