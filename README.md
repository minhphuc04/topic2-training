
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

### Nguyên tắc làm việc của DNS: 
- DNS (Domain Name System) hoạt động theo nguyên tắc phân giải tên miền (domain name) thành địa chỉ IP mà máy tính có thể hiểu và kết nối được.
1. Trình duyệt gửi truy vấn
2. Hệ thống DNS tìm bản ghi tương ứng
3. Trả về IP để trình duyệt kết nối

### Cách phân giải địa chỉ DNS:
```bash
# ping – kiểm tra kết nối và phân giải tên miền
ping vietnix.vn

# nslookup – Truy vấn bản ghi DNS cơ bản
nslookup vietnix.vn

# dig – Truy vấn DNS nâng cao (chi tiết hơn nslookup)
dig vietnix.vn

# hping3 – Gửi gói tin có kiểm soát, có thể dùng để kiểm tra phản hồi IP
hping3 vietnix.vn -S -p 80
```
**Giải thích:**
- `ttl=`: Time To Live – là một tham số quan trọng trong mạng máy tính, xác định khoảng thời gian hoặc số lần truyền mà dữ liệu được phép tồn tại trước khi bị xóa
- `time=`: thời gian phản hồi (ping delay)

---

##  SSH Command
### SSH Command là gì?

SSH (Secure Shell) là một lệnh trong Linux/Unix dùng để thiết lập kết nối bảo mật đến một máy chủ từ xa thông qua giao thức SSH. Nó cho phép người dùng điều khiển máy chủ từ xa, truyền file, và thực hiện các tác vụ quản trị từ xa một cách an toàn.

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
### SCP Command là gì?

SCP (Secure Copy Protocol) là một lệnh dòng lệnh trong Linux/Unix dùng để sao chép file hoặc thư mục giữa máy tính cục bộ và máy chủ từ xa, hoặc giữa hai máy chủ từ xa, thông qua kết nối SSH.

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
### Rsync Command là gì?

Rsync (Remote Sync) là một lệnh trong Linux/Unix dùng để đồng bộ và sao chép file hoặc thư mục giữa máy cục bộ và máy chủ từ xa, hoặc giữa hai vị trí trên cùng một hệ thống, với khả năng chỉ truyền phần thay đổi của dữ liệu, giúp tiết kiệm băng thông và thời gian.

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
### Cat Command là gì?

Cat (viết tắt của “concatenate”) là một lệnh trong Linux/Unix dùng để hiển thị nội dung của file, nối nhiều file lại với nhau, hoặc chuyển nội dung file vào đầu ra (output).
```bash
cat file.txt               # Hiển thị nội dung
sed -n 'n{p;q}' file.txt   # Dòng thứ n
cat <<EOF > file.txt
line1
line2
EOF
```

##  Echo Command
### Echo Command là gì?

Echo là một lệnh trong Linux/Unix dùng để hiển thị chuỗi văn bản ra màn hình hoặc ghi chuỗi đó vào file. Nó thường được dùng trong shell script để in thông tin, xuất giá trị biến, hoặc tạo nội dung file.

```bash
echo "new line" >> file.txt      # thêm dòng
echo "replace" > file.txt        # ghi đè
```

##  Tail/Head Command
### Tail/Head Command là gì?

Tail và Head là hai lệnh trong Linux/Unix dùng để hiển thị nội dung dòng đầu tiên hoặc dòng cuối cùng của một file văn bản.

```bash
tail file.txt                    # hiển thị 10 dòng cuối của file
tail -f file.txt                 # hiển thị nội dung cuối file theo thời gian thực
head file.txt                    # hiển thị 10 dòng đầu của file
```

##  Sed Command
### Sed Command là gì?

Sed (Stream Editor) là một công cụ dòng lệnh trong Linux/Unix dùng để xử lý và chỉnh sửa văn bản theo luồng. Nó cho phép tìm kiếm, thay thế, xóa hoặc chèn dòng trong file mà không cần mở file bằng trình soạn thảo.

```bash
sed -i 's/old/new/g' file.txt
# Thay tất cả các chuỗi "old" thành "new" trong mỗi dòng (g là global).
```

##  Traceroute/Tracert
### Traceroute/Tracert là gì?

