# Lifecycle Overview

## First Step

<p align="center">
    <img src="https://raw.githubusercontent.com/hieuvu98/translate-docs-laravel8/main/translate-docs/lifecycle.png" alt="Life Cycle" />
</p>

Đầu vào của mọi request trong Laravel nằm trong file <code>index.php</code>. Tất cả các request được điều hướng đến file
này nhờ web server (Apache/Nginx) cấu hình. File <code>index.php</code> không chứa nhiều code. Đúng hơn nó là điểm bắt
đầu để loading phần còn lại của framework.

File <code>index.php</code> load các cấu hình sinh ra từ Composer, và sau đó lấy một đối tượng của Laravel từ
bootstrap.php. Hành động đầu tiên được làm bởi Laravel là tạo một đối tượng của ứng dụng service container.

## HTTP / Console Kernel

Tiếp theo, các request được gửi đến HTTP Kernel hoặc Console Kernel, phụ thuộc vào kiểu request đi vào ứng dụng. Hai
kernel này đóng vai trò như là trung tâm điều khiển mà tất cả các request đi qua. Hiện tại, chúng ta chỉ tập trung vào
HTTP kernel nằm tại <code>app/Http/Kernel.php</code>

HTTP Kernel kế thừa từ class <code>Illuminate/Foundation/Http/Kernel</code>, class này định nghĩa một mảng
<i>bootstrappers </i> sẽ được chạy trước khi thực hiện request. Các <code>bootstrappers</code> cấu hình việc xử lý lỗi,
log, nhận biết môi trường ứng dụng và thực hiện các tác vụ khác cần được làm trước khi request thực sự được xử lý.

HTTP Kernel cũng định nghĩa danh sách HTTP Middleware do đó tất cả các request phải đi qua trước khi chúng được xử lý
bởi ứng dụng. Những middleware này sẽ đọc và ghi HTTP Session, xác định xem ứng dụng có đang ở chế độ bảo trì hay không,
xác minh CSRF token và nhiều hơn nữa.

HTTP Kernel có một phương thức handle khá đơn giản, nó nhận một request và trả về một response. Hãy nghĩ kernel như một
hộp đen lớn. Cung cấp cho nó các HTTP request và nó sẽ trả về HTTP response

## Service Providers

Một trong những hành động khởi tạo hạt nhân quan trọng nhất là load các service provider cho ứng dụng của bản. Tất cả
các service provider của ứng dụng được cấu hình trong file <code>config/app.php</code> trong mảng <code>providers</code>

Laravel sẽ duyệt qua danh sách các providers và khởi tạo mỗi cái trong chúng. Sau khi khởi tạo providers, hàm
<code>register</code> sẽ được gọi ở tất cả các providers. Sau đó, khi tất cả các providers đã được đăng ký, hàm <code>
boot</code>
sẽ được gọi ở mỗi provider.

Service providers sẽ chịu trách nhiệm khởi tạo tất cả các thành phần khác nhau của framework, như: database, queue,
validation và routing. Về cơ bản, mọi tính năng do Laravel cung cấp đều được khởi tạo và cấu hình bỏi service provider.
Bởi vì chúng khởi tạo và cấu hình nhiều tính năng của framework, service provider là khía cạnh quan trọng nhất của toàn
bộ quá trình khởi tạo Laravel

Bạn có thể tự hỏi mình rằng tại sao phương thức <code>register</code> của mọi service provider được gọi trước khi gọi
phương thức <code>boot</code> trên bất kỳ service provider nào. Câu trả lời rất đơn giản. Bằng cách gọi phương
thức <code>register</code>
của mọi service provider trước, service provider có thể phụ thuộc vào các ràng buộc được đăng ký và các giá trị vào thời
điểm hàm <code>boot</code> thực thi.

## Routing

Một trong những service providers quan trọng nhất trong ứng dụng của bạn là <code>
App/Providers/RouteServiceProvider</code>
. Service providers này các tệp định tuyến có trong thư mục <code>routes</code>. Tiếp tục, mở <code>
RouteServiceProvider </code>
và xem nó hoạt động như thế nào!

Khi ứng dụng đã được khởi động và tất cả service providers đã được đăng ký, <code>Request</code> sẽ được đưa đến cho
router. Router sẽ thực hiện đưa các request đến một route hoặc controller để xử lý, cũng như thực thi các middleware
riêng của route

Middleware cung cấp các cơ chế rất thuận tiện để lọc hoặc kiểm tra các HTTP request đi vào bên trong ứng dụng. Ví dụ,
Laravel gồm một middleware để xác minh xem người dùng ứng dụng của bạn đã xác thực (authenticated) hay chưa. Nếu người
dùng không được xác thực, middleware sẽ điều hướng người dùng đến màn hình đăng nhập. Tuy nhiên, nếu người dùng đã xác
thực, middleware cho phép request tiến sâu hơn vào ứng dụng. Một số middleware được gán cho tất cả các routes bên trong
ứng dụng, như những route được xác định bên trong thuộc tính <code>$middleware</code>, trong khi một số chỉ được chỉ
định cho các routes hoặc route groups. Bạn có thể tìm hiểu thêm về middleware bằng cách đọc tài liệu đầy đủ
về [middleware documentation](https://laravel.com/docs/8.x/middleware).

Nếu request đi qua tất cả các middleware được phân bố, route hoặc controller method sẽ được thực thi và reponse được trả
về do route hoặc controller method sẽ gửi ngược lại thông qua một chuỗi route của middleware

## Finish up

Khi route hoặc controller method trả về response, response sẽ được gửi ra ngoài thông qua route middleware, tạo điều
kiện cho ứng dụng kiểm tra và sửa đổi response.

Cuối cùng, khi response quay trở lại middleware, phương thức <code>handle</code> của HTTP kernel sẽ trả về object và
<code>index.php</code> sẽ gọi phương thức <code>send</code> và response được trả về! Phương thức <code>send</code> gửi
nội dung response đến trình duyệt của người dùng.

## Focus On Service Providers
Service Provides thực sự là chìa khóa để khởi tạo ứng dụng Laravel. Đối tượng của ứng dụng được tạo ra, các service providers
đã được đăng ký and request được đẩy tới phần được ứng dụng khởi tạo. Nó thực sự đơn giản!

Mặc định <code>AppServiceProvider</code> hoàn toàn trống rỗng. Provider này là nơi tuyệt vời để thêm các phần khởi tạo
và ràng buộc service container của riêng bạn. Dĩ nhiên là đối với các ứng dụng lớn, bạn sẽ muốn tạo vài service providers,
mỗi cái sẽ thực hiện khởi tạo một công việc khác nhau.
