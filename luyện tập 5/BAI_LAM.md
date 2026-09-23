# BÀI LÀM LUYỆN TẬP 5

## A. Câu hỏi ôn tập lý thuyết

### Câu 1. Ba luồng chính trong React Native

Khi một ứng dụng React Native khởi chạy, công việc thường được chia cho ba luồng chính:

1. **Native thread (Main/UI thread):** Đây là luồng giao diện chính của Android hoặc iOS. Luồng này tiếp nhận các sự kiện chạm của người dùng, quản lý vòng đời của ứng dụng và thực hiện các thao tác trực tiếp lên thành phần giao diện native. Sau khi có kết quả bố cục, Native thread tạo, cập nhật và vẽ các thành phần như `View` hoặc `Text` lên màn hình. Vì đây là luồng hiển thị nên không được thực hiện công việc nặng trong thời gian dài; nếu bị chặn, giao diện sẽ giật hoặc đứng.

2. **JavaScript thread:** Luồng này chạy mã JavaScript/TypeScript của ứng dụng. Nó xử lý logic nghiệp vụ, state, props, sự kiện, gọi API và quyết định cây giao diện React cần có hình dạng như thế nào. Khi người dùng nhấn nút, sự kiện từ phía native được chuyển tới JavaScript; hàm xử lý cập nhật state, React tính toán thay đổi rồi gửi mô tả cập nhật giao diện sang phía native. Nếu JavaScript thread bận xử lý tác vụ quá nặng, phản hồi logic và animation phụ thuộc JavaScript có thể bị chậm.

3. **Shadow thread:** Luồng này làm việc với cây giao diện ở dạng dữ liệu, thường gọi là Shadow Tree. Nó sử dụng Yoga để tính kích thước và vị trí của các phần tử dựa trên Flexbox, ví dụ `flex`, `padding`, `margin`, `width` và `height`. Sau khi tính xong layout, kết quả được chuyển cho Native thread để áp dụng lên các view thật.

Luồng xử lý tổng quát là: Native thread nhận thao tác → JavaScript thread xử lý sự kiện và cập nhật state → React tạo thay đổi của cây giao diện → Shadow thread/Yoga tính layout → Native thread cập nhật và vẽ giao diện. Ở kiến trúc React Native mới, JSI và Fabric giúp trao đổi trực tiếp, đồng bộ hơn giữa JavaScript và C++, nhưng ba nhóm trách nhiệm trên vẫn hữu ích để hiểu cách ứng dụng vận hành.

### Câu 2. Luồng render giao diện từ JavaScript tới native

Khi JavaScript viết `<View>` và `<Text>`, đó chưa phải là `View` của Android hay `UIView` của iOS. React trước tiên tạo một cây phần tử mô tả giao diện. Khi props hoặc state thay đổi, React so sánh cây mới với cây trước để xác định phần nào cần tạo, sửa hoặc xóa.

Trong renderer Fabric của React Native hiện đại, mô tả thay đổi được đưa qua lớp JSI/C++ để tạo và cập nhật Shadow Tree. Yoga đọc style theo mô hình Flexbox, tính toán vị trí và kích thước cho từng nút. Sau bước commit, các thay đổi cần thiết được mount lên nền tảng: `View` được ánh xạ sang thành phần hiển thị native phù hợp và `Text` được ánh xạ sang thành phần chữ native. Native thread sau đó thực hiện cập nhật thật và hệ điều hành vẽ chúng lên màn hình.

React Native tạo được cảm giác gần giống ứng dụng native vì giao diện cuối cùng được thể hiện bằng thành phần của nền tảng và sử dụng hệ thống render, nhập liệu, accessibility của Android/iOS. JavaScript chủ yếu mô tả giao diện và xử lý logic; React Native đảm nhiệm việc liên kết mô tả đó với phía native. Vì vậy lập trình viên có thể dùng một mô hình React chung nhưng người dùng vẫn tương tác với giao diện native thay vì trang HTML chạy trong WebView.

### Câu 3. Vai trò của JSI và Yoga

**JSI (JavaScript Interface)** là lớp giao tiếp giữa JavaScript runtime và mã C++/native. JSI cho phép JavaScript gọi các đối tượng hoặc hàm native trực tiếp hơn, không bắt buộc phải tuần tự hóa mọi dữ liệu thành JSON rồi gửi bất đồng bộ qua bridge cũ. Trong kiến trúc mới, JSI là nền tảng cho TurboModules và Fabric, giúp giảm chi phí giao tiếp, hỗ trợ cơ chế đồng bộ ở nơi phù hợp và cải thiện hiệu năng khởi động/render.

**Yoga** là engine bố cục đa nền tảng dùng để tính layout theo Flexbox. Yoga nhận cây giao diện cùng các thuộc tính style như `flexDirection`, `justifyContent`, `alignItems`, `padding`, `margin` và kích thước; sau đó trả về tọa độ và kích thước cuối cùng của từng phần tử. Nhờ Yoga, cùng một cách viết layout JavaScript có thể cho kết quả nhất quán trên Android và iOS.