Traceroute (trên Linux/macOS) và Tracert (trên Windows) là công cụ dòng lệnh dùng để theo dõi đường đi của gói tin từ máy tính của bạn đến một địa chỉ đích (IP hoặc domain), thông qua các router trung gian.

Nó giúp xác định số lượng “hops” (bước nhảy) và vị trí xảy ra độ trễ hoặc sự cố trong quá trình truyền dữ liệu qua mạng.


```bash
traceroute vietnix.vn
```

##  Netstat Command
### Netstat Command là gì?

Netstat (Network Statistics) là một lệnh trong Linux/Unix dùng để hiển thị các kết nối mạng, bảng định tuyến, thống kê giao thức, các cổng đang lắng nghe (listening ports), và các tiến trình sử dụng mạng.

Lệnh này thường được sử dụng để kiểm tra tình trạng mạng và phân tích các kết nối TCP/UDP đang hoạt động.

```bash
netstat -tuln
netstat -n
netstat -n -p
netstat -tunlp
netstat -t       # chỉ TCP
netstat -u       # chỉ UDP
```

##  Sort Command
### Sort Command là gì?

Sort là một lệnh trong Linux/Unix dùng để sắp xếp các dòng văn bản trong một file hoặc đầu vào theo thứ tự tăng dần hoặc giảm dần. Nó hỗ trợ sắp xếp theo bảng chữ cái, theo số, theo cột và nhiều kiểu tùy chỉnh khác.

```bash
sort file.txt
sort -r file.txt
sort -k2 file.txt
```

##  Uniq Command
### Uniq Command là gì?

Uniq là một lệnh trong Linux/Unix dùng để lọc ra các dòng trùng lặp liên tiếp trong một file hoặc đầu vào. Nó thường được dùng kết hợp với `sort` để loại bỏ hoàn toàn các dòng trùng nhau và có thể đếm số lần xuất hiện của mỗi dòng.

```bash
uniq file.txt
uniq -c file.txt
```

##  Wc Command
### Wc Command là gì?

Wc (Word Count) là một lệnh trong Linux/Unix dùng để đếm số dòng, số từ và số ký tự trong một file hoặc đầu vào. Đây là công cụ hữu ích để thống kê nhanh nội dung văn bản.

```bash
wc -l file.txt
wc -m file.txt
```

##  Chmod, Chown, Chattr
### Chmod, Chown, Chattr là gì?

- `chmod`: là lệnh dùng để thay đổi quyền truy cập (read, write, execute) của file hoặc thư mục trong Linux/Unix.

- `chown`: là lệnh dùng để thay đổi chủ sở hữu (user) và nhóm (group) của file hoặc thư mục.

- `chattr`: là lệnh dùng để thay đổi thuộc tính mở rộng (extended attributes) của file, ví dụ như đặt file ở chế độ không thể chỉnh sửa (immutable).

```bash
chmod 755 file
chmod u+x script.sh
chown user:group file
chattr +i file
```

##  Find Command
### Find Command là gì?

Find là một lệnh trong Linux/Unix dùng để tìm kiếm file và thư mục theo tên, loại, thời gian sửa đổi, quyền, kích thước hoặc các tiêu chí khác. Nó hỗ trợ tìm kiếm đệ quy trong thư mục và có thể thực hiện hành động trên các file tìm được.

```bash
find . -name "*.log"
find . -type d -name abc
find . -type f -name abc
find . -type f -name abc -exec chmod 444 {} \;
```

##  Cp/Mv Command
### Cp/Mv Command là gì?

- `cp`: là lệnh trong Linux/Unix dùng để sao chép file hoặc thư mục từ vị trí này sang vị trí khác.

- `mv`: là lệnh dùng để di chuyển hoặc đổi tên file và thư mục. Nếu đích đến là cùng tên file,

```bash
cp file.txt /dest/
cp -r folder/ /dest/
mv file.txt /dest/
```

##  Cut Command
### Cut Command là gì?

Cut là một lệnh trong Linux/Unix dùng để trích xuất các phần cụ thể của dòng văn bản, dựa trên vị trí ký tự hoặc trường (field) được phân tách bởi dấu phân cách. Nó thường được sử dụng để lấy cột dữ liệu từ file văn bản hoặc đầu ra của lệnh khác.

