# Views
## Creating & Rendering Views
Bạn có thể tạo view bằng cách đặt tên file với đuôi `.blade.php` trong thư mục `resources/views/`. Đuôi `.blade.php` 
thông báo cho framework biết rằng file chứa một blade template. Blade template chứa HTML cũng như các Blade Directive
nó cho phép dễ dàng thực hiện `echo` giá trị, tạo biểu thức `if`, lặp dữ liệu,...

Khi bạn đã tạo một view, bạn có thể trả nó về từ controller hoặc route bằng cách sử dụng helper global `view`:
```PHP
Route::get('/', function () {
    return view('greeting', ['name' => 'James']);
});
```

View cũng có thể được trả về bằng cách sử dụng facades `View`:
```PHP
use Illuminate\Support\Facades\View;

return View::make('greeting', ['name' => 'James']);
```

Đối số thứ nhất tryền vào trong helper `view` tương ứng với tên file trong thư mục `resources/views`. Đối số thứ hai là
một mảng data được truyền vào trong view để sử dụng. Trong trường hợp này, chúng ta đã truyền tên biến là `name` và nó 
sẽ được hiển trị trong view bằng cách sử dụng [Blade syntax](https://laravel.com/docs/8.x/blade)

## Nested View Directories
View cũng có thể được lồng trong các thư mục con của thư mục `resources/views`. Dấu `.` có thể được sử dụng để tham chiếu
đến các view lồng với nhau. Ví dụ, nếu view của bạn được lưu trữ tại thư mục `resources/views/admin/profile.blade.php`, 
bạn có thể trả nó về route hoặc controller bằng cách gọi như sau:
```PHP
return view('admin.profile', $data);
```

### Creating The First Available View
Sử dụng method first của facades `View`, bạn cũng có thể tạo ra một view tồn tại trong một mảng nhất định của view. Điều
này có thể hữu ích đến ứng dụng hoặc package của bạn cho phép view có thể được tùy chỉnh hoặc ghi đè:
```PHP
use Illuminate\Support\Facades\View;

return View::first(['custom.admin', 'admin'], $data);
```

### Determining If A View Exists
Nếu bạn cần xác định một view có tồn tại hay không, bạn có thể sử dụng facades `View`, method `exist` sẽ trả về true nếu
view tồn tại:
```PHP
if (View::exists('emails.customer')) {
    //
}
```

## Passing Data To Views
Bạn có thể truyền một mảng dữ liệu ra view:
```PHP
return view('greetings', ['name' => 'Victoria']);
```

Khi truyền data bằng cách này, data nên là một mảng với cặp key/value. Sau khi cung cấp data cho view, bạn có thể truy cập
đến từng giá trị  trong view bằng cách sử dụng key của data, như thế này: `<?php echo $name; ?>`.

Để thay thế cho việc truyền một mảng dữ liệu bằng helper function `view`, bạn có thể sử dụng method `with` để thêm các
dữ liệu riêng lẻ vào view:
```PHP
return view('greeting')
            ->with('name', 'Victoria')
            ->with('occupation', 'Astronaut');
```

### Sharing Data With All Views
Thỉnh thoảng, bạn cần chia sẻ data cho tất cả các view trong ứng dụng. Bạn có thể sử dụng method `share` của Facades `view`.
Thông thường, bạn nên gọi method `share` trong method `boot` của service provider. Bạn có thể thêm chúng vào trong 
`AppServiceProvider` hoặc tự tạo ra một Service Provider khác để chứa chúng:
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
View composer là các callback hoặc class method được gọi khi một view được render. Nếu bạn có một dữ liệu mà bạn cần 
ràng buộc nó lại tại thời điểm view của bạn được render, một view composer sẽ giúp bạn xử lý sắp xếp các logic 
trong view đó.

Nào chúng ta hãy đăng ký một view composer cùng với service provider. Chúng ta sẽ sử dụng hàm view để truy cập vào mẫu 
`Illuminate\Contracts\View\Factory`. Hãy nhớ rằng, Laravel sẽ không có thư mục mặc định cho view composers. Bạn có quyền 
tạo chúng bất cứ ở đâu mà bạn muốn. Ví dụ, Bạn có thể tạo thư mục `App\Http\ViewComposers` chẳng hạn:
```PHP
<?php

namespace App\Providers;

use App\Http\View\Composers\ProfileComposer;
use Illuminate\Support\Facades\View;
use Illuminate\Support\ServiceProvider;

class ViewServiceProvider extends ServiceProvider
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
        // Using class based composers...
        View::composer('profile', ProfileComposer::class);

        // Using closure based composers...
        View::composer('dashboard', function ($view) {
            //
        });
    }
}
```

Hãy nhớ rằng, nếu bạn tạo ra một Service Provider chứa các đăng ký view composer, bạn cần thêm nó vào bên trong mảng providers 
chứa tại tập tin cấu hình config/app.php.

Bây giờ chúng ta đã đăng ký thành công một view composer, phương thức `App\Http\View\Composers\ProfileComposer` sẽ thực 
thi tại thời điểm  mà view profile được rendered.
```PHP
<?php

namespace App\Http\View\Composers;

use App\Repositories\UserRepository;
use Illuminate\View\View;

class ProfileComposer
{
    /**
     * The user repository implementation.
     *
     * @var \App\Repositories\UserRepository
     */
    protected $users;

    /**
     * Create a new profile composer.
     *
     * @param  \App\Repositories\UserRepository  $users
     * @return void
     */
    public function __construct(UserRepository $users)
    {
        // Dependencies are automatically resolved by the service container...
        $this->users = $users;
    }

    /**
     * Bind data to the view.
     *
     * @param  \Illuminate\View\View  $view
     * @return void
     */
    public function compose(View $view)
    {
        $view->with('count', $this->users->count());
    }
}
```

Như vậy trước khi view đó được rendered, phương thức compose sẽ gọi `Illuminate\View\View` qua $view. Bạn có thể dụng 
phương thức with để ràng buộc dữ liệu đến view.

Chú ý: Tất các các view composers được xử lý thông qua service container, vì vậy bạn có thể thêm các phụ thuộc vào bên 
trong phương thức khởi tạo contructor của view composer.

#### Attaching A Composer To Multiple Views
Bạn có thể đính kèm nhiều view vào view composer bằng cách truyền vào một mảng chứa tất cả cá view vào thương thức composer:
```PHP
use App\Http\Views\Composers\MultiComposer;

View::composer(
    ['profile', 'dashboard'],
    MultiComposer::class
);
```

Phương thức composer có thể dùng * để đinh kèm toàn bộ view của bạn:
```PHP
View::composer('*', function ($view) {
    //
});
```

### View Creators
View creators rất giống với view composers; Tuy nhiên, nó sẽ tác động ngay lập tức vào các view thay vì chờ các view cho 
tới khi chúng được rendered. Để đăng ký một view creator, sử dụng phương thức `creator`:
```PHP
use App\Http\View\Creators\ProfileCreator;
use Illuminate\Support\Facades\View;

View::creator('profile', ProfileCreator::class);
```