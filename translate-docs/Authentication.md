<p align="center"><a href="https://laravel.com" target="_blank"><img src="https://raw.githubusercontent.com/laravel/art/master/logo-lockup/5%20SVG/2%20CMYK/1%20Full%20Color/laravel-logolockup-cmyk-red.svg" width="400"></a></p>

<p align="center">
<a href="https://travis-ci.org/laravel/framework"><img src="https://travis-ci.org/laravel/framework.svg" alt="Build Status"></a>
<a href="https://packagist.org/packages/laravel/framework"><img src="https://img.shields.io/packagist/dt/laravel/framework" alt="Total Downloads"></a>
<a href="https://packagist.org/packages/laravel/framework"><img src="https://img.shields.io/packagist/v/laravel/framework" alt="Latest Stable Version"></a>
<a href="https://packagist.org/packages/laravel/framework"><img src="https://img.shields.io/packagist/l/laravel/framework" alt="License"></a>
</p>

# Authentication

## Introduction

Nhiều ứng dụng web cung cấp một cách để người dùng của họ xác thực bằng ứng dụng và "đăng nhập".
Việc triển khai tính năng này trong các ứng dụng web có thể phức tạp và tiềm ẩn nhiều rủi ro.
Vì lý do này Laravel cố gắng cung cấp cho bạn các công cụ cần thiết để xác thực một cách nhanh chóng, 
an toàn và dễ sử dụng.

Về cốt lõi, cơ sở của Laravel's authentication được tạo thành bơi `guard` và `provides`. `Guard` xác định
cách người dùng được xác thực cho mỗi yêu cầu. Laravel cung cấp một `session` guard cái mà duy trì trạng
thái bằng cách sử dụng `session storage` và `cookies`.

Provides xác định cách người dùng được truy xuất từ bộ nhớ liên tục của bạn. Laravel hỗ trợ truy xuất người dùng bằng 
Eloquent và query builder. Tuy nhiên, bạn có thể tự do xác định bổ sung provides nếu cần cho ứng dụng của mình.

Tệp cấu hình authentication nằm ở  `config/auth.php`. File chưa một số tài liệu để điều chỉnh hành vi của Laravel's 
authentication services.

## Database Considerations

Theo mặc định, Laravel bao gồm `App\Models\User Eloquent model` trong thư mục `app/Models`. Model này có thể được sử dụng
với Eloquent Authenticate mặc định. Nếu ứng dụng cửa bạn không sử dụng Model, bạn có thể sử dụng query builder để tạo truy 
vấn Laravel.

Khi xây dựng lược đồ cơ sở dữ liệu cho model`App\Models\User`, đảm bảo cột password có độ dài ít nhất 60 ký tự.
Tất nhiên bảng `users` đã tạo ra một cột vượt quá độ dài này,

Ngoài ra bạn nên xác minh rằng bảng `users` cột bao gồm một chuỗi `remember_token`, có thể null. Cột này sẽ được sử dụng
để lưu trữ mã thông báo cho người dùng chọn tuy chọn `remember_token` khi đăng nhập vào ứng dụng của bạn.

## Retrieving The Authenticated User

Trong khi xử lý một request đến, bạn có thể truy cập xác thực người dùng thông qua phương thức `user` của `Auth Facades`:

```PHP
use Illuminate\Support\Facades\Auth;

// Retrieve the currently authenticated user...
$user = Auth::user();

// Retrieve the currently authenticated user's ID...
$id = Auth::id();
```

Ngoài ra, khi người dùng đã xác thực, bạn có thể truy cập người dùng xác thực thông qua `Illuminate\Http\Request`. Nhớ lại,
các lớp gợi ý kiểu dữ liệu sẽ tự động được đưa vào các phương thức của controller. Với đối tượng `Illuminate\Http\Request`,
bạn có thể thuận tiện truy cập vào người dùng đã xác thực từ bất kỳ phương thức của controller trong ứng dụng của bạn
thông qua phương thức `user` của request:

```PHP
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;

class FlightController extends Controller
{
    /**
     * Update the flight information for an existing flight.
     *
     * @param  \Illuminate\Http\Request  $request
     * @return \Illuminate\Http\Response
     */
    public function update(Request $request)
    {
        // $request->user()
    }
}
```
### Determining If The Current User Is Authenticated

Để xác định xem người dùng thực hiện request HTTP đến đã được xác thực chưa? Bạn có thể sử dụng phương thức `check` của
`Auth Facades`. Phương thức sẽ trả về `true` nếu user đã xác thực:

