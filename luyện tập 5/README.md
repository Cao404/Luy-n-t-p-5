# LUYỆN TẬP 5 – REACT NATIVE

Project này là bài thực hành React Native CLI, sử dụng `View` và `Text` để hiển thị nội dung **Hello React Native**. Phần trả lời đầy đủ các câu hỏi nằm trong file [BAI_LAM.md](./BAI_LAM.md).

## Yêu cầu môi trường

- Node.js từ phiên bản 22.11 trở lên.
- JDK 17.
- Android Studio, Android SDK và biến môi trường `ANDROID_HOME` đã được cấu hình.
- Thiết bị Android thật đã bật USB Debugging hoặc Android Emulator đang chạy.

## Cài đặt và chạy

```bash
npm install
npm start
```

Mở một cửa sổ terminal khác và chạy:

```bash
adb devices
npm run android
```

Nếu dùng lệnh trực tiếp của React Native CLI:

```bash
npx react-native run-android
```

## Các file chính

- `App.tsx`: giao diện sử dụng `View`, `Text` và `StyleSheet`.
- `BAI_LAM.md`: trả lời 5 câu lý thuyết và 3 bài thực hành.
- `android/`: mã native và cấu hình build Android.
- `ios/`: mã native và cấu hình build iOS.
