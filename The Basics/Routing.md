<p align="center"><a href="https://laravel.com" target="_blank"><img src="https://raw.githubusercontent.com/laravel/art/master/logo-lockup/5%20SVG/2%20CMYK/1%20Full%20Color/laravel-logolockup-cmyk-red.svg" width="400"></a></p>

<p align="center">
<a href="https://travis-ci.org/laravel/framework"><img src="https://travis-ci.org/laravel/framework.svg" alt="Build Status"></a>
<a href="https://packagist.org/packages/laravel/framework"><img src="https://img.shields.io/packagist/dt/laravel/framework" alt="Total Downloads"></a>
<a href="https://packagist.org/packages/laravel/framework"><img src="https://img.shields.io/packagist/v/laravel/framework" alt="Latest Stable Version"></a>
<a href="https://packagist.org/packages/laravel/framework"><img src="https://img.shields.io/packagist/l/laravel/framework" alt="License"></a>
</p>

# Routing

## Basic routing
Các tuyến đường cơ bản trong Laravel chấp nhận một URI và một closure, cung cấp một phương thức đơn giản và dễ hiểu để định nghĩa các tuyến đường mà không cần đến các file cấu hình phức tạp:
```PHP
use Illuminate\Support\Facades\Route;

Route::get('/greeting', function () {
    return 'Hello World';
});
```

### The Default Route Files
Tất cả các tuyến đường của Laravel được định nghĩa trong thư mục `routes`. Những tệp này được tự động load bởi `App\Providers\RouteServiceProvider`. File `routes/web.php` định nghĩa các tuyến đường dành cho giao diện web của bạn. Các tuyến đường này được phân bố cho nhóm middleware `web`, nhóm này cung cấp các tính năng như trạng thái session và bảo vệ CSRF. Các tuyến ở `routes/api.php` là các stateless và được chỉ định cho nhóm middleware `api`.

Đối với phần lớn các ứng dụng, bạn sẽ bắt đầu định nghĩa các tuyến đường ở file `routes/web.php`. Các tuyến đường được định nghĩa  ở file `routes/web.php` có thể được truy cập bằng cách nhập các URL của tuyến đường đã đươc định nghĩa trong trình duyệt của bạn:
```PHP
use App\Http\Controllers\UserController;

Route::get('/user', [UserController::class, 'index']);
```
Các tuyến đường đã được định nghĩa trong `routes/api.php` được lồng trong một nhóm tuyến đường bởi `RouteServiceProvider`. Trong nhóm này, tiến tố URI `/api` tự động được áp dụng vì thế bạn không cần làm thủ công bằng cách áp dụng nó cho mỗi tuyến đường trong tệp. Bạn có thể sửa đổi tiền tố và các tùy chọn cho nhóm tuyến đường khác bằng cách sử đổi class `RouteServiceProvider`.

### Available Router Methods
Bộ định tuyến cho phép bạn đăng ký các tuyến đáp ứng với bất kỳ hành động HTTP:
```PHP
Route::get($uri, $callback);
Route::post($uri, $callback);
Route::put($uri, $callback);
Route::patch($uri, $callback);
Route::delete($uri, $callback);
Route::options($uri, $callback);
```

Thông thường bạn cần phải đăng ký một tuyến với nhiều hành động HTTP. Bạn cũng có thể sử dụng phương thức `match`. Hoặc bạn có thể đăng ký một tuyến mà đáp ứng với tất cả hành động HTTP bằng cách sử dụng phương thức `any`:
```PHP
Route::match(['get', 'post'], '/', function () {
    //
});

Route::any('/', function () {
    //
});
```

### Dependency Injection
Bạn có thể sử dụng bất kì type-hint phụ thuộc được yêu cầu của tuyến của bạn trong callback của tuyến. Các phụ thuộc được khai báo sẽ tự động được giải quyết và được đưa vào bên trong callback bởi `service container`. Ví dụ, bạn muốn sử dụng type-hint lớp `Illuminate\Http\Request` có trong HTTP hiện tại tự động được đưa vào bên trong callback của tuyến:
```PHP
use Illuminate\Http\Request;

Route::get('/users', function (Request $request) {
    // ...
});
```

### CSRF Protection
Bất kỳ HTML form trỏ đến các tuyến `POST`,`PUT`, `PATCH` hoặc `DELETE` được xác định trong file `web.php` phải bao gồm một trường CSRF token. Nếu không thì các request sẽ bị từ chối. Bạn có thể đọc thêm về  [CSRF protection](https://laravel.com/docs/8.x/csrf): 
```PHP
<form method="POST" action="/profile">
    @csrf
    ...
</form>
```

## Redirect Routes
Nếu bạn định nghĩa một tuyến để điều hướng đến URI khác, bạn có thể sử dụng phương thức `Route::redirect`. Đây là phương thức cung cấp một cú pháp rút gọn thuận tiện bởi vì bạn không cần phải định nghĩa một tuyến đường đầy đủ hoặc controller để thực hiện một điều hướng đơn giản:
```PHP
Route::redirect('/here', '/there');
```

 Mặc định, `Route::redirect` trả về một status code 302. Bạn có thể tủy chỉnh status code bằng cách sử dụng tùy chọn tham số thứ 3:
 ```PHP
Route::redirect('/here', '/there', 301);
 ```

Hoặc bạn có thể sử dụng phương thức `Route::permanentRedirect` để trả về một status code `301`:
```PHP
Route::permanentRedirect('/here', '/there');
```

## View Routes

Nếu tuyến đường của bạn chỉ trả về một [view](https://laravel.com/docs/8.x/views), bạn có thể sử dụng phương thức `Route::view`. Giống như phương thức `Route::redirect` nó cung cấp một cú pháp rút gọn đơn giản. Phương thức `view` chấp nhận một URI như dối số đầu tiên, và tên view như đối số thứ hai của nó. Ngoài ra, bạn có thể cung cấp một mảng dữ liệu để truyền vào view như một lựa chọn đối số thứ ba:
```PHP
Route::view('/welcome', 'welcome');

Route::view('/welcome', 'welcome', ['name' => 'Taylor']);
```

## Route Parameters
### Required Parameters
