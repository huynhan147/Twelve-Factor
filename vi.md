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

Động lực của chúng tôi là nâng cao nhận thức về vài vấn đề có hệ thống chúng tôi đã gặp trong phát triển ứng dụng hiện đại, để cung cấp các từ khoá chung để bàn luận về những vấn đề này, và để cho phép 1 tập các giải pháp khái niệm rộng lớn vào các vấn đề với các thuật ngữ kèm theo. Định dạng này lấy cảm hứng bởi cuốn sách của Martin Fowler, [_Patterns of Enterprise Application Architecture][3]_ and [_Refactoring][4]_.


## Ai nên đọc tài liệu này?

Bất kỳ người phát triển nào đang xây dựng các ứng dụng chạy như là 1 dịch vụ. Thông thường các kĩ sư triển khai hoặc điều khiển những ứng dụng như vậy.

## I. Codebase

### codebase được theo dõi trong việc kiểm soát các thay đổi, nhiều triển khai.

1 ứng dụng tuân theo 12-chuẩn luôn luôn được theo dõi trong 1 hệ thống kiểm soát các phiên bản, như là [Git][5], [Mercurial][6], hoặc [Subversion][7]. 1 bản sao chéo của cơ sở dữ liệu theo dõi thay đổi được biết như là 1 code repository, thường được viết tắt là _code repo_ hoặc là _repo_.

 _codebase_ có thể là bất cứ repo đơn lẻ nào nào (trong 1 hệ thống kiểm soát sửa đổi tập trung như Subverrsion), hoặc có thể lả 1 tập các repo chia sẻ chung 1 commit gốc(trong 1 hệ thống kiểm soát sửa đổi có tính chất phân cấp như Git).

![một bản đồ codebase áp dụng cho nhiều triển khai][8]

Luôn luôn có sự tương quan giữa codebase và ứng dụng:

* Nếu có nhiều codebase, đó không phải là 1 ứng dụng - nó là 1 hệ thống phân cấp. Mỗi thành phần là 1 hệ thống được phân cấp trong 1 ứng dụng, và mỗi thành phần có thể tuân theo 12-chuẩn 1 cách riêng lẻ.
* Nhiều ứng dụng chia sẻ cùng một code là 1 sư vi phạm 12-chuẩn. Giải pháp ở đây là quản lý chia sẻ code vào các thư viện mà có thể được thêm vào thông qua [phụ thuộc vào quản lý][9].

Chỉ có duy nhất 1 codebase mỗi ứng dụng, nhưng sẽ có nhiều triển khải của ứng dụng. 1 triển khai sẽ chạy phiên bản của ứng dụng. Điều này thường là 1 phía của production, với 1 hoặc nhiều phía staging. Thêm vào đó, mỗi người phát triển có 1 bản sao của ứng dụng đang chạy trên môi trường developer nội bộ của họ, mỗi số chúng cũng được coi như 1 triển khai.

Codebase là chung giữa tất cả các triển khai, mặc dù các phiên bản khác nhau có thể hoạt động trong mỗi triển khai. Ví dụ, 1 người phát triển có vài commit chưa được triển khai đến staging, staging có vài commit chưa được triển khai tới production. Nhưng tất cả chúng đều chia sẻ cùng codebase, vì thế làm chúng có thể được định dạng như là các triển khai khác nhau của cùng 1 ứng dụng.


## II. Phụ thuộc

### Khai báo rõ ràng và tách biệt các phụ thuộc

Hầu hết các ngôn ngữ lập trình cho phép 1 hệ thống packaging phân phối các thư viện hỗ trợ, như là CPAN cho Perl hoặc [Rubygems][11] chu Ruby. Các thư viện được cài đặt thông qua hệ thống packaging có thể được cài đặt mức-hệ-thống ( được hiểu như là “các site package”) hoặc chỉ trọng phạm vi thư mục chứa ứng dụng ( được biết như là “vendoring” hoặc “bundling”).

1 ứng dụng theo 12-chuẩn không bao giờ phụ thuộc vào các tồn tại ngầm của các  package hệ thống. Nó khai báo tất cả các phụ thuộc, hoàn thiện và chính xác, thống qua 1 biểu thị khai báo phụ thuộc. Hơn nữa, nó sử dụng 1 công cụ tách biệt phụ thuộc trong quá trình thực thi để đảm bảo rằng không có phụ thuộc ngầm nào “lọt vào” từ các hệ thống xung quanh. Đặc tả phụ thuộc đầy đủ và rõ ràng được áp dụng thống nhất cho cả production và developer.

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

Trong các ứng dụng 12-chuẩn, env vars là các điều khiển chi tiết, mỗi chúng trực giao đầy đủ với các env vars khác. Chúng không vào giờ được nhóm lại với nhau như là "các môi trường", nhưng thay vào đó chúng quản lý độc lập cho từng triển khai. Đây là 1 mô hình nâng cao sự mượt mà, ứng dụng sự mở rộng 1 cách tự nhiên đến nhiều triển khai hơn trong suốt vòng đời của nó.


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

