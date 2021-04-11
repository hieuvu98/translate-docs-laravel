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
Đôi khi bạn sẽ cần nắm bắt các phân mảnh của URI trong tuyến của bạn. Ví dụ, bạn cần lấy USER ID trên URL. Bạn có thể làm việc này bằng cách định nghĩa các tham số của tuyến:
```PHP
Route::get('/user/{id}', function ($id) {
    return 'User '.$id;
});
```

Bạn có thể định nghĩa nhiều tham số cho tuyến theo yêu cầu của bạn:
```PHP
Route::get('/posts/{post}/comments/{comment}', function ($postId, $commentId) {
    //
});
```

Các tham số của tuyến đường luôn luôn được bọc trong dấu `{}` và phải bao gồm các ký tự chữ cái. Dấu gạch dưới (`_`) cũng có thể được chấp nhận trong tên tham số của tuyến.
Tham số của tuyến sẽ được đưa vào trong callback của tuyến/controller dựa trên thứ tự - 

### Parameters & Dependency Injection
Nếu tuyến đường của bạn có các phụ thuộc mà bạn muốn Service Container tự động tiêm vào bên trong callback của tuyến đường, bạn có thể liệt kê các tham số sau các phụ thuộc:
```PHP
use Illuminate\Http\Request;

Route::get('/user/{id}', function (Request $request, $id) {
    return 'User '.$id;
});
```

## Optional Parameters
Đôi khi bạn cần xác định các tham số của tuyến đường không phải lúc nào cũng có trên URI. Bạn có thể làm bằng cách đặt dấu `?` sau tên tham số. Đảm bảo cung cấp giá trị mặc định
cho các biến tương ứng:
```PHP
Route::get('/user/{name?}', function ($name = null) {
    return $name;
});

Route::get('/user/{name?}', function ($name = 'John') {
    return $name;
});
```

## Regular Expression Constraints
Bạn có thể ràng buộc định dạng tham số của tuyến bằng cách sử dụng phương thức `where` trên một tuyến. Phương thức `where` chấp nhận tên của tham số và một biếu thức chính quy
định nghĩa để ràng buộc các tham số:
```PHP
Route::get('/user/{name}', function ($name) {
    //
})->where('name', '[A-Za-z]+');

Route::get('/user/{id}', function ($id) {
    //
})->where('id', '[0-9]+');

Route::get('/user/{id}/{name}', function ($id, $name) {
    //
})->where(['id' => '[0-9]+', 'name' => '[a-z]+']);
```

Thông thường, một số biểu thức chính quy có các phương thức hỗ trợ cho phép bạn thêm các ràng buộc một cách nhanh chóng vào tuyến đường:
```PHP
Route::get('/user/{id}/{name}', function ($id, $name) {
    //
})->whereNumber('id')->whereAlpha('name');

Route::get('/user/{name}', function ($name) {
    //
})->whereAlphaNumeric('name');

Route::get('/user/{id}', function ($id) {
    //
})->whereUuid('id');
```

 Nếu các request đến không khớp với các rảng buộc thì một phản hồi HTTP 404 sẽ được trả về.

 ### Global Constraints
 Nếu bạn muốn các tham số của tuyến luôn luôn bị ràng buộc một biểu thức chính quy cho trước, bạn có thể sử dụng phương thức `pattern` trong phương thức `boot`
 của `App\Providers\RouteServiceProvide`:
 ```PHP
 /**
 * Define your route model bindings, pattern filters, etc.
 *
 * @return void
 */
public function boot()
{
    Route::pattern('id', '[0-9]+');
}
```

Khi các mẫu đã được định nghĩa, nó sẽ tự động được áp dụng cho tất cả các tuyến đường sử dụng tên tham số đó:
```PHP
Route::get('/user/{id}', function ($id) {
    // Only executed if {id} is numeric...
});
```

### Encoded Forward Slashes
Thành phần định tuyến Laravel cho phép tất cả các ký tự ngoại từ `/` hiện diện trong giá trị tham số tuyến đường. Bạn phải cho phép một cách rõ ràng `/` trở thành một phần giữ chỗ 
bằng cách sử dụng `where` với một biểu thức điều kiện:
```PHP
Route::get('/search/{search}', function ($search) {
    return $search;
})->where('search', '.*');
```

