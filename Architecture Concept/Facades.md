# Facades
## Introduction
Facades cung cấp một "static" interface cho các class sử dụng, nó đã có sẵn trong `service container`. Laravel cung cấp
nhiều facades, chúng được truy cập ở hầu hết tất cả các tính năng của Laravel.

Laravel facades làm việc giống như một "static proxies" cho các class bên dưới trong service container, cung cấp các lợi
ích của việc sử dụng cú pháp ngắn gọn trong khi vẫn duy trì khả năng kiểm tra và tính linh hoạt hơn so với các method static
truyền thống.

#### Helper Functions
Laravel cung cấp nhiều global `helper function` giúp việc tương tác với các tính năng của Laravel dễ dàng hơn. Một số
helper function phổ biến bạn có thể tương tác như `view`, `response', `url`, `config` và nhiều hơn nữa. Mỗi helper function
đều được ghi lại với các tính năng tương ứng của chúng; tuy nhiên, một danh sách đầy đủ có sẵn trong [helper document](https://laravel.com/docs/8.x/helpers)

Ví dụ, thay vì sử dụng `Illuminate\Support\Facades\Response` facade để tạo một JSON response, chúng ta có thể sử dụng
hàm `response` đơn giản hơn. Bời vì helper function có sẵn ở mọi nơi, bạn không cần import bất kì class nào để sử dụng chúng:
```PHP
use Illuminate\Support\Facades\Response;

Route::get('/users', function () {
    return Response::json([
        // ...
    ]);
});

Route::get('/users', function () {
    return response()->json([
        // ...
    ]);
});
```

### Facades Vs. Dependency Injection
Thông thường, sẽ không thể tạo một mock hoặc khai báo một static method class. Tuy nhiên, vì facades sử dụng phương thức
động để gọi method proxy tới các đổi tượng được thực thi trong service container,

TODO