# Xác thực và phân quyền trong Microservices

Xác thực và phân quyền luôn là thành phần không thể thiếu trong mọi hệ thống, nhưng mức độ áp dụng thì lại tùy thuộc vào từng giai đoạn. Nếu bạn làm mọi thứ chặt chẽ ngay từ đầu, nó có thể làm tăng độ phức tạp và làm chậm sự phát triển của công ty. Nhưng nếu bạn làm nó quá muộn, thì có thể bạn sẽ hứng chịu nguy cơ bị tấn công và rủi ro từ đó.

## Bắt đầu từ Monolithic

Thông thường ở hệ thống monolithic sẽ có 1 module chung quản lý việc xác thực và phân quyền, mỗi user sau khi đăng nhập sẽ được cấp cho 1 session ID duy nhất để định danh. Phái client có thể lưu Session ID lại lưu dưới dạng cookie và gửi kèm nó trong mọi request.

Hệ thống sau đó dùng Session ID được gửi đi để xác định danh tính của user truy cập, để người dùng không cần phải đăng nhập lại.

![Untitled](Xa%CC%81c%20thu%CC%9B%CC%A3c%20va%CC%80%20pha%CC%82n%20quye%CC%82%CC%80n%20trong%20Microservices%20506433da420d4fb08aff01881f032803/Untitled.png)

Khi Session ID được gửi lên, server sẽ xác định được danh tính của người dùng gắn với Session ID đó, đồng thời sẽ kiểm tra quyền của user xem có được truy cập tác vụ đó hay không. Giải pháp session và cookie vẫn có thể sử dụng, tuy nhiên ngày nay chúng ta có yêu cầu nhiều hơn, chẳng hạn như các ứng dụng Hybrid hoặc SPA (Single Page Application) có thể cần truy cập tới nhiều hệ thống backend khác nhau, vì vậy session và cookie lấy từ 1 server có thể không sử dụng được ở server khác.

### Bài toán khó trong Microservices

Trong kiến trúc Microservices, hệ thống được chia nhỏ thành nhiều hệ thống con, đảm nhận các nghiệp vụ và chức năng khác nhau. Mỗi hệ thống con đó cũng cần được xác thực và phân quyền, nếu xử lý theo cách của kiến trúc Monolithic ở trên chúng ta sẽ gặp các vấn đề sau:

- Mỗi service có nhu cầu phải tự thực hiện xác thực và phân quyền ở service của mình. Mặc dù có thể sử dụng các thư viện giống nhau nhưng chi phí để bảo trì thư viện chung đó với nhiều nền tảng ngôn ngữ khác nhau là quá lớn.
- Mỗi service nên tập trung vào xây dựng các nghiệp vụ của mình, việc xây dựng thêm logic về phân quyền làm giảm tốc độ phát triển và tăng độ phức tạp của các service.
- Các service thông thường sẽ cung cấp các interface dưới dạng RESTful API, sử dụng protocol HTTP. Các HTTP request sẽ được đi qua nhiều thành phần của hệ thống. Các truyền thống sử dụng session ở server (stateful) sẽ gây khó khăn cho việc mở rộng hệ thống theo chiều ngang.
- Service sẽ được truy cập từ nhiều ứng dụng và đối tượng sử dụng khác nhau, có thể là người dùng, 1 thiết bị phần cứng, 3rd - party, crontab hay 1 service khác. Việc định danh và phân quyền ở nhiều ngữ cảnh khác nhau như vậy là vô cùng phức tạp.

![Untitled](Xa%CC%81c%20thu%CC%9B%CC%A3c%20va%CC%80%20pha%CC%82n%20quye%CC%82%CC%80n%20trong%20Microservices%20506433da420d4fb08aff01881f032803/Untitled%201.png)

## Định danh

### Sử dụng JWT

Json Web Token - JWT là 1 loại token sử dụng chuẩn mở dùng để trao đổi thông tin kèm theo các HTTP request. Thông tin này được xác thực và đánh dấu 1 cách tin cậy dựa vào chữ ký. JWT có rất nhiều ưu điểm so với session.

- Stateless, thông tin không được lưu trữ trên server.
- Dễ dàng phát triển, mở rộng.
- Performance tốt hơn do server đọc thông tin ngay trong request (nếu session thì cần đọc ở storage hoặc database).

