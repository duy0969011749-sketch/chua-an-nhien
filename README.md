# Chùa An Nhiên Online

Chánh điện 2D hoạt hình. Mỗi người tạo nhân vật (nam/nữ, tên, kiểu tóc, kiểu áo, màu áo, phụ kiện), đi lại trong chùa và thấy người khác đi lại cùng lúc. Tối đa 50 người.

- Kiểu áo: áo tràng, áo dài, áo bà ba, áo thun.
- Phụ kiện (chọn nhiều): tràng hạt, kính, hoa cài, nón lá.
- Nhân vật chớp mắt, thở nhẹ khi đứng, vung tay khi đi, quay vào bàn thờ và làm động tác chi tiết khi lễ.
- Dưới tên hiện tổng số giờ online, cộng dồn qua các lần vào, lưu theo tài khoản.

Hành động: Lạy, Thắp hương, Gõ mõ, Thỉnh chuông, Xin xăm, Xin 8 số, Lời nguyện.

Động tác của nhân vật khi hành lễ (quay lưng vào bàn thờ):
- Lạy: chắp tay trước ngực, đưa lên trán, quỳ xuống, úp hai tay xuống sàn và cúi rạp đầu, đứng dậy; lặp 3 lạy.
- Thắp hương: cầm 3 nén hương đưa lên trán, vái 3 vái, đưa tay cắm hương vào bát (hương mới xuất hiện trong bát đúng lúc này), vái thêm 1 vái.
- Thỉnh chuông: vái, tay phải cầm dùi giơ lên, lấy đà rồi đánh (chuông rung và kêu đúng lúc dùi chạm), hạ tay, vái.
- Gõ mõ: quỳ xuống, tay phải cầm dùi; mỗi lần bấm dùi gõ xuống một tiếng; ngừng bấm khoảng 2,5 giây thì đứng dậy.
- Xin xăm: quỳ, vái, cầm ống xăm lắc đều, một que xăm rơi ra, đứng dậy.
- Xin 8 số: chắp tay lên trán khấn (cúi đầu 3 lần), rồi vái 3 vái.
- Lời nguyện: quỳ, chắp tay lên trán cúi đầu khấn, lạy 1 lạy, đứng dậy.
- Đang hành lễ mà bấm đi chỗ khác thì động tác dừng ngay.

Điều khiển: chạm hoặc bấm vào sàn để đi, phím mũi tên hoặc W A S D trên máy tính. Bấm vào chuông, mõ, bát hương trong cảnh cũng thực hiện hành động.

## Chat

- Máy tính: bấm Enter để mở ô chat, gõ rồi Enter để gửi, Esc để đóng. Điện thoại: bấm nút Chat ở góc trên phải.
- Tin nhắn hiện thành bong bóng trên đầu nhân vật và lưu ở khung chat góc dưới trái. Người mới vào thấy 30 tin gần nhất.
- Mỗi tin tối đa 120 ký tự. Mỗi người phải cách khoảng 2 giây mới gửi tin tiếp (quy tắc Firebase cũng chặn).
- Từ ngữ thô tục trong danh sách `BAD_WORDS` (trong `index.html`) được thay bằng dấu *. Có thể thêm từ vào danh sách này.

## Tài khoản và lưu nhân vật

- Vào trang phải đăng nhập bằng email và mật khẩu (ít nhất 6 ký tự). Ai cũng tự đăng ký được. Có nút Quên mật khẩu (Firebase gửi email đặt lại).
- Nhân vật và tổng giờ online lưu theo tài khoản trên Firebase: đăng nhập máy nào cũng ra đúng nhân vật.
- Nút Đăng xuất ở góc trên bên phải.

## Cài đặt (làm 1 lần)

### Bước 1. Tạo Firebase

1. Vào https://console.firebase.google.com, bấm Add project (tạo project).
2. Build > Realtime Database > Create database, chọn vùng Singapore (asia-southeast1), chọn Start in locked mode.
3. Build > Authentication > Get started > Sign-in method > Email/Password > Enable > Save.
4. Project settings (bánh răng) > General > Your apps > bấm biểu tượng Web, đặt tên, bấm Register. Copy đoạn `firebaseConfig`, đảm bảo có dòng `databaseURL`.
5. Mở `index.html`, tìm `const FIREBASE_CONFIG = null;` và thay bằng config vừa copy, ví dụ:

```js
const FIREBASE_CONFIG = {
  apiKey: "...", authDomain: "...", projectId: "...",
  databaseURL: "https://<ten-project>-default-rtdb.asia-southeast1.firebasedatabase.app",
  storageBucket: "...", messagingSenderId: "...", appId: "..."
};
```

6. Realtime Database > Rules, xóa hết, dán bộ quy tắc dưới đây, bấm Publish.

