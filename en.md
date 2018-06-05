
<<<<<<< HEAD
=======
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


## IV. Dịch vụ nền

### Điều chỉnh dich vụ nền dưới dạng tài nguyên đính kèm

Một _dịch vụ nền_ là một dịch vụ mà ứng dụng sử dụng trên mạng như là một phần trong các hoạt động thông thường của nó.Ví dụ như gồm có các kho dữ liệu (như là [MySQL][21] hoặc [CouchDB][22]), hệ thống nhắn tin/hàng đợi (như là [RabbitMQ][23] hoặc [Beanstalkd][24]), dịch vụ SMTP cho việc gửi đi các emai (như là [Postfix][25]), và các hệ thống caching (như mà [Memcached][26]).

dịch vụ nền giống như là một cơ sở dữ liệu được quản lý theo kiểu truyền thống giống kiểu một hệ thống administrator giống như là việc thực hiện trên thời gian thực của ứng dụng. Ngoài các dịch vụ được quản lý một cách cục bộ như này, ứng dụng cũng có thể có các dịch vụ được cung cấp và quản lý bởi 1 bên thứ ba khác. Ví dụ như dịch vụ SMTP (như là [Postmark][27]), các dịch vụ thu thập dữ liệu (như là [New Relic][28] hoặc [Loggly][29]), binary asset services (such as [Amazon S3][30]), and even API-accessible consumer services (such as [Twitter][31], [Google Maps][32], or [Last.fm][33]).

**Code của ứng dụng theo chuẩn 12 yếu tố không có sự phân biệt giữa local và dịch vụ bên thứ 3.** Đối với ứng dụng, cả 2 đều là dạng tài nguyên đính kèm, truy cập thông qua các URL hoặc phương thức định vị, xác thực khác được lưu trữ trong config [config][34]. Một [deploy][35] của ứng dụng tuân thủ 12 yếu tố có thể hoán đổi giữa cơ sở dữ liệu MySQL cục bộ với một hệ quản trị cơ sở dữ liệu cung cấp bởi bên thứ 3(như là [Amazon RDS][36]) mà không cần bất kỳ sự thay đổi nào vào trong code của ứng dụng. Tương tự vậy, một server SMTP local có thể đổi chỗ với một dịch vụ SMPT của bên thứ 3 (như là Postmark) mà không cần thay đổi code. Trong cả 2 trường hợp, chỉ có tài nguyên xử lý trong cấu hình cần phải thay đổi.

Mỗi dịch vụ nền riêng biệt là một _nguồn tài nguyên_. Ví dụ, một cơ sở dữ liệu MySQL là một nguồn tài nguyên; hai cơ sở dữ liệu MySQL (sử dụng tiến trình lưu trữ dữ liệu tại lớp ứng dụng) đủ điều kiện làm 2 nguồn tài nguyên khác nhau. ứng dụng tuân thủ 12 yếu tố xử lý các cơ sở dữ liệu này như nguồn tài nguyên đính kèm, với sự lỏng lẻo trong khớp nối giữa chúng để triển khai việc gắn chúng vào ứng dụng.

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
>>>>>>> 40358020fe93ee69495fc11292dbabff5176df11
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

**Một ứng dụng tuân thủ 12 yếu tố không bao giờ liên quan tới việc định tuyến hay luồng lưu trữ các dữ liệu output của nó .** Nó không nên cố gắng ghi hay quản lý các file logs. thay vào đó, mỗi tiến trình đang chạy tự ghi luồng sự kiện của nó, không bị cản trở bởi `stdout`. Trong quá trình phát triển tại local, nhà phát triển sẽ theo dõi luồng này qua những vấn đề xung quanh trên terminal của họ để quan sát các hành vi của ứng dụng.

