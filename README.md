# ĐỒ ÁN 1: XÂY DỰNG HỆ THỐNG CHẤM CÔNG TỰ ĐỘNG ỨNG DỤNG TRONG QUẢN LÝ NHÂN SỰ


## Giới thiệu

<div align="justify"> Hệ thống chấm công tự động thông minh là một giải pháp ứng dụng công nghệ nhúng và Internet of Things (IoT) nhằm hiện đại hóa quy trình quản lý nhân sự, thay thế các phương pháp điểm danh thủ công còn tồn tại nhiều hạn chế như gian lận, chấm hộ, sai lệch dữ liệu và khó đồng bộ.

Dự án được thiết kế với mục tiêu xây dựng một hệ thống điểm danh xác thực hai lớp: sử dụng thẻ từ RFID để nhận dạng mã định danh nhân viên và cảm biến vân tay để xác thực sinh trắc học cá nhân. Để tăng cường tính minh bạch và trực quan, hệ thống còn tích hợp ESP32-CAM để chụp ảnh người điểm danh ngay tại thời điểm xác thực. Toàn bộ thông tin (UID, họ tên, thời gian, ảnh) sau khi xử lý sẽ được tự động đồng bộ lên nền tảng lưu trữ Google Sheets thông qua kết nối Wi-Fi.

Hệ thống được triển khai trên ba vi điều khiển độc lập, mỗi khối đảm nhận một vai trò chức năng rõ ràng:

- STM32F103C8T6: điều khiển cảm biến đầu vào (RFID, vân tay), xử lý logic và hiển thị trạng thái lên màn hình OLED.

- ESP32-DevKit: trung gian giao tiếp, tiếp nhận dữ liệu từ STM32 và chuyển tiếp đến ESP32-CAM.

- ESP32-CAM: xử lý hình ảnh, lưu trữ cục bộ và truyền dữ liệu điểm danh lên Google Sheets.

Ngoài phần nhúng, hệ thống còn được bổ sung một phần mềm giao diện quản trị (UI Manager) lập trình bằng Python (PyQt5 + PySerial), cho phép người quản trị thực hiện các thao tác như thêm, sửa, xóa nhân viên, gửi lệnh điều khiển và theo dõi log hệ thống theo thời gian thực.

Với kiến trúc phần cứng phân tán, phần mềm linh hoạt và tích hợp nền tảng đám mây, hệ thống này không chỉ phù hợp triển khai trong các doanh nghiệp, nhà máy, trường học mà còn có tiềm năng mở rộng thành một sản phẩm thương mại hóa ứng dụng trong lĩnh vực quản lý nhân sự thông minh.


## Tính năng của hệ thống

- Chấm công xác thực 2 bước: Thẻ RFID + Vân tay

- Giao tiếp giữa các vi điều khiển thông qua UART

- Tự động chụp ảnh khuôn mặt người dùng bằng ESP32-CAM

- Ghi thông tin điểm danh (UID, tên, thời gian, ảnh) lên Google Sheets

- Giao diện quản trị trên máy tính lập trình bằng Python + PyQt5

- Hỗ trợ đăng ký/xóa nhân viên qua phần mềm

- Hiển thị trạng thái điểm danh qua màn hình OLED

## Kiến trúc tổng thể của hệ thống