![Untitled](Xa%CC%81c%20thu%CC%9B%CC%A3c%20va%CC%80%20pha%CC%82n%20quye%CC%82%CC%80n%20trong%20Microservices%20506433da420d4fb08aff01881f032803/Untitled%202.png)

## Mã hóa RSA cho JWT

Phần chữ ký sẽ được mã hóa lại bằng HMAC hoặc RSA:

- HMAC: đối tượng khởi tạo JWT (Token Issuer) và đầu nhận JWT (token verifier) sử dụng chung 1 private key để mã hóa và kiểm tra.
- RSA: sử dụng 1 cặp key, đối tưởng khởi tạo JWT sử dụng Private để mã hóa, đầu nhận JWT sử dụng Public Key để kiểm tra.

## Sử dụng Opaque Token khi muốn kiểm soát phiên làm việc tốt hơn

Opaque Token (còn được gọi là stateful token) là dạng token không chứa thông tin trong nó, thông thường là 1 chuỗi ngẫu nhiên và yêu cầu 1 service trung gian kiểm tra và lấy thông tin. Ví dụ:

```json
{"access_token": "c2hr8Jgp5jBn-TY7E14HRuO37hEK1o_IOfDzbnZEO-o.zwh2f8SPiLKbcMbrD_DSgOTd3FIfQ8ch2bYSFi8NwbY","expires_in": 3599,"token_type": "bearer"}
```

