
## VIII. Concurrency

### Mở rộng quy mô thông qua mô hình tiến trình

Với mọi chương trình máy tính, mỗi một lần chạy, nó đều được đại diện bởi một hoặc nhiều các tiến trình. Các ứng dụng web cũng thực hiện một loạt các hình thức thực thi tiến trình. Ví dụ,Các tiến trình của PHP khi chạy sẽ như là những tiến trình con của Apache, bắt đầu dựa theo khối lượng yêu cầu khi cần thiết . Các tiến trình của Java lại đi theo hướng ngược lại, kết hợp với JVM cung cấp một tiến trình vận tải tương đối lớn mà lưu giữ lượng lớn tài nguyên của hệ thống (CPU và memory)ngay khi khởi động, với việc quản lý đồng thời các luồng nội bộ. Trong cả 2 trường hợp, các tiến trình đang chạy chỉ hiển thị đối với các nhà phát triển của ứng dụng.

![Scale is expressed as running processes, workload diversity is expressed as process types.][66]

**Trong một ứng dụng tuân thủ 12 yếu tố, các tiến trình là thành phần số một.** Các tiến trình trong ứng dụng tuân thủ 12 yếu tố lấy cảm hứng từ [mô hình tiến trình unix để thực thi các tiến trình dịch vụ ngầm][67]. Khi sử dụng mô hình này, lập trình viên có thể  tổ chức kiến trúc ứng dụng của họ để xử lý khối lượng việc khác nhau bằng cách chỉ định mỗi loại công việc ứng với mỗi loại tiến trình. Ví dụ như, HTTP requests có thể được xử lý bởi một tiến trình web, và các task ngầm thực thi trong thời gian dài được xử lý bởi một tiến trình worker.

Việc này không loại trừ các tiến trình riêng lẻ khỏi quá trình xử lý việc ghép kênh nội bộ của chúng, thông qua các luồng bên trong VM runtime, hoặc mô hình async/evented được tìm thấy trong các công cụ như [EventMachine][68], [Twisted][69], hoặc [Node.js][70]. Nhưng một VM cá nhân chỉ có thể phát triển hơn (theo chiều sâu), do đó, ứng dụng phải có khả năng mở rộng chạy nhiều tiến trình trên nhiều máy vật lý khác nhau.

Một mô hình tiến trình thực sự toả sáng khi cuối cùng nó có khả năng mở rộn. Việc [Không chia sẻ điều gì, bản chất của việc phân chia theo chiều ngang của tiến trình trong ứng dụng tuân thủ 12 yếu tố][71] có nghĩa là thêm vào đồng thời nhiều hơn 1 hành động đơn giản và đáng tin cậy. Mảng các loại tiến trình và số lượng tiến trình của mỗi loại được biết đến như là _hệ thống tiến trình_.

Các tiến trình của ứng dụng tuân thủ 12 yếu tố [không bao giờ được là những tiến trình bất tử][72] hoặc là tiến trình ghi lại file PID. Thay vào đó, dựa vào trình quản lý các tiến trình của hệ điều hành (như là [systemd][73], một quản lý các tiến trình phân tán trên hệ thống cloud, hoặc một công cụ như là [Foreman][74] trong môi trường development) để quản lý [output streams][75], phản hồi các tiến trình bị crash, và xử lý các phiên restart và shutdown do người dùng khởi tạo.

## IX. Disposability

### Tối đa hoá sức mạnh bằng việc khởi động nhanh và shutdown 1 cách nhẹ nhàng

**[các tiến trình][76] của ứng dụng tuân thủ 12 yếu tố thường  _chỉ dùng một lần_, có nghĩa là chúng có thể bắt đầu hoặc dừng lại ngay thời điểm của thông báo.** Điều này tạo điều kiện cho việc mở rộng một cách linh hoạt, triển khai một cách nhanh chóng các thay đổi trong [code][77] hoặc [config][78], và nâng cao sức mạnh khi triển khai production.

Các tiến trình nên cố gắng **giảm thiểu thời gian khởi động**. Lý tưởng nhất là tiến trình chỉ mất một vài giây kể từ khi lệnh khởi chạy được thực thi cho tới khi quá trình này được thực hiện và sẵn sàng nhận các yếu cầu hoặc các công việc. Thời gian khởi động ngắn giúp tiến trình thực hiện nhanh hơn cho [release][79] và dễ dàng mở rộng; và nó hỗ trợ mạnh mẽ hơn, vì trình quản lý các tiến trình có thể di chuyển dễ dàng các tiến trình sang một máy vật lý mới khi được bảo trì.