## Named Routes
Đặt tên cho tuyến đường cho phép tạo URL rất thuận tiện hoặc điều hướng đến một tuyến cụ thể. Bạn có thể xác định tên của tuyến đường bằng cách sử dụng phương thức `name` khi tạo tuyến đường:
```PHP
Route::get('/user/profile', function () {
    //
})->name('profile');
```

Bạn có thể chỉ định tên tuyến cho các hành động của controller:
```PHP
Route::get(
    '/user/profile',
    [UserProfileController::class, 'show']
)->name('profile');
```

### Generating URLs To Named Routes
Khi bạn chỉ định tên cho một tuyến đường nhất định, bạn có thể sử dụng tên của tuyến đường khi tạo một URL hoặc điều hướng thông qua Laravel:
```PHP
// Generating URLs...
$url = route('profile');

// Generating Redirects...
return redirect()->route('profile');
```

Nếu tuyến được đặt tên xác định các tham số, bạn có thể truyền tham số như đối số thứ hay của hàm `route`. Các thông số đã cho sẽ tự động được chèn vào URL đã được tạo ở đúng vị trí của chúng:
```PHP
Route::get('/user/{id}/profile', function ($id) {
    //
})->name('profile');

$url = route('profile', ['id' => 1]);
```

Nếu bạn truyền các tham số bổ sung trong mảng, cặp key/value sẽ tự động được thêm vào URL đã được tạo:
```PHP
Route::get('/user/{id}/profile', function ($id) {
    //
})->name('profile');

$url = route('profile', ['id' => 1, 'photos' => 'yes']);

// /user/1/profile?photos=yes
```

### Inspecting The Current Route

Nếu bạn muốn xác định xem liệu request hiện tại được chuyển đến một tuyến đường đã được đặt tên rồi hay không, ban có thể sử dụng phương thức `named`:
```PHP
/**
 * Handle an incoming request.
 *
 * @param  \Illuminate\Http\Request  $request
 * @param  \Closure  $next
 * @return mixed
 */
public function handle($request, Closure $next)
{
    if ($request->route()->named('profile')) {
        //
    }

    return $next($request);
}
```

## Route Groups
Nhóm tuyến đường cho phép bạn chia sẻ các thuộc tính như middleware trên một lượng lớn các tuyến mà không cần định nghia cho từng tuyến đường riêng lẻ.
Các nhóm lồng nhau cố gắng hợp nhất thuộc tính với nhóm mẹ một cách thông minh nhất. Middleware và điều kiện `where` được gộp lại trong khi tên và tiền tô được thêm vào.
Dấu phân cách không gian tên và dấu `/` tự động được thêm vào một cách thích hợp.

### Middleware
Để gán middleware cho tất cả tuyến đường nằm trong nhóm, bạn cần sử dụng phương thức `middlware` trước khi khai báo nhóm. Middleware sẽ được thực thi theo thứ tự liệt kê trong mảng:  
```PHP
Route::middleware(['first', 'second'])->group(function () {
    Route::get('/', function () {
        // Uses first & second middleware...
    });

    Route::get('/user/profile', function () {
        // Uses first & second middleware...
    });
});
```

### Subdomain Routing
Nhóm các tuyến đường cũng có thể được sử dụng để xử lý định tuyến subdomain. Các Subdomain có thể được chỉ định các tham số giống như tuyến URI, cho phép bạn lấy một phần
của subdomain để sử dụng trong tuyến đường hoặc controller. Subdomain có thể được chỉ định bằng cách gọi phương thức `domain` trước khi định nghĩa một tuyến đường:
```PHP
Route::domain('{account}.example.com')->group(function () {
    Route::get('user/{id}', function ($account, $id) {
        //
    });
});
```

### Route Prefixes
Phương thức `prefix` có thể được sử dụng để đặt tiền tố cho mỗi tuyến đường trong nhóm với một URI nhất định:
```PHP
Route::prefix('admin')->group(function () {
    Route::get('/users', function () {
        // Matches The "/admin/users" URL
    });
});
```
### Route Name Prefixes