```json
{
  "rules": {
    "users": {
      "$uid": {
        ".read": "auth != null && auth.uid === $uid",
        ".write": "auth != null && auth.uid === $uid",
        "profile": { ".validate": "newData.child('name').isString() && newData.child('name').val().length <= 16" },
        "onlineMs": { ".validate": "newData.isNumber() && newData.val() >= 0" }
      }
    },
    "rooms": {
      "$room": {
        "players": {
          ".read": "auth != null",
          "$uid": {
            ".write": "auth != null && auth.uid === $uid",
            ".validate": "newData.child('name').isString() && newData.child('name').val().length <= 16"
          }
        },
        "pos": {
          ".read": "auth != null",
          "$uid": {
            ".write": "auth != null && auth.uid === $uid",
            ".validate": "newData.child('x').isNumber() && newData.child('y').isNumber()"
          }
        },
        "stats": {
          ".read": "auth != null",
          "$k": { ".write": "auth != null", ".validate": "newData.isNumber() && (!data.exists() || newData.val() >= data.val())" }
        },
        "chat": {
          ".read": "auth != null",
          "$id": {
            ".write": "auth != null && !data.exists()",
            ".validate": "newData.hasChildren(['uid', 'name', 'text', 'ts']) && newData.child('uid').val() === auth.uid && newData.child('name').isString() && newData.child('name').val().length <= 16 && newData.child('text').isString() && newData.child('text').val().length > 0 && newData.child('text').val().length <= 120 && newData.child('ts').val() === now && newData.parent().parent().child('chatLast').child(auth.uid).val() === now"
          }
        },
        "chatLast": {
          "$uid": {
            ".write": "auth != null && auth.uid === $uid",
            ".validate": "newData.val() === now && (!data.exists() || now - data.val() >= 1500)"
          }
        },
        "prayers": {
          ".read": "auth != null",
          "$id": {
            ".write": "auth != null && !data.exists()",
            ".validate": "newData.child('wish').isString() && newData.child('wish').val().length <= 200 && newData.child('name').isString() && newData.child('name').val().length <= 16"
          }
        }
      }
    }
  }
}
```

Với bộ quy tắc này, chưa đăng nhập thì không đọc hay ghi được gì; mỗi người chỉ sửa được nhân vật và vị trí của chính mình.

### Bước 2. Đưa lên GitHub Pages

1. Trên GitHub bấm New repository, đặt tên (ví dụ `chua-online`), chọn Public, bấm Create.
2. Bấm Add file > Upload files, kéo `index.html` và `README.md` vào, bấm Commit changes.
3. Settings > Pages > Source: Deploy from a branch, Branch: `main`, thư mục `/ (root)`, bấm Save.
4. Sau 1-2 phút có link dạng `https://<tên-tài-khoản>.github.io/<tên-repo>/`.

### Bước 3. Cho phép tên miền GitHub đăng nhập

Firebase > Authentication > Settings > Authorized domains > Add domain, nhập `<tên-tài-khoản>.github.io` (không có https, không có tên repo). Thiếu bước này trang sẽ báo tên miền chưa được cho phép khi đăng nhập.

## 50 người cùng lúc và chống giật lag

- Giới hạn 50 người online được kiểm tra khi vào chùa. Gói miễn phí Spark của Firebase cho tối đa 100 kết nối cùng lúc nên đủ cho 50 người.
- Vị trí đi lại tách riêng khỏi hồ sơ nhân vật: mỗi lần ai đó di chuyển, máy người khác chỉ nhận vài chục byte của đúng người đó, không tải lại cả danh sách.
- Chạm sàn để đi chỉ gửi 1 lần (điểm đến); đi bằng phím gửi khoảng 2 lần mỗi giây hoặc khi đổi hướng. Máy người khác tự vẽ chuyển động mượt giữa các lần nhận.
- Giờ online ghi lên máy chủ mỗi phút một lần.
- Gói Spark cho 10 GB dữ liệu tải về mỗi tháng. Nên theo dõi ở Realtime Database > Usage. Nếu đông người và đi lại liên tục nhiều giờ mỗi ngày thì có thể vượt, khi đó cần nâng lên gói Blaze (trả theo mức dùng).
- Giới hạn 50 người kiểm tra ở trình duyệt, người cố tình sửa mã vẫn có thể vượt; muốn chặn tuyệt đối phải dùng Cloud Functions.

## Chế độ thử

Khi `FIREBASE_CONFIG` để `null`, trang chạy chế độ thử: tài khoản chỉ lưu trên trình duyệt của máy đó, các tab trên cùng trình duyệt thấy nhau. Chỉ dùng để thử trước khi bật Firebase, không bảo mật.

## Ghi chú

Tượng Phật, nhân vật và cảnh chùa đều là hình hoạt hình vẽ bằng mã SVG, không dùng ảnh ngoài. Âm thanh mõ, chuông tạo bằng mã. Lời xăm là lời khuyên tự soạn.
