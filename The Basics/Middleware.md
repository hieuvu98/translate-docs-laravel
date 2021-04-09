<p align="center"><a href="https://laravel.com" target="_blank"><img src="https://raw.githubusercontent.com/laravel/art/master/logo-lockup/5%20SVG/2%20CMYK/1%20Full%20Color/laravel-logolockup-cmyk-red.svg" width="400"></a></p>

<p align="center">
<a href="https://travis-ci.org/laravel/framework"><img src="https://travis-ci.org/laravel/framework.svg" alt="Build Status"></a>
<a href="https://packagist.org/packages/laravel/framework"><img src="https://img.shields.io/packagist/dt/laravel/framework" alt="Total Downloads"></a>
<a href="https://packagist.org/packages/laravel/framework"><img src="https://img.shields.io/packagist/v/laravel/framework" alt="Latest Stable Version"></a>
<a href="https://packagist.org/packages/laravel/framework"><img src="https://img.shields.io/packagist/l/laravel/framework" alt="License"></a>
</p>


# Middleware

## Introduction
Middleware cung cấp một cơ chế thuận tiện để kiểm tra và học các HTTP request đi vào ứng dụng của bạn. Ví dụ Laravel chứa một middleware để xác minh user của ứng dụng đã được xác thực. Nếu user chưa được xác thực, middleware sẽ điều hướng user tới màn hình đăng nhập của ứng dụng. Tuy nhiên, nếu người dùng đã xác thực, middleware sẽ cho phép request tiến sâu vào bên trong ứng dụng.

Thêm nữa middleware có thể được viết để tiến hành nhiều tác vụ khác nhau ngoài việc xác thực. Ví dụ, một middleware logging có thể ghi lại tất cả request đi tới ứng dụng. Tất cả các middleware được đặt tại thư mục `app/Http/Middleware`

## Define Middleware
Để tạo một middleware mới, sử dụng Artisan command `make:middleware`:
```PHP
php artisan make:middleware EnsureTokenIsValid
```
Đây là command thay thế cho việc tạo một class `EnsureTokenIsValid ` mới trong thư mục `app/Http/Middleware`. Trong middleware này, chúng ta có thể chỉ cho phép truy cập vào route nếu giá trị `token` khớp mới một giá trị được chỉ định. Nếu không thì chúng ta sẽ điều hướng user quay trở lại `home` URI:
```PHP
<?php

namespace App\Http\Middleware;

use Closure;

class EnsureTokenIsValid
{
    /**
     * Handle an incoming request.
     *
     * @param  \Illuminate\Http\Request  $request
     * @param  \Closure  $next
     * @return mixed
     */
    public function handle($request, Closure $next)
    {
        if ($request->input('token') !== 'my-secret-token') {
            return redirect('home');
        }

        return $next($request);
    }
}
```
Như bạn có thể nhìn thấy, nếu `token` không khớp với `secret-token` middleware sẽ trả về  một HTTP điều hướng tới client, ngược lại request sẽ được chuyển tiếp vào bên trong ứng dụng. Để đi sâu vào trong ứng dụng, bạn cần gọi `$next` với `$request`.

### Middleware & Responses
Tất nhiên, một middleware có thể thực hiện các tác vụ trước hoặc sau khi request đi sâu vào bên trong ứng dụng. Ví dụ, middleware sẽ thực hiện một vài tác vụ trước khi request được xử lý:
```PHP
namespace App\Http\Middleware;

use Closure;

class BeforeMiddleware
{
    public function handle($request, Closure $next)
    {
        // Perform action

        return $next($request);
    }
}
```
Tuy nhiên, middleware sẽ tiến hành các tác vụ sau khi request đã được xử lý bởi ứng dụng:
```PHP
<?php

namespace App\Http\Middleware;

use Closure;

class AfterMiddleware
{
    public function handle($request, Closure $next)
    {
        $response = $next($request);

        // Perform action

        return $response;
    }
}
```
## Registering Middleware
### Global Middleware
Nếu bạn muốn middleware thực thi trong mỗi khi HTTP request tới ứng dụng, thêm tên class của middleware đó vào thuộc tính `$middleware` của class `app/Http/Kernel.php`

