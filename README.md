I. SSL

SSL là gì?

SSL là một giao thức mã hóa nhằm bảo mật truyền tải dữ liệu giữa client và server, được thay thế bởi TLS nhưng tên "SSL" vẫn phổ biến.

Có bao nhiêu cách xác thực SSL?

Theo loại chứng chỉ:

DV (Domain Validation)

OV (Organization Validation)

EV (Extended Validation)

Theo cách kết nối:

One-way SSL

Mutual SSL

CSR file dùng làm gì trong quá trình tạo SSL

CSR (Certificate Signing Request) là yêu cầu ký chứng chỉ, chứa public key và thông tin tổ chức, dùng gửi đến CA để cấp chứng chỉ.

Sử dụng OpenSSL để gen file CSR sau đó request SSL cho domain tech.training.vietnix.tech

openssl req -new -newkey rsa:2048 -nodes -keyout tech.training.vietnix.tech.key -out tech.training.vietnix.tech.csr

Pem file là gì?

PEM là định dạng file text (Base64) chứa chứng chỉ, khóa riêng, CA chain. Đuôi phổ biến: .pem, .crt, .key.

Private key SSL là gì?

Private key là khóa bí mật dùng để giải mã dữ liệu được mã hóa bởi public key hoặc để ký xác thực.

PFX file là gì? Cách chuyển từ file crt file sang PFX file.

PFX (hoặc .p12) là định dạng chứa private key, certificate và CA chain.

openssl pkcs12 -export -out certificate.pfx -inkey private.key -in certificate.crt -certfile ca_bundle.crt

📌 Ví dụ: Gộp các file .crt, .key và chuỗi CA vào một file .pfx để cài lên Windows Server hoặc IIS.

II. Domain

Domain là gì?

Domain là tên dễ nhớ dùng để truy cập website thay vì IP, ví dụ: vietnix.vn.

Các trạng thái của domain

OK, ACTIVE, CLIENT HOLD, REDEMPTION, PENDING DELETE, PENDING TRANSFER...

Subdomain là gì?

Là phần mở rộng của domain chính, ví dụ: mail.vietnix.vn, blog.google.com.

Virtual Hosts là gì?

Là kỹ thuật cấu hình nhiều website trên cùng một server thông qua tên miền.

III. Mail Server

Tìm hiểu MX Record

MX (Mail Exchange) xác định máy chủ nhận email cho domain.

Tìm hiểu DKIM, SPF, PTR

DKIM: Ký xác thực email bằng private key

SPF: Xác định IP nào được phép gửi email từ domain

PTR: Bản ghi DNS ngược (IP -> domain)

IV. DNS

DNS là gì?

DNS là hệ thống phân giải tên miền thành địa chỉ IP tương ứng.

Các loại record DNS

A, AAAA, CNAME, MX, NS, TXT, PTR

Nguyên tắc làm việc của DNS

Client gửi truy vấn → Resolver → Root → TLD → Authoritative → Trả IP

Cách phân giải địa chỉ DNS

ping vietnix.vn
hping3 vietnix.vn

ttl= là gì?

TTL (Time To Live): Số hop (bước) tối đa gói tin có thể đi qua trước khi bị loại bỏ.

time= là gì?

Thời gian (ms) để gói tin đi từ máy bạn đến đích và quay về.

V. SSH command

Dùng password

ssh user@ip

Dùng key

ssh -i privatekey.pem user@ip

Dùng port custom

ssh -p 2222 user@ip

VI. SCP command

SCP 1 file

scp file.txt user@host:/path

SCP 1 folder

scp -r folder user@host:/path

VII. RSYNC command

rsync file

rsync file.txt user@host:/path

rsync folder

rsync -avz folder user@host:/path

rsync incremental

rsync -av --delete src/ dest/

📌 Chỉ đồng bộ các file mới hoặc thay đổi, giúp tiết kiệm băng thông.

VIII. cat command

Cat nội dung 1 file

cat file.txt

Cat dòng thứ  trong file

sed -n '5p' file.txt

Cat nhiều dòng vào 1 file bằng EOF

cat <<EOF > file.txt
nội dung...
EOF

IX. echo command

Dùng echo để chèn thêm 1 dòng vào cuối file

echo "export PATH=/opt/bin:\$PATH" >> ~/.bashrc

Dùng echo để overwrite nội dung của file

echo "nội dung mới" > file.txt

X. tail/head command

tail và tailf

tail file.txt
tail -f file.txt

XI. sed command

Dùng sed để find and replace một string trong file

sed -i 's/localhost/127.0.0.1/g' nginx.conf

📌 Ví dụ: thay toàn bộ từ localhost thành 127.0.0.1 trong file cấu hình Nginx.

XII. traceroute/tracert command

Sau khi traceroute xong giải thích chi tiết kết quả trả về

traceroute vietnix.vn

