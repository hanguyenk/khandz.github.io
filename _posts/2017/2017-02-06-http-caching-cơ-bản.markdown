---
layout: "post"
title: "HTTP Caching cơ bản"
date: "2017-02-06 13:10"
description: "Tuning performance 101(3): HTTP Caching"
tags: "performance performance101 caching header"
---

**Trong một hệ thống web, có những nội dung tĩnh (ảnh, scripts, stylesheets,...) ít thay đổi và được sử dụng lại ở nhiều nơi. Bằng việc sử dụng các kỹ thuật HTTP Caching, ta có thể tạo ra các bản copy các nội dung này lưu lại ở phía client và sử dụng khi cần, từ đó giảm các HTTP request, tăng performance của hệ thống.**

![HTTP Request](/assets/images/1486370194516.png)

## HTTP Caching
`HTTP Caching` là công việc lưu cache và sử dụng lại những tài nguyên đã được tải về thông qua giao thức HTTP.
Giả sử ta có một file ảnh, hầu như không thay đổi nội dung, được lưu trên server. Nếu không có `HTTP caching`, mỗi khi client cần file này, nó lại gửi yêu cầu đến server và tải về. Ngược lại, khi sử dụng `HTTP caching`, file này sẽ được lưu lại một bản copy ở phía client và lôi ra sử dụng khi client cần.

## Vì sao lại cần sử dụng HTTP Caching ?
Ta có thể thấy việc truy xuất tài nguyên từ dữ liệu cache nhanh hơn rất nhiều so với việc yêu cầu và tải lại từ server, vốn đi qua rất nhiều bước phức tạp. Vì vậy, lợi ích đầu tiên mà `HTTP caching` đem lại chính là `giảm độ trễ của việc tải trang, tăng performance cho hệ thống`. Ngoài ra giảm yêu cầu lên server cũng đồng nghĩa với việc `tiết kiệm băng thông cho server`.

## Cấu hình HTTP Header
Các trình duyệt ngày nay đều hỗ trợ `HTTP cache`, tất cả những gì ta cần làm chỉ là đảm bảo việc HTTP response trả về đi kèm những thông tin mong muốn về việc lưu cache.

### Strong caching headers
**Strong caching headers** bao gồm `Expires` và `Cache-Control`. Khi có yêu cầu tới một tài nguyên đã từng tải về, trình duyệt sẽ kiểm tra các trường này trước khi quyết định gửi request lên server hay sử dụng bản copy trong bộ nhớ cache.

#### Expires
Cấu hình `Expires` có dạng như sau:

```
expires: Thu, 02 Feb 2017 20:00:00 GMT
```
Với header trên, server cho client biết rằng cho đến *20 giờ ngày 2 tháng 2 năm 2017*, nội dung này sẽ không có thay đổi gì lớn, client có thể tạo một bản copy và lưu vào cache để sử dụng mà không cần phải yêu cầu lại server.

#### Cache-Control
![Cache-Control](/assets/images/1486370102531.png)
Cấu hình `Cache-Control` có dạng như sau:

```
cache-control: max-age=3600
```

Với header trên, server cho client biết rằng nội dung này sẽ không có thay đổi gì lớn trong *3600 giây* nữa, client có thể tạo một bản copy và lưu vào cache để sử dụng mà không cần phải yêu cầu lại server. Ngoài `max-age` như ở trên, `Cache-Control` còn có nhiều tuỳ chọn thông tin khác, cụ thể:

  - **max-age=[seconds]**: khoảng thời gian tối đa mà nội dung có hiệu lực, được tính bằng đơn vị giây
  - **s-maxage=[seconds]**: khoảng thời gian tối đa mà nội dung có hiệu lực trong `shared caches`
  - **public** / **private** : nếu chỉ định `private`, các proxies hoặc server trung gian sẽ không cache nội dung này. Ví dụ như các ISP có thể cài đặt proxy để cache lại nội dung trang web nhằm giảm băng thông và tiết kiệm chi phí. Trong trường nợp này, nếu là response `private`, thì proxy đó sẽ không cache lại nội dung này và ngược lại nếu là response `public`, proxy có thể cache lại nội dung này.
  - **no-cache** / **no-store**: `no-cache` nghĩa là trong những lần yêu cầu sau, người dùng sẽ vẫn phải gửi request để kiểm tra xem nội dung có bị thay đổi hay không. Nếu nội dung không thay đổi, client không cần phải tải lại nội dung và ngược lại. Đơn giản hơn, `no-store` được dùng khi không cho phép nội dung được lưu lại ở bất cứ đâu, dưới bất cứ hình thức nào. `no-store` thường được dùng cho những dữ liệu nhạy cảm như thông tin cá nhân, ngân hàng,...