```bash
echo "123456" | cut -c3
echo "abcdef" | cut -c3-
echo "abcdef" | cut -c-3
```

##  Dig Command
### Dig Command là gì?

Dig (Domain Information Groper) là một công cụ dòng lệnh trong Linux/Unix dùng để truy vấn hệ thống DNS. Nó cho phép kiểm tra bản ghi DNS (như A, MX, NS, TXT...) của tên miền, hiển thị chi tiết quá trình phân giải và thời gian phản hồi từ máy chủ DNS.

```bash
dig vietnix.vn A
dig @8.8.8.8 vietnix.vn MX
```

##  Tar/Zip/Unzip Command
### Tar/Zip/Unzip Command là gì?

- `tar`: là lệnh trong Linux/Unix dùng để nén hoặc giải nén các file và thư mục thành định dạng `.tar` hoặc `.tar.gz`.

- `zip`: là lệnh dùng để nén file hoặc thư mục thành định dạng `.zip`.

- `unzip`: là lệnh dùng để giải nén các file `.zip` đã được tạo trước đó.

```bash
tar -czvf file.tar.gz folder/
tar -xzvf file.tar.gz
zip -r file.zip folder/
unzip file.zip
```

##  Mount/Umount Command
### Mount/Umount Command là gì?

- `mount`: là lệnh trong Linux/Unix dùng để gắn một thiết bị lưu trữ (như ổ cứng, USB, phân vùng) vào hệ thống file, cho phép truy cập nội dung của nó tại một thư mục cụ thể.

- `umount`: là lệnh dùng để tháo thiết bị lưu trữ ra khỏi hệ thống file một cách an toàn.

```bash
lsblk
mount /dev/sdb1 /mnt/test
umount /mnt/test
```

##  Symbolic & Hard Link
## Symbolic & Hard Link là gì?

- **Symbolic Link (Symlink)**: là một loại file đặc biệt trỏ đến một file hoặc thư mục khác. Nó giống như một “lối tắt”, có thể trỏ đến file trên một phân vùng khác.

- **Hard Link**: là một bản sao tham chiếu trực tiếp đến cùng inode của file gốc. Khi file gốc bị xóa, hard link vẫn giữ nguyên nội dung vì chúng dùng chung inode.

Cả hai đều được dùng để tạo liên kết tới file, nhưng hoạt động và tính chất khác nhau.

```bash
ln -s file link_sym
ln file link_hard
```

##  Ls Command
## Ls Command là gì?

Ls là một lệnh trong Linux/Unix dùng để liệt kê các file và thư mục trong một thư mục cụ thể. Nó hỗ trợ nhiều tùy chọn để hiển thị chi tiết quyền, kích thước, thời gian sửa đổi, file ẩn và sắp xếp theo tiêu chí khác nhau.

```bash
ls
ls -l
ls -a
```

##  Ps Command
## Ps Command là gì?

Ps (Process Status) là một lệnh trong Linux/Unix dùng để hiển thị thông tin về các tiến trình (process) đang chạy trên hệ thống. Nó cho phép người dùng theo dõi PID, trạng thái, người sở hữu, thời gian CPU, và các thông số khác của tiến trình.

```bash
ps aux
top
kill PID
```

##  Top Command 
### Top Command là gì?

Top là một lệnh trong Linux/Unix dùng để hiển thị thời gian thực (real-time) thông tin về các tiến trình đang chạy, mức sử dụng CPU, RAM, và các tài nguyên hệ thống khác. Nó giúp người dùng theo dõi hiệu suất hệ thống và quản lý tiến trình một cách hiệu quả.

- Giải thích:

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
### Free Command là gì?

Free là một lệnh trong Linux/Unix dùng để hiển thị thông tin về bộ nhớ hệ thống, bao gồm bộ nhớ đã sử dụng, còn trống, bộ nhớ đệm (buffer/cache), và swap. Đây là công cụ hữu ích để theo dõi tình trạng sử dụng RAM của hệ thống.

```bash
free -h
```
- used: đã dùng
- free: còn trống
- buff/cache: bộ nhớ cache

##  Df Command
### Df Command là gì?

Df (Disk Free) là một lệnh trong Linux/Unix dùng để hiển thị thông tin

```bash
df -h
```
- `/` là phân vùng gốc chứa hệ điều hành và file hệ thống
