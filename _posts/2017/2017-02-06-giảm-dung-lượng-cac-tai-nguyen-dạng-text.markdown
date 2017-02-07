---
layout: "post"
title: "Giảm dung lượng các tài nguyên dạng text"
date: "2017-02-06 16:16"
description: "Tuning performance 101(4): Giảm dung lượng các tài nguyên dạng text"
tags: "performance performance101"
---

**Tài nguyên dạng text là các file scripts và stylesheets trong hệ thống. Với việc UI, UX cũng như những xử lý ở phía client ngày càng được chú ý, dung lượng các tài nguyên này cũng lớn hơn trước rất nhiều. Và một trong những cách để tối ưu tốc độ tải trang đó là làm giảm dung lượng cho những tài nguyên đó.**

![Minification](/assets/images/1486437574828.png)

Có hai cách để tối ưu dung lượng cho các tài nguyên dạng text đó là **minify** (thu gọn) và **compress** (nén) file.

## Minification
Để giảm dung lượng một file text, việc đơn giản nhất là **loại bỏ những ký tự không cần thiết như các comments, khoảng trắng và các ký tự xuống dòng**. Các ký tự này có thể rất cần thiết đối với lập trình viên khi phát triển sản phẩm, tuy nhiên, khi đã đóng gói, những ký tự này lại trở nên thừa thãi và làm cho dung lượng file lớn hơn khá nhiều.
Với các file scripts và stylesheets, ta còn có thể thu gọn code thông minh hơn, phụ thuộc vào nội dung của file, ví dụ như với một đoạn css như sau:

```css
.container {
  background-color: black;
}
.container {
  color: #660066;
}
```

có thể thu gọn thành:

```css
.container {
  background-color: black;
  color: #606;
}
```
Với các file scripts, có rất nhiều kỹ thuật để thu gọn code một cách thông minh:
- Đổi tên hàm, biến, tham số ngắn gọn hơn
- Loại bỏ những đoạn code không dùng đến
- **Inlining**: nếu hàm khai báo không dùng lại nhiều, có thể thay thế việc khai báo và gọi hàm bằng thân hàm.

### Build process
Quy trình tối ưu code (ở đây chủ yếu là scripts và stylesheets) từ môi trường của lập trình viên sang môi trường khách hàng gọi là **build process**. Quy trình này có thể làm thủ công hoặc tự động, sử dụng một số công cụ như `Grunt`, `Gulp` hay `Webpack`, bao gồm những xử lý thường gặp như:

  - Gộp các file (combines files như đã đề cập trong [Giảm số lượng HTTP request](/blog/giảm-số-lượng-http-request/)
  - Thu gọn file (minify)
  - Kiểm tra lỗi (linting)
  - ...

Việc tạo một **build process** là rất cần thiết cho mỗi dự án, giúp tối ưu hệ thống, giảm lỗi và tiết kiệm thời gian rất nhiều.

Ngoài ra, [Google Closure](https://developers.google.com/closure/compiler/) là một công cụ không chỉ thu gọn mà còn tối ưu code javascript rất mạnh mẽ. Ta cũng có thể kết hợp nó cùng với các công cụ giúp tạo build process.

## Compression
Từ HTTP 1.1, các trình duyệt web bắt đầu hỗ trợ việc nén các tài nguyên web bằng header `Accept-Encoding`:

```
Accept-Encoding: gzip, deflate
```

Khi client gửi một HTTP request có header như trên, server tiếp nhận request sẽ hiểu là client chấp nhận nội dung được nén dạng `gzip` hoặc `deflate`.

Trong các phương thức nén cho các tài nguyên web, `Gzip` đang là phương thức nén phổ biến và hiệu quả nhất. Phương thức này được phát triển bởi dự án GNU và chuẩn hoá bởi *RFC 1952*. Trên thực tế, ngoài `Gzip`, chúng ta chỉ thường thấy một phương thức nén khác, đó là `deflate`, tuy nhiên, phương thức nén này cũng không phổ biến và hiệu quả bằng `Gzip`.

Tài nguyên sau khi nén hiển nhiên sẽ có dung lượng nhỏ hơn, từ đó làm giảm thời gian người dùng phải chờ tải về. Tuy nhiên, các nội dung bị nén cũng đồng nghĩa với việc phía client phải giải nén dữ liệu. Trên thực tế, công việc này tuy không nặng nề nhưng cũng tiêu tốn một lượng CPU cũng như một khoảng thời gian nhất định. Vì thế, khi thực hiện việc nén các tài nguyên web, ta nên có giới hạn dung lượng file tối thiểu (thường thì `gzip` sẽ chỉ cho hiệu quả tốt với các file có dung lượng lớn hơn 1 hoặc 2 KB).

### Cấu hình
Việc nén các tài nguyên thường được cấu hình tự động ở máy chủ HTTP (Apache, Nginx, ...). Tuy nhiên cũng có một số hệ thống cấu hình công việc này ở middleware (Rack) hay application.

## Kết luận

- Việc thu gọn nội dung các tài nguyên dạng text giúp làm giảm đáng kể dung lượng của chúng.
- Mỗi dự án nên xây dựng **build process** riêng, tốt nhất là xử lý tự động sử dụng các công cụ như `Grunt`, `Gulp` hay `Webpack`. Việc thu gọn nội dung là một trong những xử lý trong **build process** đó.
- Với các tài nguyên dạng text (HTML document, scripts, stylesheets,...) có dung lượng lớn hơn 1 đến 2 KB thì nên được nén lại.
- Phương pháp nén phổ biến nhất hiện nay là `Gzip`, các trình duyệt hiện đại ngày nay đều hỗ trợ và yêu cầu các tài nguyên nén dạng này.
- Việc nén nên được cấu hình tự động ở máy chủ HTTP (Apache, Nginx,...)
