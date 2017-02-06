---
layout: "post"
title: "Giảm số lượng HTTP request"
description: "Tuning performance 101(1): Các kỹ thuật để giảm số lượng HTTP request"
date: "2017-02-02 14:47"
tags: "performance performance101"
---

**80% thời gian chờ của người dùng đến từ những xử lý ở phía client và phần lớn trong số đó là dành để tải các nội dung như ảnh, scripts, stylesheets,... Bên cạnh đó, trên thực tế, khi tạo nhiều HTTP request để tải về các nội dung nhỏ lại chậm hơn nhiều so với khi tạo ít HTTP request để tải các nội dung lớn hơn, vì thế, một cách đơn giản để tăng performance của hệ thống là giảm các HTTP request bằng cách gộp các thành phần nội dung lại với nhau**

![google.com](/assets/images/1486031620331.png)

Có bốn kỹ thuật để gộp các nội dung và giảm số lượng HTTP request, đó là `Image Maps`, `CSS Sprites`, `Inline Images` và `Combine files`

## Image Maps
Ý tưởng của `Image maps` là gộp các ảnh nhỏ thành một ảnh lớn rồi sau đó hiển thị các ảnh nhỏ theo tọa độ được giới hạn trong bức ảnh lớn đó. Kỹ thuật này tuy không làm giảm tổng dung lượng mà người dùng phải tải về nhưng lại có thể làm giảm đáng kể số lượng HTTP request đến server.

Về mặt kỹ thuât, `Image maps` sử dụng đặc tính của 2 thẻ HTML là `<map>` và `<area>` để chỉ định vùng của bức ảnh muốn hiển thị. Bởi vậy, sử dụng kỹ thuật này, ta sẽ gặp phải một hạn chế là các ảnh nhỏ phải hiển thị cạnh nhau, đây là lý do làm cho `Image Maps` trở nên không linh hoạt và cũng không được áp dụng nhiều.

## CSS Sprites
Cùng ý tưởng với `Image Maps` nhưng `CSS Sprites` lại tỏ ra linh hoạt hơn khi sử dụng các thuộc tính `background-image` và `background-position` của CSS để chỉ định tọa độ hiển thị trong bức ảnh lớn.

Trên thực tế, `CSS Sprites` được sử dụng rất phổ biến, ta có thể thấy kỹ thuật này được áp dụng từ những website cá nhân đến các hệ thống lớn như `facebook`, `google`.


## Inline Images
Kỹ thuật này sử dụng `data: URL scheme` để đặt dữ liệu ảnh vào trong trang web. Nói một cách dễ hiểu là ta sử dụng dữ liệu ảnh đã mã hóa `base64` thay cho đường dẫn đến ảnh trên server.

Điểm hạn chế của kỹ thuật này là:

- Dung lượng ảnh sau mã hóa có thể tăng so với file ảnh gốc, vì thế tuy số lượng HTTP request tuy giảm nhưng tổng số dung lượng tải về của người dùng lại tăng.
- Ảnh không được trình duyệt lưu trong bộ nhớ cache. Mỗi khi tải lại trang web, người dùng sẽ phải tải loại toàn bộ hình ảnh.
- Làm rối code HTML / CSS cũng như việc sửa chữa cũng khó khăn hơn so với ảnh bình thường.

Kỹ thuật này cũng được sử dụng trong một số trường hợp như sử dụng trong website tĩnh, ảnh cần hiển thị ít khi thay đổi,...

## Combine files
Để giảm số lượng HTTP Request, ta có thể kết hợp tất cả các tài nguyên có cùng kiểu lại với nhau, ví dụ như các file scripts hay các file stylesheets. Kỹ thuật này ngoài giúp làm giảm số lượng HTTP request nó còn là giúp cho một số kỹ thuật khác như nén file đạt hiệu quả cao hơn từ đó làm tăng hiệu năng cuả hệ thống.
Áp dụng kỹ thuật này không có nghĩa ta phải viết toàn bộ scripts hay stylesheets vào một file duy nhất, ta có thể thực hiện việc kết hợp file này trong `build process`, sử dụng một số công cụ như `Webpack`, `Grunt`, `Gulp`,...
