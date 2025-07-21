# Tích hợp SMS cho các ứng dụng SaaS, ISV và nhiều người thuê với tính năng nhắn tin cho người dùng cuối AWS

> **📖 Bài viết gốc**: https://aws.amazon.com/blogs/messaging-and-targeting/sms-onboarding-for-saas-isv-and-multi-tenant-applications-with-aws-end-user-messaging/ > **👤 Tác giả**: Tyler Holmes
> **📅 Ngày xuất bản**: 13/5/2025  
> **🌐 Nguồn**: AWS Messaging Blog
> **👨‍💻 Người dịch**: Nguyễn Thành Long - FCJ Intern  
> **📅 Ngày dịch**: 7/7/2025
> **⏱️ Thời gian đọc**: 50 phút

---

## 📋 Tóm tắt

Blog này hướng dẫn chi tiết cách triển khai SMS cho các ứng dụng SaaS, ISV và multi-tenant sử dụng AWS End User Messaging. Nội dung tập trung vào việc giải quyết các thách thức phức tạp khi tích hợp SMS vào sản phẩm công nghệ.

**🎯 Đối tượng đọc**: Product managers, technical leads
**📊 Độ khó**: Intermediate
**🏷️ Tags**: AWS Pinpoint, SMS Gateway, Multi-Tenancy

---

## 📚 Mục lục

