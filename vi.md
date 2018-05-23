Giới thiệu
============

Trong thời đại hiện nay, phần mềm thường được phân phối như là một dịch vụ: được gọi là _web apps_, hay là _software-as-a-service_. Ứng dụng twelve-factor là một phương pháp để xây dựng các ứng dụng phần mềm như một dịch vụ:

* Sử dụng các định dạng **khai báo** cho việc thiết lập tự động để giảm thời gian và giá thành cho các developer mới khi tham gia dự án;
* Có một **quy định rõ ràng** với hệ điều hành cơ bản,cung cấp ** khả năng linh hoạt tối đa ** giữa các môi trường thực thi;
* Phù hợp cho ** triển khai ** trên nền tảng đám mây ** hiện đại **, tránh sự cần thiết cho các máy chủ và quản trị hệ thống;
* ** Giảm thiểu sự khác biệt ** giữa phát triển và sản xuất, cho phép ** triển khai liên tục ** cho sự nhanh nhẹn tối đa;
* Và có thể ** mở rộng quy mô ** mà không thay đổi đáng kể về công cụ, kiến ​​trúc hoặc thực tiễn phát triển.

Phương pháp 12 yếu tố có thể được áp dụng cho các ứng dụng được viết bằng bất kỳ ngôn ngữ lập trình nào và sử dụng bất kỳ kết hợp dịch vụ sao lưu nào (cơ sở dữ liệu, hàng đợi, bộ nhớ cache, v.v.).