TRong triển khai staging hoặc production, với mỗi luồng của tiến trình đều đưuọc theo dõi bởi tiến trình thưcj thi, đối chiếu với các luồng khác từ ứng dụng, và chuyển đến một hoặc nhiều điểm cuối để xem và  lưu trữ dài hạn. Các đích đến này thường không hiển thị hoặc được cấu hình bằng ứng dụng, và thay vào đó lại được quản lý hoàn toàn bằng môi trường thực thi. Bộ định tuyến các log mã nguồn mở (như là  [Logplex][99] và [Fluentd][100]) đã được cài đặt sẵn cho mục đích này.

Các luồng sự kiện cho 1 ứng dụng có thể được điều hướng đến 1 file, hay được theo dõi thông qua lệnh tail thời gian thực trong 1 terminal. Đáng kể nhất là , luồng này có thể gửi đến hệ thống và lập chỉ mục cho logs như là [Splunk][101], hoặc một hệ thống kho dữ liệu đa năng như là [Hadoop/Hive][102]. CÁc hệ thống này cung cấp công suất và tính linh hoạt tuyệt vời để xem xét hành vi của ứng dụng theo thời gian, kể cả:

* tìm một sự kiện cụ thể trong quá khứ.
* Vẽ lại đồ thị có quy mô lớn (chẳng hạn như là lượng yêu cầu mỗi phút).
* Cảnh báo hoạt động theo những chuẩn đoán do người dùng xác định (chẳng hạn như là cảnh báo khi số lượng lỗi trên phút vượt quá ngưỡng nhất định).


## XII. Các tiến trình Admin

### chạy các tác vụ admin/management như các tiến trình sử dụng một lần

The [Hệ thống tiến trình][03] là một loạt các tiến trìnhđược sử dụng để làm các hoạt động thông thường của ứng dụng (chẳng hạn như xử lý các yêu cầu web) khi nó chạy. Riêng biệt, các nhà phát triển chỉ muốn quản lý hay bảo trì 1 việc nào đó chỉ một lần, như là:

* chạy database migrations (e.g. `manage.py migrate` trong Django, `rake db:migrate` trong Rails).
* Chạy một console (còn được gọi là [REPL][104] shell) để chạy code tuỳ chỉnh hoặc kiểm tra các mô hình của ứng dụng dựa trên cơ sở dữ liệu trực tiếp. Hầu hết các ngôn ngữ cung cấp REPL bằng cách chạy trình thông dịch mà không có bất kì đối số nào (e.g. `python` hoặc `perl`) hoặc trong một số trường hợp có một lệnh riêng(e.g. `irb` cho Ruby, `rails console` cho Rails).
* Chạy các tập lệnh một lần được commit voà repo của ứng dụng (e.g. `php scripts/fix_bad_records.php`).

Tiến trình quản trị một lần nên được chạy trong môi trường giống hệt nhau như thường lệ [như các tiến trình dài hạn thông thường][105] của ứng dụng. Họ chạy ngược lại một bản [release][106], sử dụng như một [codebase][107] và [config][108] như bất kì quá trình nào chạy trên bản release đó. Code admin phải tương thích với code ứng dụng để tránh các vấn đề đồng bộ hóa.

Các kĩ thuật [cô lập sự phụ thuộc][109] giống nhau nên được sử dụng cho tất cả các loại tiến trình. ví dụ như , nếu tiến trình web của Ruby sử dụng câu lệnh `bundle exec thin start`, thì việc migration cơ sở dữ liệu nên sử dụng `bundle exec rake db:migrate`. Tương tự vậy, một chương trình Python sử dụng Virtualenv nên sử dụng các vendored `bin/python` để chạy cả máy chủ web Tornado và mới mọi tiến trình admin `manage.py`.

12-chuẩn đặc biệt ưa thích ngôn ngữ cung cấp REPL có hộp điều khiển, và những thứ nó dễ dàng để chạy các lệnh 1 lần. Trong triển khai cục bộ, các người phát triển gọi các tiến trình admin chạy 1 lần bằng các lệnh shell trực tiếp trong thư mục kieerm tra cả ứng dụng. Trong 1 triển khai production, các nhà phát triển có thể sử dụng ssh hay cơ chế thực thi các lệnh remote được cung cấp bởi môi trường thực thi của triển khai để chạy 1 tiến trình.


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