`Expires` được giới thiệu từ HTTP 1.0 trong khi `Cache-Control` là từ HTTP 1.1. Tuy nhiên bạn không cần phải lo lắng do hầu hết các trình duyệt ngày nay đều hỗ trợ HTTP 1.1. Thêm một điểm đáng lưu ý là khi sử dụng cả `Expires` lẫn `Cache-Control` thì `Cache-Control` được ưu tiên hơn.

### Weak caching headers
**Weak caching headers** bao gồm `Etag` và `Last-Modified`. Sau khi kiểm tra các `Strong caching headers` (nếu có) và quyết định sẽ gửi yêu cầu lên server, client sẽ thêm các thông tin này kèm theo HTTP request.

#### Etag
![Etag](/assets/images/1486370170765.png)
`Etag` là một giá trị đặc biệt gắn liền với một file và được sinh ra trên server. Giá trị này có thể được tạo ra dựa vào thời gian chỉnh sửa cuối cùng hay checksum của file và phải đảm bảo khi nội dung file thay đổi thì giá trị này cũng thay đổi theo.

Khi tạo ra một yêu cầu lên server, browser tự động gửi kèm theo tham số `If-None-Match` trong HTTP request. Sau đó, server khi nhận được yêu cầu sẽ so sánh giá trị này với giá trị `Etag` của file. Nếu trùng khớp (tức là file đó không thay đổi), server sẽ trả về mã `304 Not Modified` và trình duyệt có thể tải dữ liệu từ trong cache. Ngược lại, nếu có sai khác thì server sẽ trả về file mới với giá trị `Etag` mới.

#### Last-Modified
Cũng cùng một ý tưởng với `Etag` tức là kiểm tra file đang có trong cache và file trên server có giống nhau không, nhưng thay vì tạo ra một giá trị đặc biệt, `Last-Modified` sử dụng giá trị thời gian cuối cùng mà file được chỉnh sửa.
Khi tạo ra một yêu cầu lên server, browser tự dộng gửi kèm theo tham số `If-Modified-Since` trong HTTP request. Sau đó, server khi nhận được yêu cầu sẽ so sánh giá trị này với giá trị ngày thay đổi của file. Nếu trùng khớp (tức là file đó không thay đổi), server sẽ trả về mã `304 Not Modified` và trình duyệt có thể tải dữ liệu từ trong cache. Ngược lại, nếu có sai khác thì server sẽ trả về file mới với giá trị `Last-Modified` mới.

Về cơ bản, `Etag` và `Last-Modified` không có nhiều điểm khác biệt. Tuy nhiên, có một đặc điểm là một file trên nhiều server khác nhau sẽ có những `Etag` khác nhau, vì thế khi triển khai hệ thống đặt trên nhiều server thì `Last-Modified` được khuyên dùng hơn.

## Kết luận:

- `HTTP Caching` được chia làm 2 loại:
  - **Strong caching headers** với `Expires` và `Cache-Control`
  - **Weak caching headers** với `Etag` và `Last-Modified`
- Hệ thống nên cấu hình một giải pháp **Strong caching headers** kết hợp với một giải pháp **Weak caching headers**, trong đó:
  - `Cache-Control` nên dùng hơn so với `Expires` bởi tính linh hoạt và khả năng tuỳ biến cao.
  - `Last-Modified` nên dùng hơn so với `Etag` bởi khả năng mở rộng trên nhiều server và việc cấu hình cũng dễ dàng hơn.
- Với các giải pháp **Strong caching headers**, nên kết hợp việc đánh version cho file để đảm bảo việc chỉnh sửa trước thời hạn cache.
