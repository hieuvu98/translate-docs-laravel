<p align="center"><a href="https://laravel.com" target="_blank"><img src="https://raw.githubusercontent.com/laravel/art/master/logo-lockup/5%20SVG/2%20CMYK/1%20Full%20Color/laravel-logolockup-cmyk-red.svg" width="400"></a></p>

<p align="center">
<a href="https://travis-ci.org/laravel/framework"><img src="https://travis-ci.org/laravel/framework.svg" alt="Build Status"></a>
<a href="https://packagist.org/packages/laravel/framework"><img src="https://img.shields.io/packagist/dt/laravel/framework" alt="Total Downloads"></a>
<a href="https://packagist.org/packages/laravel/framework"><img src="https://img.shields.io/packagist/v/laravel/framework" alt="Latest Stable Version"></a>
<a href="https://packagist.org/packages/laravel/framework"><img src="https://img.shields.io/packagist/l/laravel/framework" alt="License"></a>
</p>

# Validation

## Introduction

Laravel cung cấp một số cách tiếp cận khác nhau để thẩm định dữ liệu gửi đến ứng dụng của bạn. Cách phổ biến nhất là sử
dụng phương thức `validate` có sẵn trên tất cả các HTTP request đến. Tuy nhiên, chúng ta cũng sẽ thảo luận về các cách
tiếp cận khác.

Laravel bao gồm nhiều quy tắc thẩm định rất thuận tiện mà bạn có thể áp dụng cho dữ liệu, thậm chí cung cấp khả năng xác
thực nếu dữ liệu là giá trị duy nhất trong một bảng cơ sở dữ liệu nhất định. Chúng tôi sẽ trình bày chi tiết từng quy tắc
thẩm định này để bạn làm quen với tất cả các tính năng này.

## Validation Quickstart
### Defining The Routes

Đầu tiên, giả sử chúng ta có các route sau được xác định:
```PHP
use App\Http\Controllers\PostController;

Route::get('/post/create', [PostController::class, 'create']);
Route::post('/post', [PostController::class, 'store']);
```

Route `GET` sẽ hiển thị một form để user tạo một bài viết mới, trong khi route `POST` sẽ lưu trữ bài viết mới vào database.

### Creating The Controller

Tiếp theo, hãy xem một controller để xử lý các request đến route này. Chúng ta sẽ để trống phương thức `store`:
```PHP
<?php

namespace App\Http\Controllers;

use App\Http\Controllers\Controller;
use Illuminate\Http\Request;

class PostController extends Controller
{
    /**
     * Show the form to create a new blog post.
     *
     * @return \Illuminate\View\View
     */
    public function create()
    {
        return view('post.create');
    }

    /**
     * Store a new blog post.
     *
     * @param  \Illuminate\Http\Request  $request
     * @return \Illuminate\Http\Response
     */
    public function store(Request $request)
    {
        // Validate and store the blog post...
    }
}
```

### Writing The Validation Logic

Bây giờ chúng ta sẽ điền vào phương thức store với logic thẩm định một bài viết mới. Để thực hiện việc này, chúng ta sẽ 
sử dụng phương thức `validate` được cung cấp bởi đối tượng `Illuminate\Http\Request`. Nếu các quy tắc thẩm định được thông
qua, code của bạn sẽ tiếp tục thực thi bình thường; tuy nhiên nếu thẩm định không thành công, một ngoại lệ sẽ được đưa ra
và phản hồi lỗi thích hợp sẽ tự động được gửi lại cho người dùng.

Nếu thẩm định không thành công, một chuyển hướng đến URL trước đó sẽ được tạo ra. Nếu request đến là một XHR request, một
phản rồi dạng JSON chưa các thông báo lỗi sẽ được trả về.

Để hiểu rõ hơn về phương thức `validate`, hãy quay trở lại phương thức `store`:

```PHP
/**
 * Store a new blog post.
 *
 * @param  \Illuminate\Http\Request  $request
 * @return \Illuminate\Http\Response
 */
public function store(Request $request)
{
    $validated = $request->validate([
        'title' => 'required|unique:posts|max:255',
        'body' => 'required',
    ]);

    // The blog post is valid...
}
```