Transparent Token (còn được gọi là stateless token thông thường chính là dạng JWT, token bản thân chứa thông tin và không cần 1 service trung gian để kiếm tra. Hãy so sanh 2 loại token này:

| Tiêu chí | Opaque Token | Transparent Token |
| --- | --- | --- |
| Tốc độ | Kém, vì phải kiểm tra thông tin qua auth server | Nhanh, chỉ cần private key là được |
| Mã hóa | Không lộ thông tin nhạy cảm trong nội dung token | Có thể xem được thông tin trong nội dung token |
| Tính hợp lệ | Mọi token đều phải được auth server kiểm tra | Token sẽ hợp lệ cho tới khi hết hạn |

Như vậy chúng ta có thể thấy Transparent Token mang lại tốc độ tốt hơn, đơn giản dễ sử dụng với cả 2 phía, không phụ thuộc vào 1 server trung tâm để kiểm tra. Còn Opaque Token kiểm soát tốt hơn các phiên làm việc của đối tượng, chẳng hạn khi bạn muốn thoát tất cả các thiết bị đang đang nhập.

## OAuth 2

Các tokens sẽ được khởi tạo thông qua OAuth 2, là phương thức xác thực phổ biến nhất hiện nay, mà qua đó một service, hay 1 ứng dụng bên thứ 3 có thể đại diện (delegation) cho người dùng truy cập vào 1 tài nguyên của người dùng nằm trong 1 dịch vụ nào đó.

OAuth 2 là chuẩn mở, có đầy đủ tư liệu, thư viện ở tất cả ngôn ngữ khác nhau giúp cho việc tích hợp, phát triển dựa trên nó trở nên dễ dàng và nhanh chóng

## Kiến trúc cho xác thực và phân quyền

Sau khi đã có định danh và giao thức dùng để giao tiếp, câu hỏi tiếp theo là cần trả lời câu hỏi đối tượng với định danh đó có quyền thực hiện 1 hành động, truy cập 1 tài nguyên nào đó hay không. Bên cạnh các service được xây dựng mới, vẫn còn tồn tại các hệ thống cũ (legacy) chạy song song, thế nên hiện nay có 2 cách tổ chức phân quyền như dưới đây.

## Xác thực, phân quyền tại lớp rìa

![Untitled](Xa%CC%81c%20thu%CC%9B%CC%A3c%20va%CC%80%20pha%CC%82n%20quye%CC%82%CC%80n%20trong%20Microservices%20506433da420d4fb08aff01881f032803/Untitled%203.png)

Theo mô hình tất cả mọi request sẽ được xác thực khi đi qua API Gateway hoặc BFF (backend for frontend). BFF chính là lớp service ở rìa được thiết kế riêng cho từng ứng dụng. Chúng ta sẽ đặt xác thực và phân quyền ở lớp rìa này.

- API Gateway sẽ bắt buộc tất cả request sẽ cần gửi kèm token để định danh
- Nếu token này là JWT (đối với OpenID Connect), Gateway có thể kiểm tra tính hợp lệ của token thông qua chữ ký (signature), thông tin (claim) hoặc đối tượng khởi tạo (issuer)
- Nếu token này là Opaque Token, Gateway có thể phân tích token, đổi lấy JWT và truyền tiếp vào trong cho các services.
- API Gateway hoặc BFF kiểm tra các policy xem có hợp lệ hay không thông qua authorization server trung tâm.
- Các microservices không thực hiện lớp xác thực và phân quyền nào, có thể tự do truy cập bên trong vùng nội bộ (internal network)

![Untitled](Xa%CC%81c%20thu%CC%9B%CC%A3c%20va%CC%80%20pha%CC%82n%20quye%CC%82%CC%80n%20trong%20Microservices%20506433da420d4fb08aff01881f032803/Untitled%204.png)

Mô hình này có điểm tương đồng với kiến trúc Monolithic khi đặt xác thực phân quyền tại 1 số service nhất định, việc xây dựng và bảo trì sẽ tốn chi phí nhỏ hơn, tuy nhiên sẽ để lộ 1 khoảng trống bảo mật rất lớn ở lớp trong do các service có thể tự do truy cập lập nhau.

## Xác thực, phân quyền tại các service

Ở mô hình này, mỗi service khi được thiết kế và xây dựng các giao tiếp APIs mở rộng được và có thể phục vụ cho public internet. 

Để làm được việt này, vài trò rất lớn sẽ nằm ở service IAM (Identity Access Management), IAM nắm giữ các định danh của toàn bộ đối tượng (user, service, command…) cùng với các bộ luật phân quyen2 chi tiết.

![Untitled](Xa%CC%81c%20thu%CC%9B%CC%A3c%20va%CC%80%20pha%CC%82n%20quye%CC%82%CC%80n%20trong%20Microservices%20506433da420d4fb08aff01881f032803/Untitled%205.png)

Việc mỗi service phải tự thực hiện việc xác thực, phân quyền sẽ làm tăng chi phí xây dựng các service, bên ngoài các nghiệp vụ chính thì cần thêm lớp middleware để giao tiếp với IAM. Tuy nhiên các service sẽ có được sự tự chủ hoàn toàn, chủ động về việc cung cấp tài nguyên cho các đối tượng, và tăng tốc phát triển hơn vì nhiều trường hợp client có thể truy cập thẳng tới các service mà không cần phát triển thêm lớp BFF ở giữa

## Access Control

Xây dựng hệ thống (rules) hiệu quả không bao giờ là dễ dàng, khi yêu cầu về nghiệp vụ tăng cao kéo theo yêu cầu về phân quyền càng phức tạp. Hãy lấy 1 ví dụ cụ thể để làm rõ, mỗi ứng dụng thông thường sẽ gán quyền cho 1 thành viên cụ thể. Mở rộng ra trong 1 hệ thống Microservices, đối tượng ở đây có thể là người dụng, service, crontab,…

Có 1 vài cách tiếp cập cho việc phân quyền như trên, hãy thử đi qua cách khác nhau để có nhiều góc nhìn khác nhau.

## Access control list (ACL)

![Untitled](Xa%CC%81c%20thu%CC%9B%CC%A3c%20va%CC%80%20pha%CC%82n%20quye%CC%82%CC%80n%20trong%20Microservices%20506433da420d4fb08aff01881f032803/Untitled%206.png)

Trong ví dụ trên bạn có thể thấy 1 ma trận của đối tượng và quyền, nó gần tương đương với cách quản lý file trên Linux (chmod) và phù hợp với những ứng dụng có ít đối tượng. Khi hệ thống lớn lên mô hình này sẽ không thể quản lý nổi bởi ma trận được tạo ra quá lớn và phức tạp. Do vậy mô hình này không còn phổ biến hiện tại.

## Role-based access control (RBAC)

RBAC liên kết đối tượng với các vai trò (role), và từ vai trò tới các quyền. Chẳng hạn vai trò Administrator có thể thừa hưởng mọi quyền và vài trò Manager, có điều này giúp làm giảm độ phức tạp của ma trận quyền, thay vì gán toàn bộ quyền cho Administrator thì chỉ cần cho Administrator thừa hưởng các quyền của Manager.

![Untitled](Xa%CC%81c%20thu%CC%9B%CC%A3c%20va%CC%80%20pha%CC%82n%20quye%CC%82%CC%80n%20trong%20Microservices%20506433da420d4fb08aff01881f032803/Untitled%207.png)

RBAC rất phổ biến và bạn có thể thấy ở mọi nơi, so với ACL thì RBAC giảm thiểu độ phức tạp khi số lượng đối tượng + quyền tăng cao. Tuy nhiên RBAC chưa thỏa mãn được 1 số trường hợp, ví dụ khi cấp quyền 1 sản phẩm được sửa bởi người tạo, người dùng nằm trong 1 phòng ban xác định hoặc quyền phân biệt với người dùng từ nhiều hệ thống (tenant) khác nhau.

## Policy-Based Access Control (PBAC)

PBAC được xây dựng trên Attribute Based Access Control (ABAC), qua đó định nghĩa các quyền để diễn đạt mộ yêu cầu được cho phép hay từ chối. ABAC sử dụng các thuộc tính (attribute) để mô tả cho đối tượng cần kiểm tra, mỗi thuộc tính là 1 cặp key-value, ví dụ Department = Marketing. Nhờ đó ABAC có thể giúp phân quyền mịn hơn, phù hợp với nhiều ngữ cảnh (context) và nghiệp vụ (business rule) khác nhau.

![Untitled](Xa%CC%81c%20thu%CC%9B%CC%A3c%20va%CC%80%20pha%CC%82n%20quye%CC%82%CC%80n%20trong%20Microservices%20506433da420d4fb08aff01881f032803/Untitled%208.png)

PBAC được định nghĩa thông qua các policy được viết dưới dạng 1 ngôn ngữ chung XACML (Extensible Access Control Markup Language). Một policy định nghĩa 4 đối tượng subject, effect, action và resource. Nhìn qua thì nó gần giống với cách định nghĩa 1 ACL

```json
{"subjects": ["user:john"],"effect": "allow","actions": ["catalog:delete"]"resources": ["product:john-leman"],}
```

Chúng ta có thể bổ sung subject, action cũng như resource thêm vào policy nếu muốn.

```json
{"subjects": ["user:john", "user:katy", "user:perry"],"effect": "allow","actions": ["catalog:delete", "catalog:update", "catalog:publish"]"resources": ["product:john-leman", "product:john-doe"]}
```

### Luật ưu tiên

- Mặc định nếu không có policy phù hợp, yêu cầu sẽ bị từ chối
- Nếu không có policy nào **deny**, có ít nhất 1 policy **allow** thì yêu cầu được cho phép
- Nếu có 1 policy là **deny**, thì yêu cầu luôn bị từ chối.

### Regular Expression

Các policy cho phép khai báo sử dụng regular expression, như ở ví dụ này cho phép tất cả người dùng được xem thông tin product.

```json
{"subjects": ["user:<.*>"],"effect": "allow","actions": ["catalog:read],"resources": ["product:<.*>"]}
```

### Điều kiện

Các policy có thể bổ sung các điều kiện để thu hẹp phạm vi của quyền, ví dụ như chỉ áp dụng cho 1 dải IP nhất định, hoặc chỉ cho phép người tạo sản phẩm được sửa sản phẩm đó.

```json
{"subjects": ["user:ken"],"actions" : ["catalog:delete", "catalog:create", "catalog:update"],"effect": "allow","resources": ["products:<.*>"],"conditions": {"IpAddress": {"addresses": ["192.168.0.0/16"]}}}
```

### Tổng kết

Việc liên tục mở rộng nghiệp vụ và hệ thống đòi hỏi các service phải tự xác thực, qua đó không phân biệt các service đó là bên trong (internal) hay bên ngoài (external), giúp các team dễ dàng mở rộng tích hợp với nhau. Việc này đòi hỏi mô hình xác thực dung phải hoạt động ổng định, tối ưu và đáp ứng được hiệu năng cao.
