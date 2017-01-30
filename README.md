# init-system
Một số ‘init’ System  trong Linux

Trong Linux/UNIX, tiến trình khởi tạo (initalization) là tiến trình được thực thi bởi kernel trong lúc boot và nhận process ID (PID) là 1. Nó thực thi ở chế độ nền đến khi hệ thống shutdown.

Init process khởi động tất cả các process khác. Đó là các daemon, service và các  background processe khác.

Nó là tiến trình mẹ của các tiến trình khác. Một tiến trình có thể khởi động nhiều tiến trình con khác trên hệ thống, nhưng khi một tiến trình cha die, init sẽ trở thành cha của các tiến trình con đó.

Trong những năm gần đây, có nhiều hệ thống init được phát triển cho các bản phân phối của Linux, sau đây là các hệ thống nổi bật nhất:

## 1. System V

System V (SysV) là một hệ thống phổ biến trong Linux/UNIX. Hầu hết các distro Linux đầu tiên đều sử dụng SysV ngoại trừ Gentoo và Slackware.

Trải qua nhiều năm, do có một vài khiếm khuyết, một vài hệ thống SysV init được phát triển để tăng tính hiệu quả và hoàn hảo hơn.

Mặc dù có các lựa chọn thay thế để cải thiện SysV  với các tính năng mới, nhưng vẫn có tương thích với các script gốc.



## 2. Upstart

Upstart là hệ thống init được phát triển bởi Ubuntu như một sự thay thế cho SysV. Nó khởi đọng các tiến trình các , kiểm tra chúng khi đang chạy và stop chúng khi hệ thống shutdown.

Nó là một hệ thống hybrid init, trong đó sử dụng cả hai script của cả SysV và Systemd. 

Một số tính năng đáng chú ý của Upstart bao gồm :

- Được phát triển cho Ubuntu nhưng có thể chạy trên tất cả các distro Linux
- Các event được tạo trong suốt quá trình start, stop
- Các event có thể được gửi bởi tiến trình hệ thống khác
- Truyền thông với tiến trình init qua D-Bus
- Các user có thể start, stop tiến trình của chính họ
- Tái tạo các dịch vụ ngừng đột ngột

## 3. SystemD

SystemD là một hệ thống init mới cho Linux. Được giới thiệu ở Fedora 15, nó là một tập hợp các công cụ để quản lý hệ thống dễ dàng. Mục đích chính là để khởi tạo, quản lý và theo dõi tất cả các tiến trình của hệ thống trong quá trình khởi động và khi hệ thống đang chạy.

Nó hoàn toàn khác biệt với hệ thống init truyền thống. Nó tương thích SysV và LBS init.

Một số tính năng của SystemD:
 
- Rõ ràng, đơn giản, hiệu quả
- Xử lý song song khi khởi động
- API tốt hơn 
- Cho phép loại bỏ các quy trình tùy chọn
- Hỗ trợ journal trong quá trình logging
- Hỗ trợ lập lịch làm việc 
- Lưu trữ log trong file nhị phân
- Tốt hơn với GNOME



## 4. OpenRC

## 5. runit




*Tham khảo*

http://www.tecmint.com/best-linux-init-systems/

https://www.digitalocean.com/community/tutorials/how-to-configure-a-linux-service-to-start-automatically-after-a-crash-or-reboot-part-1-practical-examples

https://www.digitalocean.com/community/tutorials/how-to-configure-a-linux-service-to-start-automatically-after-a-crash-or-reboot-part-2-reference






