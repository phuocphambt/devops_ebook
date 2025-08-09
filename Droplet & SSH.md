# Bài 1: Chuẩn bị Droplet & SSH
## Mục tiêu :
``` bash
- Tạo Droplet Ubuntu LTS trên DigitalOcean.
- Kết nối bằng SSH key (không dùng mật khẩu).
- Tạo user thường (không phải root) có quyền sudo.
- Bật tường lửa (UFW), đặt timezone và cập nhật hệ thống.
```
## I. Tạo SSH key trên máy cá nhân:
``` bash
ssh-keygen -t ed25519 -C "you@example.com"
# Nhấn Enter để lưu tại ~/.ssh/id_ed25519 và đặt passphrase (khuyến khích)
eval "$(ssh-agent -s)"
ssh-add ~/.ssh/id_ed25519
cat ~/.ssh/id_ed25519.pub
```
## II. Tạo Droplet trên DigitalOcean
``` bash
1. Vào Create → Droplets.
2. Image: Ubuntu 22.04 LTS hoặc 24.04 LTS.
3. Size: 1 vCPU / 1–2 GB RAM.
4. Authentication: SSH Keys → Add SSH Key → dán khóa public ở trên.
5. Tạo Droplet và ghi lại Public IP.
```
## III. SSH vào máy & tạo user "devops"
### Đăng nhập lần đầu (user root)
``` bash
ssh root@<DROPLET_PUBLIC_IP>
```
### Tạo user thường + cấp sudo + chép key:
```bash
adduser devops
usermod -aG sudo devops
rsync -av ~/.ssh/ /home/devops/.ssh
chown -R devops:devops /home/devops/.ssh
chmod 700 /home/devops/.ssh
chmod 600 /home/devops/.ssh/authorized_keys 2>/dev/null || true
```
### Update hệ thống & timezone
```bash
apt update && apt upgrade -y
timedatectl set-timezone Asia/Ho_Chi_Minh
```
### Bật UFW, chỉ mở SSH
```bash
ufw allow OpenSSH
ufw --force enable
ufw status
```
## IV. SSH Hardening 
**(Quan trọng: Chỉ làm bước này sau khi xác nhận đăng nhập bằng devops@IP đã OK.)**

Sửa cấu hình SSH để chặn root login & mật khẩu:
``` bash
sudo cp /etc/ssh/sshd_config /etc/ssh/sshd_config.bak
sudo nano /etc/ssh/sshd_config
```
Sửa lại các thông số : 
``` bash
PasswordAuthentication no
PermitRootLogin no
PubkeyAuthentication yes
```
Kiểm tra & reload:
``` bash
sudo sshd -t && sudo systemctl reload sshd
```
