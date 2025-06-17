
#  Kiến thức tổng hợp về SSL, Domain, DNS, và các lệnh Linux

---

##  SSL

### SSL là gì?
SSL (Secure Sockets Layer) là một giao thức mã hóa được sử dụng để bảo vệ việc truyền dữ liệu giữa máy chủ (server) và trình duyệt (client). Hiện nay, SSL đã được thay thế bằng TLS (Transport Layer Security), nhưng thuật ngữ "SSL" vẫn được dùng phổ biến.

### Có bao nhiêu cách xác thực SSL?
1. **Domain Validation (DV)**
2. **Organization Validation (OV)**
3. **Extended Validation (EV)**

### CSR file dùng làm gì trong quá trình tạo SSL?
CSR (Certificate Signing Request) là tệp chứa thông tin domain và public key. CSR được gửi lên CA (Certificate Authority) để yêu cầu cấp SSL.

### Tạo CSR bằng OpenSSL:
```bash
openssl req -new -newkey rsa:2048 -nodes -keyout tech.training.vietnix.tech.key -out tech.training.vietnix.tech.csr
```

### PEM file là gì?
PEM (Privacy Enhanced Mail) là định dạng văn bản (Base64) cho các chứng chỉ SSL, khóa riêng (private key), chuỗi CA. Phổ biến với đuôi: `.pem`, `.crt`, `.key`

### Private Key là gì?
Private Key là khóa riêng được tạo cùng với public key, dùng để giải mã dữ liệu mã hóa từ client.

