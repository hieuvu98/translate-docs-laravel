<p align="center"><a href="https://laravel.com" target="_blank"><img src="https://raw.githubusercontent.com/laravel/art/master/logo-lockup/5%20SVG/2%20CMYK/1%20Full%20Color/laravel-logolockup-cmyk-red.svg" width="400"></a></p>

<p align="center">
<a href="https://travis-ci.org/laravel/framework"><img src="https://travis-ci.org/laravel/framework.svg" alt="Build Status"></a>
<a href="https://packagist.org/packages/laravel/framework"><img src="https://img.shields.io/packagist/dt/laravel/framework" alt="Total Downloads"></a>
<a href="https://packagist.org/packages/laravel/framework"><img src="https://img.shields.io/packagist/v/laravel/framework" alt="Latest Stable Version"></a>
<a href="https://packagist.org/packages/laravel/framework"><img src="https://img.shields.io/packagist/l/laravel/framework" alt="License"></a>
</p>

# Blade Templates

## Introduction
Blade là một templating engine đơn giản nhưng rất mạnh mẽ được tạo ra và đi cùng với Laravel. Không giống như các templating engine khác, Blade không hạn chế
việc bạn sử dụng PHP thuần túy ngoài giao diện. Thực tế, tất cả Blade template được biên dịch thành các mã PHP thuần và được cache lại cho tới khi chúng bị chỉnh
sửa, nghĩa là Blade không phát sinh thêm các chi phí nào trong ứng dụng của bạn. Tệp Blade template sử dụng đuôi là `.blade.php` và thông thường được lưu trong thư
mục `resource/views`.

Blade view có thể được trả về từ các tuyến đường hoặc controller bằng cách sử dụng `view`. Tất nhiên, như đã đề cập trong tài liệu 
phần [views](https://laravel.com/docs/8.x/views), dữ liệu có thể đường truyền đến Blade view bằng cách sử dụng đối số thứ hai của helpler `view`:
```PHP
Route::get('/', function () {
    return view('greeting', ['name' => 'Finn']);
});
```

## Displaying Data