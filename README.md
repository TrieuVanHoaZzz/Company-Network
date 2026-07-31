# Company Networking Project #6

## 📖 Bối cảnh dự án (Project Overview)
Dự án thiết kế và triển khai mô hình mạng với quy mô **600 nhân viên**. Tòa nhà gồm 3 tầng với 6 phòng ban chức năng khác nhau. Yêu cầu đặt ra là xây dựng một hệ thống mạng hoàn toàn mới, đảm bảo đáp ứng nhu cầu truy cập hiện tại, có tính dự phòng cao và sẵn sàng mở rộng trong tương lai.

Công cụ mô phỏng: **Cisco Packet Tracer**

---

##  Sơ đồ mạng (Network Topology)
<img width="2032" height="1286" alt="bai6_company" src="https://github.com/user-attachments/assets/39362ca8-13f9-4b56-bf49-934fd3978561" />


---

##  Phân bổ không gian & Thiết kế mạng phân cấp
Mô hình tuân thủ kiến trúc phân cấp (Hierarchical Network Design) với các lớp:
- **Core Layer:** 2 Router (CORE-R1, CORE-R2 - dòng 2911) kết nối định tuyến tốc độ cao và liên kết ra 2 Router ISP (IPS1, IPS2 - dòng 2811) qua các dải IP Public (`195.136.17.x/30`).
- **Distribution Layer:** 2 Multilayer Switch L3 (dòng 3650-24PS) hoạt động định tuyến nội bộ, Inter-VLAN Routing và cung cấp Gateway dự phòng. Các switch này kết nối Point-to-Point với Core Router qua các dải IP từ `172.16.3.144/30` đến `172.16.3.156/30`.
- **Access Layer:** Phân bổ tại 3 tầng (F1, F2, F3) với các Access Switch (dòng 2960) kết nối trực tiếp với thiết bị đầu cuối.

**Chi tiết phòng ban và VLAN (dựa trên sơ đồ):**
- **Tầng 1 (F1):** Phòng Sales & Marketing (VLAN 10) | Phòng HR & Logistics (VLAN 20)
- **Tầng 2 (F2):** Phòng Finance & Accounting (VLAN 30) | Phòng Admin & PR (VLAN 40)
- **Tầng 3 (F3):** Phòng IT (VLAN 50) | Phòng Server (VLAN 60)

---

##  Bảng quy hoạch địa chỉ IP (IP Addressing Scheme)
Dựa trên dải mạng nội bộ `172.16.0.0/16`, kỹ thuật chia mạng con **VLSM (Variable Length Subnet Mask)** được áp dụng để tối ưu không gian địa chỉ IP.

| Phòng ban | VLAN | Số Host | Network IP | Subnet Mask | Gateway (L3 SW) | Dải IP khả dụng (Usable IP) | Broadcast IP |
|:---|:---:|:---:|:---|:---|:---|:---|:---|
| **Sales & Marketing** | 10 | 120 | `172.16.1.0/25` | `255.255.255.128` | `172.16.1.1` | `172.16.1.1 - 172.16.1.126` | `172.16.1.127` |
| **HR & Logistics** | 20 | 120 | `172.16.1.128/25` | `255.255.255.128` | `172.16.1.129` | `172.16.1.129 - 172.16.1.254` | `172.16.1.255` |
| **Finance & Accounting**| 30 | 120 | `172.16.2.0/25` | `255.255.255.128` | `172.16.2.1` | `172.16.2.1 - 172.16.2.126` | `172.16.2.127` |
| **Admin & PR** | 40 | 120 | `172.16.2.128/25` | `255.255.255.128` | `172.16.2.129` | `172.16.2.129 - 172.16.2.254` | `172.16.2.255` |
| **IT (ICT)** | 50 | 120 | `172.16.3.0/25` | `255.255.255.128` | `172.16.3.1` | `172.16.3.1 - 172.16.3.126` | `172.16.3.127` |
| **Server Room** | 60 | 12 | `172.16.3.128/28` | `255.255.255.240` | `172.16.3.129` | `172.16.3.129 - 172.16.3.142` | `172.16.3.143` |

---

##  Các công nghệ & Yêu cầu kỹ thuật chính đã triển khai
- **Cấu hình cơ bản:** Đặt Hostname (Switch0-5, CORE-R1, L3-Switch, ...), tắt `no ip domain-lookup`, thiết lập mật khẩu bảo vệ (Console, Enable secret, VTY) và Banner MOTD.
- **Bảo mật truy cập:** Cấu hình SSH trên tất cả Router và Switch Layer 3 để quản trị từ xa an toàn.
- **VLAN & Trunking:** Khởi tạo VLAN theo ID, gán cổng Access cho thiết bị đầu cuối và cấu hình Trunking (dot1q) trên các đường uplink giữa Switch Access và Switch Distribution.
- **Inter-VLAN Routing:** Cấu hình Switch Virtual Interface (SVI) trên các Switch Layer 3 (3650) đóng vai trò là Default Gateway để các VLAN khác mạng có thể giao tiếp với nhau.
- **Cấp phát IP động (DHCP):** Thiết lập DHCP Server tập trung tại Server Room (VLAN 60). Cấu hình lệnh `ip helper-address` trên cổng Gateway SVI của các L3 Switch để forward các gói tin DHCP Broadcast từ Client về máy chủ.
- **Định tuyến động (OSPF):** Triển khai giao thức OSPF trên Core Routers và Distribution L3 Switches để tự động cập nhật bảng định tuyến nội bộ, giúp hệ thống hoạt động linh hoạt khi có sự cố đứt cáp.
- **Server Services:** Cấu hình IP tĩnh cho hệ thống thiết bị cốt lõi trong Server Room gồm DHCP Server, DNS Server, Mail Server và máy PC Sys-Admin.
- **Mạng không dây (WLAN):** Triển khai Access Point ở mỗi tầng/phòng ban (VLAN 10-50), phát sóng Wi-Fi cho các thiết bị di động (Laptop, Tablet-PC) giúp người dùng kết nối linh hoạt và tự động nhận IP từ DHCP.
- **NAT/PAT & ACL:** Cấu hình Overload NAT (PAT) kết hợp Access Control List (ACL) để biên dịch địa chỉ IP Private thành Public IP đi Internet, đồng thời xử lý Load Balancing (chia tải) và dự phòng đường truyền qua 2 liên kết kết nối với ISP.
- **Switchport Security:** Cấu hình bảo mật lớp 2 trên các cổng Access của switch bộ phận Finance & Accounting (giới hạn địa chỉ MAC, thiết lập violation restrict/shutdown) nhằm ngăn chặn truy cập trái phép ở cấp độ vật lý.

---