Các tiến trình **shutdown một cách nhẹ nhàng khi chúng nhận được một tín hiệu [SIGTERM][80]** từ trình quản lý các tiến trình. Đối với tiến trình web, shutdown một cách nhẹ nhàng bằng việc dừng lắng nghe trên cổng dịch vụ (do đó từ chối mọi yêu cầu mới nào), cho phép mọi yêu cầu hiện tại kết thúc, và sao đó thì tắt chúng. Bản chất trong mô hình này là ở các yêu cầu HTTP nhỏ ( không quá một vài giây), hoặc trong trường hợp yêu cầu dài hơn, client sẽ cố gắng kết nối lại khi kết nối đó bị mất.

Đối với một tiến trình worker, shutdown một cách nhẹ nhàng bằng việc trả lại công việc hiện tại vào hàng đợi các công việc. Ví dụ, trên [RabbitMQ][81],một worker có thể gửi một [`NACK`][82]; trên [Beanstalkd][83], một job có thể được trả về hàng đợi một cách tự động khi worker bị mất kết nối. Các hệ thống dựa trên khoá như là [Delayed Job][84] càn phải chắn chắn là khoá phát hành của họ có trên job record. Bản chất của mô hình này cho mọi công việc đều là [reentrant][85], thường đạt được bằng cách đóng gói các kết quả trả về vào trong một transaction, hoặc làm cho một [idempotent][86] hoạt động.

Các tiến trình cũng phải **chống lại việc sudden death một cách mạnh mẽ**, trong trường hợp lỗi cơ bản về phần cứng. Mặc dù đây là trường hợp ít khi xảy ra hơn so với việc shutdown một cách nhẹ nhàng như `SIGTERM`, nó vẫn có thể sảy ra. Các tiếp cận được đề xuất ở đây là sử dụng một backend hỗ trợ hàng đợi một cách mạnh mẽ, như là Beanstalkd, nó trả về hàng đợi những công việc khi client mất kết nối hoặc hết thời gian chờ.Dù với cách nào đi nữa, một ứng dụng tuân thử 12 yếu tố đều được tổ chức kiến trúc để xử lý sự cố bất ngờ, chấm dứt việc không mềm mại. Khái niệm [Thiết kế chỉ có sự cố][87] dẫn tới [các kết luận logic][88].

## X. Tính tương hỗ của Dev/prod

### Giữ môi trường development, staging, và production càng giống nhau càng tốt

Trong lịch sử, đã có những khoảng cách đáng kể giữa môi trường development ( một nhà phát triển chỉnh sửa trực tiếp trên phiên bản [deploy][89] cục bộ của ứng dụng) và production (phiên bản triển khai của ứng dụng đang chạy được người dùng cuối truy cập). Những khoảng cách được thể hiện trong 3 khu vực:

* **Khoảng cách về thời gian:** Nhà phát triển có thể làm việc trên code mất vài ngày, vài tuần, hoặc thậm chí vài tháng để đưa vào production.
* **khoảng cách về nhân sự**: các nhà phát triển thì viết code, còn các kỹ sư ops mới deploy nó.
* **Khoảng cách về công cụ**: Các nhà phát triển có thể sử dụng các công cụ như Nginx, SLQite, và OSX trong khi môi trường triển khai production lại sử dụng Apache, MySQL và Linux.

**Ưng dụng tuân thủ 12 yếu tố được thiết kế để [Tiếp tục phát triển][90] bằng cách giữ cho môi trường của development và production có khoảng cách là nhỏ nhất.** Nhìn vào 3 khoảng cách được mô tả ở trên:
* Làm cho khoảng cách về thời gian nhỏ nhất: một nhà phát triển có thể viết code và nó được deployed sau một vài giờ hoặc thậm chí chỉ vài phút sau đó.
* Làm cho khoảng cách về nhân sự nhỏ lại: Các nhà phát triển mà viết ra đoan code thì liên quan chặt chẽ tới việc triển khai chúng và theo dõi chúng trong môi trường production.
* Làm cho khoảng cách về cách công cụ nhỏ lại: giữ môi trường development và production càng giống nhau càng tốt.

Tóm tắt các điều trên thành một bảng như sau:

| ----- |
|  |  Ứng dụng truyền thống |  ứng dụng tuân thủ 12 yếu tố |  
| thời gian giữa việc triển khai |  Vài tuần |  vài giờ |  
| Người viết code và người triển khai code |  2 Người khác |  cùng một người |  
| Môi trường Dev và production  |  Khác nhau |  Càng giống càng tốt | 