Trng trường hợp đơn giản nhất, code là một tập lệnh độc lập, môi trường thực thi là laptop cục bộ của lập trình viên với một ngôn ngữ được cài đặt chạy theo thời gian, và quá trình này được đưa ra thông qua chế độ command line (ví dụ như, `python my_script.py`). Ở mặt ngược lại, triển khai một production của ứng dụng phức tạp có thể sử dụng nhiều [loại tiến trình, khởi tạo từ 0 haowjc nhiều hơn các tiền trình cần chạy][44].

**Các tiến trình tuân thủ 12 yếu tố thường không có trạng thái và [không chia sẻ bất cứ thứ gì][45].** Bất kỳ dữ liệu nào cần lưu trữ đều phải được lưu trong một [dịch vụ lưu trữ][46] có trạng thái, thường là một cơ sở dữ liệu.

Không gian bộ nhớ hoặc file hệ thống của tiến trình có thể được sử dụng như là bộ đệm nhỏ gọn, một chiều. Ví dụ, việc tải xuống 1 file lớn, vận hành trên nó, và lưu trữ kết quả của hoạt động này vào trong cơ sở dữ liệu. Ứng dụng tuân thủ 12 yếu tố không bao giờ giả định bất cứ thứ gì được cache trong bộ nhớ hoặc trên đĩa sẽ khả dụng đối với các request hoặc công việc trong tương lai – với nhiều tiến trình của từng loại đang chạy, cơ hội cao là các request trong tương lai sẽ được phục vụ bởi một tiến trình khác.Ngay cả khi chỉ chạy một tiến trình, việc khởi động lại (được kích hoạt bằng cách triển khai trong code, thay đổi cấu hình, hoặc di chuyển môi trường thực thi tiến trình sang một vị trí vật lý khác) thường sẽ xoá sạch các trạng thái cục bộ (ví dụ như bộ nhớ và file hệ thống).

Các đóng gói như [django-assetpackager][47] sử dụng các file hệ thống làm bộ nhớ đệm để biên dịch các asset. Ứng dụng tuân thủ 12 quy tắc thích việc biên dịch này hơn là là thực hiện trong [giai đoạn xây dựng][48]. Các đóng gói như [Jammit][49] và [Rails asset pipeline][50]có thể được cấu hình thành các asset package trong giai đoạn xây dựng.

Một vìa hệ thống web dựa vào ["sticky sessions"][51] – đó là, lưu trữ dữ liệu về các phiên người dùng trong bộ nhớ đệm của tiến trình thuộc ứng dụng và đợi các yêu cầu trong tương lai từ người truy cập tương tự khi điều hướng vào cùng một tiến trình. Các session cố định là vi phạm nguyên tắc 12 yếu tố và không bao giờ được sử dụng hay phụ thuộc vào nó. Các dữ liệu ở trang thái session là ứng cử viên tốt cho kho dữ liệu có khả năng cung cấp thời gian hết hạn, như là [Memcached][52] hoặc [Redis][53].

## VII. Ràng buộc cổng

### Cung cấp dịch vụ thông qua cổng ràng buộc

Các ứng dụng web đôi khi được thực hiện bên trong một webserver container. Ví dụ, Các ứng dụng PHP có thể chạy dưới dạng module bên trong [Apache HTTPD][54], hoặc các ứng dụng Java web có thể chạy trong [Tomcat][55].

**ứng dụng tuân thủ 123 yêu tố hoàn toàn khép kín** và không dựa vào thời gian chạy của máy chủ trong môi trường thực thi để tạo ra một dịch vụ web-facing. Một ứng dụng web **cung cấp HTTP như là một dịch vụ bằng cách ràng buộc nó với một cổng**,và lắng nghe các request trên cổng đó.

Trong môi trường phát triển cục bộ, lập trình viên truy cập vào một dịch vụ URL như `http://localhost:5000/` để truy cập vào dịch vụ được cung cấp bởi ứng dụng của họ. Trong triển khai, một lớp định tuyến xử lý các yêu cầu định tuyến từ tên máy chủ công khai tới tiến trình web đã được ràng buộc các cổng.

Điều này thường được cài đặt bằng cách sử dụng [khai báo phụ thuộc][56] để thêm thư việc webserver voà ứng dụng, như là  [Tornado][57] cho Python, [Thin][58] cho Ruby, hoặc [Jetty][59] cho Java và một vài ngôn ngữ JVM-based khác. Điều này sảy ra hoàn toàn trong _không gian người dùng_, đó là, trong code của ứng dụng. Các quy ước với môi trương thực thi được kết nối tới một cổng để xử lý các yêu cầu.

HTTP không phải dịch vụ duy nhất có thể được cung cấp bằng cách kết nối qua cổng. Gần đây thì bất kỳ loại phần mềm máy chủ nào cũng có thể chạy thông qua một tiến trình kết nối với một cổng và đợi các yêu cầu. Những ví dụ bao gồm có  [ejabberd][60] (thông qua [XMPP][61]), và [Redis][62] (thông qua [Redis protocol][63]).

Cần lưu ý rằng phương pháp kết nối cổng cũng có nghĩa là một ứng dụng có thể trở thành [dịch vụ nền][64] cho ứng dụng khác, bằng cách cung cấp URL cho ứng dụng nền như một tài nguyên để xử lý trong [cấu hình][65] dành cho ứng dụng sử dụng.


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