Như bạn thấy, các quy tắc thẩm định được đưa vào phương thức `validate`. Đừng lo lắng, tất cả những quy tắc có sẵn nằm ở
đây [documented](https://laravel.com/docs/8.x/validation#available-validation-rules). Ngoài ra, các quy tắc thẩm định có
thể được chỉ định dưới dạng mảng thay vì một chuỗi phân cách.

Ngoài ra bạn có thể sử dụng phương thức `validateWithBag` để thẩm định một request và lưu bất kỳ thông tin báo lỗi nào:

```PHP
$validatedData = $request->validateWithBag('post', [
    'title' => ['required', 'unique:posts', 'max:255'],
    'body' => ['required'],
]);
```

### Stopping On First Validation Failure

Đôi khi bạn có thể muốn dừng các quy tắc thẩm định đang chạy sau lần thẩm định đầu tiên không thành công. để làm như vậy
hãy chỉ định quy tắc `bail` cho thuộc tính:

```PHP
$request->validate([
    'title' => 'bail|required|unique:posts|max:255',
    'body' => 'required',
]);
```

### A Note On Nested Attributes
Nếu HTTP request đến chưa các trường dữ liệu lồng nhau, bạn có thể chỉ định các trường này trong quy tắc thẩm định của bạn
bằng cách sử dụng dấu `.`:
```PHP
$request->validate([
    'title' => 'required|unique:posts|max:255',
    'author.name' => 'required',
    'author.description' => 'required',
]);
```

Mặt khác, nếu tên trường của bạn có chưa dấu `.` bạn có thể sử dụng dấu `\` để tránh việc hiểu nhầm với cú pháp có dấu `.`:
```PHP
$request->validate([
    'title' => 'required|unique:posts|max:255',
    'v1\.0' => 'required',
]);
```

## Displaying The Validation Errors

Vậy nên, điều gì sẽ xảy ra nếu các trường trong request đến không thỏa mãn các quy tắc thẩm định đã cho? Như đã đề cập ở
trước Laravel sẽ tự động điều hướng người dùng quay trở lại vị trí trước đó của họ. Ngoài ra, tất cả các lỗi thẩm định và
giá trị input của request sẽ tự động được chuyển sang [flash session](https://laravel.com/docs/8.x/session#flash-data)

Một biến `$errors` được chia sẻ với tất cả view của ứng dụng bởi middleware `Illuminate\View\Middleware\ShareErrorsFromSession`,
cái mà được cung cấp bởi nhóm middlewảe `group`. Khi middleware này được áp dụng, biến `$errors` sẽ luôn luôn có sẵn trong
view, cho phép bạn sử dụng biến `$errors` một cách an toàn.

```PHP
<!-- /resources/views/post/create.blade.php -->

<h1>Create Post</h1>

@if ($errors->any())
    <div class="alert alert-danger">
        <ul>
            @foreach ($errors->all() as $error)
                <li>{{ $error }}</li>
            @endforeach
        </ul>
    </div>
@endif

<!-- Create Post Form -->
```

### Customizing The Error Messages
Mỗi quy tắc thẩm định được xây dựng trong Laravel có một thông báo lỗi nằm trong file `resources/lang/en/validation.php`.
Trong tệp này, bạn sẽ tìm thấy một mục dịch ý nghĩa cho từng quy tắc. Bạn có thể tự do thay đổi hoặc sửa đổi các thông báo
này dựa trên nhu cầu của ứng dụng.

Mặt khác, bạn có thể sao chép sang một thư mục dịch ngôn ngữ khác để áp dụng đa ngôn ngữ cho ứng dụng.

### XHR Requests & Validation

Trong ví dụ này, chúng ta sử dụng một form truyền thống để gửi dữ liệu đến ứng dụng. Tuy nhiên, nhiều ứng dụng nhận được
XHR request từ bên FE. Khi sử dụng phương thức `validate` trong một XHR request, Laravel sẽ không tạo mới một phản hồi
điều hướng. Thay vào đó Laravel tạo một JSON response chứa tất cả các lỗi thẩm định. JSON response sẽ trả về HTTP status
code là 422.

### The @error Directive

Bạn có thể sừ dụng chỉ chị `@error blade` để nhanh chóng xác định nếu thông báo lỗi thẩm định có thuộc tính nhất định.
Trong chỉ thị `@directive`, bạn có thể echo biến `$message` để hiển thị lỗi:
```PHP
<!-- /resources/views/post/create.blade.php -->

<label for="title">Post Title</label>

<input id="title" type="text" class="@error('title') is-invalid @enderror">

@error('title')
    <div class="alert alert-danger">{{ $message }}</div>
@enderror
```

### Repopulating Forms

Khi Laravel tạo ra một phản hồi điều hướng do lỗi thẩm định, Laravel sẽ tự động chuyển tất cả input request vào session.
Điều này được làm để bạn có thể thuận tiện truy cập vào input trong request tiếp theo và tạo lại biết mẫu mà người dùng
đã cố gắng gửi.

Để truy xuất đến đầu vào của request trước đó bạn gọi phương thức `old`:
```PHP
$title = $request->old('title');
```

### A Note On Optional Fields

```PHP
$request->validate([
    'title' => 'required|unique:posts|max:255',
    'body' => 'required',
    'publish_at' => 'nullable|date',
]);
```

## Form Request Validation

### Creating Form Requests

Để tạo một đối tượng request, bạn có thể sử dụng `make:request` của Artisan command CLI:
```PHP
php artisan make:request StorePostRequest
```

Form request đã được tạo ra sẽ đặt ở thư mục `app/Http/Requests`. Nếu thư mục không tồn tại, nó sẽ được tạo mới khi bạn
chạy command `make:request`. 
```PHP
/**
 * Get the validation rules that apply to the request.
 *
 * @return array
 */
public function rules()
{
    return [
        'title' => 'required|unique:posts|max:255',
        'body' => 'required',
    ];
}
```

Tất cả những gì bạn cần làm là nhập gợi ý kiểu request trên các phương thức của controller. Form request đến sẽ được xác
thực theo phương thức cntroller được gọi, nghĩa là bạn không cần làm lộn xộn controller với bất kỳ logic nào:
```PHP
/**
 * Store a new blog post.
 *
 * @param  \App\Http\Requests\StorePostRequest  $request
 * @return Illuminate\Http\Response
 */
public function store(StorePostRequest $request)
{
    // The incoming request is valid...

    // Retrieve the validated input data...
    $validated = $request->validated();
}
```

## Authorizing Form Requests

Lớp form request cũng chứa một phương thức `authorize`. Trong phương thức này, bạn có thể xem xét nếu người dùng xác đã thực
thực sự có quyền cập nhật một tài nguyển nhất định. Ví dụ bạn có thể xác định, nếu người dụng thực sự sử hữu comment của
bài viết mà họ đang cố gắng cập nhật. Rất có thể bạn sẽ tương tác với [authorization gates and policies](https://laravel.com/docs/8.x/authorization)
trong phương thức này:
```PHP
use App\Models\Comment;

/**
 * Determine if the user is authorized to make this request.
 *
 * @return bool
 */
public function authorize()
{
    $comment = Comment::find($this->route('comment'));

    return $comment && $this->user()->can('update', $comment);
}
```

Nếu phương thức `authorize` trả về false, một HTTP response với status code 403 sẽ tự động được trả về và phương thức trong
controller của bạn không được thực hiện.

Nếu bạn định xử lý logic authorization cho request, bạn chỉ cần trả về `true` từ phương thức `authorize`:
```PHP
/**
 * Determine if the user is authorized to make this request.
 *
 * @return bool
 */
public function authorize()
{
    return true;
}
```
### Customizing The Error Messages
Bạn có thể tùy chỉnh thông báo lỗi được sử dụng bởi form request bằng cách ghi đề lên moethod `message`. Phương thức sẽ
trả về một mảng  các cặp attribute/rule thông báo lỗi trong ứng dụng.

```PHP
/**
 * Get the error messages for the defined validation rules.
 *
 * @return array
 */
public function messages()
{
    return [
        'title.required' => 'A title is required',
        'body.required' => 'A message is required',
    ];
}
```