Mỗi dòng là 1 hop (router trung gian)

Mỗi thời gian là độ trễ đến router đó (ms)

XIII. netstat command

Hiển thị các socket đang listen

netstat -tuln

Don't resolve hostname

netstat -n

Don't resolve portname

netstat -n

Display process name/PID

netstat -tulpn

Only show TCP socket

netstat -tn

Only show UDP socket

netstat -un

XIV. sort command

sort theo thứ tự tăng dần

sort file.txt

sort theo thứ tự giảm dần

sort -r file.txt

sort theo column

sort -k 2 file.txt

XV. uniq command

Lọc ra các dòng lặp lại trong một file

uniq file.txt

Lọc ra các dòng lặp lại trong file và đếm số lượng các dòng lặp lại

uniq -c file.txt

XVI. wc command

Đếm số dòng trong file

wc -l file.txt

Đếm số kí tự trong file

wc -m file.txt

XVII. chmod, chown, chattr command

Phân quyền bằng số, phân quyền bằng chữ

chmod 755 file
chmod u+x file

Đổi owner user/group

chown user:group file

Set Immutable Attribute

chattr +i file.txt

📌 Không thể xóa hoặc sửa file khi đã gán thuộc tính này.

XVIII. find command

Find các file có đuôi .log

find /var/log -name "*.log"

Find các folder có tên abc

find / -type d -name "abc"

Find các file có tên abc

find / -type f -name "abc"

Find các file có tên abc và thực hiện phân quyền read only cho file

find / -type f -name "abc" -exec chmod 444 {} \;

XIX. cp command

cp file

cp file.txt /backup/

cp folder

cp -r config/ /backup/

XX. mv command

mv file, folder

mv test.txt /tmp/
mv folder1/ /tmp/

XXI. cut command

cut kí tự thứ  trong string

echo "abcdef" | cut -c3

📌 In ra ký tự thứ 3 → c

cut từ kí tự thứ  trở về sau

echo "abcdef" | cut -c4-

📌 In ra từ ký tự thứ 4 trở đi → def

cut từ kí tự thứ  trở về trước

echo "abcdef" | cut -c-4

📌 In ra từ ký tự đầu đến ký tự thứ 4 → abcd

XXII. dig command

Dùng Dig command để kiểm tra resolv record A, MX, NS

dig vietnix.vn A

Dùng Dig command để kiểm tra resolv record A, MX, NS với custom DNS

dig @8.8.8.8 vietnix.vn MX

XXIII. tar/zip/unzip command

Nén/Giải nén file tar.gz

tar -czvf file.tar.gz folder/
tar -xzvf file.tar.gz

Nén/Giải nén file .zip

zip -r archive.zip folder/
unzip archive.zip

XXIV. mount/umount command

Add thêm một ổ cứng sdb ~ 5gb

lsblk

Kiểm tra được có bao nhiêu ổ cứng trên máy chủ

fdisk -l

Mount ổ cứng vào /mnt/test

mount /dev/sdb1 /mnt/test

Umount /mnt/test

umount /mnt/test

XXV. Symbolic Links, Hard Links command

Định nghĩ Sym Link

Là một lối tắt (shortcut) trỏ tới file thật. Nếu file gốc bị xóa, symlink sẽ hỏng.

Định nghĩ Hard Link

Là một bản sao cấp hệ thống tệp. Nếu file gốc bị xóa, hard link vẫn tồn tại.

Ví dụ về Sym Link và Hard Link

ln -s /etc/nginx/nginx.conf nginx-link  # symbolic link
ln /etc/nginx/nginx.conf nginx-hard     # hard link

XXVI. ls command

Liệt kê danh sách file/thư mục

ls

Liệt kê danh sách file/thư mục và thuộc tính

ls -l

Show file ẩn

ls -a

XXVII. ps command

show tiến trình

ps aux

kill tiến trình

kill <PID>

XXVIII. top command

Kiểm tra tài nguyên cpu đang sử dụng của một vài process đang chạy

top

Giải thích về Load average, us, sy, ni, id, wa, hi, si, st, zombie process, sleeping process

us: CPU cho user

sy: CPU cho system

ni: ưu tiên thấp

id: idle

wa: đợi I/O

hi/si: ngắt phần cứng/phần mềm

st: bị steal (ảo hóa)

zombie: tiến trình chết chưa bị dọn

sleeping: tiến trình đang chờ

XXIX. free command

Giải thích ram used, free, shared, buff/cache, free

free -h

used: RAM đang sử dụng

free: RAM chưa dùng

shared: RAM dùng chung giữa tiến trình

buff/cache: bộ đệm I/O

XXX. df command

Xem dung lượng disk

df -h

Phân vùng / là gì

Là phân vùng gốc chứa toàn bộ hệ điều hành và các thư mục hệ thống Linux (/bin, /etc, /home, ...).
