# Setup FreePBX

# Table of contents

- [1. Cài đặt FreePBX](#install-freepbx)
- [2. Tạo extension - máy nhánh](#add-extensions)
- [3. Chuyển hướng cuộc gọi](#follow-me)
- [4. Cấu hình NAT](#configure-nat)

# Contents

# <a name="install-freepbx" >1. Cài đặt FreePBX</a>

Download Freepbxdistro at [http://downloads.freepbxdistro.org/ISO/](http://downloads.freepbxdistro.org/ISO/)

FreePBX distro các bản mới có tên gọi là SNG7-FPBX, nó dựa trên bản phân phối RHEL7. Bản mới nhất hiện đang là SNG7-FPBX-1805.

Download tại: [SNG7-FPBX-64bit-1805-2.iso](https://downloads.freepbxdistro.org/ISO/SNG7-FPBX-64bit-1805-2.iso)

Sau khi cài đặt xong, truy cập FreePBX server và cấu hình network. Cách cấu hình network tương tự với CentOS/RHEL.

```
DEVICE="eth0"
ONBOOT="yes"
IPADDR=192.168.10.118
NETMASK=255.255.255.0
GATEWAY=192.168.10.1
DNS1=8.8.8.8
DNS2=1.1.1.1
```

Sau đó thực hiện restart network

`systemctl restart network`

Tiếp theo mở web browser và truy cập FreePBX theo địa chỉ [http://192.168.10.118](http://192.168.10.118)

Màn hình khởi tạo đầu tiên yêu cầu tạo tài khoản cho quản trị hệ thống FreePBX.

<p align="center"> 
<img src="../images/initial-FreePBX-Administration.png" />
</p>

Sau khi vào thông tin tài khoản, nhấn Create Account để tạo tài khoản quản trị.

Màn hình chính FreePBX, xuất hiện gồm 04 options:

<p align="center"> 
<img src="../images/home-login.png" />
</p>

- FreePBX Administration: cho phép chúng ta cấu hình FreePBX, với thông tin tài khoản quản trị vừa tạo ở trên.

- User Control Panel: Là nơi người dùng có thể login và thực hiện gọi qua giao diện web, thiết lập nút phone, xem voicemail, gửi tin nhắn SMS & XMPP, … (Thông tin tài khoản login cho option này nằm ở module User Management

- Operator Panel: Là màn hình mà cho phép operator để kiểm soát cuộc gọi

- Get Support: Chuyển hướng support sang trang chủ FreePBX

Chúng ta chọn “FreePBX Administration” để login vào trang quản trị FreePBX

Tại màn hình chính là Dashboard của FreePBX với các thông tin tổng quan, thống kê số liệu hệ thống, network, trạng thái dịch vụ, ...

<p align="center"> 
<img src="../images/dashboard-freepbx.png" />
</p>

## <a name="add-extensions">2. Tạo extension - máy nhánh</a>

Chúng ta thực hiện tạo số máy nhánh (hay SIP Account hoặc Extension)

Trong giao diện chính mở: Applications> Extensions> Add Extension> Add New Chan_SIP Extension

<p align="center"> 
<img src="../images/add-extensions.png" />
</p>

Thực hiện tạo một extension Chan_SIP với thông tin:

- User Extension: Vào thông tin số máy nhánh dùng khi dial, ở đây là 100

- Display Name: Thiết lập tên hiển thị KeepWalking cho số nhánh này

- Secret: Vào thông tin mật khẩu cho tài khoản này (Ở đây tôi đặt P@ssw0rd)

**Note**: Cả Chan_SIP và PJSIP đều cho phép tạo extension number nhưng Chan_SIP cho phép hỗ trợ NAT. Hiện tại thì PJSIP được sử dụng cho default SIP (với port 5060), Chan_SIP sử dụng port 5160.

Để tạo các tài khoản khoản cho SIP extension gồm cả Chan_SIP và PJSIP, chúng ta thực hiện như các bước ở trên.

<p align="center"> 
<img src="../images/show-extensions.png" />
</p>

## <a name="follow-me">3. Thiết lập chuyển hướng cuộc gọi</a>

Trên giao diện web của FreePBX, chúng ta có thể cấu hình để cho phép chuyển hướng cuộc gọi (với tùy chọn Follow Me) từ một số khi số đó không answer hoặc busy.

Chúng ta cấu hình chuyển hướng cuộc gọi từ số 100 đến 101 khi số máy 100 không answer hoặc busy.

**Step1: Enable Follow Me**

Mở Applications → Follow Me

<p align="center"> 
<img src="../images/enable-follow-me.png" />
</p>

Khi đó chọn **Yes** cho số extension cần enabled tính năng **Follow Me**

**Step2: Edit extensions**

Mở Applications → Extensions (Hoặc mở Application → Follow Me)

Khi đó chọn số extension cần edit, ví dụ ở đây là **100**

<p align="center"> 
<img src="../images/edit-follow-me.png" />
</p>

Khi đó vào các thông số sau:

- Ring Time (max 60 sec): thiết lập khoảng thời gian tối đa rung chuông, nếu sau khoảng thời gian này extension 100 không trả lời thì sẽ forward sang Extensions 101. Ta thiết lập khoảng time là 20sec.

- Destination if no answer: chọn Extension và chọn số nhánh sẽ forward. Ở đây, chúng ta thiết lập khi số 100 không answer hoặc busy thì sẽ forward cuộc gọi đến số 101.
Cuối cùng nhấn Submit → Apply Config

## <a name="configure-nat">4. Cấu hình NAT</a>

Trong trường hợp công ty có nhiều văn phòng hoặc người dùng ở ngoài văn phòng và có nhu cầu sử dụng hệ thống tổng đài. Khi đó, chúng ta cần publish tổng đài ra internet hoặc cấu hình truy cập với kết nối VPN.

Trong phần này chúng ta sẽ giới thiệu phần cấu hình publish tổng đài ra internet

<p align="center"> 
<img src="../images/FreePBX-NAT-Diagram.png" />
</p>

**Step1**: NAT từ Router đến FreePBX

Trên Router chúng ta sẽ NAT các ports sau đến FreePBX

- 5060 UDP - Port SIP

- 10000-20000 UDP - Dải ports cho RTP Media

**Step2**: Thêm thông tin NAT trên FreePBX

Mở Settings → Assterisk SIP Settings

Tại tab General SIP Settings

<p align="center"> 
<img src="../images/freepbx-nat01.png" />
</p>

Trong phần **NAT Settings** vào các thông tin sau:

- External Address: Vào địa chỉ Public IP của router NAT đến FreePBX, ở đây ví dụ là 1.2.3.4

- Local Networks: Vào thông tin subnet của LAN của FreePBX

Trong phần **RTP Settings** vào thông tin sau:

- RTP Port Ranges: Vào dải ports 10000-20000 để cho phép truyền tệp tin, video, audio qua mạng

Tại tab Chan SIP Settings

<p align="center"> 
<img src="../images/freepbx-nat02.png" />
</p>

- NAT: tích chọn yes để luôn thực hiện NAT

- IP Configuration: Chọn Static IP và vào thông tin Public IP trên router sẽ NAT xuống FreePBX

Sau khi vào sau các thông tin NAT cho FreePBX thì thực hiện Submit → Apply Config

Updating ...