[Dịch vụ chạy ngầm][91], như là cơ sở dữ liệu của ứng dụng, hệ thống hàng đợi, hoặc cache, là một phần trong tính tương hỗ của dev/prod parity khá quan trọng. Nhiều ngôn ngữ cung cấp các thư viện giúp đơn giản hoá việc truy cập tới các dịch vụ chạy ngầm, bao gồm có _adapters_ đến các loại dịch vụ khác nhau. Một vài ví dụ như trong bảng bên dưới.

| ----- |
| Loại |  Ngôn ngữ |  Thư viện |  Adapters |  
| Database |  Ruby/Rails |  ActiveRecord |  MySQL, PostgreSQL, SQLite |  
| Queue |  Python/Django |  Celery |  RabbitMQ, Beanstalkd, Redis |  
| Cache |  Ruby/Rails |  ActiveSupport::Cache |  Memory, filesystem, Memcached | 

Các nhà phát triển thường bị hấp dẫn khi sử dụng các dịch vụ chạy ngầm nhẹ trên môi trường local của họ, trong khi các dịch vụ chạy ngầm mạnh mẽ và hoàn chỉnh thường được sử dụng trong môi trường production. Ví dụ như, sử dụng SQLite trên môi trường local và PostgreSQL trên môi trường production; hoặc các bộ nhớ local dành cho bộ nhớ đệm trên môi trường development và Memcached trong production.

**Nhà phát triển theo 12 yếu tố chống lại việc thôi thúc sử dụng các dịch vụ ngầm khác nhau giữa môi trường development và production**, ngay cả khi adapter có thể thích nghi với bất kì sự thay đổi nào trong các dịch vụ ngầm. Khác biệt giữa các dịch vụ ngầm là những điểm không tương thích rất nhỏ, khiến cho việc code có thể làm việc và pass các bài test trên môi trường development hoặc staging nhưng lại fail trên môi trường production. Các lỗi này tạo ra các xung đột làm mất tập trung khi triển khai liên tục. Chí phí của các xung đột này và giảm thiểu hiệu năng trong việc triển khai liên tục là rất lớn khi xem xét một cách tổng quan trong suốt thời gian tồn tại của một ứng dụng.

Các dịch vụ local loại nhẹ đã kém hấp dẫn hơn so với trước đây. Các dịch vụ ngầm hiện đại như Memcached, PostgreSQL, và RabbitMQ không khó trong việc cài đặt và chạy thông qua hệ thống đóng gói, như là [Homebrew][92] và [apt-get][93]. Một cách khác, các công cụ cung cấp khai báo như là [Chef][94] và [Puppet][95] kết hợp với các môi trường ảo có dung lượng nhẹ như là [Docker][96] và [Vagrant][97] cho phép các nhà phát triển chạy các môi trường local mà gần như giống với môi trường production. Chi phí để cài đặt và sử dụng các hệ thống như này thường rất thấp so với lợi ích mà việc tính tương hỗ dev/prod và triển khai liên tục mang lại.

Các bộ Adapter cho cá dịch vụ chạy ngầm khác nhau vẫn rất hữu dụng, vì chúng giúp cho việc chuyển sang một dịch vụ chạy ngầm khác ảnh hưởng tương đối ít. Nhưng đối với tất cả các triển khia của ứng dụng  (developer environments, staging, production) nên sử dụng cùng loại và phiên bản với mỗi dịch vụ chạy ngầm.

## XI. Logs

### Đối xử với logs như một luồng sự kiện

_Logs_ cung cấp khả năng hiển thị các hành vi của ứng dụng đang chạy. Trên môi trường server-based chugns thường được ghi vào một file trên disk (một "logfile"); nhưng đây chỉ là một định đạng đầu ra.

Logs thường là các [luồng tổng hợp][98], các sự kiện được sắp xếp theo thời gian thu thập từ luồng output của tất cả ác tiến trình và các dịch vụ nền. Logs ở dạng thô thường là những đoạn văn bản với mỗi sự kiện trên 1 dòng(mặc dù backtrace của các trường hợp ngoại lệ thường có nhiều dòng). Log đều không có đầu hay cuối cố định, nhưng luồng của nó sẽ liên tục miễn là ứng dụng còn hoạt động.

**A twelve-factor app never concerns itself with routing or storage of its output stream.** It should not attempt to write to or manage logfiles. Instead, each running process writes its event stream, unbuffered, to `stdout`. During local development, the developer will view this stream in the foreground of their terminal to observe the app's behavior.

In staging or production deploys, each process' stream will be captured by the execution environment, collated together with all other streams from the app, and routed to one or more final destinations for viewing and long-term archival. These archival destinations are not visible to or configurable by the app, and instead are completely managed by the execution environment. Open-source log routers (such as [Logplex][99] and [Fluentd][100]) are available for this purpose.

