---
layout: "post"
title: "Giảm dung lượng các tài nguyên dạng text"
date: "2017-02-06 16:16"
description: "Tuning performance 101(4): Giảm dung lượng các tài nguyên dạng text"
tags: "performance performance101"
---

**Tài nguyên dạng text là các file scripts và stylesheets trong hệ thống. Với việc UI, UX cũng như những xử lý ở phía client ngày càng được chú ý, dung lượng các tài nguyên này cũng lớn hơn trước rất nhiều. Và một trong những cách để tối ưu tốc độ tải trang đó là làm giảm dung lượng cho những tài nguyên đó.**

Có hai cách để tối ưu dung lượng cho các tài nguyên dạng text đó là **minify** (thu gọn) và **compress** (nén) file.

## Minification
Để giảm dung lượng một file text, việc đơn giản nhất là loại bỏ những ký tự không cần thiết như các comments hay khoảng trắng. Comments và khoảng trắng là những ký tự rất cần thiết đối với lập trình viên khi phát triển sản phẩm, tuy nhiên, một khi đã đóng gói, những ký tự này lại trở nên thừa thãi và làm cho dung lượng file lớn hơn khá nhiều.
Với các file scripts và stylesheets, ta còn có thể thu gọn ở mức thông minh hơn, phụ thuộc vào nội dung của file, ví dụ:

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
