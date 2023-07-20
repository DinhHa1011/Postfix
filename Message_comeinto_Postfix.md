- Message come into Postfix bằng 4 cách:
  + Message có thể được chấp nhận vào Postfix locally (gửi từ một user trên cùng máy)
  + Message có thể được chấp nhận vào Postfix qua network
  + Message có thể được chấp nhận vào Postfix bằng phương thức khác là chuyển tiếp từ địa chỉ khác
  + Postfix tự tạo thông báo về khi nó gửi thông báo bị trả về hoặc bị trì hoãn
    => luôn có khả năng message bị reject trước khi nó vào postfix hoặc một vài message defer để gửi sau
### Local Email Submission
- Các thành phần postfix khác hoạt động cùng nhau bằng cách viết message và đọc message từ queue. Trình quản lý queue có trách nhiệm quản lý 
