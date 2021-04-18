# Controller

## Introduction
Thay vì định nghĩa tất cả các logic xử lý request trong tệp tin chứa các tuyến đường, bạn cũng có thể tổ chức hành vi này 
bằng cách sử dụng lớp controller. Controller có thể nhóm logic xử lý request liên quan đến nhau vào một lớp duy nhất.Ví dụ, một lớp `UserController` có thể xử lý tất cả các request liên quan đến user, bao gồm việc hiển thị, thêm, cập nhật và xóa user. Controller được lưu trữ ở thư mục `app/Http/Controllers`.

## Writing Controllers
### Basic Controllers
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

Khi một request đến khớp với URI của tuyến đường được chỉ định, method `show` trong controller `App\Http\Controllers\UserController` 
sẽ được gọi và tham số của tuyến sẽ được truyền đến method.

## Single Action Controllers
Nếu một hành động controller phức tạo, bạn có thể thấy thuận tiện khi dành toàn bộ controller để xử lý 1 hành động. Để 
làm được điều này, bạn cần khai báo `__invoke()` bên trong controller:
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

Khi đăng ký các tuyến đường cho một hành động controller, bạn không cần chỉ định một method controller. Thay vào đó, bạn
chỉ cần truyền tên controller vào trong tuyến:
```PHP
use App\Http\Controllers\ProvisionServer;

Route::post('/server', ProvisionServer::class);
```

## Controller Middleware
Middleware có thể được chỉ định cho controller của các tuyến:
```PHP
Route::get('profile', [UserController::class, 'show'])->middleware('auth');
```

Hoặc, bạn sẽ thấy tuận hiện hơn khi chỉ định middleware bên trong hàm khởi tạo `constructor` của controller. Sử dụng method
`middleware` trong hàm tạo của controller, bạn có thể chỉ định middleware cho mỗi hành động của controller:
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

Controller cũng cho phép bạn đăng ký middleware sử dụng closure. Điều này cung cấp một cách thuận tiện để định nghĩa một
middleware cho một controller duy nhất mà không cần
xác định toàn bộ middleware của lớp:
```PHP
$this->middleware(function ($request, $next) {
    return $next($request);
});
```

## Resource Controllers
Định tuyến tài nguyên Laravel gán các tuyến điển hình như: tạo, đọc, cập nhật và xóa (CRUD) tới một controller với một 
dòng code duy nhất. Để bắt đầu, chúng ta cần sử dụng artisan conmmand  `make:controller` với lựa chọn `--resource` sẽ tạo controller nhanh nhẩt để xử lý các hành động này:
```PHP
php artisan make:controller PhotoController --resource
```

Command sẽ tạo một controller tại `app/Http/Controllers/PhotoController.php`. Controller sẽ chứa các method cho mỗi hoat
động tài nguyên có sẵn. Tiếp theo, bạn có thể đăng ký một tuyến tài nguyên trỏ đến controller:
```PHP
use App\Http\Controllers\PhotoController;

Route::resource('photos', PhotoController::class);
```

Khai báo một tuyến đường đơn này tạo ra các tuyến đường để xử lý các hành động khác nhau trên tài nguyên. Controlelr được
tạo ra sẽ có sẵn các method cho mỗi hành động. Nhớ lại, bạn luôn luôn có thể xem tổng quan nhanh về các tuyến đường trong ứng dụng của bạn bằng cách chạy Artisan command `route:list`.

Bạn cũng có thể đăng ký nhiều resource controller cùng một lúc bằng cách truyền một mảng vào method `resource`:
```PHP
Route::resources([
    'photos' => PhotoController::class,
    'posts' => PostController::class,
]);
```

#### Actions Handled By Resource Controller
| Verb      |  URI                      | Action    | Route Name    |
| ----------|:-------------------------:|----------:|:--------------|
| GET       | `/photos`                 | index     | photos.index  |
| GET       | `/photos/create`          | create    | photos.create |
| POST      | `/photos`                 | store     | photos.store  |
| GET       | `/photos/{photo}`         | store     | photos.show   |
| GET       | `/photos/{photo}/edit`    | edit      | photos.edit   |
| PUT/PATCH | `/photos/{photo}`         | update    | photos.update |
| DELETE    | `/photos/{photo}`         | destroy   | photos.destroy|