### Assigning Middleware To Routes
Nếu bạn muốn chỉ định middleware cho mỗi route riêng, trước tiên bạn nên gán key cho middleware trong ứng dụng ở file `app/Http/Kernel.php`. Theo mặc định, thuộc tính `$routeMiddleware` của lớp này chứa các mục cho middleware đi kèm với Laravel.
Bạn cần thêm middleware của mình vào danh sách và chỉ định key do bạn chọn:
```PHP
// Within App\Http\Kernel class...

protected $routeMiddleware = [
    'auth' => \App\Http\Middleware\Authenticate::class,
    'auth.basic' => \Illuminate\Auth\Middleware\AuthenticateWithBasicAuth::class,
    'bindings' => \Illuminate\Routing\Middleware\SubstituteBindings::class,
    'cache.headers' => \Illuminate\Http\Middleware\SetCacheHeaders::class,
    'can' => \Illuminate\Auth\Middleware\Authorize::class,
    'guest' => \App\Http\Middleware\RedirectIfAuthenticated::class,
    'signed' => \Illuminate\Routing\Middleware\ValidateSignature::class,
    'throttle' => \Illuminate\Routing\Middleware\ThrottleRequests::class,
    'verified' => \Illuminate\Auth\Middleware\EnsureEmailIsVerified::class,
];
```

Khi middleware đã được định nghĩa ở HTTP kernel, bạn có thể sử dụng bạn có thể sử dụng phương thức `middleware` để chỉ định cho một route:
```PHP
Route::get('/profile', function () {
    //
})->middleware('auth');
```

Bạn có thể chỉ định nhiều middleware cho route bằng cách truyền một mảng tên middleware và phương thức `middleware`:
```PHP
Route::get('/', function () {
    //
})->middleware(['first', 'second']);
```

Khi chỉ đỉ middleware, bạn cũng có thể truyền tên đầy đủ của middleware:
```PHP
use App\Http\Middleware\EnsureTokenIsValid;

Route::get('/profile', function () {
    //
})->middleware(EnsureTokenIsValid::class);
```

Khi chỉ định middleware cho một group route, đôi khi bạn có thể cần ngăn chặn middleware áp dụng cho một route riêng lẻ trong nhóm. Bạn có thể làm điều này bằng cách sử dụng phương thức `withoutMiddleware`:
```PHP
Route::middleware([EnsureTokenIsValid::class])->group(function () {
    Route::get('/', function () {
        //
    });

    Route::get('/profile', function () {
        //
    })->withoutMiddleware([EnsureTokenIsValid::class]);
});
```

Phương thức `withoutMiddleware` chỉ có thể loại bỏ cho route middleware, không áp dụng cho global middleware.

### Middleware Groups
Đôi khi bạn muốn nhóm một số middleware dưới một key duy nhất để gán chúng cho route một cách dễ dàng hơn. Bạn có thể thực hiện bằng cách sử dụng thuộc tính `middlewareGroups` trong HTTP kernel.

Ngoài ra, Laravel có 2 group middleware `web` và `api`, chứa middleware phổ biến có thể bạn muốn áp dụng vào web hoặc API route của bạn. Nhớ lại, các nhóm middleware được tự dụng áp dụng vào các tuyến bới service provider `App\Providers\RouteServiceProvider` trong các tệp định tuyến `web` và `api`:
```PHP
/**
 * The application's route middleware groups.
 *
 * @var array
 */
protected $middlewareGroups = [
    'web' => [
        \App\Http\Middleware\EncryptCookies::class,
        \Illuminate\Cookie\Middleware\AddQueuedCookiesToResponse::class,
        \Illuminate\Session\Middleware\StartSession::class,
        // \Illuminate\Session\Middleware\AuthenticateSession::class,
        \Illuminate\View\Middleware\ShareErrorsFromSession::class,
        \App\Http\Middleware\VerifyCsrfToken::class,
        \Illuminate\Routing\Middleware\SubstituteBindings::class,
    ],

    'api' => [
        'throttle:api',
        \Illuminate\Routing\Middleware\SubstituteBindings::class,
    ],
];
```

Nhóm middleware có thể được chỉ định cho các route và hành động của controller bằng cách sử dụng cú phát tương tự với middleware riêng lẻ. Nhắc lại, nhóm middleware giúp thuận tiện hơn cho việc chỉ định nhiều middleware cho một route cùng lúc:
```PHP
Route::get('/', function () {
    //
})->middleware('web');

Route::middleware(['web'])->group(function () {
    //
});
```

### Sorting Middleware