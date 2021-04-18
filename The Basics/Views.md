# Views
# Introduce

Tất nhiên, có một điều không thực tế là trả về toàn bộ HTML một cách trực tiếp từ cá tuyến đường hoặc controller. Rất may
mắn, views cung cấp một cách thuận tiện để đặt tất cả các mã HTML vào trong các tệp riêng biệt. Views được lưu trữ ở
thư mục `resources/views`. Một view đơn giản trông giống như sau:
```PHP
<!-- View stored in resources/views/greeting.blade.php -->

<html>
    <body>
        <h1>Hello, {{ $name }}</h1>
    </body>
</html>
```

Bởi vì view được lưu trữ tại `resources/views/greeting.blade.php`, chúng ta có thể trả về nó bằng cách sử dụng helper
`view` như sau:
```PHP
Route::get('/', function () {
    return view('greeting', ['name' => 'James']);
});
```

## Creating & Rendering Views
Bạn có thể tạo một view bằng cách đặt một file có đuôi `.blade.php` trong thư mục `resources/views/`. Phần đuôi mở rộng
`.blade.php` thông báo cho framework rằng file chứa một [Blade template](https://laravel.com/docs/8.x/blade). Blade template
chứa HTML giống như các chỉ thị Blade cho phép bạn dễ dàng lặp lại các giá trị, tạo câu lệnh "if", lặp qua dữ liệu và nhiều
hơn.

View cũng có thể được trả về bằng cách sử dụng facade `View`:
```PHP
use Illuminate\Support\Facades\View;

return View::make('greeting', ['name' => 'James']);
```

Như bạn đã thấy, đối số đầu tiên được truyền vào helper `view` tương ứng với tên tệp trong thư mục `resources/views`.
Đối số thứ hai là một mảng dữ liệu cần dùng trong view. Trường hơp này chúng ta truyền vào biến `name`, nó sẽ được hiển
thị trong view khi sử dụng [Blade syntax](https://laravel.com/docs/8.x/blade).

### Nested View Directories
View cũng có thể được lồng trong các thư mục con của thư mục `recources/views`. Dấu `.` có thể được sử dụng để tham chiếu
đến các view nồng nhau. Ví dụ, nếu view được lưu tại `resources/views/admin/profile.blade.php`, bạn có thể trả về nó từ
các tuyến đường hoặc controller như sau:
``PHP
return view('admin.profile', $data);
``

### Creating The First Available View
### Determining If A View Exists
## Passing Data To Views
Như bạn đã thấy ở các ví dụ trước, ban có thể truyền một mảng dữ liệu vào views, và các biến này sẽ có sẵn ngoài view:
```PHP
return view('greetings', ['name' => 'Victoria']);
```

Khi truyền dữ liệu theo cách này, dữ liệu phải là dạng mảng với cặp key/value. Sau khi cung cấp dữ liệu cho view, bạn có
thể truy xuất đến từng giá trị trong view bằng các key của dữ liệu giống như `<?php echo $name; ?>`.

Có một cách khác để truyền dữ liệu vào view bằng cách sử dụng helper function `with`:
```PHP
return view('greeting')
            ->with('name', 'Victoria')
            ->with('occupation', 'Astronaut');
```

### Sharing Data With All Views
Thỉnh thoảng bạn cần chia sẻ dữ liệu với tất cả các view trong ứng dụng. Bạn có thể sử dụng method `share` của `View` facade.
Thông thường bạn nên gọi method `share` trong method `boot` của service provider. Bạn có thể tự do thêm chúng vào class
`App\Providers\AppServiceProvider` hoặc tạo một service provider riêng biệt để chứa nó:
```PHP
<?php

namespace App\Providers;

use Illuminate\Support\Facades\View;

class AppServiceProvider extends ServiceProvider
{
    /**
     * Register any application services.
     *
     * @return void
     */
    public function register()
    {
        //
    }

    /**
     * Bootstrap any application services.
     *
     * @return void
     */
    public function boot()
    {
        View::share('key', 'value');
    }
}
```

## View Composers