Mối liên hệ giữa các phần là:

1. JavaScript thread chạy component, xử lý logic UI/UX và tạo mô tả cây giao diện.
2. Fabric/JSI đưa mô tả cần cập nhật tới phần renderer C++ mà không phụ thuộc hoàn toàn vào bridge kiểu cũ.
3. Shadow Tree đại diện cho giao diện chưa được vẽ; Yoga tính layout cho các nút trong cây này.
4. Kết quả sau commit được mount; Native thread cập nhật các thành phần nền tảng thật.
5. Android hoặc iOS thực hiện đo, vẽ và hiển thị kết quả cuối cùng lên màn hình.

### Câu 4. Chuẩn bị build và chạy trên thiết bị Android thật

Các bước thực hiện:

1. Cài Node.js, JDK 17, Android Studio, Android SDK, Android SDK Platform-Tools và cấu hình `ANDROID_HOME`/`PATH` để dùng được `adb`.
2. Mở **Settings → About phone**, chạm nhiều lần vào **Build number** để bật Developer Mode. Tên mục có thể khác tùy hãng điện thoại.
3. Vào **Developer options** và bật **USB Debugging**. Chế độ này cho phép ADB trên máy tính giao tiếp với điện thoại để cài, khởi chạy và gỡ lỗi ứng dụng.
4. Dùng cáp USB có truyền dữ liệu để nối điện thoại với máy tính; chọn chế độ truyền tệp nếu thiết bị yêu cầu.
5. Mở khóa màn hình. Khi hộp thoại hỏi cho phép USB debugging xuất hiện, chọn cho phép và có thể đánh dấu luôn tin cậy máy tính này.
6. Chạy `adb devices`. Thiết bị phải xuất hiện với trạng thái `device`, không phải `unauthorized` hoặc `offline`.
7. Trong thư mục project, chạy `npm install`, khởi động Metro bằng `npm start`, sau đó mở terminal khác chạy `npx react-native run-android`.
8. Nếu thiết bị không truy cập được Metro, chạy `adb reverse tcp:8081 tcp:8081`, sau đó reload ứng dụng.

Phải bật Developer Mode vì tùy chọn USB Debugging chỉ có trong nhóm tùy chọn dành cho nhà phát triển. USB Debugging mở kênh ADB để máy tính gửi file APK, cài ứng dụng, đọc log và điều khiển quá trình debug. `adb devices` giúp xác nhận cả kết nối vật lý lẫn quyền tin cậy. Giữ thiết bị mở khóa là cần thiết vì Android có thể đang chờ người dùng xác nhận khóa RSA; nếu màn hình khóa, yêu cầu này không được chấp nhận và thiết bị thường hiện `unauthorized`.

### Câu 5. So sánh build React Native trên Android và iOS

| Nội dung | Android | iOS |
|---|---|---|
| Hệ điều hành máy phát triển | Windows, macOS hoặc Linux | Bắt buộc dùng macOS để build native chính thức |
| IDE/công cụ chính | Android Studio, Android SDK, Gradle, ADB và JDK | Xcode, iOS SDK, Simulator, CocoaPods và công cụ dòng lệnh của Xcode |
| Thiết bị thử nghiệm | Điện thoại Android bật USB Debugging hoặc Android Emulator | iPhone/iPad thật hoặc iOS Simulator |
| Lệnh thường dùng | `npx react-native run-android` | `npx react-native run-ios` |
| Ký ứng dụng | Debug keystore có sẵn; bản phát hành cần signing key/keystore | Cần Apple ID, certificate và provisioning profile tùy cách cài/phát hành |
| Giới hạn đáng chú ý | Có thể build trên nhiều hệ điều hành; cần đúng SDK/JDK | Xcode chỉ chạy trên macOS, nên không thể build iOS native trực tiếp trên Windows/Linux |

Android Studio cung cấp SDK Manager, emulator, Logcat và bộ công cụ build Android. SDK chứa API nền tảng, build-tools và platform-tools cần để Gradle biên dịch, đóng gói và cài ứng dụng. Với iOS, Xcode chứa compiler, iOS SDK, Simulator, công cụ ký mã và hệ thống build. Apple ID miễn phí có thể dùng cho một số hoạt động thử nghiệm trên thiết bị cá nhân nhưng có giới hạn về thời hạn/chức năng; để phân phối qua TestFlight hoặc App Store cần tham gia Apple Developer Program. Dù phần lớn JavaScript có thể dùng chung, mỗi nền tảng vẫn có cấu hình, quyền truy cập, quy trình ký và đôi khi có mã native riêng.

## B. Bài tập luyện tập thực hành

### Bài tập mức dễ