```PHP
use Illuminate\Support\Facades\Auth;

if (Auth::check()) {
    // The user is logged in...
}
```
Mặc dù có thể xác định người dùng có được xác thực hay không bằng phương thức `check` nhưng bạn thường sẽ sử dụng middleware
để xác minh người dụng đã xác thực hay chưa trước khi cho phép người dùng truy cập vào `route` hoặc `controller`.

## Protecting Routes

Route middleware có thể được sử dụng để chỉ cho phép người dùng đã xác thực truy cập và một route. Laravel cung cấp một 
`auth` middleware, nó tham chiếu đến lớp `Illuminate\Auth\Middleware\Authenticate`. Vì middleware đã được đăng ký trong
HTTP kernel của ứng dụng, tất cả những gì của bạn cần gắn middleware và route đã định nghĩa:

```PHP
Route::get('/flights', function () {
    // Only authenticated users may access this route...
})->middleware('auth');
```

### Redirecting Unauthenticated Users

Khi middleware `auth` phát hiện user chưa được xác thực, nó sẽ chuyển hướng người dùng đến tuyến `login`.
Bạn có thể sửa đổi hành vì này bằng cách cập nhập hàm `redirectTo ` ở file `app/Http/Middleware/Authenticate.php`:

```PHP
/**
 * Get the path the user should be redirected to.
 *
 * @param  \Illuminate\Http\Request  $request
 * @return string
 */
protected function redirectTo($request)
{
    return route('login');
}
```

### Specifying A Guard

Khi đính kèm `auth` middleware vào một route, bạn cũng có thể chỉ định `guard` nào sử dụng để xác thực người dùng.
Guard được chỉ định phải tương ứng với một trong các keys có trong mảng `guards` trong tệp cấu hình `auth.php `:

```PHP
Route::get('/flights', function () {
    // Only authenticated users may access this route...
})->middleware('auth:admin');
```

## Manually Authenticating Users

Chúng ta sẽ truy cập vào các Laravel authentication services thông qua `Auth` facade, vì vậy chúng ra cần thêm `Auth` 
facade ở đầu lớp. Tiếp theo hãy kiểm tra bằng phương thức `attemp`. Phương thức `attemp` thưởng được sử dụng để xử lý xác 
thực từ form đăng nhập. Nếu xác thực thành công, bạn sẽ tạo lại một session của người dụng để ngăn chặn `session fixation`:

```PHP
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;
use Illuminate\Support\Facades\Auth;

class LoginController extends Controller
{
    /**
     * Handle an authentication attempt.
     *
     * @param  \Illuminate\Http\Request $request
     * @return \Illuminate\Http\Response
     */
    public function authenticate(Request $request)
    {
        $credentials = $request->only('email', 'password');

        if (Auth::attempt($credentials)) {
            $request->session()->regenerate();

            return redirect()->intended('dashboard');
        }

        return back()->withErrors([
            'email' => 'The provided credentials do not match our records.',
        ]);
    }
}
```
Phương thức `attempt` chấp nhận một mảng các cặp key/value như là tham số đầu tiên của nó. Các giá trị trong mảng sẽ được
sử dụng để tìm user trong cơ sở dữ liệu. Bởi vì theo ví dụ đã nêu ở trên, user sẽ được truy xuất bời giá trị của cột email.
Nếu tìm thấy user, mật khẩu đã hash trong cơ sở dữ liệu sẽ được so sánh với giá trị `password` được truyền cho phương thức
thông qua mảng. Bạn không nên mã hóa mật khẩu trước khi gửi request, bơi vì framework sẽ tự động mã hóa giá trị trước khi
so sánh nó với mật khẩu đã được mã hóa trong database. Một session đã xác thực sẽ được bắt đầu cho người dùng nếu hai mật
khẩu mã hóa khớp nhau.

Phương thức intend được cung cấp bởi trình điều hướng Laravel sẽ điều hướng user đến URL mà họ cố gắng truy cập mà trước
 đó đã bị chặn lại bởi middleware. Một URI dự phòng có thể được cung cấp cho phương thức này trong trường hợp điểm đến
dự định không có sẵn.

### Specifying Additional Conditions

Nếu muốn, ban có thể thêm điều kiện vào câu lệnh truy vấn ngoài 2 giá trị email và password. Để thực hiện nó, chúng ta có
thể chỉ thêm các điều kiện truy vấn vào mảng rồi truyền vào phương thức `attempt`:

```PHP
if (Auth::attempt(['email' => $email, 'password' => $password, 'active' => 1])) {
    // Authentication was successful...
}
```

Trong những ví dụ này, email không phải là một tùy chọn bắt buộc, nó chỉ được dùng làm ví dụ. Bạn nên sử dụng bất kỳ tên
cột nào tương ứng với "tên người dùng" trong bảng cơ sở dữ liệu của bạn.

## Remembering Users
