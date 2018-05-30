[Source](https://12factor.net/ "Permalink to The Twelve-Factor App")

# The Twelve-Factor App

Trong thời đại hiện đại, phần mềm thường được phân phối dưới dạng dịch vụ: được gọi là  _web apps_, hoặc  _software-as-a-service_. 12 yếu tố về ứng dụng là phương pháp xây dựng ứng dụng về phần mềm như 1 dịch vụ như là:

* Sử dụng các chuẩn **khai báo** cho việc tự động cài đặt (thiết lập), để giảm thiểu thời gian và chi phí cho những lập trình viên mới tham gia vào dự án;
* Có các giao ước **rõ ràng** với hệ điều hành bên ,cung cấp **tối đa tính di động** giữa các môi trường thực thi;
* Phù hợp cho việc  **phát triển** trên các **nền tảng đám mây** hiện đại, giảm bớt sự cần thiết cho máy chủ và các hệ thống quản lý;
* **Giảm thiểu sự khác nhau** giữa môi trường phát triển và môi trường sản phẩm, cho phép **triển khai liên tục**  một cách linh động nhất;
* Và có thể  **mở rộng quy mô** mà không có sự thay đổi đáng kể nào về mặt công cụ,kiến trúc hay thói quen phát triển phần mềm.

Phương pháp luận 12-chuẩn có thể áp dụng cho các ứng dụng viết bằng bất kỳ ngôn ngữ lập trình nào, và sử dụng bất kỳ tập dịch vụ nền nào (database, queue, memory cache, etc).

## Background

Các tác giả của tài liệu đã áp dụng trực tiếp trong quá trình phát triển và triển khai hàng trăm ứng dụng, và giản tiếp chứng kiến quá trình phát triển, điều hành và mở rộng của hàng trăm nghìn ứng dụng qua công việc trên nền tảng [Heroku][1].


Tài liệu này tổng hợp tất cả kinh nghiệm và quan sát của chúng tôi trên rất nhiều các ứng dụng phần-mềm-như-1-dịch-vụ trong cuộc sống. Nó là 3 khía cạnh chính trên các ý tưởng thực hành cho phát triển ứng dụng, tập trung cụ thể vào tính linh động của phát triển liên tục 1 cách có tổ chức của ứng dụng, tính linh động trong cộng tác giữa các người phát triển làm việc trên codebase của ứng dụng, và [tránh các chi phí của vận hành ứng dụng][2].

Động lực của chúng tôi là nâng cao nhận thức về vài vấn đề có hệệ thống chúng tôi đã gặp trong phát triển ứng dụng hiện đại, để cung cấp các từ vựng chung để bàn luận về những vấn đề này, và để cho phép 1 tập các giải pháp khái niệm rộng lớn vào các vấn đề với các thuật ngữ kèm theo. Định dạn này lấy cảm hứng bởi cuốn sách của Martin Fowler, [_Patterns of Enterprise Application Architecture][3]_ and [_Refactoring][4]_.


## Ai nên đọc tài liệu này?

Bất kỳ người phát triển nào đang xây dựng các ứng dụng chạy như là 1 dịch vụ. Thông thường các kĩ sư triển khai hoặc điều khiển những ứng dụng như vậy.

## I. Codebase

### codebase được theo dõi trong việc kiểm soát các thay đổi, nhiều triển khai.

1 ứng dụng tuân theo 12-chuẩn luôn luôn được theo dõi trong 1 hệ thống kiểm soát các phiên bản, như là [Git][5], [Mercurial][6], hoặc [Subversion][7]. 1 bản sao chéo của cơ sở dữ liệu theo dõi thay đổi được biết như là 1 code repository, thường được viết tắt là _code repo_ hoặc là _repo_.

 _codebase_ có thể là bất cứ repo đơn lẻ nào nào (trong 1 hệ thống kiểm soát sửa đổi tập trung như Subverrsion), hoặc có thể lả 1 tập các repo chia sẻ chung 1 commit gốc(trong 1 hệ thống kiểm soát sửa đổi phân cấp như Git).

![một bản đồ codebase cho nhiều triển khai][8]

Luôn luôn có sự tương quan 1-1 giữa codebase và ứng dụng:

* Nếu có nhiều codebase, đó không phải là 1 ứng dụng - nó là 1 hệ thống phân cấp. Mỗi thành phần là 1 hệ thống được phân cấp trong 1 ứng dụng, và mỗi thành phần có thể tuân theo 12- chuẩn riêng 1 cách riêng lẻ.
* Nhiều ứng dụng chia sẻ cùng code là 1 sư vi phạm 12- chuẩn. Giải pháp ở đây là quản lý chia sẻ code vào các thư viện mà có thể được thêm vào thông qua [phụ thuộc vào quản lý][9].

Chỉ có duy nhất 1 codebase mỗi ứng dụng, nhưng sẽ có nhiều triển khải của ứng dụng. 1 triển khai sẽ chạy phiên bản của ứng dụng. Điều này thường là 1 phía của production, với 1 hoặc nhiều phía staging. Thêm vào đó, mỗi người phát triển có 1 bản sao của ứng dụng đang chạy trên môi trường developer nội bộ của họ, mỗi số chúng cũng được coi như 1 triển khai.

Codebase là chung giữa tất cả các triển khai, mặc dù các phiên bản khác nhau có thể hoạt động trong mỗi triển khai. Ví dụ, 1 người phát triển có vài commit chưa đượcđược triển khai đến staging, staging có vài commit chưa được triển khai tới production. Nhưng tất cả chúng đều chia sẻ cùng codebase, vì thế làm chún có thể được định dạng như là các triển khai khác nhau của cùng 1 ứng dụng.


## II. Phụ thuộc

### Khai báo rõ ràng và tách biệt các phụ thuộc

Hầu hết các ngôn ngữ lập trình cho phép 1 hệ thống packaging phân phối các thư viện hỗ trợ, như là CPAN cho Perl hoặc [Rubygems][11] chu Ruby. Các thư viện được cài đặt thông qua hệ thống packaging có thể được cài đặt mức-hệ-thống ( được hiểu như là “các site package”) hoặc chỉ trọng phạm vi thư mục chứa ứng dụng ( được biết như là “vendoring” hoặc “bundling”).

1 ứng dụng theo 12-chuẩn không bao giờ phụ thuộc vào các tồn tại ngầm của các gọi mức-hệ-thống. Nó khai báo tất cả các phụ thuộc, hoàn thiện và chính xác, thống qua 1 biểu thị khai báo phụ thuộc. Hơn nữa, nó sử dụng 1 công cụ tách biệt phụ thuộc trong quá trình thực thi để đảm bảo rằng không có phụ thuộc ngầm nào “lọt vào” từ các hệ thống xung quanh. Đặc tả phụ thuộc đầy đủ và rõ ràng được áp dụng thống nhất cho cả productiont và developer.

Để ví dụ, [Bundler][12] cho Ruby cung cấp một `Gemfile` để định dạng một khai báo phụ thuộc và `bundle exec` để cô lập sự phụ thuộc đó. Trong Python có 2 công cụ riêng biệt cho các bước này 

- [Pip][13] được sử dụng để khai báo và [Virtualenv][14] để tách biệt chúng. Và C cũng có [Autoconf][15] để khai báo phụ thuộc, đường dẫn tĩnh có thể cung cấp việc tách biệt các phụ thuộc. Bất kể dùng tập công cụ nào, khai báo và tách biệt phụ thuộc luôn phải được sử dụng cùng nhau - chỉ một là không đủ để thỏa mãn 12-chuẩn.

1 lợi ích của việc khai báo phụ thuộc rõ ràng là nó đơn giản hóa việc cài đặt cho những người phát triển mới của ứng dụng. Người phát triển mới có thể lấy codebase của ứng dụng về máy phát triển của họ, với điều kiện tiên quyết chỉ yêu cầu ngôn ngữữ chạy và quản lý phụ thuộc được cài đặt. Họ sẽ cóó thể cài đặt bất cứ thứ gì cần thiết để chạy code của ứng dụng với 1 lệnh _xây dựng xác định_. Ví dụ, lấy xây dựng cho Ruby/Bundler là `bundle install`, trong khi đó với Clojure/[Leiningen][16] lại là `lein deps`.

Các ứng dụng theo 12-chuẩn cũng không phụ thuộc vào các tồn tại ngầm của bất kỳ công cụ hệ thống nào. Ví dụ bao gồm cả với ImageMagick hay `curl`. Trong khi nhưng công cụ này có thể tồn tại trên nhiều hay thậm chí là hầu hết các hệ thống, không có gì bảo đảm rằng chúng sẽ tồn tại trên tất cả các hệ thống mà ứng dụng có thể chạy trong tương lại, hay 1 phiên bản được tìm thấy trong hệ thống tương lai có thể tương thích với ứng dụng. Nếu ứng dụng cần 1 công cụ hệ thống, công cụ đó cần được gán vào ứng dụng.



## III. Cấu hình

### Lưu cấu hình trong môi trường

_Cấu hình_ của 1 ứng dụng là bất cứ thứ gì giống như là sự thay đổi giữa [deploys][17] (môi trường staging, production, developer, etc). Nó bao gồm:

* Nguồn xử lý cơ sở dữ liệu, Memcaches, và các  [backing services][18]
* Thông tin đăng nhập cho các dịch vụ bên ngoài như Amazon S3 hoặc Twitter
* Các giá trị của mỗi-triển-khai như là tên máy chủ của triển khai

Các ứng dụng đôi khi lưu trữ cấu hình như các hằngằng số trong code. Điều này xung đột với 12-chuẩn, khi nó yêu cầu  **tính tách biệt 1 cách nghiêm ngặt trong cấu hình ở code**. Cấu hình thay đổi giữa các triển khai, còn code thì không.

1 kiểm thử thí nghiệm kiểm tra ứng dụng có tất cả cấuấu hình chính xác với code là khi codebae có thể tạo mã nguồn mở bất cứ khi nào, mà không ảnh hưởng bất cứ thông tin xác thực nào.


Lưu ý rằng định nghĩa "config"  **không** bao gồm các cấu hình nội bộ ứng dụng, như là `config/routes.rb` trong Rails, hoặc cách mà [các code modules kết nối][19] trong [Spring][20]. Loại cấu hình này không thay đổi giữa các triển khai, và ví thể hoàn thiện nhất trong code.

1 cách tiếp cận khác với cấu hình là sử dụng các file cấu hình không được kiểm soát trong kiểm soát thay đổi, như là  `config/database.yml` trong Rails. Đây là 1 cải thiện lớn so với việc sử dụng các hằng số được kiểm soát trongong code repo, nhưng vẫn còn những nhược điểm: nó rất dễ kiểm soát lỗi 1 file cấu hình trong repo, có 1 xu hướng là các file cấu hình sẽ nằm rải rác ở những chỗ khác nhau và trong những định dạng khác nhau, khiên nó khó để thấy và kiểm soát tất cả các cấu hình trong 1 chỗ. Hơn nữa những định dạng này có xu hướng riêng biệt với từng ngôn ngữ hay framework.

**Ứng dụng theo 12-chuẩn lưu trữ cấu trình trong _các biến môi trường_** (thường gọi tắt mà _env vars_ hoặc _env_). ENV vars thường dễ thay đổi giữa các triển khai mà không phải thay đổi code, không giống như các file cấu hình, chỉ 1 chút thay đổi của chúng được kiểm soát phụ thuocj bởi code repo; và không giống các file cấu hình thông thường, các cơ chế cấu hình khác như Java System Properties, chúng là các chuẩn ngôn ngữ và OS-agnostic.

1 khía cạnh khác của quản lý cấu hình là gom nhóm. Đôi khi các ứng dụng gom cấu hình thành các tên nhóm ( thường gọi là “các môi trường”) được đặt tên sau các triển khai cụ thể, như là môi trường `development`, `test`, và  `production` trong Rails. Phương pháp này không được gọn cho lắm: vì sẽ có nhiều triển khai của ứng dụng được tạo ra, các tên môi trường mới sẽ cần thiết, như là `staging` hoặc `qa`. Vì dự án sẽ càng phát triển hơn, các người phát triển có thể sẽ thêm các môi trường đặc biệt riêng của họ như `joes-staging`, kết quả của việc kết hợp đống cấu hình này sẽ khiến việc quản lý triển khải của ứng dụng trở nên mong manh.

Trong các ứng dụng 12-chuẩn, env vars là các điều khiển chi tiết, mỗi chúng trực giao đầy đủ với các env vars khác. Chúng không vào giờ được nhóm lại với nhau như làà "các môi trường", nhưng thay vào đó chúng quản lý độc lập cho từng triển khai. Đây là 1 mô hình nâng cao sự mượt mà, ứng dụng sự mở rộng 1 cách tự nhiên đến nhiều triển khai hơn trong suốt vòng đời của nó.


## IV. Dịch vụ sao lưu

### Điều chỉnh dich vụ sao lưu dưới dạng tài nguyên đính kèm

Một _dịch vụ sao lưu_ là một dịch vụ mà ứng dụng sử dụng trên mạng như là một phần trong các hoạt động thông thường của nó.Ví dụ như gồm có các kho dữ liệu (như là [MySQL][21] hoặc [CouchDB][22]), hệ thống nhắn tin/hàng đợi (như là [RabbitMQ][23] hoặc [Beanstalkd][24]), dịch vụ SMTP cho việc gửi đi các emai (như là [Postfix][25]), và các hệ thống caching (như mà [Memcached][26]).

dịch vụ sao lưu giống như là một cơ sở dữ liệu được quản lý theo kiểu truyền thống giống kiểu một hệ thống administrator giống như là việc thực hiện trên thời gian thực của ứng dụng. Ngoài các dịch vụ được quản lý một cách cục bộ như này, ứng dụng cũng có thể có các dịch vụ được cung cấp và quản lý bởi 1 bên thứ ba khác. Ví dụ như dịch vụ SMTP (như là [Postmark][27]), các dịch vụ thu thập dữ liệu (như là [New Relic][28] hoặc [Loggly][29]), binary asset services (such as [Amazon S3][30]), and even API-accessible consumer services (such as [Twitter][31], [Google Maps][32], or [Last.fm][33]).

**Code của ứng dụng theo chuẩn 12 yếu tố không có sự phân biệt giữa local và dịch vụ bên thứ 3.** Đối với ứng dụng, cả 2 đều là dạng tài nguyên đính kèm, truy cập thông qua các URL hoặc phương thức định vị, xác thực khác được lưu trữ trong config [config][34]. Một [deploy][35] của ứng dụng tuân thủ 12 yếu tố có thể hoán đổi giữa cơ sở dữ liệu MySQL cục bộ với một hệ quản trị cơ sở dữ liệu cung cấp bởi bên thứ 3(như là [Amazon RDS][36]) mà không cần bất kỳ sự thay đổi nào vào trong code của ứng dụng. Tương tự vậy, một server SMTP local có thể đổi chỗ với một dịch vụ SMPT của bên thứ 3 (như là Postmark) mà không cần thay đổi code. Trong cả 2 trường hợp, chỉ có tài nguyên xử lý trong cấu hình cần phải thay đổi.

Mỗi dịch vụ sao lưu riêng biệt là một _nguồn tài nguyên_. Ví dụ, một cơ sở dữ liệu MySQL là một nguồn tài nguyên; hai cơ sở dữ liệu MySQL (sử dụng tiến trình lưu trữ dữ liệu tại lớp ứng dụng) đủ điều kiện làm 2 nguồn tài nguyên khác nhau. ứng dụng tuân thủ 12 yếu tố xử lý các cơ sở dữ liệu này như nguồn tài nguyên đính kèm, với sự lỏng lẻo trong khớp nối giữa chúng để triển khai việc gắn chúng vào ứng dụng.

![A production deploy attached to four backing services.][37]

Các tài nguyên có thể được đính kèm và tách ra khỏi việc deploy nếu muốn. Ví dụ, nếu cơ sở dữ liệu của ứng dụng  hoạt động không chuẩn do các vấn đề về phần cứng, người quản trị của ứng dụng có thể chuyển sang một máy chủ cơ sở dữ liệu mới đã được restore từ bản sao lưu gần nhất. Cơ sở dữ liệu production hiện tại có thể được tách ra và đính kèm một cơ sở dữ liệu mới - tất cả đều không cần thay đổi bất kỳ code nào.

## V. Build, release, run

### các giai đoạn xây dựng và chạy hoàn toàn khác biệt

Một [codebase][38] được chuyển lên giai đoạn triển khai ( không phát triển ) trải qua 3 giai đoạn:

* _Giai đoạn xây dựng_ là việc chuyển đổi mà nó convert từ các tập code vào một gói thực thi được gọi là _xây dựng_. Sử dụng các phiên bản code tại các commit xác định bởi quá trình triển khai, giai đoạn xây dựng tìm nạp các gói cung cấp [phụ thuộc][39], biên dịch các tệp nhị phân và các đính kèm.
*  _Giai đoạn phát hành_ xây dựng bản dựng bởi giai đoạn xây dựng và kết hợp nó với[config][40] hiện tại của phiên bản triển khai. Kết quả của _phát hành_ chứa cả phiên bản dựng và các cấu hình và nó sẵn sàng thực hiện ngay lập tức trong môi trường thực thi.
* _Giai đoạn chạy_ (được biết như là "runtime") chạy ứng dụng trên môi trường thực thi, bằng cách đưa ra một số [quy trình][41] của ứng dụng khác biệt so với 1 phiên bản phát hành đã chọn.

![Code becomes a build, which is combined with config to create a release.][42]

**Ứng dụng tuân theo 12 yếu tố sử dụng phân chia chặt chẽ giữa các giai đoạn bản dựng, giai đoạn phát hành và giai đoạn chạy.** Ví dụ, nó không thể thay đổi mã nguồn khi đã chạy, vì không có cách nào để công bố những thay đổi đó quay trở lại giai đoạn xây dựng.

Các công cụ triển khai tường cung cấp cả các công cụ quản lý việc phát hành,đáng chú ý nhất là khả năng quay trở lại phiên bản phát hành trước đó. Ví dụ, [Capistrano][43] triển khai công cụ lưu trữ các phiên bản phát hành trong 1 thư mục con có tên là  `releases`, nơi mà phiên bản phát hành hiện tạilà một liên kết tượng trưng đến thư mục của bản phát hành hiện tại. Nó giúp cho lệnh `rollback` thực hiện dễ dàng hơn để quay lại phiên bản trước nó 1 các nhanh chóng.

Mọi bản phát hành phải luôn có ID duy nhất, như là mốc thời gian của bản phát hành (ví dụ như `2011-04-06-20:32:17`) hoặc 1 số tự tăng (như là `v100`). Bản phát hành là phiên bản chỉ có thể thêm vào và bản phát hành không thể  có những đột biến khi nó được tạo ra. Mọi thay đổi đều phải tạo ra 1 phiên bản phát hành mới.

Các phiên bản đựng được bắt đầu bơi các lập trình viên của ứng dụng đó khi code mới được triển khai.Theo thời gian chạy, ngược lại, có thể xảy ra trong các trường hợp tự động như là khởi động lại server, hoặc quá trình bị lỗi đang được các tiến trình quản lý quy trình khởi động lại. Vì thế, giai đoạn chạy nên giữ các thành phần ít bị di chuyển nhất có thể,vì các vấn đề ngăn chặn một ứng dụng đang chạy có thể bị gián đoạn ngay giữa đêm khi không có lập trình viên nào có mặt. Giai đoạn xây dựng có thể phức tạp hơn, vì các lỗi luôn ở trước mặt cho một nhà phát triển có thể thúc đẩy việc triển khai.


## VI. Tiến trình

### Thực thi ứng dụng dưới dạng một hoặc nhiều tiến trình không có trạng thái

ứng dụng được thực thi trong môi trường thực thi như là 1 hoặc nhiều  _tiến trình_.

Trng trường hợp đơn giản nhất, code là một tập lệnh độc lập, môi trường thực thi là laptop cục bộ của lập trình viên với một ngôn ngữ được cài đặt chạy theo thời gian, and the process is launched via the command line (for example, `python my_script.py`). On the other end of the spectrum, a production deploy of a sophisticated app may use many [process types, instantiated into zero or more running processes][44].

**Twelve-factor processes are stateless and [share-nothing][45].** Any data that needs to persist must be stored in a stateful [backing service][46], typically a database.

The memory space or filesystem of the process can be used as a brief, single-transaction cache. For example, downloading a large file, operating on it, and storing the results of the operation in the database. The twelve-factor app never assumes that anything cached in memory or on disk will be available on a future request or job – with many processes of each type running, chances are high that a future request will be served by a different process. Even when running only one process, a restart (triggered by code deploy, config change, or the execution environment relocating the process to a different physical location) will usually wipe out all local (e.g., memory and filesystem) state.

Asset packagers like [django-assetpackager][47] use the filesystem as a cache for compiled assets. A twelve-factor app prefers to do this compiling during the [build stage][48]. Asset packagers such as [Jammit][49] and the [Rails asset pipeline][50] can be configured to package assets during the build stage.

Some web systems rely on ["sticky sessions"][51] – that is, caching user session data in memory of the app's process and expecting future requests from the same visitor to be routed to the same process. Sticky sessions are a violation of twelve-factor and should never be used or relied upon. Session state data is a good candidate for a datastore that offers time-expiration, such as [Memcached][52] or [Redis][53].

## VII. Port binding

### Export services via port binding

Web apps are sometimes executed inside a webserver container. For example, PHP apps might run as a module inside [Apache HTTPD][54], or Java apps might run inside [Tomcat][55].

**The twelve-factor app is completely self-contained** and does not rely on runtime injection of a webserver into the execution environment to create a web-facing service. The web app **exports HTTP as a service by binding to a port**, and listening to requests coming in on that port.

In a local development environment, the developer visits a service URL like `http://localhost:5000/` to access the service exported by their app. In deployment, a routing layer handles routing requests from a public-facing hostname to the port-bound web processes.

This is typically implemented by using [dependency declaration][56] to add a webserver library to the app, such as [Tornado][57] for Python, [Thin][58] for Ruby, or [Jetty][59] for Java and other JVM-based languages. This happens entirely in _user space_, that is, within the app's code. The contract with the execution environment is binding to a port to serve requests.

HTTP is not the only service that can be exported by port binding. Nearly any kind of server software can be run via a process binding to a port and awaiting incoming requests. Examples include [ejabberd][60] (speaking [XMPP][61]), and [Redis][62] (speaking the [Redis protocol][63]).

Note also that the port-binding approach means that one app can become the [backing service][64] for another app, by providing the URL to the backing app as a resource handle in the [config][65] for the consuming app.
## VIII. Concurrency

### Scale out via the process model

Any computer program, once run, is represented by one or more processes. Web apps have taken a variety of process-execution forms. For example, PHP processes run as child processes of Apache, started on demand as needed by request volume. Java processes take the opposite approach, with the JVM providing one massive uberprocess that reserves a large block of system resources (CPU and memory) on startup, with concurrency managed internally via threads. In both cases, the running process(es) are only minimally visible to the developers of the app.

![Scale is expressed as running processes, workload diversity is expressed as process types.][66]

**In the twelve-factor app, processes are a first class citizen.** Processes in the twelve-factor app take strong cues from [the unix process model for running service daemons][67]. Using this model, the developer can architect their app to handle diverse workloads by assigning each type of work to a _process type_. For example, HTTP requests may be handled by a web process, and long-running background tasks handled by a worker process.

This does not exclude individual processes from handling their own internal multiplexing, via threads inside the runtime VM, or the async/evented model found in tools such as [EventMachine][68], [Twisted][69], or [Node.js][70]. But an individual VM can only grow so large (vertical scale), so the application must also be able to span multiple processes running on multiple physical machines.

The process model truly shines when it comes time to scale out. The [share-nothing, horizontally partitionable nature of twelve-factor app processes][71] means that adding more concurrency is a simple and reliable operation. The array of process types and number of processes of each type is known as the _process formation_.

Twelve-factor app processes [should never daemonize][72] or write PID files. Instead, rely on the operating system's process manager (such as [systemd][73], a distributed process manager on a cloud platform, or a tool like [Foreman][74] in development) to manage [output streams][75], respond to crashed processes, and handle user-initiated restarts and shutdowns.

## IX. Disposability

### Maximize robustness with fast startup and graceful shutdown

**The twelve-factor app's [processes][76] are _disposable_, meaning they can be started or stopped at a moment's notice.** This facilitates fast elastic scaling, rapid deployment of [code][77] or [config][78] changes, and robustness of production deploys.

Processes should strive to **minimize startup time**. Ideally, a process takes a few seconds from the time the launch command is executed until the process is up and ready to receive requests or jobs. Short startup time provides more agility for the [release][79] process and scaling up; and it aids robustness, because the process manager can more easily move processes to new physical machines when warranted.

Processes **shut down gracefully when they receive a [SIGTERM][80]** signal from the process manager. For a web process, graceful shutdown is achieved by ceasing to listen on the service port (thereby refusing any new requests), allowing any current requests to finish, and then exiting. Implicit in this model is that HTTP requests are short (no more than a few seconds), or in the case of long polling, the client should seamlessly attempt to reconnect when the connection is lost.

For a worker process, graceful shutdown is achieved by returning the current job to the work queue. For example, on [RabbitMQ][81] the worker can send a [`NACK`][82]; on [Beanstalkd][83], the job is returned to the queue automatically whenever a worker disconnects. Lock-based systems such as [Delayed Job][84] need to be sure to release their lock on the job record. Implicit in this model is that all jobs are [reentrant][85], which typically is achieved by wrapping the results in a transaction, or making the operation [idempotent][86].

Processes should also be **robust against sudden death**, in the case of a failure in the underlying hardware. While this is a much less common occurrence than a graceful shutdown with `SIGTERM`, it can still happen. A recommended approach is use of a robust queueing backend, such as Beanstalkd, that returns jobs to the queue when clients disconnect or time out. Either way, a twelve-factor app is architected to handle unexpected, non-graceful terminations. [Crash-only design][87] takes this concept to its [logical conclusion][88].

## X. Dev/prod parity

### Keep development, staging, and production as similar as possible

Historically, there have been substantial gaps between development (a developer making live edits to a local [deploy][89] of the app) and production (a running deploy of the app accessed by end users). These gaps manifest in three areas:

* **The time gap:** A developer may work on code that takes days, weeks, or even months to go into production.
* **The personnel gap**: Developers write code, ops engineers deploy it.
* **The tools gap**: Developers may be using a stack like Nginx, SQLite, and OS X, while the production deploy uses Apache, MySQL, and Linux.

**The twelve-factor app is designed for [continuous deployment][90] by keeping the gap between development and production small.** Looking at the three gaps described above:
* Make the time gap small: a developer may write code and have it deployed hours or even just minutes later.
* Make the personnel gap small: developers who wrote code are closely involved in deploying it and watching its behavior in production.
* Make the tools gap small: keep development and production as similar as possible.

Summarizing the above into a table:

| ----- |
|  |  Traditional app |  Twelve-factor app |  
| Time between deploys |  Weeks |  Hours |  
| Code authors vs code deployers |  Different people |  Same people |  
| Dev vs production environments |  Divergent |  As similar as possible | 

[Backing services][91], such as the app's database, queueing system, or cache, is one area where dev/prod parity is important. Many languages offer libraries which simplify access to the backing service, including _adapters_ to different types of services. Some examples are in the table below.

| ----- |
| Type |  Language |  Library |  Adapters |  
| Database |  Ruby/Rails |  ActiveRecord |  MySQL, PostgreSQL, SQLite |  
| Queue |  Python/Django |  Celery |  RabbitMQ, Beanstalkd, Redis |  
| Cache |  Ruby/Rails |  ActiveSupport::Cache |  Memory, filesystem, Memcached | 

Developers sometimes find great appeal in using a lightweight backing service in their local environments, while a more serious and robust backing service will be used in production. For example, using SQLite locally and PostgreSQL in production; or local process memory for caching in development and Memcached in production.

**The twelve-factor developer resists the urge to use different backing services between development and production**, even when adapters theoretically abstract away any differences in backing services. Differences between backing services mean that tiny incompatibilities crop up, causing code that worked and passed tests in development or staging to fail in production. These types of errors create friction that disincentivizes continuous deployment. The cost of this friction and the subsequent dampening of continuous deployment is extremely high when considered in aggregate over the lifetime of an application.

Lightweight local services are less compelling than they once were. Modern backing services such as Memcached, PostgreSQL, and RabbitMQ are not difficult to install and run thanks to modern packaging systems, such as [Homebrew][92] and [apt-get][93]. Alternatively, declarative provisioning tools such as [Chef][94] and [Puppet][95] combined with light-weight virtual environme9nts such as [Docker][96] and [Vagrant][97] allow developers to run local environments which closely approximate production environments. The cost of installing and using these systems is low compared to the benefit of dev/prod parity and continuous deployment.

Adapters to different backing services are still useful, because they make porting to new backing services relatively painless. But all deploys of the app (developer environments, staging, production) should be using the same type and version of each of the backing services.

## XI. Logs

### Treat logs as event streams

_Logs_ provide visibility into the behavior of a running app. In server-based environments they are commonly written to a file on disk (a "logfile"); but this is only an output format.

Logs are the [stream][98] of aggregated, time-ordered events collected from the output streams of all running processes and backing services. Logs in their raw form are typically a text format with one event per line (though backtraces from exceptions may span multiple lines). Logs have no fixed beginning or end, but flow continuously as long as the app is operating.

**A twelve-factor app never concerns itself with routing or storage of its output stream.** It should not attempt to write to or manage logfiles. Instead, each running process writes its event stream, unbuffered, to `stdout`. During local development, the developer will view this stream in the foreground of their terminal to observe the app's behavior.

In staging or production deploys, each process' stream will be captured by the execution environment, collated together with all other streams from the app, and routed to one or more final destinations for viewing and long-term archival. These archival destinations are not visible to or configurable by the app, and instead are completely managed by the execution environment. Open-source log routers (such as [Logplex][99] and [Fluentd][100]) are available for this purpose.

The event stream for an app can be routed to a file, or watched via realtime tail in a terminal. Most significantly, the stream can be sent to a log indexing and analysis system such as [Splunk][101], or a general-purpose data warehousing system such as [Hadoop/Hive][102]. These systems allow for great power and flexibility for introspecting an app's behavior over time, including:

* Finding specific events in the past.
* Large-scale graphing of trends (such as requests per minute).
* Active alerting according to user-defined heuristics (such as an alert when the quantity of errors per minute exceeds a certain threshold).


## XII. Admin processes

### Run admin/management tasks as one-off processes

The [process formation][03] is the array of processes that are used to do the app's regular business (such as handling web requests) as it runs. Separately, developers will often wish to do one-off administrative or maintenance tasks for the app, such as:

* Running database migrations (e.g. `manage.py migrate` in Django, `rake db:migrate` in Rails).
* Running a console (also known as a [REPL][104] shell) to run arbitrary code or inspect the app's models against the live database. Most languages provide a REPL by running the interpreter without any arguments (e.g. `python` or `perl`) or in some cases have a separate command (e.g. `irb` for Ruby, `rails console` for Rails).
* Running one-time scripts committed into the app's repo (e.g. `php scripts/fix_bad_records.php`).

One-off admin processes should be run in an identical environment as the regular [long-running processes][105] of the app. They run against a [release][106], using the same [codebase][107] and [config][108] as any process run against that release. Admin code must ship with application code to avoid synchronization issues.

The same [dependency isolation][109] techniques should be used on all process types. For example, if the Ruby web process uses the command `bundle exec thin start`, then a database migration should use `bundle exec rake db:migrate`. Likewise, a Python program using Virtualenv should use the vendored `bin/python` for running both the Tornado webserver and any `manage.py` admin processes.

Twelve-factor strongly favors languages which provide a REPL shell out of the box, and which make it easy to run one-off scripts. In a local deploy, developers invoke one-off admin processes by a direct shell command inside the app's checkout directory. In a production deploy, developers can use ssh or other remote command execution mechanism provided by that deploy's execution environment to run such a process.


[1]: http://www.heroku.com/
[2]: http://blog.heroku.com/archives/2011/6/28/the_new_heroku_4_erosion_resistance_explicit_contracts/
[3]: https://books.google.com/books/about/Patterns_of_enterprise_application_archi.html?id=FyWZt5DdvFkC
[4]: https://books.google.com/books/about/Refactoring.html?id=1MsETFPD3I0C
[5]: http://git-scm.com/
[6]: https://www.mercurial-scm.org/
[7]: http://subversion.apache.org/
[8]: https://12factor.net/images/codebase-deploys.png
[9]: https://12factor.net/dependencies
[10]: http://www.cpan.org/
[11]: http://rubygems.org/
[12]: https://bundler.io/
[13]: http://www.pip-installer.org/en/latest/
[14]: http://www.virtualenv.org/en/latest/
[15]: http://www.gnu.org/s/autoconf/
[16]: https://github.com/technomancy/leiningen#readme
[17]: https://12factor.net/codebase
[18]: https://12factor.net/backing-services
[19]: http://docs.spring.io/spring/docs/current/spring-framework-reference/html/beans.html
[20]: http://spring.io/
[21]: http://dev.mysql.com/
[22]: http://couchdb.apache.org/
[23]: http://www.rabbitmq.com/
[24]: http://kr.github.com/beanstalkd/
[25]: http://www.postfix.org/
[26]: http://memcached.org/
[27]: http://postmarkapp.com/
[28]: http://newrelic.com/
[29]: http://www.loggly.com/
[30]: http://aws.amazon.com/s3/
[31]: http://dev.twitter.com/
[32]: https://developers.google.com/maps/
[33]: http://www.last.fm/api
[34]: https://12factor.net/config
[35]: https://12factor.net/codebase
[36]: http://aws.amazon.com/rds/
[37]: https://12factor.net/images/attached-resources.png
[38]: https://12factor.net/codebase
[39]: https://12factor.net/dependencies
[40]: https://12factor.net/config
[41]: https://12factor.net/processes
[42]: https://12factor.net/images/release.png
[43]: https://github.com/capistrano/capistrano/wiki
[44]: https://12factor.net/concurrency
[45]: http://en.wikipedia.org/wiki/Shared_nothing_architecture
[46]: https://12factor.net/backing-services
[47]: http://code.google.com/p/django-assetpackager/
[48]: https://12factor.net/build-release-run
[49]: http://documentcloud.github.com/jammit/
[50]: http://ryanbigg.com/guides/asset_pipeline.html
[51]: http://en.wikipedia.org/wiki/Load_balancing_%28computing%29#Persistence
[52]: http://memcached.org/
[53]: http://redis.io/
[54]: http://httpd.apache.org/
[55]: http://tomcat.apache.org/
[56]: https://12factor.net/dependencies
[57]: http://www.tornadoweb.org/
[58]: http://code.macournoyer.com/thin/
[59]: http://www.eclipse.org/jetty/
[60]: http://www.ejabberd.im/
[61]: http://xmpp.org/
[62]: http://redis.io/
[63]: http://redis.io/topics/protocol
[64]: https://12factor.net/backing-services
[65]: https://12factor.net/config
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