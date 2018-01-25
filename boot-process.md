# Quá trình khởi động của Linux

## 1. Bắt đầu khởi động : trạng thái OFF

### 1.1 Wake-on-LAN

Trạng thái OFF nghĩa là không có nguồn điện ? Điều này không hẳn đúng. Ví dụ cho điều này là đèn Ethernet vẫn sáng vì tính năng Wake-on-LAN (WOL) được kích hoạt.

Để kiểm tra nó có được kích hoạt hay không thực hiện :

```
sudo ethtool <interface name>
```

Nếu "Wake-on" được show ra, các remote host khác có thể khởi động hệ thống bằng cách gửi [MagicPacket](https://en.wikipedia.org/wiki/Wake-on-LAN#Magic_packet). Nếu không muốn khởi động từ xa, hãy tắt WOL bằng cách:

```
sudo ethtool -s <interface name> wol d
```

Bộ xử lý để đáp ứng MagicPacket có thể là 1 phần của NIC hoặc  [Baseboard Management Controller (BMC)](https://lwn.net/Articles/630778/).


### 1.2 Intel Management Engine, Platform Controller Hub, và Minix

Không phải chỉ có BMC là microcontroller (MCU) lắng nghe khi hệ thống đang OFF. Các hệ thống x86_64 cũng bao gồm bộ phần mềm Intel Management Engine (IME) để quản lý từ xa.
Một loạt các thiết bị, từ máy chủ đến máy tính xách tay đều sử dụng công nghệ này, cho phép các chức năng như KVM Remote Control và  Intel Capability Licensing Service.
Theo Intel, IME có lỗ hổng chưa được vá, và thật tệ khi IME rất khó để vô hiệu hóa.
Trammell Hudson đã tạo ra [me_cleaner](https://github.com/corna/me_cleaner) để dọn dẹp một số thành phần IME như máy chủ hệ thống nhúng nhưng nó cũng có thể ảnh hưởng tới hệ thống đang chạy.

IME firmware và phần mềm System Management Mode (SMM) theo kèm khởi động dựa trên hệ điều hành Minix và chạy trên bộ xử lý nền tảng riêng biệt Platform Controller Hub chứ không phải là CPU.
Sau đó SMM khởi chạy Universal Extensible Firmware Interface (UEFI) - được viết và chạy trên CPU.
Nhóm Coreboot ở Google đã bắt đầu một dự án NERF (Non-Extensible Reduced Firmware) có tham vọng thay thế không chỉ của UEFI mà còn các thành phần người dùng Linux ban đầu như systemd
Trong khi chờ đợi kết quả của những nỗ lực mới này, người dùng Linux bây giờ có thể mua máy tính xách tay từ Purism, System76 hoặc Dell với IME bị vô hiệu, ngoài ra chúng ta có thể hy vọng cho máy tính xách tay có bộ vi xử lý ARM 64-bit.


### 1.3 Bootloaders

Công việc của Bootloader là đảm bảo cho bộ xử lý được hỗ trợ đủ tài nguyên nó cần để chạy hệ điều hành.
Khi bật nguồn, không chỉ RAM ảo mà DRAM cũng chưa có cho đến khi controller của nó được bật lên.
Bootloader bật nguồn, quét các bus và các interface để xác định kernel image và root filesystem.
Các Bootloader phổ biến như U-boot, GRUB hỗ trợ các giao diện quen thuộc như USB, PCI, NFS và các thiết bị nhúng đặc biệt như NOR- và NAND-flash.
Nó cũng tương tác với các thiết bị bảo mật phần cứng như Trusted Platform Modules (TPMs).

![Running the U-boot bootloader in the sandbox on the build host](https://github.com/locvx1234/init-system/blob/master/images/linuxboot_1.png)

U-boot được sử dụng rộng rãi trên các hệ thống Raspberry Pi, các thiết bị Nitendo, bảng điều khiển ô tô, Chromebooks.
Nó không có syslog, khi có sự kiện nào đó, nó cũng không có output console nào.
Để hỗ trợ cho debug, nhóm phát triển Uboot cung cấp một sandbox, trong đó các bản vá có thể được kiểm tra trên máy chủ hoặc trên  Continuous Integration system.
Việc sử dụng U-Boot's sandbox tương đối đơn giản trên các hệ thống mà có các công cụ phát triển như Git, GNU Compiler Collection (GCC) được cài đặt.

```
$# git clone git://git.denx.de/u-boot; cd u-boot
$# make ARCH=sandbox defconfig
$# make; ./u-boot
=> printenv
=> help
```

Nếu bạn đang chạy U-Boot trên x86_64, bạn có thể sử dụng các tính năng như mô phỏng phân vùng lại thiết bị lưu trữ, thao tác khóa bí mật dựa trên TMP và hotplug của các thiết bị USB.

## 2. Khởi động kernel

### 2.1 Cung cấp một booting kernel

Sau khi hoàn thành nhiệm vụ, bootloader khởi động sẽ thực hiện lệnh nhảy tới kernel code mà nó đã nạp vào bộ nhớ chính và bắt đầu thực thi, đi qua bất kỳ tùy chọn dòng lệnh nào mà người dùng đã chỉ định.

File `/boot/vmlinuz` chỉ ra rằng nó là một `bzImage` có nghĩa là file nén lớn

Linux source tree chứa một công cụ  [extract-vmlinux](https://github.com/torvalds/linux/blob/master/scripts/extract-vmlinux) để giải nén file:

```
$# scripts/extract-vmlinux /boot/vmlinuz-$(uname -r) > vmlinux
$# file vmlinux
vmlinux: ELF 64-bit LSB executable, x86-64, version 1 (SYSV), statically
linked, stripped
```

Kernel là một chương trình nhị phân [thực thi và liên kết](http://man7.org/linux/man-pages/man5/elf.5.html) như các chương trình người dùng Linux.
Điều đó có nghĩa là chúng ta có thể sử dụng từ gói `binutils` như `readelf` để kiểm tra nó :

```
$# readelf -S /bin/date
$# readelf -S vmlinux
```

Danh sách các phần trong các chương trình nhúng cũng tương tự nhau.

Vì vậy, kernel phải bắt đầu một cái gì đó giống như các chương trình Linux khác ELF  ... nhưng làm thế nào để các chương trình người sử dụng thực sự bắt đầu? Trong hàm `main()`, phải không? Không chính xác.

Trước khi hàm `main()` có thể chạy, các chương trình cần thực thi ngữ cảnh bao gồm bộ nhớ `heap` và `stack` cùng các chương trình mô tả tập tin cho `stdio`, `stdout`, `stderr`. Các chương trình người dùng (userspace) lấy tài nguyên từ  các thư viện chuẩn, hầu hết là `glibc` trong các hệ thống Linux.

```
$# file /bin/date
/bin/date: ELF 64-bit LSB shared object, x86-64, version 1 (SYSV), dynamically
linked, interpreter /lib64/ld-linux-x86-64.so.2, for GNU/Linux 2.6.32,
BuildID[sha1]=14e8563676febeb06d701dbee35d225c5a8e565a,
stripped
```

Các chương trình ELF có một thông dịch viên, giống như các tập lệnh Bash và Python, nhưng thông dịch viên không cần phải được chỉ định với #! như trong các ngôn ngữ kịch bản, như ELF là định dạng gốc của Linux.
Trình thông dịch ELF cung cấp một số nhị phân với các tài nguyên cần thiết bằng cách gọi `_start()`, một chức năng có sẵn từ gói nguồn glibc có thể được kiểm tra qua GDB.
Kernel rõ ràng là không có thông dịch viên và phải cung cấp chính nó, nhưng làm thế nào?

Việc kiểm tra khởi động của hạt nhân với GDB giúp đưa ra câu trả lời. Đầu tiên cài đặt gói gỡ lỗi cho kernel có chứa một phiên bản chưa được vmlinux, ví dụ `apt-get install linux-image-amd64-dbg`, hoặc biên dịch và cài đặt kernel của riêng bạn từ nguồn, ví dụ bằng cách làm theo các hướng dẫn trong [Debian Kernel Handbook](http://kernel-handbook.alioth.debian.org/).
`gdb vmlinux` theo sau bởi các tệp tin hiển thị phần init.text của ELF. Danh sách bắt đầu thực hiện chương trình trong init.text với l * (địa chỉ)
GDB sẽ chỉ ra rằng kernel x86_64 khởi động trong tệp tin của kernel `/x86/kernel/head_64.S`
ARM 32-bit kernel có cấu trúc tương tự `arch/arm/kernel/head.S`

## 3. Từ start_kernel() cho tới PID 1

### 3.1 Phần kê khai phần cứng của kernel: cây thiết bị và các bảng ACPI

Khi khởi động, kernel cần thông tin về phần cứng ngoại trừ bộ xử lý mà nó đã được biên dịch.
Các hướng dẫn trong mã được gia tăng bởi dữ liệu cấu hình được lưu trữ riêng.
Có hai phương pháp chính để lưu trữ dữ liệu này: cây thiết bị và bảng ACPI.
Hạt nhân học được phần cứng nào nó phải chạy ở mỗi lần khởi động bằng cách đọc các tập tin này.

Đối với thiết bị nhúng, cây thiết bị là biểu hiện của phần cứng được cài đặt.
Cây thiết bị chỉ đơn giản là một tập tin được biên dịch cùng lúc với nguồn kernel và thường nằm trong `/boot` cùng với `vmlinux`.
Để xem những gì trong cây thiết bị nhị phân trên một thiết bị ARM, chỉ cần sử dụng lệnh `strings` từ gói `binutils` trên một tệp có tên tương ứng `/boot/*.dtb`, vì `dtb` đề cập đến một cây nhị phân cây thiết bị.
Rõ ràng cây thiết bị có thể được sửa đổi đơn giản bằng cách chỉnh sửa các tệp JSON giống như tạo tệp và rerunning trình biên dịch dtc đặc biệt được cung cấp cùng với mã nguồn kernel.
Mặc dù cây thiết bị là một tệp tin tĩnh mà đường dẫn tệp thường được hạt nhân truyền đến bởi trình nạp khởi động trên dòng lệnh, một cơ chế che phủ cây thiết bị đã được thêm vào trong những năm gần đây, nơi kernel có thể tự động nạp các mảnh bổ sung để đáp ứng sự kiện hotplug sau khi khởi động.

x86-family và nhiều thiết bị ARM64 cấp doanh nghiệp sử dụng Advanced Configuration and Power Interface (ACPI). Trái ngược với cây thiết bị, thông tin ACPI được lưu trữ trong hệ thống tập tin ảo của hệ thống `/sys/firmware/acpi/` được tạo bởi kernel khi khởi động bằng cách truy cập vào ROM trên máy.
Cách dễ dàng để đọc bảng ACPI là với lệnh `acpidump` từ gói công cụ acpica. Đây là một ví dụ:

![ACPI tables on Lenovo laptops are all set for Windows 2001](https://github.com/locvx1234/init-system/blob/master/images/linuxboot_2.png)

ACPI có cả phương pháp và dữ liệu, không giống như cây thiết bị, vốn là ngôn ngữ mô tả phần cứng.
Các phương pháp của ACPI tiếp tục hoạt động sau khi khởi động.
Ví dụ, bắt đầu lệnh `acpi_listen` (từ gói apcid) và mở và đóng nắp máy tính xách tay sẽ cho thấy rằng chức năng ACPI đều chạy.
Có thể tạm thời và tự động ghi đè lên các bảng ACPI, thay đổi vĩnh viễn chúng liên quan đến tương tác với trình đơn BIOS lúc khởi động hoặc reflashing ROM.
Nếu bạn đang gặp rắc rối đó, có lẽ bạn nên cài đặt coreboot, phần mềm nguồn mở thay thế.

### 3.2 Từ start_kernel() tới userspace

Code `init/main.c` thật đáng kinh ngạc, nó vẫn mang bản quyền gốc của Linus Torvalds từ năm 1991-1992.
Các dòng tìm thấy trong `dmesg` - đầu vào một hệ thống mới khởi động bắt nguồn chủ yếu từ tập tin nguồn này.
CPU đầu tiên được đăng ký với hệ thống, các cấu trúc dữ liệu toàn cục được khởi tạo, và trình tự lập trình, bộ xử lý gián đoạn (IRQs), bộ đếm thời gian và giao diện điều khiển được đặt theo thứ tự nghiêm ngặt, trực tuyến.
Cho đến khi chức năng `timekeeping_init()` chạy, tất cả các dấu thời gian là số không.
Phần khởi tạo kernel này là đồng bộ, có nghĩa là sự thực hiện xảy ra trong chính xác một luồng và không có chức năng nào được thực hiện cho đến khi kết thúc và trả về kết quả cuối cùng.
Kết quả là, đầu ra `dmesg` sẽ được tái tạo hoàn toàn, ngay cả giữa hai hệ thống, miễn là họ có cùng một thiết bị cây hoặc bảng ACPI.
Linux đang hoạt động giống như một trong những hệ điều hành RTOS (hệ điều hành thời gian thực) chạy trên các MCU, ví dụ như QNX hoặc VxWorks.
Tình huống vẫn tồn tại trong hàm `rest_init()`, được gọi bởi `start_kernel()` tại thời điểm kết thúc.

![Summary of early kernel boot process](https://github.com/locvx1234/init-system/blob/master/images/linuxboot_3.png)

`rest_init()` tạo ra một thread mới chạy `kernel_init()`, nó sẽ gọi `do_initcalls()`.
Người dùng có thể theo dõi initcalls bằng cách thêm `initcall_debug` vào dòng lệnh kernel, dẫn đến các mục dmesg mỗi khi một chức năng `initcall` chạy.
`initcalls` đi qua bảy cấp độ tuần tự: early, core, postcore, arch, subsys, fs, device, và late.
Phần người dùng có thể dễ thấy nhất là các thiết bị ngoại vi: bus, mạng, lưu trữ, màn hình hiển thị ... cùng với việc tải các mô-đun kernel của chúng.
`rest_init()` cũng tạo ra một thread thứ hai trên bộ vi xử lý khởi động bắt đầu bằng cách chạy `cpu_idle()` trong khi nó chờ cho `scheduler ` lên lịch cho nó làm việc.

`kernel_init()` cũng thiết lập multiprocessing đối xứng (SMP).
Với các kernel gần đây, tìm thấy trong output `dmesg` bằng cách tìm kiếm "Bringing up secondary CPUs.." SMP tiến hành bằng các "hotplugging" CPU, có nghĩa là nó quản lý vòng đời của chúng bằng một máy trạng thái tương tự như các thiết bị USB cắm nóng.
Hệ thống quản lý năng lượng của nhân thường lấy các lõi riêng lẻ, sau đó đánh thức chúng khi cần thiết, do đó cùng một mã CPU hotplug được gọi lặp đi lặp lại trên một máy không bận. Quan sát lời gọi hệ thống quản lý nguồn bằng công cụ [BCC](http://www.brendangregg.com/ebpf.html) được gọi là `offcputime.py`.

Lưu ý rằng mã trong `init/main.c` gần như đã hoàn thành khi chạy `smp_init()`: Bộ xử lý khởi động đã hoàn thành hầu hết việc khởi tạo một lần mà các lõi khác không cần phải lặp lại. Tuy nhiên, mỗi luồng CPU phải được sinh ra cho mỗi lõi để quản lý các sự cố ngắt (IRQs), workqueues, timer, và power.
Ví dụ, xem mỗi luồng CPU phục vụ softirqs và workqueues đang hoạt động thông qua lệnh `ps -o psr`.

```
$\# ps -o pid,psr,comm $(pgrep ksoftirqd)
 PID PSR COMMAND
   7   0 ksoftirqd/0
  16   1 ksoftirqd/1
  22   2 ksoftirqd/2
  28   3 ksoftirqd/3

$\# ps -o pid,psr,comm $(pgrep kworker)
PID  PSR COMMAND
   4   0 kworker/0:0H
  18   1 kworker/1:0H
  24   2 kworker/2:0H
  30   3 kworker/3:0H
[ . .  . ]
```

### 3.3 Early userspace: Ai ra lệnh cho initd ?

Bên cạnh cây thiết bị, một đường dẫn file khác được cung cấp cho kernel khi khởi động là file `initrd`.
Initrd thường ở trong `/boot` cùng với tệp `bmlIcon` của `bzImage` trên `x86` hoặc cùng với `uImage` và cây thiết bị tương tự cho ARM.
Liệt kê nội dung của `initrd` bằng công cụ `lsinitramfs` là một phần của gói `initramfs-tools-core`.
Các sơ đồ `initrd` chứa các thư mục `/bin`, `/sbin`, và `/etc` cùng với các mô-đun kernel, cộng với một số tệp trong `/scripts`.
Tất cả những thứ này trông khá quen thuộc, vì `initrd` phần lớn chỉ đơn giản là một hệ thống tập tin gốc Linux.
Gần như tất cả các file thực thi trong `/bin` và `/sbin` bên trong ramdisk là các liên kết đến binary BusyBox, kết quả là `/bin` và `/sbin` nhỏ hơn 10x so với glibc's.

Tại sao lại phải tạo ra một initrd nếu tất cả những gì nó làm là nạp một số mô-đun và sau đó bắt đầu init trên hệ thống tập tin gốc thường xuyên?
Xem xét một hệ thống tập tin gốc được mã hóa. Giải mã có thể dựa vào việc tải một mô-đun kernel được lưu trữ trong `/lib/modules` trên hệ thống tập tin gốc ... và initrd làm tốt điều đó.
Môđun `crypto` có thể được biên dịch thành kernel thay vì được tải từ một tệp tin, nhưng có nhiều lý do khiến bạn không muốn làm như vậy.
Ví dụ, việc biên dịch kernel bằng các mô đun có thể làm cho nó quá lớn, không phù hợp với dung lượng có sẵn, hoặc việc biên dịch tĩnh có thể vi phạm các điều khoản của một giấy phép phần mềm.
Các driver lưu trữ, mạng và thiết bị đầu vào con người (HID) cũng có thể có trong `initrd`. Về cơ bản bất kỳ mã nào không phải là một phần của kernel, thích hợp để gắn kết hệ thống tập tin gốc.
`Initrd` cũng là nơi người dùng có thể lưu trữ mã bảng ACPI tuỳ chỉnh của riêng họ.


![Having some fun with the rescue shell and a custom initrd](https://github.com/locvx1234/init-system/blob/master/images/linuxboot_4.png)

`initrd` cũng tuyệt vời cho việc thử nghiệm các hệ thống tập tin và các thiết bị lưu trữ dữ liệu.

Cuối cùng, khi init chạy, hệ thống đang bật! Kể từ khi các bộ vi xử lý thứ cấp đang chạy, máy đã trở nên không đồng bộ, không thể dự đoán trước, hiệu năng cao.
Thật vậy, `ps -o pid, psr, comm -p 1` có thể cho thấy tiến trình `init` của `userspace` không còn chạy trên bộ xử lý khởi động nữa.
