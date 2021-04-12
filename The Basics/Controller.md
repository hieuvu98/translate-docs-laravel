<p align="center"><a href="https://laravel.com" target="_blank"><img src="https://raw.githubusercontent.com/laravel/art/master/logo-lockup/5%20SVG/2%20CMYK/1%20Full%20Color/laravel-logolockup-cmyk-red.svg" width="400"></a></p>

<p align="center">
<a href="https://travis-ci.org/laravel/framework"><img src="https://travis-ci.org/laravel/framework.svg" alt="Build Status"></a>
<a href="https://packagist.org/packages/laravel/framework"><img src="https://img.shields.io/packagist/dt/laravel/framework" alt="Total Downloads"></a>
<a href="https://packagist.org/packages/laravel/framework"><img src="https://img.shields.io/packagist/v/laravel/framework" alt="Latest Stable Version"></a>
<a href="https://packagist.org/packages/laravel/framework"><img src="https://img.shields.io/packagist/l/laravel/framework" alt="License"></a>
</p>

# Controller

## Giới thiệu
Thay vì định nghĩa tất cả các logic xử lý request trong tệp tin chứa các tuyến đường, bạn cũng có thể tổ chức hành vi này bằng cách sử dụng lớp controller. Controller có thể nhóm logic xử lý request liên quan đến nhau vào một lớp duy nhất.Ví dụ, một lớp `UserController` có thể xử lý tất cả các request liên quan đến user, bao gồm việc hiển thị, thêm, cập nhật và xóa user. Controller được lưu trữ ở thư mục `app/Http/Controllers`.

## Viết controller
### Cơ bản về controller
Hãy nhìn vào ví dụ về một controller cơ bản. Lưu ý rằng controller này được kế thừa từ controlelr cơ sở nằm trong Laravel:
```PHP
<?php

namespace App\Http\Controllers;

use App\Http\Controllers\Controller;
use App\Models\User;

class UserController extends Controller
{
    /**
     * Show the profile for a given user.
     *
     * @param  int  $id
     * @return \Illuminate\View\View
     */
    public function show($id)
    {
        return view('user.profile', [
            'user' => User::findOrFail($id)
        ]);
    }
}
```

Bạn có thể định nghĩa một tuyến đường trỏ đến method controller:
```PHP
use App\Http\Controllers\UserController;

Route::get('/user/{id}', [UserController::class, 'show']);
```

Khi một request đến khớp với URI của tuyến đường được chỉ định, method `show` trong controller `App\Http\Controllers\UserController` sẽ được gọi và tham số của tuyến sẽ được truyền đến method.

## Single Action Controllers
Nếu một hành động controller phức tạo, bạn có thể thấy thuận tiện khi dành toàn bộ controller để xử lý 1 hành động. Để làm được điều này, bạn cần khai báo `__invode()` bên trong controller:
```PHP
<?php

namespace App\Http\Controllers;

use App\Http\Controllers\Controller;
use App\Models\User;

class ProvisionServer extends Controller
{
    /**
     * Provision a new web server.
     *
     * @return \Illuminate\Http\Response
     */
    public function __invoke()
    {
        // ...
    }
}
```

Khi đăng ký các tuyến đường cho một hành động controller, bạn không cần chỉ định một method controller. Thay vào đó, bạn chỉ cần truyền tên controller vào trong tuyến:
```PHP
use App\Http\Controllers\ProvisionServer;

Route::post('/server', ProvisionServer::class);
```

## Controller Middleware
Middleware có thể được chỉ định cho controller của các tuyến:
```PHP
Route::get('profile', [UserController::class, 'show'])->middleware('auth');
```

Hoặc, bạn sẽ thấy tuận hiện hơn khi chỉ định middleware bên trong hàm khởi tạo `constructor` của controller. Sử dụng method `middleware` trong hàm tạo của controller, bạn có thể chỉ định middleware cho mỗi hành động của controller:
```PHP
class UserController extends Controller
{
    /**
     * Instantiate a new controller instance.
     *
     * @return void
     */
    public function __construct()
    {
        $this->middleware('auth');
        $this->middleware('log')->only('index');
        $this->middleware('subscribed')->except('store');
    }
}
```

Controller cũng cho phép bạn đăng ký middleware sử dụng closure. Điều này cung cấp một cách thuận tiện để định nghĩa một middleware cho một controller duy nhất mà không cần
xác định toàn bộ middleware của lớp:
```PHP
$this->middleware(function ($request, $next) {
    return $next($request);
});
```

## Resource Controllers
Định tuyến tài nguyên Laravel