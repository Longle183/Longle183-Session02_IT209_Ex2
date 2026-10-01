# Bài 2: Khởi tạo User thường và thiết lập đặc quyền quản trị

## 1. Mục tiêu

Tạo một tài khoản người dùng thường có tên `devops` trên Ubuntu Server để thực hiện các công việc quản trị hệ thống hàng ngày thay vì sử dụng trực tiếp tài khoản `root`.

User `devops` được thêm vào nhóm `sudo`, đồng thời được cấu hình SSH Key để có thể đăng nhập từ máy tính cá nhân.

---

## 2. Môi trường

- Server: DigitalOcean Droplet
- Operating System: Ubuntu 22.04 LTS
- User ban đầu: `root`
- User mới: `devops`
- Authentication: SSH Key
- Quyền quản trị: `sudo`

---

## 3. Tạo user devops

Đăng nhập vào Droplet bằng tài khoản `root`:

```bash
ssh -i $env:USERPROFILE\.ssh\id_ed25519 root@<IP_DROPLET>
```

Sau đó tạo user:

```bash
adduser devops
```

Hệ thống yêu cầu nhập mật khẩu cho user `devops`.

Kiểm tra user:

```bash
id devops
```

Kết quả cho thấy user `devops` đã được tạo thành công.

---

## 4. Thêm devops vào nhóm sudo

Thực hiện:

```bash
usermod -aG sudo devops
```

Kiểm tra:

```bash
groups devops
```

Kết quả có nhóm:

```text
sudo
```

Có thể kiểm tra bằng:

```bash
id devops
```

Kết quả tương tự:

```text
uid=1001(devops) gid=1001(devops) groups=1001(devops),27(sudo)
```

---

## 5. Sao chép SSH Key

SSH Key của tài khoản `root` nằm trong:

```text
/root/.ssh
```

Sử dụng `rsync` để sao chép sang thư mục home của `devops`:

```bash
rsync --archive --chown=devops:devops /root/.ssh /home/devops/
```

Nếu hệ thống chưa có `rsync`, cài đặt bằng:

```bash
apt update
apt install rsync -y
```

Sau đó thực hiện lại lệnh sao chép:

```bash
rsync --archive --chown=devops:devops /root/.ssh /home/devops/
```

---

## 6. Thiết lập quyền SSH

Thiết lập quyền cho thư mục `.ssh`:

```bash
chmod 700 /home/devops/.ssh
```

Thiết lập quyền cho `authorized_keys`:

```bash
chmod 600 /home/devops/.ssh/authorized_keys
```

Đảm bảo toàn bộ thư mục SSH thuộc về `devops`:

```bash
chown -R devops:devops /home/devops/.ssh
```

Kiểm tra:

```bash
ls -la /home/devops/.ssh
```

Kết quả mong muốn tương tự:

```text
drwx------ 2 devops devops ... .
-rw------- 1 devops devops ... authorized_keys
```

---

## 7. Kiểm tra cấu hình SSH

Kiểm tra cú pháp SSH:

```bash
sshd -t
```

Nếu không có thông báo lỗi, cấu hình SSH hợp lệ.

Kiểm tra SSH service:

```bash
systemctl status ssh
```

SSH service phải ở trạng thái:

```text
Active: active (running)
```

---

## 8. Đăng nhập bằng devops

Thoát khỏi phiên root:

```bash
exit
```

Từ máy tính cá nhân, thực hiện:

```powershell
ssh -i $env:USERPROFILE\.ssh\id_ed25519 devops@<IP_DROPLET>
```

Sau khi đăng nhập thành công, kiểm tra:

```bash
whoami
```

Kết quả:

```text
devops
```

Điều này chứng minh tài khoản `devops` có thể đăng nhập thông qua SSH Key.

---

## 9. Kiểm tra quyền sudo

Kiểm tra nhóm:

```bash
groups
```

Kết quả phải có:

```text
sudo
```

Sau đó chạy:

```bash
sudo whoami
```

Hệ thống có thể yêu cầu nhập mật khẩu của user `devops`.

Kết quả:

```text
root
```

Điều này chứng minh `devops` là user thường nhưng được cấp quyền quản trị thông qua nhóm `sudo`.

---

## 10. Kết quả

Đã hoàn thành các yêu cầu:

- Tạo user `devops`.
- Thêm `devops` vào nhóm `sudo`.
- Sao chép SSH Key từ `/root/.ssh` sang `/home/devops/.ssh`.
- Thiết lập owner `devops:devops`.
- Thiết lập quyền `700` cho thư mục `.ssh`.
- Thiết lập quyền `600` cho `authorized_keys`.
- Đăng nhập thành công bằng `devops` thông qua SSH Key.
- Thực hiện thành công `sudo whoami` và nhận kết quả `root`.

---

## 11. Cấu trúc thư mục nộp bài

```text
ex2/
└── README.md
```

Không đưa Private Key vào GitHub.

Không commit các file như:

```text
id_ed25519
```

Chỉ cần nộp `README.md` và log/ảnh chụp màn hình cần thiết.