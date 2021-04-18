# Request

## Introduction
Class `Illuminate\Http\Request` cung cấp một cách object-oriented để tương tác với HTTP request hiện tại đang được xử lý
bởi ứng dụng của bạn cũng như truy xuất input, cookies và file được gửi lên cùng với request

## Interacting With The Request
### Accessing The Request
Để có được một instance của HTTP request hiện tại thông qua dependency injection, bạn nên sử dụng type-hint  `Illuminate\Http\Request`
class trong closure của route hoặc method controller. Một instance của request đến sẽ tự động được đưa vào bởi service container:
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
        $name = $request->input('name');

        //
    }
}
```

Bạn cũng có thể sử dụng type-hint `Illuminate\Http\Request` trên closure của routes. Service container sẽ tự động đẩy
request đến vào trong closure khi nó được thưc thi:
```PHP
use Illuminate\Http\Request;

Route::get('/', function (Request $request) {
    //
});
```

#### Dependency Injection & Route Parameters