### PFX file là gì? Cách chuyển từ .crt sang .pfx
PFX (PKCS#12) là định dạng lưu trữ cả certificate và private key trong 1 file. 
```bash
openssl pkcs12 -export -out certificate.pfx -inkey private.key -in certificate.crt -certfile CA-bundle.crt
```

---

##  Domain

### Domain là gì?
Domain là tên miền đại diện cho một địa chỉ IP trên Internet, giúp người dùng dễ nhớ và truy cập.


# Trạng thái tên miền và hướng xử lý

## Trạng thái chung

### OK / active
- Ý nghĩa: Domain hoạt động bình thường.
- Nên làm: Thêm các trạng thái bảo vệ như `clientTransferProhibited`, `clientDeleteProhibited`, `clientUpdateProhibited`.

### addPeriod
- Ý nghĩa: Vừa được đăng ký, đang trong giai đoạn đầu.
- Nên làm: Không cần xử lý.

### autoRenewPeriod
- Ý nghĩa: Tên miền tự động gia hạn.
- Nên làm: Nếu không muốn, liên hệ nhà đăng ký để hủy.

### inactive
- Ý nghĩa: Chưa cấu hình Name Server.
- Nên làm: Kiểm tra DNS, liên hệ nhà đăng ký.

### pendingCreate
- Ý nghĩa: Đang xử lý đăng ký tên miền.
- Nên làm: Chờ hoàn tất.

### pendingDelete
- Ý nghĩa: Sắp bị xóa.
- Nên làm: Chờ tên miền được giải phóng để đăng ký lại.

### pendingRenew
- Ý nghĩa: Đang chờ gia hạn.
- Nên làm: Đợi xử lý xong hoặc liên hệ xác nhận.

### pendingRestore
- Ý nghĩa: Đang khôi phục tên miền đã hết hạn.
- Nên làm: Theo dõi, nếu về `redemptionPeriod` thì liên hệ khẩn.

### pendingTransfer
- Ý nghĩa: Đang chuyển nhà đăng ký.
- Nên làm: Không yêu cầu chuyển thì liên hệ hủy và khóa lại (`clientTransferProhibited`).

### pendingUpdate
- Ý nghĩa: Đang cập nhật thông tin.
- Nên làm: Nếu không yêu cầu, hãy kiểm tra lại.

### redemptionPeriod
- Ý nghĩa: Domain hết hạn, cần chuộc lại (có phí cao).
- Nên làm: Liên hệ nhà đăng ký ngay để chuộc lại.

### renewPeriod
- Ý nghĩa: Vừa gia hạn, đang chờ xác nhận.
- Nên làm: Chờ xử lý.

### serverDeleteProhibited
- Ý nghĩa: Không thể xóa domain (thường do tranh chấp).
- Nên làm: Liên hệ nhà đăng ký để gỡ bỏ.

### serverHold
- Ý nghĩa: Không được kích hoạt trong DNS.
- Nên làm: Liên hệ nhà đăng ký kiểm tra lý do.

### serverRenewProhibited
- Ý nghĩa: Không thể gia hạn.
- Nên làm: Liên hệ nhà đăng ký, xử lý mất thời gian hơn client*.

### serverTransferProhibited
- Ý nghĩa: Không thể transfer.
- Nên làm: Liên hệ nhà đăng ký.

### serverUpdateProhibited
- Ý nghĩa: Không thể cập nhật thông tin.
- Nên làm: Liên hệ nhà đăng ký.

### transferPeriod
- Ý nghĩa: Sau khi chuyển tên miền, chờ xác nhận hoàn tất.
- Nên làm: Nếu không chủ động chuyển thì kiểm tra ngay.

## Trạng thái tại Nhà đăng ký (Registrar)

| Trạng thái | Ý nghĩa | Hành động |
|-----------|---------|-----------|
| clientDeleteProhibited | Cấm xóa domain | Giúp bảo vệ domain khỏi bị xóa trái phép |
| clientHold | Tạm dừng DNS | Liên hệ nhà đăng ký để xử lý |
| clientRenewProhibited | Cấm gia hạn | Thường do tranh chấp, cần liên hệ để gỡ bỏ |
| clientTransferProhibited | Cấm chuyển đổi NĐK | Bảo vệ domain khỏi bị chiếm đoạt |
| clientUpdateProhibited | Cấm cập nhật thông tin | Ngăn thay đổi trái phép, liên hệ nhà đăng ký nếu cần gỡ |

### Subdomain là gì?
Là miền con của một domain chính, ví dụ: `blog.example.com` là subdomain của `example.com`

### Virtual Hosts là gì?
Là một tính năng trong web server và cũng là một phương thức lưu trữ, cho phép nhiều trang web hoặc tên miền hoạt động trên cùng một máy chủ vật lý hoặc một địa chỉ IP duy nhất.

---

##  Mail Server

### Tìm hiểu MX Record
MX (Mail Exchange) là bản ghi DNS xác định mail server nhận email cho domain.

### Tìm hiểu DKIM, SPF, PTR
- **SPF**: Xác định IP nào được phép gửi email từ domain
- **DKIM**: Ký xác thực nội dung email bằng private key
- **PTR**: Bản ghi ngược ánh xạ IP → domain, hỗ trợ xác minh gửi mail

---

##  DNS

### DNS là gì?
DNS (Domain Name System) là hệ thống phân giải tên miền thành địa chỉ IP.

### Các loại record DNS:
- A: ánh xạ domain → IPv4
- AAAA: ánh xạ domain → IPv6
- CNAME: alias (bí danh)
- MX: email server
- NS: name server
- PTR: ánh xạ ngược
- TXT: chứa thông tin xác thực (SPF, DKIM...)

### Nguyên tắc làm việc của DNS: DNS (Domain Name System) hoạt động theo nguyên tắc phân giải tên miền (domain name) thành địa chỉ IP mà máy tính có thể hiểu và kết nối được.
1. Trình duyệt gửi truy vấn
2. Hệ thống DNS tìm bản ghi tương ứng
3. Trả về IP để trình duyệt kết nối

### Cách phân giải địa chỉ DNS:
```bash
ping vietnix.vn
hping3 vietnix.vn
```
**Giải thích:**
- `ttl=`: Time To Live – số hop còn lại
- `time=`: thời gian phản hồi (ping delay)

---

##  SSH Command
- Dùng password:
```bash
ssh user@host
```
- Dùng key:
```bash
ssh -i ~/.ssh/id_rsa user@host
```
- Dùng custom port:
```bash
ssh -p 2222 user@host
```

---

##  SCP Command
- SCP 1 file:
```bash
scp file.txt user@host:/path
```
- SCP 1 folder:
```bash
scp -r folder/ user@host:/path
```

---

##  Rsync Command
- Rsync file:
```bash
rsync -av file.txt user@host:/path
```
- Rsync folder:
```bash
rsync -av folder/ user@host:/path
```
- Rsync incremental:
```bash
rsync -av --progress --delete source/ dest/
```

---

##  Cat Command
```bash
cat file.txt               # Hiển thị nội dung
sed -n 'n{p;q}' file.txt   # Dòng thứ n
cat <<EOF > file.txt
line1
line2
EOF
```

##  Echo Command
```bash
echo "new line" >> file.txt      # thêm dòng
echo "replace" > file.txt        # ghi đè
```

##  Tail/Head Command
```bash
tail file.txt
tail -f file.txt
head file.txt
```

##  Sed Command
```bash
sed -i 's/old/new/g' file.txt
```

##  Traceroute/Tracert
```bash
traceroute vietnix.vn
```

##  Netstat Command
```bash
netstat -tuln
netstat -n
netstat -n -p
netstat -tunlp
netstat -t       # chỉ TCP
netstat -u       # chỉ UDP
```

##  Sort Command
```bash
sort file.txt
sort -r file.txt
sort -k2 file.txt
```

##  Uniq Command
```bash
uniq file.txt
uniq -c file.txt
```

##  Wc Command
```bash
wc -l file.txt
wc -m file.txt
```

##  Chmod, Chown, Chattr
```bash
chmod 755 file
chmod u+x script.sh
chown user:group file
chattr +i file
```

##  Find Command
```bash
find . -name "*.log"
find . -type d -name abc
find . -type f -name abc
find . -type f -name abc -exec chmod 444 {} \;
```

##  Cp/Mv Command
```bash
cp file.txt /dest/
cp -r folder/ /dest/
mv file.txt /dest/
```

##  Cut Command
```bash
echo "123456" | cut -c3
echo "abcdef" | cut -c3-
echo "abcdef" | cut -c-3
```

##  Dig Command
```bash
dig vietnix.vn A
dig @8.8.8.8 vietnix.vn MX
```

##  Tar/Zip/Unzip Command
```bash
tar -czvf file.tar.gz folder/
tar -xzvf file.tar.gz
zip -r file.zip folder/
unzip file.zip
```

##  Mount/Umount Command
```bash
lsblk
mount /dev/sdb1 /mnt/test
umount /mnt/test
```

##  Symbolic & Hard Link
```bash
ln -s file link_sym
ln file link_hard
```

##  Ls Command
```bash
ls
ls -l
ls -a
```

##  Ps Command
```bash
ps aux
top
kill PID
```

##  Top Command - Giải thích:

- Load average: trung bình tải hệ thống
- us: user CPU
- sy: system CPU
- ni: nice
- id: idle
- wa: wait IO
- hi: hardware interrupt
- si: software interrupt
- st: stolen time
- zombie: tiến trình chết
- sleeping: tiến trình đang chờ

##  Free Command
```bash
free -h
```
- used: đã dùng
- free: còn trống
- buff/cache: bộ nhớ cache

##  Df Command
```bash
df -h
```
- `/` là phân vùng gốc chứa hệ điều hành và file hệ thống
