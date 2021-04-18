# Service Provider
## Introduction

Service provider là nơi trung tâm của tất cả quá trình khởi tạo ứng dụng Laravel. Ứng dụng của bạn cũng như các thành phần
cốt lõi của Laravel được khởi tạo thông qua service provider.

Nhưng, "bootstrapped" là gì? Đơn giản, chúng ta muốn đăng ký nhiều thứ, bao gồm đăng ký các liên kết tới service provider,
event listens, middleware and even route. Service provider là nơi trung tâm để cấu hình ứng dụng của bạn.

Nếu bạn mở file `config/app.php` nàm trong Laravel, bạn sẽ nhìn thấy một mảng `providers`. Đây là tất cả class service
provider sẽ được load vào trong ứng dụng. Theo mặc định, tập hợp các service provider cốt lõi của Laravel được liệt kê 
trong mảng. Các service provider này sẽ tiến hành khởi động các thành phần cốt lõi của Laravel như là mailer, queue, cache,
và nhiều thứ khác. Một trong số đó được gọi là `deferred` providers, nghĩa là chúng sẽ không được load trong mọi request,
nhưng chỉ khi service nào cần sử dụng thì cung cấp nó.

## Writing Service Providers

Tất cả service provider kế thừa từ class `Illuminate\Support\ServiceProvider`. Hầu hết các service provider bao gồm method
`register` và `boot`. Trong method `register`, bạn chỉ nên ràng buộc mọi thứ vào trong service container. Bạn không nên
cố gắng đăng ký bất kỳ event liseners, routes, hoặc bất kỳ các chức năng nào khác vào bên trong method `register`.

Artisan CLI có thể dễ dàng sinh ra một provider mới thông qua câu lệnh `make:provider`:
```PHP
php artisan make:provider RiakServiceProvider
```

### The Register Method
Như đã đề cập ở trước, bên trong hàm register, bạn chỉ nên thực hiện đăng kí vào trong service container. Bạn không nên
cố gắng đăng kí bất kì event listeners, routes hay bất kì các chức năng nào khác vào trong hàm register. Nếu không, 
bạn có thể vô tình sử dụng một service được cung cấp bởi một service provider mà chưa được load.
```PHP
<?php

namespace App\Providers;

use App\Services\Riak\Connection;
use Illuminate\Support\ServiceProvider;

class RiakServiceProvider extends ServiceProvider
{
    /**
     * Register any application services.
     *
     * @return void
     */
    public function register()
    {
        $this->app->singleton(Connection::class, function ($app) {
            return new Connection(config('riak'));
        });
    }
}
```
Các service provider chỉ được định nghĩa ở method `register` và sử dụng phương thức đó để triển khai `App\Services\Riak\Connection`
 trong service container.

#### The bindings And singletons Properties
Nếu service provider của bạn đăng ký nhiều ràng buộc đơn giản, bạn có thể sử dụng thuộc tính `bindings` và `singletons`
thay vì đăng ký thủ công từng từng ràng buộc. Khi service provider được load nó sẽ tự động kiểm tra các thuộc tính này và
kiểm tra các ràng buộc:
```PHP
<?php

namespace App\Providers;

use App\Contracts\DowntimeNotifier;
use App\Contracts\ServerProvider;
use App\Services\DigitalOceanServerProvider;
use App\Services\PingdomDowntimeNotifier;
use App\Services\ServerToolsProvider;
use Illuminate\Support\ServiceProvider;

class AppServiceProvider extends ServiceProvider
{
    /**
     * All of the container bindings that should be registered.
     *
     * @var array
     */
    public $bindings = [
        ServerProvider::class => DigitalOceanServerProvider::class,
    ];

    /**
     * All of the container singletons that should be registered.
     *
     * @var array
     */
    public $singletons = [
        DowntimeNotifier::class => PingdomDowntimeNotifier::class,
        ServerProvider::class => ServerToolsProvider::class,
    ];
}
```

### The Boot Method
Vậy nếu chúng ta cần đăng ký một [View composer](https://laravel.com/docs/8.x/views#view-composers) trong service provider
thì sao? Việc này nên được làm trong method `boot`. Method này sẽ được gọi sau khi tất cả service provider đã được đăng ký,
nghĩa là bạn có thể truy cập vào tất cả các service đã được đăng ký vào trong framework:
```PHP
<?php

namespace App\Providers;

use Illuminate\Support\Facades\View;
use Illuminate\Support\ServiceProvider;

class ComposerServiceProvider extends ServiceProvider
{
    /**
     * Bootstrap any application services.
     *
     * @return void
     */
    public function boot()
    {
        View::composer('view', function () {
            //
        });
    }
}
```

#### Boot Method Dependency Injection
Bạn có thể type-hint dependencies cho service provider của bạn ở hàm boot. Service container sẽ tự động inject bất cứ 
dependencies nào bạn cần:
```PHP
use Illuminate\Contracts\Routing\ResponseFactory;

/**
 * Bootstrap any application services.
 *
 * @param  \Illuminate\Contracts\Routing\ResponseFactory  $response
 * @return void
 */
public function boot(ResponseFactory $response)
{
    $response->macro('serialized', function ($value) {
        //
    });
}
```

## Registering Providers
Tất cả các service provider được đăng kí bên trong file cấu hình config/app.php. File này chứa một mảng các providers
danh sách tên của các service providers. Mặc định, một tập hợp các core service provider của Laravel nằm trong mảng 
này. Những provider này làm nhiệm vụ khởi tạo các thành phần core của Laravel, ví dụ như mailer, queue, cache, và các
thành phần khác.

Để đăng kí provider, đơn giản chỉ cần thêm vào trong mảng đó:
```PHP
'providers' => [
    // Other Service Providers

    App\Providers\ComposerServiceProvider::class,
],
```

## Deferred Providers
Nếu bạn muốn provider chỉ đăng kí liên kết vào trong service container, bạn có thể chọn trì hoãn việc đăng kí cho tới 
khi nào cần thiết. Việc trì hoãn quá trình load một provider sẽ cải thiện performance của ứng dụng, vì nó không load
từ filesystem trong mọi yêu cầu.

Để trì hoàn việc load một provider, set thuộc tính defer thành true và khai báo một hàm provides. Hàm này sẽ trả về
liên kết tới service container mà provider này đăng kí:
```PHP
<?php

namespace App\Providers;

use App\Services\Riak\Connection;
use Illuminate\Contracts\Support\DeferrableProvider;
use Illuminate\Support\ServiceProvider;

class RiakServiceProvider extends ServiceProvider implements DeferrableProvider
{
    /**
     * Register any application services.
     *
     * @return void
     */
    public function register()
    {
        $this->app->singleton(Connection::class, function ($app) {
            return new Connection($app['config']['riak']);
        });
    }

    /**
     * Get the services provided by the provider.
     *
     * @return array
     */
    public function provides()
    {
        return [Connection::class];
    }
}
```