![image](https://github.com/user-attachments/assets/0f338bc4-bb6e-4527-bbe3-edd5faf5c69e)


Trong đó: 

- Khối hiển thị: Có nhiệm vụ hiển thị các thông báo, trạng thái và hướng dẫn thao tác cho người dùng trong quá trình điểm danh và đăng ký.
- Khối nhận diện thẻ từ: Đảm nhận vai trò nhận diện mã UID từ thẻ MIFARE Classic của nhân viên. Đây là bước xác thực đầu tiên trong quy trình điểm danh. Khối này giao tiếp với Khối vi điều khiển 1 bằng giao thức SPI
- Khối sinh trắc học: Thực hiện nhận dạng vân tay của nhân viên sau khi đã quét thẻ thành công. Đây là bước xác thực thứ hai, giúp tăng cường độ tin cậy. Khối này giao tiếp với Khối vi điều khiển 1 bằng giao thức UART
- Khối vi điều khiển 1: Chịu trách nhiệm thu thập và xử lý dữ liệu từ các cảm biến đầu vào (RFID, vân tay), đồng thời giao tiếp UART với Khối vi điều khiển 2. Sau khi xử lý, khối này điều khiển khối hiển thị để phản hồi tương tác với người dùng.
- Khối vi điều khiển 2: Là cầu nối trung gian giữa khối vi điều khiển 1, khối vi điều khiển 3 (UART) và người quản trị hệ thống (UART). Đảm bảo dữ liệu được truyền tải liền mạch và nhất quán.
- Khối vi điều khiển 3: Thực hiện các chức năng chuyên biệt bao gồm điều khiển khối nhận diện (camera) và khối lưu trữ (thẻ nhớ SD). Đồng thời, khối này chịu trách nhiệm gửi thông tin điểm danh thành công lên Google Sheets thông qua kết nối Wi-Fi.
- Khối lưu trữ: Dùng để lưu trữ toàn bộ thông tin nhân viên bao gồm: ID, tên, mã UID (4 byte), và ảnh chụp tại thời điểm điểm danh. Khối này được tích hợp sẵn trên vi điều khiển 3, sử dụng giao tiếp SPI để trao đổi dữ liệu.
- Khối nhận diện: Tích hợp trực tiếp trên vi điều khiển 3, bao gồm camera điều khiển thông qua giao thức SCCB. Có chức năng chụp ảnh nhân viên sau khi xác thực thành công.
- Khối mạng: Kết nối hệ thống với mạng Internet thông qua Wi-Fi, phục vụ việc gửi dữ liệu lên hệ thống lưu trữ trực tuyến như Google Sheets.
- Khối Google Sheets: Là nền tảng lưu trữ đám mây nơi tập hợp dữ liệu các nhân viên đã điểm danh thành công. Đồng thời cung cấp giao diện trực quan, dễ truy cập cho người quản trị.
- Khối giao diện: Cho phép người quản trị tương tác với hệ thống, theo dõi hoạt động điểm danh, cũng như thực hiện các thao tác như thêm, xóa nhân viên hoặc gửi lệnh điều khiển hệ thống.
- Khối nguồn: Cung cấp điện năng ổn định cho toàn bộ hệ thống hoạt động liên tục và hiệu quả.

## Phần cứng sử dụng cho hệ thống

| Thiết bị        | Mô tả                                    |
| --------------- | ---------------------------------------- |
| STM32F103C8T6   | Thu thập cảm biến, điều khiển OLED       |
| ESP32 DevKit    | Xử lý trung gian và giao tiếp UART       |
| ESP32-CAM       | Chụp ảnh + gửi ảnh và dữ liệu lên Sheets |
| Cảm biến AS608  | Nhận dạng vân tay                        |
| RFID RC522      | Đọc thẻ MIFARE 13.56 MHz                 |
| OLED SH1106     | Hiển thị trạng thái                      |
| Thẻ nhớ microSD | Lưu ảnh tạm thời từ ESP32-CAM            |


## Phần mềm của hệ thống

Phía vi điều khiển:

- STM32: code lập trình bằng STM32CubeIDE (ngôn ngữ C)

- ESP32: lập trình bằng Arduino IDE (C++)

- ESP32-CAM: xử lý hình ảnh + Wi-Fi + HTTP

Phía giao diện người quản trị:

- Ngôn ngữ: Python 3

- Giao diện: PyQt5

- Giao tiếp: PySerial

- Tính năng: thêm, xóa, sửa nhân viên, gửi lệnh, mở Google Sheets

## Kết quả đạt được

### Hình ảnh board mạch thực tế

![image](https://github.com/user-attachments/assets/7c5502e5-559d-420b-999c-bcc5a956447f)



### Hình ảnh giao diện quản trị

![image](https://github.com/user-attachments/assets/349dd757-8830-4faa-ad90-d40a1894c11e)

## Hướng dẫn sử dụng
- Bước 1, các bạn truy cập vào folder đã được tải về: DoAn1-main.zip (đã giải nén) -> mở folder VoMinhThai-22139063-PhanThanhThao-22139062.
- Bước 2, tiến hành mở /hardware/schematic.pdf.
- Bước 3, đọc sơ đồ nguyên lý và kết nối mạch như sơ đồ, có thể bỏ qua khối nguồn (khối này cấp nguồn hệ thống), bạn có thể nối các khối trong đó lại bỏ qua khối nguồn, thay vào đó cắm nguồn vào chân USB Type C trên vi điều khiển ESP32 Devkit.
- Bước 4, truy cập vào /firmware và tiến hành tải các file .ino tương ứng về rồi nạp có cho vi điều khiển tuy nhiên trước khi nạp code cần thay đổi một số thứ như sau (tại code ESP32-CAM):
  - Đầu tiên thông tin wifi: ``` const char* ssid = "Minh Thai";``` 
                                <br>```const char* password = "01202728759";```
  - Thay ``` const char* googleScriptId ="https://script.google.com/macros/s/AKfycbx5QKuQZAvtdk5Q6iZ9LmWkVbuHb3sVTyVAgXXFDLXu2BUw-lPJx8uBxhpI-P928gM/exec";```
    - Các bước thay đổi:
      - Truy cập trang: Google Sheets vào tạo một Sheets mới, đồng thời đặt tên các header: Date, Time, ID, Fullname
      - Sau đó chọn vào tiện ích/ extensions -> Apps Script để mở trang tích hợp Script vào cho bảng tính
        ![image](https://github.com/user-attachments/assets/dbb79fef-1e7e-4b1e-905b-714e9c9413d9)
      - Tại trang giao diện chính của Apps Script truy cập Deploy -> Manage Deployment -> và copy đoạn dưới đây:
        ![image](https://github.com/user-attachments/assets/dca2f4aa-d2e3-495c-8296-8e4ca5ca8e59)
  - Tại /ui có file UI.py bạn có thể vào VS code bấm run hoặc build thành file .exe bằng lệnh ```pyinstaller --onefile --add-data "hcmute.png;." UI.py``` (Đây là giao diện quản trị viên)

  
## Tác giả 
- Họ và tên: Võ Minh Thái, Phan Thanh Thảo
- Mã Số sinh viên: 22139063, 22139062
- Sinh viên ngành: Hệ thống nhúng và IoT
- Trường: Đại học Sư Phạm Kỹ Thuật Thành Phố Hồ Chí Minh
- Giảng viên hướng dẫn: ThS. Trương Quang Phúc
- Youtube: https://www.youtube.com/watch?v=RMcZxA5p-UM
- Liên hệ: thaivm14072004@gmail.com
<br>
This document was written by [ThaiVM2004]https://github.com/ThaiVM2004 and [thaoIot2004]https://github.com/thaoIot2004
</div>