The event stream for an app can be routed to a file, or watched via realtime tail in a terminal. Most significantly, the stream can be sent to a log indexing and analysis system such as [Splunk][101], or a general-purpose data warehousing system such as [Hadoop/Hive][102]. These systems allow for great power and flexibility for introspecting an app's behavior over time, including:

* Finding specific events in the past.
* Large-scale graphing of trends (such as requests per minute).
* Active alerting according to user-defined heuristics (such as an alert when the quantity of errors per minute exceeds a certain threshold).


## XII. Các tiến trình Admin

### chạy các tác vụ admin/management như các tiến trình sử dụng một lần

The [process formation][03] is the array of processes that are used to do the app's regular business (such as handling web requests) as it runs. Separately, developers will often wish to do one-off administrative or maintenance tasks for the app, such as:

* Running database migrations (e.g. `manage.py migrate` in Django, `rake db:migrate` in Rails).
* Running a console (also known as a [REPL][104] shell) to run arbitrary code or inspect the app's models against the live database. Most languages provide a REPL by running the interpreter without any arguments (e.g. `python` or `perl`) or in some cases have a separate command (e.g. `irb` for Ruby, `rails console` for Rails).
* Running one-time scripts committed into the app's repo (e.g. `php scripts/fix_bad_records.php`).

One-off admin processes should be run in an identical environment as the regular [long-running processes][105] of the app. They run against a [release][106], using the same [codebase][107] and [config][108] as any process run against that release. Admin code must ship with application code to avoid synchronization issues.

The same [dependency isolation][109] techniques should be used on all process types. For example, if the Ruby web process uses the command `bundle exec thin start`, then a database migration should use `bundle exec rake db:migrate`. Likewise, a Python program using Virtualenv should use the vendored `bin/python` for running both the Tornado webserver and any `manage.py` admin processes.

Twelve-factor strongly favors languages which provide a REPL shell out of the box, and which make it easy to run one-off scripts. In a local deploy, developers invoke one-off admin processes by a direct shell command inside the app's checkout directory. In a production deploy, developers can use ssh or other remote command execution mechanism provided by that deploy's execution environment to run such a process.


[66]: https://12factor.net/images/process-types.png
[67]: https://adam.herokuapp.com/past/2011/5/9/applying_the_unix_process_model_to_web_apps/
[68]: http://rubyeventmachine.com/
[69]: http://twistedmatrix.com/trac/
[70]: http://nodejs.org/
[71]: https://12factor.net/processes
[72]: http://dustin.github.com/2010/02/28/running-processes.html
[73]: https://www.freedesktop.org/wiki/Software/systemd/
[74]: http://blog.daviddollar.org/2011/05/06/introducing-foreman.html
[75]: https://12factor.net/logs
[76]: https://12factor.net/processes
[77]: https://12factor.net/codebase
[78]: https://12factor.net/config
[79]: https://12factor.net/build-release-run
[80]: http://en.wikipedia.org/wiki/SIGTERM
[81]: http://www.rabbitmq.com/
[82]: http://www.rabbitmq.com/amqp-0-9-1-quickref.html#basic.nack
[83]: http://kr.github.com/beanstalkd/
[84]: https://github.com/collectiveidea/delayed_job#readme
[85]: http://en.wikipedia.org/wiki/Reentrant_%28subroutine%29
[86]: http://en.wikipedia.org/wiki/Idempotence
[87]: http://lwn.net/Articles/191059/
[88]: http://docs.couchdb.org/en/latest/intro/overview.html
[89]: https://12factor.net/codebase
[90]: http://avc.com/2011/02/continuous-deployment/
[91]: https://12factor.net/backing-services
[92]: http://mxcl.github.com/homebrew/
[93]: https://help.ubuntu.com/community/AptGet/Howto
[94]: http://www.opscode.com/chef/
[95]: http://docs.puppetlabs.com/
[96]: https://www.docker.com/
[97]: http://vagrantup.com/

[98]: https://adam.herokuapp.com/past/2011/4/1/logs_are_streams_not_files/
[99]: https://github.com/heroku/logplex
[100]: https://github.com/fluent/fluentd
[101]: http://www.splunk.com/
[102]: http://hive.apache.org/
[103]: https://12factor.net/concurrency
[104]: http://en.wikipedia.org/wiki/Read-eval-print_loop
[105]: https://12factor.net/processes
[106]: https://12factor.net/build-release-run
[107]: https://12factor.net/codebase
[108]: https://12factor.net/config
[109]: https://12factor.net/dependencies
