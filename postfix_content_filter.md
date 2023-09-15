# Content Check Postfix
# Table of contents

- [Content Check Configuration](#content-check-configuration)
  - [Content Check](#content-check)
  - [Write Regex Patern](#write-regex-patern)
  - [Config Content Checking in Postfix](#config-content-checking-in-postfix)
  - [References](#references)
## Content Check 
- Postfix có thể lọc nội dung mail thông qua 4 trường (parameter) sau
    + **header_checks**: Check Header
    + **mime_header_checks**: Check MIME Header
    + **nested_header_checks**: Check Header của tệp đính kèm
    + **body_checks**: Check nội dung thư

- Các trường trên hoạt động dựa trên 1 bảng chứa các patterns định trước kèm hành động theo nó
    + Ví dụ 1 dòng trong bảng
    ```
    /hi/ REJECT
    ```
    + Hiểu là match có ký tự `hi` thì sẽ `REJECT`
    + Mặc định `mime_header_checks` và `nested_header_checks` sẽ dùng chung bảng check với `header_checks` nếu muốn tách lẻ -> Tự cấu hình file tách lẻ của từng parameter
    ```
    header_checks = regexp:/etc/postfix/header_checks
    body_checks = regexp:/etc/postfix/body_checks
    mime_header_checks=regexp:/etc/postfix/mime_header_checks
    nested_header_checks=regexp:/etc/postfix/nested_header_checks
    ```
- Các Actions hỗ trợ trong bảng check
    + **REJECT** message : Nếu match điền kiện thì Reject 
    + **WARN** message : Hiển thị logs Warning nếu match điều kiện: Dùng test trước khi quyết định chuyển sang sử dụng action Reject
    + **IGNORE**: Nếu match điều kiện thì xóa. Với header thì sẽ remove cả header. Với body sẽ xóa dòng chứa điều kiện đã match.Dùng trong trường hợp không muốn để lộ thông tin nội bộ khi gửi qua mailrs or lines from the body of a message. Khi dùng chú ý vì match sẽ xóa Header (gây khó khăn trong việc match tìm lỗi khi gặp vấn đề)
    + **HOLD** message: Nếu match điều kiện thì đưa tin nhắn vào HOLD Queue
        - Lệnh xem queue hold
        ```
         postqueue -p
        ```
        - Lệnh xem nội dung mail bị HOLD
        ```
        postcat -bh -q <Queue-ID>
        ```
        - Lệnh move khỏi hold trở lại normal queue 
        ```
        postsuper -H <Queue-ID>
        ```
    + **DISCARD** message: Nếu match điều kiện => Discard
    + **FILTER** transport:nexthop : Nếu match điều kiện => Đưa sang cho content filter xử lí 
## Write Regex Patern 
- Bảng cú pháp regex cơ bản
![](https://i.imgur.com/RF1atPM.png)
- Quy tắc match chuỗi của regex
    + `[xyz]` Tìm và so sánh tất cả ký tự nằm trong dấu ngoặc vuông và trùng khớp với 1 ký tự trong dấu ngoặc vuông. Ví dụ: [31] sẽ trùng khớp với 3 hoặc 1, [0123456789] sẽ trùng khớp với bất kỳ một ký tự nào trong khoảng từ 0 đến 9.
    + ``[a-z]`` So sánh và trùng khớp với một ký tự nằm trong khoảng chỉ định. Ví dụ: [a-z] sẽ trùng khớp với một ký tự trong khoảng từ a đến z nằm trong chuỗi cần test. [0-9] sẽ trùng khớp với bất kỳ một ký tự nào trong khoảng từ 0 đến 9.
    + ``[^xyz]`` So sánh và không trùng khớp với những ký tự nằm trong khoảng chỉ định. Dấu ^ (dấu mũ) nằm trong dấu ngoặc vuông là một dấu phủ định. Ví dụ: [^a-z] sẽ không trùng khớp với tất cả các ký tự nằm trong khoảng từ a đến z.
    + ``^`` Trùng khớp với phần đầu của chuỗi đích. Ví dụ: ^a sẽ trùng khớp với chữ a trong chuỗi abc, ^\w+ sẽ trùng khớp với chữ đầu tiên – chữ "the" của chuỗi "The quick brown fox jumps over the lazy dog".
    + ``$`` Trùng khớp với phần cuối của chuỗi đích. Ví dụ: c$ sẽ trùng khớp với chữ c trong chuỗi abc, \w+$ sẽ trùng khớp với chữ cuối – chữ "dog" của chuỗi "The quick brown fox jumps over the lazy dog".
    + ``+`` Trùng khớp với 1 hoặc nhiều lần ký tự đứng trước nó. Ví dụ \d+ sẽ chỉ trùng với chuỗi có từ 1 con số trở lên.
    + ``*`` Trùng khớp với 0 hoặc nhiều lần ký tự đứng trước nó. Ví dụ \d* sẽ trùng với chuỗi có chứa 1 chữ số hoặc k có chữ số nào cũng đc.
    + ``?`` Trùng khớp với 0 hoặc 1 lần ký tự đứng trước nó. Tương tự như * nhưng nó lại chỉ nhân lên 1 lần. * thì nhân lên nhiều lần.
    + ``.`` Trùng khớp với 1 ký tự đơn bất kỳ ngoại trừ ký tự ngắt dòng (line-break) và cũng không lấy được ký tự có dấu (unicode). Ví dụ: . sẽ trùng khớp với ký tự a hoặc b hoặc c trong chuỗi abc. Nhưng . sẽ không bắt được các chữ ă hoặc ê.
    + `x{n}` Trùng khớp đúng với n lần ký tự đứng trước nó. n là một số không âm. Ví dụ \d{2} sẽ bắt đc các số có 2 chữ số đứng liền nhau.
    + `x{n,}` Trùng khớp với ít nhất n lần ký tự đứng trước nó. n là một số không âm.Ví dụ \d{2,} sẽ bắt đc các số có từ 2 chữ số trở lên đứng liền nhau.
    + `x{n,m}` Trùng khớp với ít nhất n lần và nhiều nhất là m lần ký tự đứng trước nó. n và m là một số không âm và n <= m. Ví dụ: a{1,3} sẽ khớp với hah, haah, haaah nhưng không khớp với haaaah.
    + `x|y` Trùng khớp với x hoặc y. Ví dụ: slow|fast sẽ khớp với chữ slow hoặc fast trong chuỗi đích.
    + `\b` Trùng khớp với toàn bộ ký tự đứng trước nó. Ví dụ: hello\b sẽ trùng khớp với toàn bộ từ hello trong chuỗi hello world nhưng sẽ không khớp với chuỗi helloworld.
    + `\B` Ngược lại với \b, \B sẽ không khớp với toàn bộ mà chỉ 1 phần ký tự đứng trước nó. Ví dụ: hello\B sẽ trùng khớp với chữ hello trong chuỗi helloworld nhưng sẽ không khớp với chuỗi hello world.
    + `\d `Trùng khớp 1 ký tự số (digit).
    + `\D` Trùng khớp 1 ký tự không phải số (non-digit).
    + `\s` Trùng khớp 1 ký tự khoảng trắng (whitespace) bao gồm khoảng trắng tạo ra bởi phím Tab.
    + `\S `Trùng khớp với 1 ký tự không phải là khoảng trắng (non-whitespace).
    + `\w` Trùng khớp với các ký tự là từ (word) bao gồm dấu _ (underscore) và chữ số.
    + `\W` Trùng khớp với các ký tự không phải là từ (non-word). Ví dụ: \W sẽ khớp với ký tự % trong chuỗi "100%".
    + `\uxxxx` Trùng khớp với 1 ký tự unicode. Ví dụ: \u00FA sẽ khớp với ký tự "ú", \u00F9 sẽ khớp với ký tự "ù". Bảng tra unicode tiếng Việt [tại đây](https://vietunicode.sourceforge.net/charset/) 
    + `\pL` Trùng khớp với một ký tự Unicode bất kỳ ngoại trừ dấu cách. Đây chính là cú pháp viết hoàn hảo hơn của dấu .,Ví dụ \pL+ sẽ lấy được chuỗi truyền, thuyết trong chuỗi "truyền thuyết".
- Website để thực hành viết regex[ tại đây ](https://regexr.com/)
- Ví dụ một file mẫu sẽ có những dòng như sau 
    ```
    /increase your sales by/ REJECT
    /lowest rates.*\!/ REJECT
    /in compliance (with|of) strict/ REJECT
    /[:alpha:]<!--.*-->[:alpha:]/ REJECT Suspicious embedded HTML comments
    ```
    + Dòng 1 : Match chuỗi "increase your sales by"
    + Dòng 2 : Match chuỗi bắt đầu bằng "lowest rates" kèm theo một đoạn ký tự theo sau nó và kết thúc bằng "!" . Đoạn text như sau sẽ bị match với điều kiện này "`("We have our lowest rates in 40 years!")`
    + Dòng 3: Match chuỗi `in compliance with strict` hoặc `in compliance of strict` 
    + Dòng 4: Match comment HTML trong text
## Config Content Checking in Postfix
- Thêm trường trong file `main.cf`
    Ví dụ với header các trường còn lại add tương tự
    +  Mở file `main.cf` của Postfix
    +  Thêm dòng sau. Lưu và đóng file
    ```
    header_checks = regexp:/etc/postfix/header_checks
    ```
- Tạo file chứa content check 
    + Tạo file
    ```
    vim /etc/postfix/header_checks
    ``` 
    + Thêm nội dung và action: Ví dụ ở đây if match `hi` then action `Reject`
    ```
    /hi/    REJECT
    ```
    + Postmap cho file 
    ```
    postmap /etc/postfix/header_checks
    ```
- Restart Postfix để nhận cấu hình 
```
systemctl restart postfix
```

## References
* [How to check queue status to hold, deferred and active](https://dilliganesh.wordpress.com/2016/11/04/how-to-check-queue-status-to-hold-deferred-and-active/)
* [Regex là gì?](https://topdev.vn/blog/regex-la-gi/)
* [Học Regular Expression và cuộc đời bạn sẽ bớt khổ](https://viblo.asia/p/hoc-regular-expression-va-cuoc-doi-ban-se-bot-kho-updated-v22-Az45bnoO5xY)
* [Kyle D. Dent - Postfix_ The Definitive Guide (2003, O'Reilly Media) Chapter 11](https://drive.google.com/file/d/1qgfeHMxqyThf6BI5T0blsc37QJV8BGzv/view)