#### Customizing Missing Model Behavior
Thông thường, một phản hồi HTTP 404 sẽ được tạo ra nếu một model tài nguyên bị ràng buộc ngầm không tìm thấy. Tuy nhiên,
bạn có thể tùy chỉnh hành vi này bằng cách gọi method `missing` khi định nghĩa một tuyến tài nguyên. Method `missing` chấp
nhận một closure, nó sẽ được gọi nếu một model ràng buộc ngầm không được tìm thấy ở bất kỳ tuyến tài nguyên nào:
```PHP
use App\Http\Controllers\PhotoController;
use Illuminate\Http\Request;
use Illuminate\Support\Facades\Redirect;

Route::resource('photos', PhotoController::class)
        ->missing(function (Request $request) {
            return Redirect::route('photos.index');
        });
```

#### Specifying The Resource Model
Nếu bạn đang sử dụng [route model binding ](https://laravel.com/docs/8.x/routing#route-model-binding) và muốn các phương 
thức của controller gợi ý một model instance, bạn có thể sử dụng lựa chọn `--model` khi tạo controller:
```PHP
php artisan make:controller PhotoController --resource --model=Photo
```

### Partial Resource Routes
Khi khai báo một tuyến tài nguyên, bạn có thể chỉ định một tập hợp con của các hành động của controller sẽ xử lý thay vì
sử dụng trọn bộ hành động mặc định:
```PHP
use App\Http\Controllers\PhotoController;

Route::resource('photos', PhotoController::class)->only([
    'index', 'show'
]);

Route::resource('photos', PhotoController::class)->except([
    'create', 'store', 'update', 'destroy'
]);
```

#### API Resource Routes
Khi khai báo tuyến tài nguyên nó sẽ được sử dụng bởi API, thông thường bạn muốn loại trừ các tuyến mà nó đưa đến HTML
template như `create` và `edit`. Để thuận tiện hơn, bạn có thể sử dụng method `apiResource` để tự động loại từ hai tuyến
đường này:
```PHP
use App\Http\Controllers\PhotoController;

Route::apiResource('photos', PhotoController::class);
```

Bạn có thể đăng ký nhiều tài nguyên API controller cùng lúc bằng cách truyền một mảng vào method `apiResource`:
```PHP
use App\Http\Controllers\PhotoController;
use App\Http\Controllers\PostController;

Route::apiResources([
    'photos' => PhotoController::class,
    'posts' => PostController::class,
]);
```

Để tạo một tài nguyên API controller nhanh chóng mà không có method `create` và `edit`, sử dụng lựa chọn `--api` khi thực
hiện command `make:controller`:
```PHP
php artisan make:controller PhotoController --api
```

### Nested Resources
Đôi khi bạn cần khai báo các tuyến đường đến một tài nguyên lồng nhau. Ví dụ, một tài nguyên photo có thể có nhiều comment,
nó được đính kèm vào photo. Để lồng các tài nguyên controller, bạn hãy sử dụng ký hiệu `.` khi khai báo tuyến:
```PHP
use App\Http\Controllers\PhotoCommentController;

Route::resource('photos.comments', PhotoCommentController::class);
```

Tuyến đường này sẽ đăng ký một tài nguyên lồng ghép, nó có thể được truy cập với URI như sau:
```PHP
/photos/{photo}/comments/{comment}
```

#### Scoping Nested Resources
//TODO Scoping Nested Resources
#### Shallow Nesting
//TODO Shallow Nesting

### Naming Resource Routes 
Theo mặc định, tất cả các hành động controller đều có tên tuyến; tuy nhiên, bạn có thể ghi đè các tên này bằng cách truyền
một mảng `names` với các tên mong muốn:
```PHP
use App\Http\Controllers\PhotoController;

Route::resource('photos', PhotoController::class)->names([
    'create' => 'photos.build'
]);
```

### Naming Resource Route Parameters
Theo mặc định, `Route::resource` sẽ tạo các tham số tuyến đường cho các tuyến tài nguyên của bạn dựa trên tên tài nguyên.
Bạn có thể dễ dàng ghi đè lên trên mỗi tài nguyên cơ sở bằng cách sử dụng phương thức `parameters`. Một mảng được truyền
vào trong method `parameters` nên là một mảng liên kết tên tài nguyên và tên tham số:
```PHP
use App\Http\Controllers\AdminUserController;

Route::resource('users', AdminUserController::class)->parameters([
    'users' => 'admin_user'
]);
```

Ví dụ trên sẽ tạo ra những URI sau cho định tuyến `show` của tài nguyên:
```PHP
/users/{admin_user}
```

### Scoping Resource Routes

Tính năng ràng buộc ngầm model của Laravel có thể tự động được ràng buộc lồng nhau sao cho model con đã giải quyết được xác
nhận là thuộc về một model cha. Bằng cách sử dụng method `scoped` khi định nghĩa các tài nguuyeen lồng nhau bạn có thể kích
hoạt xác định phạm vi tự động, xác định tài nguyên con nào nên được truy xuất bằng cách:
```PHP
Route::resource('photos.comments', PhotoCommentController::class)->scoped([
    'comment' => 'slug',
]);
```

Tuyến đường này sẽ đăng ký một tài nguyên lồng nhau có phạm vi có thể được truy cập bằng URI sau:
```PHP
/photos/{photo}/comments/{comment:slug}
```

### Localizing Resource URIs

### Supplementing Resource Controllers
Nếu bạn cần thêm các tuyến đường bổ sung và một controller tài nguyên ngoài bộ các tuyến mặc định, bạn nên xác định tuyến đường 
đó trước khi gọi đến method `Route::resource`, nếu không các tuyến được xác định với method `resource` có thể vô tình ưu tiên hơn
các tuyến đường bổ sung của bạn:
```PHP
use App\Http\Controller\PhotoController;

Route::get('/photos/popular', [PhotoController::class, 'popular']);
Route::resource('photos', PhotoController::class);
```

## Dependency Injection & Controllers
#### Constructor Injection
Phần service container của Laravel được dùng để xử lý tất cả các controller của Laravel. Kết quả là, bạn có thể "type-hint" bất 
cứ thành phần phụ thuộc nào mà controller của bạn cần vào trong constructor của controller. Các thành phần phụ thuộc sẽ được tự 
động xử lý và được thêm vào trong "instance" của controller:
```PHP
<?php

namespace App\Http\Controllers;

use App\Repositories\UserRepository;

class UserController extends Controller
{
    /**
     * The user repository instance.
     */
    protected $users;

    /**
     * Create a new controller instance.
     *
     * @param  \App\Repositories\UserRepository  $users
     * @return void
     */
    public function __construct(UserRepository $users)
    {
        $this->users = $users;
    }
}
```

### Method Injection
Ngoài constructor injection, bạn cũng có thể "type-hint" các thành phần phụ thuộc trong các method của controller. 
Ví dụ, hãy "type-hint" instance của `Illuminate\Http\Request` vào một trong những method của ta:
```PHP
<?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;

class UserController extends Controller
{
    /**
     * Store a new user.
     *
     * @param  \Illuminate\Http\Request  $request
     * @return \Illuminate\Http\Response
     */
    public function store(Request $request)
    {
        $name = $request->name;

        //
    }
}
```

Nếu như method của controller của bạn cũng chờ đợi đầu vào từ tham sổ của định tuyến, thì đơn giản là liệt kê các đối số của 
định tuyến vào phía sau các thành phần phụ thuộc khác. Ví dụ, nếu định tuyến của bạn được định nghĩa như sau:
```PHP
use App\Http\Controllers\UserController;

Route::put('/user/{id}', [UserController::class, 'update']);
```

Bạn vẫn có thể "type-hint" `Illuminate\Http\Request` và truy cập vào tham số định tuyến của bạn id bằng cách định nghĩa method 
controller của bạn như sau:
```PHP
?php

namespace App\Http\Controllers;

use Illuminate\Http\Request;

class UserController extends Controller
{
    /**
     * Update the given user.
     *
     * @param  \Illuminate\Http\Request  $request
     * @param  string  $id
     * @return \Illuminate\Http\Response
     */
    public function update(Request $request, $id)
    {
        //
    }
}
```