Những người đóng góp cho tài liệu này đã trực tiếp tham gia vào việc phát triển và triển khai hàng trăm ứng dụng, và gián tiếp chứng kiến ​​sự phát triển, vận hành và mở rộng hàng trăm nghìn ứng dụng thông qua công việc của chúng tôi trên nền tảng [Heroku] (http: // www. heroku.com/).

Tài liệu này tổng hợp tất cả các trải nghiệm và quan sát của chúng tôi về một loạt các ứng dụng phần mềm như một dịch vụ trong tự nhiên. Nó là một __triangulation__ về thực hành ý tưởng cho phát triển ứng dụng, đặc biệt chú ý đến sự năng động của sự phát triển hữu cơ của một ứng dụng theo thời gian, sự cộng tác giữa các nhà phát triển làm việc trên codebase của ứng dụng và [tránh chi phí xói mòn phần mềm] (http : //blog.heroku.com/archives/2011/6/28/the_new_heroku_4_erosion_resistance_explicit_contracts/).

Động lực của chúng tôi là nâng cao nhận thức về một số vấn đề hệ thống mà chúng tôi đã thấy trong phát triển ứng dụng hiện đại, để cung cấp vốn từ vựng chung cho việc thảo luận những vấn đề đó và cung cấp một loạt giải pháp khái niệm rộng cho những vấn đề kèm theo thuật ngữ. Định dạng được lấy cảm hứng từ sách của Martin Fowler _ [Mẫu Kiến trúc Ứng dụng Doanh nghiệp] (https://books.google.com/books/about/Patterns_of_enterprise_application_archi.html?id=FyWZt5DdvFkC) _ và _ [Tái cấu trúc] (https: // books.google.com/books/about/Refactoring.html?id=1MsETFPD3I0C)_.

Ai nên đọc tài liệu này?
==============================

Bất kỳ developer xây dựng ứng dụng nào hoạt động như một dịch vụ.Các kỹ sư ops triển khai hoặc quản lý các ứng dụng như vậy.

I. Codebase
-----------

### Một codebase được theo dõi trong kiểm soát sửa đổi, nhiều triển khai


Một ứng dụng twelve-factor luôn được theo dõi trong một hệ thống kiểm soát phiên bản, chẳng hạn như [Git] (http://git-scm.com/), [Mercurial] (https://www.mercurial-scm.org/), hoặc [Subversion] (http://subversion.apache.org/). Bản sao của cơ sở dữ liệu theo dõi sửa đổi được gọi là _code repository_, thường được rút ngắn thành _code repo_ hoặc chỉ _repo_.

Một _codebase_ là bất kỳ repo duy nhất (trong một hệ thống kiểm soát sửa đổi tập trung như Subversion), hoặc bất kỳ bộ repos cùng chia sẻ một cam kết gốc (trong một hệ thống kiểm soát sửa đổi phân cấp như Git).

![Một bản đồ codebase cho nhiều triển khai](/images/codebase-deploys.png)


Luôn có mối tương quan một-một giữa codebase và ứng dụng:

* Nếu có nhiều codebases, nó không phải là một ứng dụng - đó là một hệ thống phân tán. Mỗi thành phần trong một hệ thống phân tán là một ứng dụng và mỗi thành phần có thể tuân thủ riêng với twelve-factor.
* Nhiều ứng dụng chia sẻ cùng một code là vi phạm twelve-factor. Giải pháp ở đây là để chuyển code chia sẻ thành các thư viện có thể được đưa vào thông qua [trình quản lý phụ thuộc] (./dependencies).


Chỉ có một codebase cho mỗi ứng dụng, nhưng sẽ có nhiều triển khai ứng dụng. _deploy_ là phiên bản đang chạy của ứng dụng. Đây thường là một trang web production và một hoặc nhiều trang web staging. Ngoài ra, mọi nhà phát triển đều có một bản sao của ứng dụng đang chạy trong môi trường phát triển cục bộ của họ, mỗi một trong số đó cũng đủ điều kiện để triển khai.

Các codebase là như nhau trên tất cả các triển khai, mặc dù các phiên bản khác nhau có thể hoạt động trong mỗi triển khai. Ví dụ, một nhà phát triển có một số commit chưa triển khai đến staging; staging có một số commit chưa được triển khai đến production. Nhưng tất cả đều chia sẻ cùng một codebase, do đó làm cho chúng có thể nhận dạng như các triển khai khác nhau của cùng một ứng dụng.

II. Các phụ thuộc
----------------

### Khai báo rõ ràng và cô lập phụ thuộc

Hầu hết các ngôn ngữ lập trình đều cung cấp một hệ thống đóng gói để phân phối các thư viện hỗ trợ, chẳng hạn như [CPAN] (http://www.cpan.org/) cho Perl hoặc [Rubygems] (http://rubygems.org/) cho Ruby. Các thư viện được cài đặt thông qua hệ thống đóng gói có thể được cài đặt trên toàn hệ thống (được gọi là “gói trang web”) hoặc được đưa vào thư mục chứa ứng dụng (được gọi là “bán hàng” hoặc “gói”).

** Một ứng dụng 12-factor không bao giờ dựa vào sự tồn tại ngầm của các gói toàn hệ thống. ** Nó khai báo tất cả các phụ thuộc, hoàn toàn và chính xác, thông qua một biểu thị _dependency declaration_. Hơn nữa, nó sử dụng công cụ tách biệt _dependency trong quá trình thực hiện để đảm bảo rằng không có phụ thuộc ngầm nào bị "rò rỉ" từ hệ thống xung quanh. Đặc tả phụ thuộc đầy đủ và rõ ràng được áp dụng thống nhất cho cả sản xuất và phát triển.

Ví dụ, [Bundler] (https://bundler.io/) dành cho Ruby cung cấp định dạng biểu thị `Gemfile` để khai báo phụ thuộc và` bundle exec`, cô lập chúng. Trong Python có hai công cụ riêng biệt cho các bước này - [Pip] (http://www.pip-installer.org/en/latest/) được sử dụng để khai báo và [Virtualenv] (http://www.virtualenv.org / vi / mới nhất /), cách ly. Thậm chí C có [Autoconf] (http://www.gnu.org/s/autoconf/) để khai báo phụ thuộc, và liên kết tĩnh có thể cung cấp sự cô lập phụ thuộc. Bất kể chuỗi công cụ, khai báo phụ thuộc và cách ly phải luôn luôn được sử dụng cùng nhau - chỉ có một hoặc khác là không đủ để đáp ứng mười hai yếu tố.


Một lợi ích của khai báo phụ thuộc rõ ràng là nó đơn giản hóa việc thiết lập cho các nhà phát triển mới cho ứng dụng. Nhà phát triển mới có thể kiểm tra codebase của ứng dụng trên máy phát triển của họ, chỉ yêu cầu trình quản lý phụ thuộc và thời gian chạy ngôn ngữ được cài đặt làm điều kiện tiên quyết. Họ sẽ có thể thiết lập mọi thứ cần thiết để chạy mã của ứng dụng bằng lệnh _build xác định. Ví dụ, lệnh xây dựng cho Ruby / Bundler là `bundle install`, trong khi cho Clojure / [Leiningen] (https://github.com/technomancy/leiningen#readme) nó là` lein deps`.

Các ứng dụng 12-factor cũng không phụ thuộc vào sự tồn tại tiềm ẩn của bất kỳ công cụ hệ thống nào. Các ví dụ bao gồm việc kích hoạt ImageMagick hoặc `curl`. Mặc dù các công cụ này có thể tồn tại trên nhiều hoặc thậm chí hầu hết các hệ thống, không có gì đảm bảo rằng chúng sẽ tồn tại trên tất cả các hệ thống mà ứng dụng có thể chạy trong tương lai hoặc phiên bản được tìm thấy trên hệ thống tương lai sẽ tương thích với ứng dụng. Nếu ứng dụng cần trình bao ra một công cụ hệ thống, công cụ đó sẽ được đưa vào ứng dụng.

III. Cấu hình
-----------

### Lưu trữ cấu hình trong môi trường

Một _cấu hình_ của ứng dụng là mọi thứ có khả năng thay đổi trong khi [triển khai] (./ codebase) (dàn dựng, sản xuất, môi trường nhà phát triển, v.v.). Điều nay bao gôm:


* Tài nguyên xử lý cơ sở dữ liệu, Memcached và [dịch vụ sao lưu] khác (./backing-services)
* Thông tin đăng nhập cho các dịch vụ bên ngoài như Amazon S3 hoặc Twitter
* Giá trị mỗi lần triển khai chẳng hạn như tên máy chủ chuẩn cho việc triển khai

Đôi khi, các ứng dụng lưu trữ cấu hình như là hằng số trong code. Đây là vi phạm của 12-factor, yêu cầu **tách cấu hình chính xác khỏi mã**. Cấu hình khác nhau đáng kể trên triển khai, code thì không.

Một bài kiểm tra litmus xem một ứng dụng có tất cả các cấu hình một cách chính xác với các yếu tố code là liệu codebase có thể được làm nguồn mở tại bất kỳ thời điểm nào, mà không ảnh hưởng đến bất kỳ thông tin đăng nhập nào.

Lưu ý rằng định nghĩa "config" này **không** bao gồm cấu hình ứng dụng nội bộ, chẳng hạn như `config / routes.rb` trong Rails, hoặc cách [mô-đun mã được kết nối] (http://docs.spring.io/ spring / docs / current / spring-framework-reference / html / beans.html) trong [Spring] (http://spring.io/). Loại cấu hình này không thay đổi giữa triển khai và do đó được thực hiện tốt nhất trong code.


Một cách tiếp cận khác để cấu hình là sử dụng các tệp cấu hình không được kiểm tra trong điều khiển sửa đổi, chẳng hạn như `config / database.yml` trong Rails. Đây là một cải tiến lớn so với việc sử dụng các hằng số được kiểm tra trong code repo, nhưng vẫn có điểm yếu: rất dễ nhầm lẫn khi kiểm tra tệp cấu hình cho repo; có một xu hướng cho các tập tin cấu hình được phân tán ở những nơi khác nhau và các định dạng khác nhau, làm cho nó khó khăn để xem và quản lý tất cả các cấu hình ở một nơi. Hơn nữa, các định dạng này có xu hướng đặc biệt về ngôn ngữ hoặc khung.


** Cửa hàng ứng dụng 12-factor cấu hình trong các biến _environment_ ** (thường được rút ngắn thành _env vars_ hoặc _env_). Env vars dễ thay đổi giữa triển khai mà không thay đổi bất kỳ mã nào; không giống như các tập tin cấu hình, có rất ít khả năng chúng được kiểm tra vào mã repo một cách vô tình; và không giống như các tệp cấu hình tùy chỉnh, hoặc các cơ chế cấu hình khác như thuộc tính hệ thống Java, chúng là một tiêu chuẩn bất khả tri về ngôn ngữ và hệ điều hành.


Một khía cạnh khác của quản lý cấu hình là nhóm. Đôi khi, ứng dụng định cấu hình hàng loạt thành các nhóm được đặt tên (thường được gọi là "môi trường") được đặt tên theo các triển khai cụ thể, chẳng hạn như môi trường `development`,` test` và `production` trong Rails. Phương pháp này không quy mô rõ ràng: vì nhiều triển khai ứng dụng được tạo, các tên môi trường mới là cần thiết, chẳng hạn như `staging` hoặc` qa`. Khi dự án phát triển hơn nữa, các nhà phát triển có thể thêm các môi trường đặc biệt của riêng mình như `joes-staging`, kết quả là một vụ nổ tổ hợp của cấu hình giúp quản lý triển khai ứng dụng rất dễ vỡ.

Trong một ứng dụng 12-factor, env vars là điều khiển chi tiết, mỗi hoàn toàn trực giao với các env vars khác. Chúng không bao giờ được nhóm lại với nhau thành “môi trường”, mà thay vào đó được quản lý độc lập cho từng triển khai. Đây là một mô hình có quy mô thuận lợi khi ứng dụng tự nhiên mở rộng sang nhiều triển khai hơn suốt thời gian tồn tại của nó.