- [Tích hợp SMS cho các ứng dụng SaaS, ISV và nhiều người thuê với tính năng nhắn tin cho người dùng cuối AWS](#tích-hợp-sms-cho-các-ứng-dụng-saas-isv-và-nhiều-người-thuê-với-tính-năng-nhắn-tin-cho-người-dùng-cuối-aws)
  - [📋 Tóm tắt](#-tóm-tắt)
  - [📚 Mục lục](#-mục-lục)
    - [Tích hợp SMS cho các ứng dụng SaaS, ISV và nhiều người thuê với tính năng nhắn tin cho người dùng cuối AWS](#tích-hợp-sms-cho-các-ứng-dụng-saas-isv-và-nhiều-người-thuê-với-tính-năng-nhắn-tin-cho-người-dùng-cuối-aws-1)
      - [Giới thiệu](#giới-thiệu)
      - [Kiến trúc Multi-Tenant cho SMS](#kiến-trúc-multi-tenant-cho-sms)
      - [Định hình SMS của bạn cung cấp: Cân nhắc chiến lược](#định-hình-sms-của-bạn-cung-cấp-cân-nhắc-chiến-lược)
      - [Kết luận](#kết-luận)
  - [📖 Glossary - Thuật ngữ](#-glossary---thuật-ngữ)
  - [🔗 Tài liệu tham khảo](#-tài-liệu-tham-khảo)
    - [Tài liệu gốc](#tài-liệu-gốc)
    - [Tài liệu tiếng Việt](#tài-liệu-tiếng-việt)
    - [Tools và Services](#tools-và-services)
  - [💬 Ghi chú của người dịch](#-ghi-chú-của-người-dịch)
    - [Challenges trong quá trình dịch](#challenges-trong-quá-trình-dịch)
    - [Insights gained](#insights-gained)
  - [🤝 Đóng góp và Feedback](#-đóng-góp-và-feedback)

---

### Tích hợp SMS cho các ứng dụng SaaS, ISV và nhiều người thuê với tính năng nhắn tin cho người dùng cuối AWS

#### Giới thiệu

Nhắn tin SMS vẫn là một trong những kênh giao tiếp đáng tin cậy và hiệu quả nhất. Tuy nhiên, đối với các công ty Phần mềm như một Dịch vụ (SaaS), Nhà cung cấp Phần mềm Độc lập (ISV) và các nhà cung cấp giải pháp đa tenant muốn tích hợp khả năng SMS vào sản phẩm của mình, hành trình này có thể phức tạp và đầy thách thức.
Hướng dẫn này được thiết kế riêng cho các nhà cung cấp công nghệ – cho dù bạn là công ty SaaS, ISV hay bất kỳ nền tảng nào cho phép khách hàng của bạn gửi tin nhắn SMS đến người dùng cuối. Trong suốt bài viết, các thuật ngữ sau sẽ được sử dụng:
Nhà cung cấp (Provider): Tổ chức cung cấp khả năng SMS như một phần của sản phẩm/dịch vụ.
Khách hàng (Customer): Các đơn vị sử dụng công nghệ của Nhà cung cấp để gửi tin nhắn SMS.
Người dùng cuối (End User): Người nhận chọn tham gia nhận tin nhắn SMS từ Khách hàng. Việc triển khai SMS có thể phức tạp do các quy định riêng theo từng quốc gia, quy trình đăng ký kéo dài hàng tuần hoặc thậm chí hàng tháng, các loại trình gửi (Long Code, Short Code, Sender ID, v.v.) với khả năng khác nhau, cùng nhu cầu đa dạng của Khách hàng và Người dùng cuối. Những thách thức này càng lớn hơn khi bạn là Nhà cung cấp dịch vụ SMS cho Khách hàng của mình, và họ lại phục vụ Người dùng cuối của họ.

#### Kiến trúc Multi-Tenant cho SMS

Dưới đây là các mô hình kiến trúc để triển khai SMS, tùy theo nhu cầu kinh doanh và mối quan hệ với Khách hàng:

1. Mô hình "Bring Your Own AWS Account"
   Ai đăng ký và cấu hình?
   Khách hàng kết nối tài khoản AWS của họ, nên việc đăng ký và cấu hình diễn ra trong tài khoản Khách hàng.
   Thông tin đăng ký thường là của Khách hàng.
   Trách nhiệm của Khách hàng:
   Tự xử lý đăng ký, cấu hình.
   Tích hợp tài khoản với dịch vụ của Nhà cung cấp.
   Quản lý gửi tin, danh sách opt-out, v.v.
   Thanh toán hóa đơn AWS.
   Trách nhiệm của Nhà cung cấp:
   Cung cấp giao diện thân thiện gọi API AWS End User Messaging bằng thông tin xác thực của Khách hàng.
   Phù hợp cho: Khách hàng kỹ thuật muốn kiểm soát toàn bộ và đã sử dụng AWS.
2. Tài khoản Nhà cung cấp – Đăng ký & Cấu hình
   Thủ công Ai đăng ký và cấu hình?
   Nhà cung cấp sở hữu tài khoản và nhập thông tin Khách hàng thủ công.
   Trách nhiệm của Khách hàng:
   Cung cấp thông tin cần thiết cho Nhà cung cấp.
   Trách nhiệm của Nhà cung cấp:
   Thu thập thông tin đăng ký từ Khách hàng.
   Quản lý quy trình phức tạp thay mặt Khách hàng.
   Phù hợp cho: Nhà cung cấp có ít Khách hàng giá trị cao cần hỗ trợ đặc biệt.
3. Giải pháp Bán Tự động – Khách hàng Gửi Tin
   Ai đăng ký và cấu hình?
   Nhà cung cấp xây dựng cách để Khách hàng nộp thông tin đăng ký, sau đó tự động gửi đến nhà mạng/cơ quan quản lý.
   Trách nhiệm của Nhà cung cấp:
   Cung cấp quy trình nộp thông tin đơn giản (webhooks, biểu mẫu, API).
   Tự động gửi dữ liệu đăng ký.
   Quản lý cấu hình kỹ thuật và khả năng gửi tin.
   Phù hợp cho: Nhà cung cấp có trình độ kỹ thuật trung bình, muốn giảm rào cản nhưng vẫn tách biệt trách nhiệm tuân thủ.
4. Giải pháp Tự động Hoàn toàn – Nhà cung cấp Gửi Tin
   Ai đăng ký và cấu hình?
   Thông tin Khách hàng được sử dụng để đăng ký, Nhà cung cấp xử lý tự động
   Trách nhiệm của Nhà cung cấp:
   Cung cấp Điều khoản & Chính sách Bảo mật tuân thủ sẵn.
   Cung cấp quy trình opt-in đạt chuẩn (biểu mẫu web, kịch bản thoại, v.v.)
   . Xử lý mọi khía cạnh kỹ thuật của đăng ký.
   Phù hợp cho: Nhà cung cấp quy mô lớn phục vụ nhiều Khách hàng với trình độ kỹ thuật khác nhau.
5. Tin nhắn Tự động Giới hạn Mẫu
   Ai đăng ký và cấu hình?
   Thông tin Khách hàng được dùng để đăng ký, Nhà cung cấp xử lý tự động.
   Trách nhiệm của Nhà cung cấp:
   Cung cấp bộ mẫu tin nhắn được phê duyệt trước.
   Quản lý tuân thủ tập trung.
   Phù hợp cho: Các trường hợp có nhu cầu tin nhắn dễ dự đoán (nhắc hẹn, thông báo giao hàng, OTP).
6. Chương trình Quản lý Hoàn toàn
   Ai đăng ký và cấu hình?
   Khách hàng ủy quyền cho Nhà cung cấp gửi tin thay mặt họ, nên Nhà cung cấp sở hữu mối quan hệ với Người dùng cuối.
   Ví dụ: Dịch vụ thông báo giao hàng gửi tin: "ShipTrack: Đơn hàng từ ACME Corp sẽ giao vào ngày mai. Theo dõi tại [link]"
   Phù hợp cho: Các trường hợp chuyên biệt mà nền tảng của bạn đóng vai trò trung gian quan trọng.

#### Định hình SMS của bạn cung cấp: Cân nhắc chiến lược

1. Chiến lược Định giá
   Chi phí SMS thay đổi theo quốc gia, loại trình gửi và khối lượng. AWS tính phí dựa trên lượng tin gửi theo từng quốc gia. Bạn cần cân nhắc:
   Tín dụng SMS: Khách hàng mua tín dụng không phụ thuộc vào quốc gia đích.
   Phân bổ theo USD: Khách hàng có ngân sách cố định, chi phí tin nhắn trừ dần.
   Giá theo nhóm quốc gia: Chia thành các tầng (ví dụ: Bắc Mỹ, Tây Âu) với giá khác nhau.
   Tin nhắn đóng gói: Bao gồm một số lượng tin nhắn nhất định trong gói đăng ký.
   Cân nhắc Địa lý
   Các quốc gia có quy định khác nhau về:
   Loại trình gửi được hỗ trợ.
   Yêu cầu đăng ký.
   Giờ giới hạn gửi tin (quiet hours).
   Hạn chế nội dung (cờ bạc, rượu, nội dung người lớn, v.v.).
   Chiến lược Giảm Thiểu Rào cản Triển khai
   Chính sách Bảo mật & Điều khoản được Host: Cung cấp mẫu tuân thủ sẵn.
   Webform Đăng ký: Thu thập thông tin đăng ký một cách đơn giản.
   Widget Opt-in: Tích hợp dễ dàng vào website/app của Khách hàng.
   Thư viện Mẫu Tin nhắn: Giảm rủi ro tuân thủ.
   Môi trường Kiểm thử: Sandbox để Khách hàng thử nghiệm trước khi chính thức.
   Tài liệu & Đào tạo: Hướng dẫn rõ ràng theo từng loại trình gửi và use case.
   Là nhà cung cấp, bạn cần quyết định những quốc gia nào bạn sẽ hỗ trợ và cách đảm bảo tuân thủ giữa các thị trường. Quyết định này không chỉ ảnh hưởng đến giá cả của bạn mà cả kiến ​​trúc sản phẩm của bạn, đặc biệt nếu bạn phục vụ khách hàng toàn cầu.
2. Các chiến lược để giảm ma sát thực hiện

- Việc thực hiện SMS có thể phức tạp cho khách hàng của bạn. Dưới đây là một số chiến lược có thể đơn giản hóa và/hoặc hợp lý hóa quá trình. Một số trong số này có thể được trộn lẫn và kết hợp và cũng có thể được sử dụng như một giá trị gia tăng hoặc thậm chí là một đề nghị trả tiền cho khách hàng của bạn:
  **Chính sách bảo mật do nhà cung cấp bảo quản và/hoặc Điều khoản & Điều kiện**
  Tạo ra các mẫu tuân thủ, có thể tùy chỉnh cho các chính sách bảo mật và điều khoản và điều kiện mà khách hàng của bạn có thể sử dụng. Điều này đảm bảo tiết lộ đúng các thực tiễn SMS mà không yêu cầu khách hàng cập nhật các tài liệu pháp lý của riêng họ.
  **Đăng ký các biểu mẫu và quy trình công việc**
  Phát triển các dạng web thân thiện với người dùng thu thập tất cả thông tin đăng ký cần thiết trong một quy trình có hướng dẫn. Những điều này có thể đơn giản hóa đáng kể các đăng ký phức tạp như thương hiệu 10DLC và đăng ký chiến dịch. Dưới đây, Hình 1-3, bạn sẽ tìm thấy một số ví dụ về các hình thức tuân thủ có thể được tùy chỉnh cho việc sử dụng của bạn
  **Các tiện ích chọn tham gia được phê duyệt trước**
  Tạo các tiện ích có thể nhúng, chẳng hạn như Hình 1-3 ở trên, mà khách hàng của bạn có thể thêm vào trang web hoặc ứng dụng của họ để thực hiện các quy trình chọn tham gia tuân thủ. Chúng có thể bao gồm tất cả các tiết lộ và xác nhận cần thiết trong khi dễ dàng tích hợp.
  ![Figure 1](\SMS-Opt-In-Forms-2-1.png)
  ![Figure 1](\SMS-Opt-In-Forms-1.png)
  ![Figure 1](\SMS-Opt-In-Forms-3.png)
  **Thư viện mẫu**
  Cung cấp một thư viện các mẫu tin nhắn được phê duyệt trước cho các trường hợp sử dụng phổ biến. Điều này làm giảm rủi ro tuân thủ và đơn giản hóa quá trình gửi cho khách hàng của bạn.
  **Môi trường thử nghiệm**
  Tạo môi trường hộp cát nơi khách hàng có thể kiểm tra triển khai SMS của họ trước khi phát hành. Điều này giúp nắm bắt các vấn đề với định dạng, các quy trình chọn tham gia hoặc tuân thủ nội dung.
  **Tài liệu và đào tạo**
  Phát triển tài liệu rõ ràng và các tài nguyên đào tạo cụ thể cho từng loại người khởi tạo và trường hợp sử dụng. Điều này trao quyền cho khách hàng của bạn trong khi giảm gánh nặng hỗ trợ.

#### Kết luận

Kết hợp các khả năng SMS vào nền tảng của bạn có thể tăng cường sự tham gia của khách hàng, nhưng hành trình có thể phức tạp. Hướng dẫn này đã khám phá những cân nhắc chính để giúp bạn điều hướng nó thành công.

Bài đăng này đã kiểm tra các mô hình kiến ​​trúc khác nhau, mỗi mô hình có trách nhiệm về trách nhiệm của khách hàng và trách nhiệm của nhà cung cấp. Bài đăng này đã xem xét các yếu tố chiến lược như giá cả, quy định địa lý và các loại người khởi tạo phải được xem xét cẩn thận.
Cuối cùng, các chiến lược thực tế để giảm ma sát thực hiện cho các khách hàng như tài liệu tuân thủ được lưu trữ, quy trình đăng ký hợp lý và các mẫu được phê duyệt trước, bạn có thể sử dụng để đơn giản hóa quy trình tích hợp đã được thảo luận.

Tuy nhiên, bước đầu tiên quan trọng là hiểu mối quan hệ giữa bạn với tư cách là nhà cung cấp, khách hàng và người dùng cuối của họ. Hình dạng này có thông tin được sử dụng để đăng ký người khởi tạo, từ đó xác định trải nghiệm SMS.

Cuối cùng, một giải pháp SMS thành công đòi hỏi phải cân bằng các yếu tố kỹ thuật, quy định và lấy khách hàng làm trung tâm. Tận dụng hướng dẫn này sẽ trang bị cho bạn thiết kế và triển khai một đề nghị làm hài lòng khách hàng của bạn và người dùng cuối của họ

---

## 📖 Glossary - Thuật ngữ

| English    | Tiếng Việt                    | Định nghĩa                                                  |
| ---------- | ----------------------------- | ----------------------------------------------------------- |
| ISV        | Nhà cung cấp phần mềm độc lập | Công ty phát triển và bán phần mềm                          |
| Opt-in     | Đồng ý tham gia               | Quá trình người dùng chủ động đồng ý nhận tin nhắn          |
| Originator | Nguồn gửi tin nhắn            | Entity được sử dụng để gửi tin nhắn (Long Code, Short Code) |
| ...        | ...                           | ...                                                         |

## 🔗 Tài liệu tham khảo

### Tài liệu gốc

- [Original Article](https://aws.amazon.com/vi/blogs/containers/amazon-eks-pod-identity-streamlines-cross-account-access/): Bài viết gốc
- [Author's Profile](link): Thông tin tác giả
- [Related Articles](link): Bài viết liên quan

### Tài liệu tiếng Việt

- [AWS Documentation VN](link): Tài liệu AWS tiếng Việt
- [AWS Learning Resources](link): Tài nguyên học tập AWS
- [Community Discussions](link): Thảo luận cộng đồng

### Tools và Services

- [AWS Service 1](link): Mô tả service
- [AWS Service 2](link): Mô tả service
- [Third-party Tools](link): Tools bổ sung

---

## 💬 Ghi chú của người dịch

[Ghi chú về quá trình dịch, challenges gặp phải, insights gained]

### Challenges trong quá trình dịch

- **Technical Terms**: [Thuật ngữ khó dịch và cách giải quyết]
- **Cultural Context**: [Context cần adapt cho VN]
- **Complex Concepts**: [Khái niệm phức tạp và cách giải thích]

### Insights gained

- **Technical Learning**: [Kiến thức kỹ thuật học được]
- **Language Skills**: [Kỹ năng ngôn ngữ phát triển]
- **Industry Knowledge**: [Hiểu biết ngành nghề]

---

## 🤝 Đóng góp và Feedback

Bài dịch này được thực hiện trong khuôn khổ **FCJ Internship Program**.
**📧 Liên hệ**: nguyenboo2018@gmail.com  
**💬 Feedback**: Mọi góp ý để cải thiện chất lượng dịch thuật xin gửi về email trên  
**🔄 Updates**: Bài dịch sẽ được cập nhật dựa trên feedback từ cộng đồng

---

_© 2024 - Bản dịch thuộc về Vũ Quang Huy. Vui lòng credit khi sử dụng._