Khi người dùng mở ứng dụng React Native, hệ điều hành khởi tạo tiến trình và Native thread tạo cửa sổ giao diện đầu tiên, đồng thời quản lý vòng đời của màn hình. JavaScript runtime được khởi động và JavaScript thread tải bundle, chạy component gốc rồi tạo mô tả giao diện cần hiển thị. Từ mô tả này, Shadow Tree được tạo; Yoga tính kích thước và vị trí các phần tử dựa trên style Flexbox. Kết quả layout được commit và Native thread tạo hoặc cập nhật các thành phần native thật, sau đó Android/iOS vẽ khung hình đầu tiên lên màn hình. Ba phần phối hợp theo vai trò: JavaScript thread quyết định nội dung và hành vi, Shadow thread/Yoga tính bố cục, còn Native thread tiếp nhận sự kiện và hiển thị kết quả.

### Bài tập mức dễ đến trung bình

Mã nguồn hoàn chỉnh nằm trong `App.tsx`. Phần cốt lõi sử dụng đúng hai thành phần giao diện `View` và `Text` như sau:

```tsx
<View style={styles.card}>
  <Text style={styles.eyebrow}>LUYỆN TẬP 5</Text>
  <Text style={styles.title}>Hello React Native</Text>
  <Text style={styles.description}>
    Giao diện này được tạo bằng hai thành phần cơ bản là View và Text.
  </Text>
</View>
```

- `View` là vùng chứa dùng để nhóm các thành phần, sắp xếp layout và tạo bề mặt của thẻ nội dung. Nó có vai trò tương tự một container và được chuyển thành view tương ứng của nền tảng.
- `Text` dùng để hiển thị chuỗi văn bản. Trong React Native, nội dung chữ phải được đặt trong `Text`; thành phần này hỗ trợ font, màu, cỡ chữ, line-height và accessibility của nền tảng.
- `StyleSheet.create` tập hợp các đối tượng style, giúp tên style rõ ràng và có kiểm tra kiểu tốt hơn. `flex`, `alignItems`, `justifyContent`, `padding` và `margin` tham gia tính layout; `color`, `fontSize`, `fontWeight`, `backgroundColor`, `borderRadius` và shadow quyết định phần nhìn.
- React Native/Fabric chuyển cây component sang Shadow Tree, Yoga tính layout, rồi renderer mount kết quả thành các view native trên Android hoặc iOS. Vì vậy mã JavaScript/TypeScript dùng chung có thể hiển thị thành giao diện nền tảng thật.

### Bài tập mức trung bình – Checklist thiết bị Android thật

- [ ] Cài Node.js đúng phiên bản mà project yêu cầu.
- [ ] Cài JDK 17 và kiểm tra bằng `java -version`.
- [ ] Cài Android Studio cùng Android SDK, SDK Platform và Platform-Tools.
- [ ] Cấu hình `ANDROID_HOME` và thêm `platform-tools` vào biến `PATH`.
- [ ] Trên điện thoại, vào About phone và chạm Build number nhiều lần để bật Developer Mode.
- [ ] Mở Developer options và bật USB Debugging.
- [ ] Kết nối điện thoại bằng cáp USB có hỗ trợ truyền dữ liệu.
- [ ] Chọn File Transfer/MTP nếu điện thoại chỉ sạc và chưa được nhận.
- [ ] Mở khóa điện thoại và chọn **Allow** trong hộp thoại cấp quyền USB debugging/RSA.
- [ ] Chạy `adb devices` và kiểm tra serial của máy có trạng thái `device`.
- [ ] Vào thư mục project và chạy `npm install`.
- [ ] Mở terminal thứ nhất và chạy `npm start` để khởi động Metro.
- [ ] Mở terminal thứ hai và chạy `npx react-native run-android`.
- [ ] Nếu ứng dụng không nối được Metro qua USB, chạy `adb reverse tcp:8081 tcp:8081` rồi reload.
- [ ] Xác nhận ứng dụng được cài và thấy dòng “Hello React Native” trên thiết bị.

#### Lỗi thường gặp và cách xử lý

- **Không có thiết bị trong `adb devices`:** đổi cáp hoặc cổng USB, bật chế độ truyền tệp, cài USB driver của hãng trên Windows, rồi chạy lại `adb kill-server` và `adb start-server`.
- **Trạng thái `unauthorized`:** mở khóa màn hình và chấp nhận hộp thoại RSA. Nếu không thấy hộp thoại, chọn **Revoke USB debugging authorizations**, tắt/bật lại USB Debugging rồi kết nối lại.
- **Trạng thái `offline`:** rút cáp, khởi động lại ADB bằng `adb kill-server` và `adb start-server`, sau đó cắm lại thiết bị.
- **Thiết bị đang khóa:** mở khóa màn hình trong lúc kết nối và chấp nhận quyền; một số máy chặn cài đặt qua USB khi đang khóa.
- **Build không tìm thấy SDK:** kiểm tra Android SDK Path trong Android Studio, biến `ANDROID_HOME` và file `android/local.properties` nếu môi trường yêu cầu.
- **Ứng dụng báo không tải được bundle/không kết nối Metro:** bảo đảm Metro đang chạy ở cổng 8081 và dùng `adb reverse tcp:8081 tcp:8081`.
- **Có nhiều thiết bị hoặc emulator:** chỉ định thiết bị bằng `npx react-native run-android --deviceId <serial>`, trong đó serial lấy từ `adb devices`.
