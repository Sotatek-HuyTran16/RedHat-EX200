# Note lại 1 vài thứ hay ho hoặc vài nội dung hay bị quên

## Storage

Khi tạo partition chưa gắn flag lvm mà đã tạo PV -> gán vào VG thì sẽ bị ít dung lượng hơn vì bị phí các section đầu

Nếu đang thi mà quên ko gắn flag -> size nhỏ hơn yêu cầu của VG -> failed

Giải quyết như nào:
- Gán thêm pv vào cho đủ (maybe) nhưng chưa bt bn mới đủ
- Gỡ ra gán lại

## NFS automount - autofs

Cơ chế:
- Khi client truy cập file qua NFS, kernel sẽ gửi request kèm UID/GID của process trên client
- NFS server không biết username bên client, chỉ biết số UID/GID
- NFS server sẽ kiểm tra UID/GID đó có quyền trên file không (theo metadata trên server)

Ví dụ:
- Nếu file trên NFS server thuộc userX có UID=1001
- Thì để client truy cập đúng quyền, user trên client cũng phải có UID=1001
- Nếu client có user userX nhưng UID=2001 thì sẽ bị mismatch → quyền sẽ sai

Một số tình huống
1. Autofs dùng root squashing (mặc định)

- Nếu client dùng root (UID=0), NFS sẽ ánh xạ thành nobody (UID không đặc quyền) để tránh root từ client có toàn quyền trên server
- Có thể tắt bằng no_root_squash trong /etc/exports nhưng rất nguy hiểm

2. Khi UID/GID không khớp → user thấy file nhưng quyền không đúng (có thể chỉ đọc hoặc hoàn toàn không có quyền)

### Lỗi khi ssh vào user có home directory là NFS mounted

-> SElinux policy chặn sshd process truy cập các file NFS dẫn đến permission denied

-> Bật booleans để fix

```bash
setsebool -P use_nfs_home_dirs on
```