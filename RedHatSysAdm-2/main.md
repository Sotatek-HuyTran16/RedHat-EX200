# Khóa học Red Hat System Admin II (RH134)

## Chapter 1: Cải thiện năng suất command line

Kỹ năng viết shell script là điều cần thiết đối với quản trị viên hệ thống trong bất kỳ môi trường vận hành nào. Shell script giúp cải thiện hiệu quả và độ chính xác của việc hoàn thành các tác vụ thường xuyên.

Nên viết shell script với các trình soạn thảo nâng coa như vim hoặc emacs

### 1.1. Viết bash scripts cơ bản

Dòng đầu tiên của scripts bắt đầu với '#!'
```bash
#!/usr/bin/bash
```

Nếu scripts được lưu trong các thư mục bin chứa câu lệnh (các folder bin thường được chỉ định trong biến môi trường PATH), có thể chạy trực tiếp scripts bằng tên của scripts như 1 command thông thường

```bash
[user@host ~]$ which hello
~/bin/hello
[user@host ~]$ echo $PATH
/home/user/.local/bin:/home/user/bin:/sbin:/bin:/usr/sbin:/usr/bin:/usr/local/sbin:/usr/local/bin
```

Một vài ký tự và từ có thể có ý nghĩa đặc biệt, để viết các ký tự này bthg, dùng '\' ở trước chúng hoặc thêm '' và ""

```bash
[user@host ~]$ echo # not a comment #

[user@host ~]$ echo \# not a comment #
# not a comment
[user@host ~]$ echo \# not a comment \#
# not a comment #
[user@host ~]$ echo '# not a comment #'
# not a comment #
```

Dấu '' sẽ diễn giải toàn bộ câu lệnh theo đúng nghĩa đen, trong khi dấu "" vẫn cho phép chèn các lệnh và tên biến.
```bash
[user@host ~]$ echo "Will variable $var evaluate to $(hostname -s)?"
Biến host có được đánh giá là host không? 
[user@host ~]$ echo 'Will variable $var evaluate to $(hostname -s)?'
Biến $var có được đánh giá là $(hostname -s) không? 
```

Để cung cấp output cho shell scripts, sử dụng 'echo' để gửi các message cho STDOUT (ở đâu là terminal) hoặc có thể sử dụng redirect để lưu STDOUT vào các file (> và >>)

```bash
[user@host ~]$ cat ~/bin/hello
#!/usr/bin/bash

echo "Hello, world"

[user@host ~]$ hello
Hello, world
```

Lệnh 'echo' cũng được sử dụng để hiển thị thông tin hoặc thông báo lỗi, good practice là redirect các lỗi đến STDERR để tách chúng khỏi các STDOUT thông thường.

```bash
[user@host ~]$ cat ~/bin/hello
#!/usr/bin/bash

echo "Hello, world"
echo "ERROR: Houston, we have a problem." >&2

[user@host ~]$ hello 2> hello.log
Hello, world
[user@host ~]$ cat hello.log
ERROR: Houston, we have a problem.
```

### 1.2. Vòng lặp và điều kiện trong shell scripts

Vòng lặp trong scripts có syntax như sau:
```bash
for VARIABLE in LIST; do
COMMAND VARIABLE
done
```

Các giá trị của LIST được lưu như là một VARIBALE trong shell, và các câu lệnh lấy giá trị từ các VARIABLE đó

```bash
[user@host ~]$ for HOST in host{1,2,3}; do echo $HOST; done
host1
host2
host3


[user@host ~]$ for PACKAGE in $(rpm -qa | grep kernel); \
do echo "$PACKAGE was installed on \
$(date -d @$(rpm -q --qf "%{INSTALLTIME}\n" $PACKAGE))"; done
kernel-tools-libs-5.14.0-70.2.1.el9_0.x86_64 was installed on Thu Mar 24 10:52:40 PM EDT 2022
kernel-tools-5.14.0-70.2.1.el9_0.x86_64 was installed on Thu Mar 24 10:52:40 PM EDT 2022
kernel-core-5.14.0-70.2.1.el9_0.x86_64 was installed on Thu Mar 24 10:52:46 PM EDT 2022
kernel-modules-5.14.0-70.2.1.el9_0.x86_64 was installed on Thu Mar 24 10:52:47 PM EDT 2022
kernel-5.14.0-70.2.1.el9_0.x86_64 was installed on Thu Mar 24 10:53:04 PM EDT 2022
```

Sau khi 1 scripts đã chạy toàn bộ nỗi dung, process sẽ thoát và trả quyền cho tiến trình cha của nó. Tuy nhiên, có thể thoát scripts trước khi chạy hết các lệnh bằng 'exit'

Sử dụng exit với giá trị 0 -> 255, các giá trị này biểu diễn exit code. Exit code này sẽ được trả về cho tiến trình cha để biểu thị trạng thái khi thoát.

Exit code 0 biểu duễn scripts đã hoàn thành mà ko có lỗi, tất cả các exit code khác đều thể hiện scripts đã gặp lỗi. Exit code nào đại diện cho lỗi nào thì tùy vòa việc định nghĩa của người lập trình scripts.

Để nhận exit code của câu lệnh cuối cùng đã chạy xong, sử dụng biến ? như sau: '$?'
```bash
[user@host bin]$ cat hello
#!/usr/bin/bash
echo "Hello, world"
exit 0
[user@host bin]$ ./hello
Hello, world
[user@host bin]$ echo $?
0
```

Để đảm bảo các điều kiện bất ngờ không làm gián đoạn scripts, cần phải kiểm tra các input của scripts. Có thể xác định logic của scripts đã đúng chưa bằng lệnh 'test'.

| Option          | Ý nghĩa                                      |
| --------------- | ---------------------------------------------|
|  gt             | Lớn hơn (greater than)                       |
|  ge             | Lớn hơn hoặc bằng (greater than or equal)    |
|  lt             | Nhỏ hơn (lower than)                         |
|  le             | Lớn hơn (lower than or equal)                |
|  =or==          | Bằng nhau (giống nhau)                       |
|  !=             | Khác nhau                                    |
|  z              | Độ dài bằng 0                                |
|  n              | Độ dài khác 0                                |
|  f              | file tồn tại                                 |
|  d              | folder tồn tại                               |
|  L              | file có symbolic link                        |
|  r              | user có quyền đọc                            |
|  ne             | không bằng                                   |
|  eq             | bằng                                         |

Ví dụ:
```bash
[user@host ~]$ test 1 -gt 0 ; echo $?
0
[user@host ~]$ test 0 -gt 1 ; echo $?
1
[user@host ~]$ [[ abc = abc ]]; echo $?
0
[user@host ~]$ [[ abc == def ]]; echo $?
1
[user@host ~]$ STRING=''; [[ -z "$STRING" ]]; echo $?
0
[user@host ~]$ STRING='abc'; [[ -n "$STRING" ]]; echo $?
0
```

**if-else**

Syntax của điều kiện if:
```bash
if <CONDITION>; then
      <STATEMENT>
      ...
      <STATEMENT>
fi

# ví dụ
[user@host ~]$ systemctl is-active psacct > /dev/null 2>&1
[user@host ~]$ if  [[ $? -ne 0 ]]; then sudo systemctl start psacct; fi
```

Syntax của if-else:
```bash
if <CONDITION>; then
      <STATEMENT>
      ...
      <STATEMENT>
else
      <STATEMENT>
      ...
      <STATEMENT>
fi

# ví dụ
[user@host ~]$ systemctl is-active psacct > /dev/null 2>&1
[user@host ~]$ if  [[ $? -ne 0 ]]; then \
sudo systemctl start psacct; \
else \
sudo systemctl stop psacct; \
fi
```

Syntax của if-then-elif-then-else:
```bash
if <CONDITION>; then
      <STATEMENT>
      ...
      <STATEMENT>
elif <CONDITION>; then
      <STATEMENT>
      ...
      <STATEMENT>
else
      <STATEMENT>
      ...
      <STATEMENT>
fi

# ví dụ
[user@host ~]$ systemctl is-active mariadb > /dev/null 2>&1
[user@host ~]$ MARIADB_ACTIVE=$?
[user@host ~]$ sudo systemctl is-active postgresql > /dev/null 2>&1
[user@host ~]$ POSTGRESQL_ACTIVE=$?
[user@host ~]$ if  [[ "$MARIADB_ACTIVE" -eq 0 ]]; then \
mysql; \
elif  [[ "$POSTGRESQL_ACTIVE" -eq 0 ]]; then \
psql; \
else \
sqlite3; \
fi
```

trong case này, bash sẽ thực hiện TH nào match đầu tiên từ trên xuống dưới theo thứ tự và skip các điều kiện còn lại. Nếu ko có điều kiện nào match thì sẽ thực hiện lệnh else.

**Match text với simple regular expression**

ví dụ:
```bash
cat
dog
concatenate
dogma
category
educated
boondoggle
vindication
chilidog
```

Tìm các từ có bắt đầu bằng cat -> ^cat
```bash
cat
category
```

Tìm các từ có kết thúc bằng dog -> dog$
```bash
dog
chilidog
```

Tìm các dòng chứa duy nhất từ cat -> ^cat$
```bash
cat
```

**Match text với extended regular expression**

Tìm từ có chứa cụm từ c theo sau là 1 ký tự bất kỳ rồi đến ký tự t -> c.t
```bash
cat
concatenate
c$t
```

Tìm các từ chứa cụm cat, cot hoặc cut -> c[aou]t

Tìm các từ có chứa cụm bắt đầu bằng c và kết thúc bằng t, không quan trọng đoạn giữa -> c.*t
```bash
cat
coat
culvert
ct # đoạn giữa là rỗng 
```

Tìm các từ có chứa cụm bắt đầu bằng c và kết thúc bằng t, đoạn giữa chỉ gồm các ký tự trong ds [a,o,u] -> c[aou]*t

Tìm các từ chúa cụm bắt đầu bằng c kết thúc bằng t, ở giữa có đúng 2 ký tự -> c.\{2\}t

**Match regular expression ở command line**

Sử dụng lệnh 'grep'
```bash
[user@host ~]$ grep '^computer' /usr/share/dict/words
computer
computerese
computerise
computerite

# sử dụng nhiều match expression
[root@host ~]# cat /var/log/secure | grep -e 'pam_unix' \
-e 'user root' -e 'Accepted publickey' | less
Mar  4 03:31:41 localhost passwd[6639]: pam_unix(passwd:chauthtok): password changed for root
Mar  4 03:32:34 localhost sshd[15556]: Accepted publickey for devops from 10.30.0.167 port 56472 ssh2: RSA SHA256:M8ikhcEDm2tQ95Z0o7ZvufqEixCFCt+wowZLNzNlBT0
Mar  4 03:32:34 localhost systemd[15560]: pam_unix(systemd-user:session): session opened for user devops(uid=1001) by (uid=0)
```

Trong vim, để tìm kiếm các dòng match với expression, nhập '/' sau đó nhập expression và Enter để tìm.

## Chapter 2: Lập lịch cho các task trong tương lai

### 2.1. Lập lịch một user job đã bị hoãn (deferred jobs)
Một số câu lệnh/pipeline có thể cần chạy tại một thời điểm cụ thể (vd: tác vụ bảo trì chỉ chạy vào ban đêm), các lệnh được lập lịch này gọi là các tasks hay các jobs. Deferred xác định rằng các task này sẽ chạy trong tương lai.

Để lập lịch deferred job, sử dụng lệnh 'at'. 'At' cung cấp một system daemon atd và các câu lệnh atq để tương tác với daemon

User có thể xếp các tác vụ vào atd daemon bằng lệnh 'at', atd daemon gồm 26 queue từ a -> z theo thứ tự chữ cái càng sau thì có độ ưu tiên hệ thống càng thấp

Sử dụng cú pháp 'at TIMESPEC' để lên lịch chạy các task

TIMESPEC chỉ định thời gian để tác vụ chạy

Sau khi nhập lệnh at, nhập các câu lệnh cần chạy, hoàn tất bằng cách nhấn Ctrl D

Có thể dùng shell scripts thay cho các lệnh nhập thủ công

```bash
at now +5min < myscript

at 21:03 < myscript
```

TIMESPEC có thể sử dụng các kiểu thời gian:
- now +5min - chạy sau 5 phút
- teatime tomorrow - chạy vào teatime (16h) ngày mai
- noon +4days - chạy vào buổi trưa 4 ngày sau
- chạy vào 5h chiều ngày 03/08/2021

Nếu chỉ có thời gian, ngày sẽ tự động chỉ định là hôm nay hoặc ngày mai tùy TH
```bash
at 21:03 < myscript   # Nếu giờ hiện tại là 21:01, tác vụ sẽ chạy lúc 21:03 cùng ngày.
at 21:00 < myscript   # Nếu giờ hiện tại đã qua 21:00, tác vụ sẽ chạy vào ngày hôm sau.
```

**Quản lý các deferred jobs**

Sử dụng lệnh atq hoặc 'at -l' để xem danh sách các jobs đang pending của user hiện tại
```bash
[user@host ~]$ atq
28  Mon May 16 05:13:00 2022 a user
29  Tue May 17 16:00:00 2022 h user
30  Wed May 18 12:00:00 2022 a user

# 28 là job number
# Mon May 16 05:13:00 2022 - thòi điểm chạy jobs
# a - job được lập lịch trong queue a (queue mặc định là a)
# user - người sở hữu job

# xóa job
atrm <JOBNUMBER>
```

### 2.2. Lập lịch cho các user jobs định kỳ (Recurring jobs)

RHEL cung cấp daemon crond được bật mặc định, 'crond' đọc nhiều file cấu hình: 1 file cấu hình mỗi user và một set các file hệ thống.

Mỗi user có 1 file cá nhân mà họ edit cấu hình bằng lệnh 'crontab -e'

Khi chạy các job định kỳ, các file này cung cấp các điều khiển chi tiết cho admin và user. Nếu job không có chuyển hướng vào file, crond sẽ email bất kỳ output hay lỗi nào đến job owner.

![cronjob](pic/cronjob.png)

User có đặc quyền có thể sử dụng 'crontab -u' đẻ sưa job của user khác

Lệnh crontab ko được sử dụng để quản lý các job hệ thống. Không nên sử dụng crontab với quyền root

**Mô tả format của user job**

Sử dụng lệnh 'crontab -e' để chỉnh sửa file crontab, lệnh này sẽ gọi trình soạn thảo mặc định vim để sửa đổi file.

Các quy tắc trong file crontab:
- Mỗi job chỉ sử dụng 1 dòng duy nhất trong file crontab
- Các dòng trống giúp dễ đọc hơn
- Dòng chú thích bắt đầu bằng '#'
- Biến môi trường có định dạng NAME=value, có ảnh hưởng đến tất cả các dòng sau đó (biến mặc định là SHELL). Biến MAILTO chỉ định địa chỉ email sẽ nhận output.

Cấu trúc file crontab:
```bash
# phút giờ ngày tháng thứ lệnh
```

Crontab không hỗ trợ chỉ định năm nên nếu muốn chỉ định năm cụ thể cần dùng 'at' hoặc systemd timers

```bash
# ví dụ
0 0 1 1 * [ "$(date +\%Y)" = "2026" ] && echo "Happy New Year 2026"
```

Cách sử dụng các ký tự:
- \* chạy ở tất cả các giá trị có thể có
- Số - chỉ định thời gian cụ thể
- x-y - chạy trong phạm vi x -> y
- x,y - danh sách các giá trị
- */x - chạy theo chu kỳ x (vd '*/7' là chạy mỗi 7 phút)
- 3 chữ cái viết tắt cho tháng (Jan, Feb...) và thứ (Mon, Tue...)

Ví dụ:
```bash
# lệnh này sẽ chạy định kỳ bakup vào 9:00 ngày 3/2 hàng năm
0 9 3 2 * /usr/local/bin/yearly_backup

# Gửi email đến mail với nội dung "Chime" mỗi 5 phút từ 9:00 đến 16:00 mỗi thứ 6 trong tháng 7
*/5 9-16 * Jul 5 echo "Chime"

# Từ thứ hai -> chủ nhật được đánh index 0 -> 7 và thứ sáu = 5
# Tương tự, với tháng, thay vì dùng 3 chữ cũng có thể dùng 1-12 cho mỗi tháng

# chạy báo cáo ngày vào 23:58 từ t2 -> t6 
58 23 * * 1-5 /usr/local/bin/daily_report

# Gửi email với tiêu đề checking in vào đại chỉ mail developer@example.com vào 9:00 mỗi ngày trong tuần
0 9 * * 1-5 mutt -s "Checking in" developer@example.com % Hi there, just checking in.
```

### 2.3. Lập lịch các system job định kỳ (Recurring system jobs)

Các task hệ thống định kỳ nên được chạy từ tài khoản root thay vì từ user

Lập lịch các task hệ thống bằng cách sửa file hệ thống crontab thay vì sử dụng lệnh crontab

Crontab toàn cục trong file /etc/crontab cho phép cấu hình các task định kỳ của hệ thống với trường bổ sung để chỉ định tài khoản mà job sẽ chạy

Cấu trúc file crontab:
```bash
SHELL=/bin/bash
PATH=/sbin:/bin:/usr/sbin:/usr/bin
MAILTO=root

# For details see man 4 crontabs

# Example of job definition:
# .---------------- minute (0 - 59)
# |  .------------- hour (0 - 23)
# |  |  .---------- day of month (1 - 31)
# |  |  |  .------- month (1 - 12) OR jan,feb,mar,apr ...
# |  |  |  |  .---- day of week (0 - 6) (Sunday=0 or 7) OR sun,mon,tue,wed,thu,fri,sat
# |  |  |  |  |
# *  *  *  *  * user-name  command to be executed
```

Nên có các file cấu hình trong /etc/cron.d/ chứa các tùy chỉnh của từng cron job, để tránh việc update gói sẽ ghi đè file /etc/crontab và mất jobs

Cron cũng có các thư mục riêng để chứa các tập lệnh được thực hiện định kỳ:
- /etc/cron.hourly/ - Chạy mỗi giờ
- /etc/cron.daily/ - Chạy mỗi ngày
- /etc/cron.weekly/ - Chạy mỗi tuần
- /etc/cron.monthly/ - Chạy mỗi tháng

Các file trong các thư mục này phải là các scripts đã được cấp quyền thực thi, không phải các file crontab (sử dụng chmod +x <script-name>)

**Sử dụng Anacron để quản lý các job định kỳ**

Anacron là công cụ giúp chạy các công việc định kỳ mà không bị bỏ qua nếu hệ thống bị tắt hoặc hibernated

File cấu hình /etc/anacrontab đảm bảo các job đã lên lịch luôn chạy cà không bị skip kể cả khi hệ thống bị tắt.

Ví dụ, khi một system job chạy daily không được thực hiện tại thời gian nó được chỉ định do hệ thống đang reboot thì sau khi hệ thống ready sẽ thực hiện system job đó.

Tham số 'Delay in minutes' trong file /etc/anacrontab có thể chỉ định thời gian tối đa job cho phép delay

Thư mục /var/spool/anacron/ giúp lưu timestamp của các job đã được chạy trong /etc/anacrontab để biết lần cuối cùng job chạy.

Cấu trúc file anacrontab:
- period-in-day: số ngày giữa 2 lần chạy của job, có thể sử dụng macro thay vì số (@daily = 1, @wweekly = 7, @monthly = 30)
- delay-in-minutes: thời gian trễ để chạy job sau khi hệ thống sẵn sàng, giúp tránh việc job chạy ngay lập tức sau khi khởi động
- job-identifier: tên định danh
- command: lệnh thực thi

```bash
<period-in-days> <delay-in-minutes> <job-identifier> <command>

# vd
1 5 daily_backup /usr/local/bin/backup.sh
```

File anacrontab cũng có các biến môi trường với syntax NAME=value. Biến START_HOURS_RANGE chỉ định thời gian job có thể chạy trong khoảng nào, nếu hệ thống bật ngoài giờ này, job sẽ thực hiện vào ngày kế tiếp.

**Systemd timer**

Là một đơn vị dùng để kích hoạt các đơn vị khác (ví dụ file .service) theo thời gian định sẵn.

Systemd timer giúp thay thế cron hoặc anacron trong nhiều tình huống, đồng thời ghi log vào journal giúp dễ giám sát và debug.

Systemd timer service cũng là một dịch vụ được gọi bởi sysstat-collect.timer để thu thấp thống kê hệ thống mỗi 10 phút, file cấu hình là /usr/lib/systemd/system/sysstat-collect.timer

Cấu trúc file:
```bash
...output omitted...
[Unit]
Description=Run system activity accounting tool every 10 minutes

[Timer]
OnCalendar=*:00/10

[Install]
WantedBy=sysstat.service

# OnCalendar=*:00/10 -> kích hoạt service mỗi 10 phút
```

Tên của file .timer phải trùng với tên file .service nó sẽ gọi (vd: backup.timer sẽ gọi unit backup.service)

Một số cấu hình timer:

OnCalendar: lên lịch cố định theo ngày, giờ

- OnCalendar=daily: mỗi ngày 
- OnCalendar=Mon *-*-* 12:00:00 → mỗi thứ Hai lúc 12:00
- OnCalendar=2022-04-* 12:35,37,39:16 → mỗi ngày trong tháng 4/2022 lúc 12:35:16, 12:37:16, 12:39:16

OnUnitActiveSec=15min → chạy sau 15 phút kể từ lần cuối kích hoạt

OnBootSec=1min → chạy sau 1 phút kể từ lúc máy khởi động

Lưu ý: không sửa trưc tiếp các file trong /usr/lib/systemd/system vì sẽ bị ghi đè ngay khi update.

Thay vào đó, hãy tạo 1 bản copy ở /etc/systemd/system và sửa bản copy.

Sau khi đã chỉnh sửa file tại /etc/systemd/system, chạy systemctl reload để nạp cấu hình mới.

Cuối cùng, enable timer bằng 'systemctl --now'

Các file .timer và .service thường được lưu ở 1 trong 2 thư mục:
- /usr/lib/systemd/system - chứa các file cấu hình của systemd do các gói phần mềm cài đặt, dễ bị ghi đè khi update gói
- /etc/systemd/system - chứa các file cấu hình do người dùng tạo, không bị thay đổi theo hệ thống. Các file trong thư mục này sẽ ghi đè các file của thư mục trên nếu có cùng tên

### 2.4. Quản lý các file tạm thời

Các ứng dụng và dịch vụ quan trọng thường sd các file tạm thời để lưu dữ liệu như thư mục /tmp hay lưu các vị trí cụ thể cho các task ở /run, chỉ tồn tại trong bộ nhớ. Các thư mục này sẽ bị xóa khi hệ thống reboot

Việc các thư mục bị xóa có thể khiến các daemon và scripts hoạt động không chính xác.

Việc quản lý các file tạm thời là quan trọng để tránh việc chiếm dụng quá nhiều dung lượng đĩa và dữ liệu cũ không cần thiết.

RHEL sử dụng systemd-tmpfiles, nó cung cấp các phương pháp có cấu trúc và có thể cấu hình để quản lý các folder và file tạm thời

Khi hệ thống khởi động, một trong các dịch vụ đầu tiên là systemd-tmpfiles-setup. Dịch vụ này sẽ chạy lệnh 'systemd-tmpfiles' với option --create và --remove giúp đọc hướng dẫn từ các file cấu hình:
- /usr/lib/tmpfiles.d/*.conf
- /run/tmpfiles.d/*.conf
- /etc/tmpfiles.d/*.conf

Các cấu hình này sẽ liệt kê các file, folder mà systemd-tmpfiles-setup được tạo/sửa/xóa và bảo vệ file/folder với các quyền

**Dọn dẹp file tạm thời bằng systemd timer**

Để tránh việc hệ thống lâu dài chiếm dụng hết không gian đĩa với dữ liệu cũ, systemd-tmpfiles-clean.timer là một timer systemd được sử dụng để dọn dẹp các tệp tin tạm theo định kỳ. Timer này kích hoạt dịch vụ systemd-tmpfiles-clean.service, chạy lệnh systemd-tmpfiles --clean để dọn dẹp tệp tin tạm.

Timer này được cấu hình trong file systemd-tmpfiles-clean.timer.
```bash
# /usr/lib/systemd/system/systemd-tmpfiles-clean.timer
[Unit]
Description=Daily Cleanup of Temporary Directories
Documentation=man:tmpfiles.d(5) man:systemd-tmpfiles(8)
ConditionPathExists=!/etc/initrd-release

[Timer]
OnBootSec=15min
OnUnitActiveSec=1d
```

OnBootSec=15min: Timer sẽ kích hoạt dịch vụ systemd-tmpfiles-clean.service 15 phút sau khi hệ thống khởi động.

OnUnitActiveSec=1d: Timer sẽ tiếp tục kích hoạt dịch vụ này mỗi 24 giờ (1 ngày) sau lần kích hoạt trước đó.

**Dọn dẹp file tạm thời thủ công**

Câu lệnh systemd-tmpfiles --clean là công cụ để dọn dẹp các tệp tạm thời trong hệ thống, giúp giải phóng không gian ổ đĩa và loại bỏ những tệp không còn được sử dụng.

Câu lệnh này sẽ quét qua các tệp cấu hình đã được chỉ định, kiểm tra các tệp và thư mục, và tự động xóa các tệp không còn được truy cập hoặc thay đổi trong một khoảng thời gian nhất định

Syntax của các file cấu hình .conf:
```bash
Type, Path, Mode, UID, GID, Age, and Argument

# vd
D /home/student 0700 student student 1d

# Tạo thư mục /home/student nếu chưa tồn tại, nếu đã tồn tại, sẽ xóa tất cả các tệp trong thư mục này.

# Các tệp không được thay đổi hoặc truy cập trong vòng một ngày (1d) sẽ bị xóa khi thực hiện câu lệnh systemd-tmpfiles --clean
```

Type: chỉ định hành động mà hệ thống phải thực hiện
- d: Tạo thư mục nếu chưa tồn tại.
- D: Tạo thư mục nếu chưa tồn tại và xóa nội dung bên trong nếu tồn tại.
- L: Tạo liên kết tượng trưng (symlink).
- Z: Đặt lại quyền sở hữu, quyền truy cập và các bối cảnh SELinux cho tệp hoặc thư mục.

Path: đường dẫn đến file/folder cần thao tác

Mode: Quyền truy cập (permissions) cho tệp hoặc thư mục.

UID/GID: Người sở hữu (user ID) và nhóm sở hữu (group ID) cho tệp hoặc thư mục.

Age: Thời gian tối đa mà tệp có thể không được thay đổi trước khi bị xóa (ví dụ: 1d cho một ngày).

Argument: Tham số bổ sung cho hành động cần thực hiện (nếu có).

**Quy tắc ưu tiên**
Câu lệnh sẽ đọc cấu hình trong các folder chứa quy tắc tạo, sửa, xóa và bảo vệ tệp:
- /etc/tmpfiles.d/*.conf - chứa các cấu hình tùy chỉnh của user, có ưu tiên cao nhất 
- /run/tmpfiles.d/*.conf - chứa các tệp cấu hình tạm thời được sd bới các daemon hoặc app trong hệ thống. Có mức ưu tiên trung bình và thường là các tệp cấu hình tạm thời
- /usr/lib/tmpfiles.d/*.conf - đây là các tệp cấu hình gốc từ các gói phần mềm và có ưu tiên thấp nhất (ko nên chỉnh sửa trực tiếp các file này)

## Chapter 3: Phân tích và lưu trữ logs

### 3.1. Mô tả kiến trúc ghi log của RHEL

Khi hệ điều hành hoạt động, kernel (nhân hệ điều hành) và các tiến trình khác sẽ ghi lại những sự kiện (events) xảy ra – như khởi động, lỗi, đăng nhập, khởi chạy dịch vụ, v.v.

Có thể xem nội dung log bằng các lệnh như:
- less
- tail

RHEL sử dụng kiến trúc ghi log chuẩn dựa trên giao thức syslog, gồm 2 dịch vụ chính:
- systemd-journald (dịch vụ trung tâm)
- rsyslog (dịch vụ truyền thống)

**systemd-journald**

Thu thập log từ nhiều nguồn:
- Kernel
- Quá trình khởi động hệ thống
- Output từ các daemon (chạy nền)
- Các sự kiện syslog

Ghi log theo định dạng cấu trúc, có chỉ mục (structured, indexed journal)

Mặc định lưu trong RAM → không tồn tại sau khi khởi động lại.

**rsyslog**

Đọc các log từ journald

Xử lý và ghi log ra các file trong hệ thống

Có thể:
- Ghi ra các file khác nhau tùy theo loại sự kiện
- Gửi log tới máy chủ log bên ngoài (centralized logging)

Các thư mục lưu trữ log trong /var/log
- /var/log/messages	- Hầu hết log syslog thông thường
- /var/log/secure - Log về bảo mật, xác thực (login, sudo, SSH,...)
- /var/log/maillog - Log liên quan đến hệ thống mail
- /var/log/cron - Log các tác vụ theo lịch (cron jobs)
- /var/log/boot.log - Log quá trình khởi động (boot)

Một số ứng dụng không sử dụng syslog, mà tự lưu log riêng (vd: Apache lưu tại /var/log/httpd/ hoặc /var/log/apache2/)

### 3.2. File cấu hình rsyslog

File cấu hình rsyslog nằm ở /etc/rsyslog.conf và các cấu hình bổ sung (nếu có) nằm ở /etc/rsyslog.d/*.conf

```bash
#### RULES ####

# Log all kernel messages to the console.
# Logging much else clutters up the screen.
#kern.*                                                 /dev/console

# Log anything (except mail) of level info or higher.
# Don't log private authentication messages!
*.info;mail.none;authpriv.none;cron.none                /var/log/messages

# The authpriv file has restricted access.
authpriv.*                                              /var/log/secure

# Log all the mail messages in one place.
mail.*                                                  -/var/log/maillog


# Log cron stuff
cron.*                                                  /var/log/cron

# Everybody gets emergency messages
*.emerg                                                 :omusrmsg:*

# Save news errors of level crit and higher in a special file.
uucp,news.crit                                          /var/log/spooler

# Save boot messages also to boot.log
local7.*                                                /var/log/boot.log
```

Mỗi dòng gồm: <nguồn-sự-kiện>.<mức-độ-qtrong>    đường_dẫn_log

Nguồn sk (Facilities)

![facilities](pic/facilities.png)

Mức độ quan trọng (Priorities)

![priorities](pic/priorities.png)

Có thể dùng ký tự \* để đại diện cho tất cả mức độ quan trọng
```bash
authpriv.*    /var/log/secure
```

**Log Rotation**

Lệnh 'logrotate' giúp giới hạn dung lượng log bằng cách:
- Đổi tên file log cũ (thêm ngày, ví dụ: messages-20220320)
- Tạo file log mới
- Thông báo cho dịch vụ ghi log (nếu cần)

Các log cũ sẽ bị xóa sau 1 khoảng thời gian

'logrotate' chạy hàng ngày (qua cron hoặc systemd timer)

Các file logrotate:
- /etc/logrotate.conf - Cấu hình tổng (global)
- /etc/logrotate.d/ - Thư mục chứa cấu hình riêng cho từng dịch vụ
- /var/lib/logrotate/status - Theo dõi trạng thái log đã xoay vòng

Ví dụ:
```bash
/var/log/httpd/*.log {
    daily                # Xoay log mỗi ngày
    missingok            # Không báo lỗi nếu log không tồn tại
    rotate 7             # Giữ lại 7 bản log cũ
    compress             # Nén log cũ (gzip)
    delaycompress        # Nén log từ lần xoay kế tiếp
    notifempty           # Không xoay nếu log rỗng
    create 0640 root root # Tạo log mới với quyền
    sharedscripts        # Chạy script sau khi xoay 1 lần cho toàn bộ file
    postrotate
        /bin/systemctl reload httpd.service > /dev/null 2>/dev/null || true
    endscript
}
```

Kiểm tra hoạt động logrotate:
```bash
# chạy thử
logrotate -d /etc/logrotate.conf

# chạy thật
logrotate -f /etc/logrotate.conf
```

Logrotate thường được chạy định kỳ thông qua:
- cronjob hàng ngày
```bash
/etc/cron.daily/logrotate
```
- systemd timer (chỉ có trên RHEL 9+)
```bash
systemctl status logrotate.timer
```

**Cấu trúc 1 dòng log syslog**

```bash
Mar 20 20:11:48 localhost sshd[1433]: Failed password for student from 172.25.0.10 port 59344 ssh2

# Mar 20 20:11:48	Dấu thời gian khi sự kiện xảy ra
# localhost	Tên máy chủ (host) gửi thông điệp log
# sshd[1433]	Tên tiến trình (sshd) và PID (1433) gửi thông điệp
# Failed password for student...	Nội dung log – báo lỗi đăng nhập thất bại cho user student
```

Sử dụng lệnh 'tail -f' để theo dõi log đang ghi theo tgian thực
```bash
# Lệnh này sẽ hiển thị 10 dòng cuối trong file log và liên tục in ra dòng mới được ghi.
tail -f /var/log/secure
```

**Tạo syslog thủ công**

Mục đích:
- Giúp kiểm tra xem cấu hình ghi log đã hoạt động đúng chưa
- Kiểm tra phân quyền file log
- Mô phỏng sự kiện cho debug
- Chèn logger vào shell sscripts để ghi log hoạt động của scripts vào syslog

Câu lệnh:
```bash
logger -p authpriv.notice "Testing secure log"
```

### 3.3. Review System journal

Systemd-journald là dịch vụ lưu các dữ liệu log trong một file nhị phân có cấu trúc, được lập chỉ mục gọi là journal. Dữ liệu này bao gồm các thông tin bổ sung về log của sự kiện.

Để xem các log của từ journal, sử dụng lệnh journalctl để xem tất cả các log hoặc tìm kiếm một vài sự kiện đặc biệt.

Nếu chạy lệnh với quyền root, sẽ xem được tất cả log. User thường cũng có thể xem log nhưng sẽ bị hạn chế xem 1 số log nhất định

```bash
[root@host ~]# journalctl -n 5
Mar 15 04:42:17 host.lab.example.com systemd[1]: Started Hostname Service.
Mar 15 04:42:47 host.lab.example.com systemd[1]: systemd-hostnamed.service: Deactivated successfully.
Mar 15 04:47:33 host.lab.example.com systemd[2127]: Created slice User Background Tasks Slice.
Mar 15 04:47:33 host.lab.example.com systemd[2127]: Starting Cleanup of User's Temporary Files and Directories...
Mar 15 04:47:33 host.lab.example.com systemd[2127]: Finished Cleanup of User's Temporary Files and Directories.
```

Để có thể xem thêm nội dung chi tiết hơn của journal log, bật verbose output
```bash
[root@host ~]# journalctl -o verbose
```

### 3.4. Lưu trữ các system journal

Mặc định RHEL 9 lữu trữ các system journal trong thư mục /run/log và bị hệ thống xóa khi reboot

Có thể đổi cấu hình của systemd-journald trong file /etc/systemd/journald.conf để các journal log được lưu lại sau khi reboot

Thông số Storage trong file sẽ quy định cách lưu trữ log của hệ thống:
- persistent - Lưu log vĩnh viễn tại /var/log/journal (nếu chưa có thì systemd-journald sẽ tạo thư mục này)
- volatile - Lưu tạm thời tại /run/log/journal (mặc định nếu chưa cấu hình gì)
- auto - Nếu tồn tại /var/log/journal thì dùng chế độ persistent, ngược lại dùng volatile
- none - Không lưu log, nhưng vẫn có thể gửi log đi nơi khác (ví dụ qua mạng)

Phải kết hợp lưu log vĩnh viễn và log rotate để không bị đầy dung lượng

Để kiểm tra dung lượng log hiện tại:
```bash
journalctl | grep -E 'Runtime Journal|System Journal'
Mar 15 04:21:14 localhost systemd-journald[226]: Runtime Journal (/run/log/journal/4ec03abd2f7b40118b1b357f479b3112) is 8.0M, max 113.3M, 105.3M free.
```

Để lưu log vĩnh viễn:
- tạo folder /var/log/journal (nếu chưa có)
- Sửa file cấu hình /etc/systemd/journald.conf - đổi Storage=Persistent
- khởi động lại systemd-journald

Khi sd journalctl sẽ hiển thị log của tất cả các lần boot, để phân biệt cần thêm option -b
```bash
journalctl -b # xem log của lần boot hiện tại 
journalctl -b -1 # xem log của lần boot trước đó
journalctl --list-boots # xem ds các lần khởi động mà hệ thống ghi nhận
```

### 3.5. Duy trì thời gian chính xác

Để xem thông tin về thời gian hiện tại, múi giờ, đồng bộ NTP:
```bash
timedatectl

Local time:       Wed 2022-03-16 05:53:05 EDT
Universal time:   Wed 2022-03-16 09:53:05 UTC
RTC time:         Wed 2022-03-16 09:53:05
Time zone:        America/New_York (EDT, -0400)
System clock synchronized: yes
NTP service:      active
RTC in local TZ:  no
```

Thiết lập múi giờ:
```bash
timedatectl list-timezones # Xem danh sách các múi giờ

tzselect # Xác định múi giờ theo vị trí địa lý , đưa ra tên múi giờ phù hợp (không thay đổi hệ thống).

timedatectl set-timezone <múi_giờ> # Đặt múi giờ cho hệ thống

timedatectl set-timezone Asia/Ho_Chi_Minh

timedatectl set-ntp true/false # Bật/Tắt đồng bộ hóa NTP

timedatectl set-time "2025-08-07 09:00:00" # Cập nhật thủ công thời gian hệ thống khi đã tắt NTP
```

**Cấu hình và giám sát dịch vụ chronyd**

chronyd giúp đồng bộ thời gian hệ thống bằng cách điều chỉnh Real-Time Clock (RTC) của máy, so với các máy chủ NTP được cấu hình.

Nếu không có mạng, chronyd sẽ tính độ lệch (drift) của RTC và lưu thông tin này vào file do driftfile chỉ định trong /etc/chrony.conf.

Mặc định, chronyd sử dụng các máy chủ NTP từ NTP Pool Project (các máy chủ công cộng) để đồng bộ thời gian, không cần cấu hình thêm.

Nếu máy nằm trong mạng nội bộ (không ra Internet), cần thay đổi máy chủ NTP trong file /etc/chrony.conf.

Stratum là tầng (độ sâu) của máy chủ NTP trong chuỗi đồng bộ:
- Stratum 0: Nguồn đồng hồ chuẩn độ chính xác cao (ví dụ: đồng hồ nguyên tử).
- Stratum 1: Máy chủ trực tiếp kết nối với Stratum 0.
- Stratum 2: Máy tính lấy đồng bộ từ Stratum 1, v.v.

Các loại nguồn thời gian trong chrony.conf
- server: máy chủ NTP ở tầng trên (stratum thấp hơn), được ưu tiên.
- peer: máy đồng cấp (stratum ngang bằng).

Có thể khai báo nhiều server hoặc peer, mỗi dòng một địa chỉ máy chủ hoặc peer.

Sử dụng chronyc để giám sát chronyd
```bash
chronyc sources # Kiểm tra các nguồn thời gian mà chronyd đang sử dụng

chronyc sources -v # Kiểm tra chi tiết các nguồn thời gian mà chronyd đang sử dụng

  .-- Source mode  '^' = server, '=' = peer, '#' = local clock.
 / .- Source state '*' = current best, '+' = combined, '-' = not combined,
| /             'x' = may be in error, '~' = too variable, '?' = unusable.
||                                                 .- xxxx [ yyyy ] +/- zzzz
||      Reachability register (octal) -.           |  xxxx = adjusted offset,
||      Log2(Polling interval) --.      |          |  yyyy = measured offset,
||                                \     |          |  zzzz = estimated error.
||                                 |    |           \
MS Name/IP address         Stratum Poll Reach LastRx Last sample
===============================================================================
^* 172.25.254.254                3   6    17    26  +2957ns[+2244ns] +/-   25ms
```

chronyd rất quan trọng để giữ thời gian chính xác cho hệ thống, đặc biệt trong các môi trường cần đồng bộ log hoặc các dịch vụ nhạy cảm với thời gian

## Chapter 4: Lưu trữ và chuyển các file

### 4.1. Quản lý lưu trữ các file đã nén

Archive là một file duy nhất (file bình thường hoặc thiết bị lưu trữ như ổ đĩa, USB, v.v) chứa nhiều file con khác

Archive giống như 1 hộp đựng nhiều file để dễ dàng quản lý, di chuyển hoặc sao lưu

Vậy, điểm khác nhau giữa archive file và folder là gì?

| Tiêu chí             | Folder (Thư mục) | Archive (Tệp lưu trữ) |
|----------------------|------------------|---------------------------|
|Tồn tại               | Trực tiếp trên hệ thống tập tin | Là một tệp duy nhất trên hệ thống |
|Có thể chứa file/thư mục con? | Có | Có |
|Có thể truy cập trực tiếp?	   | Có (cd, ls, open...)|Không (cần giải nén hoặc dùng lệnh để xem)|
|Có thể nén không?	           | Không (tự thân thư mục không nén được)	|Có thể nén để giảm dung lượng|
|Dùng để backup/chuyển file?   | Không tiện (nhiều file riêng lẻ)	|Rất tiện (gom lại 1 file duy nhất)|
|Dễ truyền qua mạng/email?     | Không (nhiều file rời rạc)|	Có (1 file duy nhất)|
|Thay đổi nội dung dễ không?   | Rất dễ (copy, move, delete…)	|Không dễ (phải giải nén rồi nén lại)|


File archive có thể lưu trên hệ thống file-system hoặc lưu trực tiếp vào các thiết bị lưu trữ như:
- ổ đĩa bằng (tape drive)
- USB (flash drive)
- Đĩa CD/DVD, ổ cứng rời ...

File archive có thể được tạo ra mà không cần nén hoặc nén tùy nhu cầu sd

Archive là một thuật ngữ phổ biến và có rất nhiều các loại file archive khác nhau. Một số loại archive thường gặp:
- .zip - sd chủ yếu trên Win, có thể nén và chứa nhiều file
- .tar - là định dạng archive ko nén trên Linux, thường được kết hợp với các thuật toán nén như gzip, xz để tạo ra các file như .tar.gz, .tar.xz
- .rar - một định dạng archive phổ biến trên Win

Nén (compression) có thể được sd để giảm kích thước của file archive

Trên Linux, tar là công cụ chuẩn để làm việc cới archive. Chức năng của tar:
- Tạo file archive từ các file
- Nén file archive
- Liệt kê nội dung file archive
- Giải nén

Cấu trúc file archive dạng .tar:
- Metadata chứa thông tin của file (tên, kích thước, quyền...)
- Dữ liệu
- Index giúp tìm và trích xuất file 

Tar cũng có thể xem nội dung của file archive đã nén mà không cần giải nén

Các option:

![tar](pic/tar.png)

Câu lệnh:
```bash
# tạo file archive
tar -cf <tên-file-archive> <các-file/folder-trong-đó>

# vd
tar -cf mybackup.tar myapp1.log myapp2.log myapp3.log
```

Khi tạo file archive, các file gốc sẽ được copy bản sao vào file archive chứ ko bị di chuyển.

Nếu sử dụng đường dẫn tuyệt đối từ thư mục '/', tar sẽ loại bỏ dấu '/' đi (ví dụ: /etc/mess -> etc/mess).

Điều này giúp tránh ghi đè lên file hệ thống khi giải nén, các file giải nén sẽ nằm đúng vị trí file archive. Các nội dung vẫn sẽ được đưa vào file archive đầy đủ.

```bash
tar -cf etc-backup.tar /etc

# nội dung file .tar
etc/
etc/fstab
etc/hosts
...
```

Để liệt kê nội dung file
```bash
[root@host ~]# tar -tf /root/etc.tar
etc/
etc/fstab
etc/crypttab
etc/mtab
...output omitted...
```

Để giải nén file .tar và lấy các nội dung, cần có 1 folder trống để đưa nội dung vào đó, tránh việc bị ghi đè 1 số file lên các file hiện tại (nếu trùng tên)

Việc giải nén sẽ đưa các file trong archive ra thư mục đang làm việc

```bash
[user@host ~]# pwd
/home/user
[user@host ~]# ls
archive.tar
[user@host ~]# tar -tf /archive.tar
documents/file1.txt
documents/file2.txt
...output omitted...
[user@host ~]# tar -xf archive.tar
[user@host ~]# ls
archive.tar documents
[user@host ~]# ls ./documents
file1.txt file2.txt
```

Nếu cần chỉ định nơi giải nén
```bash
tar -xf archive.tar -C /đường/dẫn/đích
```

**Tạo 1 file archive đã bị nén (compression)**

|Loại nén|Tùy chọn|Đuôi file|Ưu điểm|Nhược điểm|
|--------|--------|---------|-------|----------|
|gzip	|-z	|.tar.gz|	Nhanh, tương thích cao|	Tỉ lệ nén thấp hơn|
|bzip2	|-j|	.tar.bz2|	Nén tốt hơn gzip|	Chậm hơn|
|xz	|-J|	.tar.xz	|Tỉ lệ nén cao nhất|	Chậm nhất, không luôn có sẵn|

```bash
# gzip
tar -czf /root/etcbackup.tar.gz /etc

# bzip2
tar -cjf /root/logbackup.tar.bz2 /var/log

# xz
tar -cJf /root/sshconfig.tar.xz /etc/ssh
```

**Giải nén file archive đã nị nén bởi các thuật toán**

Cú pháp:
```bash
tar -xf file.tar.gz
```

Mặc định, tar sẽ tự xem đuôi file để biết thuật toán nén và giải nén, không cần chỉ định option

Nếu có chỉ định option mà không đúng với thuật toán nén sẽ gây lỗi

```bash
tar -xzf file.tar.xz
# kqua
gzip: stdin: not in gzip format
tar: Child returned status 1
tar: Error is not recoverable: exiting now
```

### 4.2. Truyền file có bảo mật giữa các hệ thống

Sử dụng SSH để truyền file, 2 công cụ phổ biến:
- sftp - an toàn, giống ftp + ssh
- tcp - copy nhanh qua ssh

Với sftp, phải tạo kết nối trước
```bash
[user@host ~]$ sftp remoteuser@remotehost
remoteuser@remotehost's password: password
Connected to remotehost.
sftp>
```

Sau khi kết nối thành công, bạn sẽ ở sftp và trên máy remote. Lúc này, các câu lệnh tạo file sẽ tạo trên máy remote

```bash
# upload file từ local lên remote
sftp> put /path/to/localfile /remote/path/targetfile

# download file từ remote về local
sftp> get /remote/path/remotefile /local/path/localfile

# Tải file từ remote đến thư mục làm việc tại local
# chỉ hỗ trợ down về, cách này ko put file lên được
sftp user@host:/remote/path/file


```

![sftp](pic/sftp.png)

Với scp thì đơn giản hơn, chỉ cần gõ lệnh để download hoặc upload file
```bash
# syntax
scp [source] [destination]

# download từ remote -> local
scp user@remotehost:/etc/yum.conf /home/user

# upload local -> remote
scp myscript.sh user@remotehost:/home/user/

# thêm option -r cho folder
scp -r folder/ user@remotehost:/home/user/
```

Trước RHEL 9, scp dùng giao thức cũ rcp – có lỗ hổng bảo mật (CVE-2020-15778).

Từ RHEL 9, scp mặc định dùng giao thức sftp (an toàn hơn).

Đừng dùng option -0 vì nó ép dùng giao thức cũ → có thể bị tấn công

### 4.3. Đồng bộ hóa file giữa các hệ thống 1 cách bảo mật

Lệnh rsync là 1 cách khác để copy file từ hệ thống này sang hệ thống khác, giúp đồng bộ dữ liệu giữa máy local và remote.

Tool này dùng thuật toán để thay đổi lượng dữ liệu copy ít nhất có thể bằng cách chỉ đồng bộ hóa các phần đã bị thay đổi

rsync bảo toàn quyền, chủ sở hữu, thời gian chỉnh sửa... nếu dùng đúng tùy chọn.

```bash
# syntax
rsync [options] SOURCE DESTINATION

# kết nối chiều nào cũng như nhau
rsync -av /local/path user@remote:/remote/path

rsync -av user@remote:/remote/path /local/path
```

Các option của rsync
![rsync1](pic/rsync-1.png)

Nếu có option -a (archive mode) thì sẽ có thêm các option bổ sung

![rsync2](pic/rsync-2.png)

Để sd các flag con của option -a thì nên ghi rõ từng flag. Nếu chỉ gõ -a thì tương đương với -rlptgoD
```bash 
rsync -rlt --no-p --no-g --no-o /src/ /dest/

rsync -rltv --no-p --no-g --no-o /src/ /dest/
```

Dấu '/' ở cuối đường dẫn có ảnh hưởng đến cách đông bộ giữa 2 file:
```bash
# Chỉ nội dung thư mục /src sẽ được chép vào /dest
rsync -av /src/ /dest/

# Thư mục /src sẽ được chép vào dưới dạng /dest/src/
rsync -av /src /dest/
```

Có thêm 2 option giúp bảo toàn các thuộc tính mở rộng
- -A bảo toàn Access Control Lists (ACLs)
- -X bảo toàn SELinux file contexts

## Chapter 5: Điều chỉnh (Tune) hiệu suất hệ thống

### 5.1. Điều chỉnh Tuning Profile

tuned là một daemon (dvu ngầm) của hệ thống dùng để điều chỉnh các tham số của hệ thống (điều chỉnh động hoặc tĩnh) bằng cách sử dụng tuning profile - phản ánh các yêu cầu cụ thể của workload

tuned điều chỉnh cả statically (cố định) và dynamically (động, thay đổi theo tải hệ thống)

|Static Tuning|	Dynamic Tuning|
|---|-----|
|Thiết lập một lần khi khởi động	|Theo dõi và điều chỉnh trong thời gian thực|
|Không thay đổi khi hệ thống bận	|Thay đổi phù hợp với workload hiện tại|
|Dễ cấu hình, ít tiêu tốn tài nguyên|	Phức tạp hơn nhưng tối ưu hiệu suất tốt hơn|

Các loại tuning profile (RHEL 9)
![tuning-profile](pic/tuning-profile.png)

**Static Tuning**

Điều chỉnh tĩnh apply các cài đặt hệ thống khi dịch vụ tuning start hoặc khi áp dụng profile mới

```bash
# Cài tuned
dnf install tuned -y
systemctl enable --now tuned

# chọn profile
tuned-adm profile throughput-performance

# lúc này, các tham số trong profile sẽ được áp dụng ngay lập tức và sẽ ko đổi đến khi apply profile mới

# ktra profile đang hoạt động
tuned-adm active
```

**Dynamic Tuning**

Mặc định, dynamic tuning tắt, cần cấu hình thủ công
```bash
# File cấu hình: /etc/tuned/tuned-main.conf
dynamic_tuning = 1         # bật dynamic tuning
update_interval = 10       # cập nhật mỗi 10 giây
```

Mặc định cấu hình của hệ thống là file /usr/lib/tuned. Các profile do user tùy chỉnh nằm ở /etc/tuned

Tương tự các file cấu hình khác, ko nên sử ở /usr/lib vì dễ bị thay đổi khi package update. Hãy copy sang /etc/tuned và sửa.

Hệ thống sẽ ưu tiên sd các file ở /etc/tuned/

Một số câu lệnh:
```bash
# Danh sách profile có sẵn
tuned-adm list

# Kiểm tra profile đang dùng
tuned-adm active

# Kích hoạt một profile
tuned-adm profile throughput-performance

# xem mô tả profile
tuned-adm profile_info profile-name

# gợi ý profile phù hợp với hệ thống
tuned-adm recommend

# tắt tuned và hủy áp dụng các profile
tuned-adm off
```

### 5.2. Cấu hình việc lập lịch cho các tiến trình (Influence Process Scheduling)

Các hệ thống hiện đại sd CPU multi-cores, multi-thread giúp thực thi đồng thời nhiều tiến trình song song, gây ra bão hòa CPU

Linux sử dụng kỹ thuật gọi là time-slicing hoặc multitasking để quản lý các tiến trình

Hệ điều hành sử dụng bộ lập lịch tiến trình (process scheduler) để liên tục chuyển đổi giữa các process (các thread) đang chờ được xử lý

Trên mỗi lõi CPU, các tiến trình không chạy đồng thời thực sự, mà được chia thời gian rất nhỏ (time slices) — thường là vài mili-giây.

Mỗi tiến trình đều có một mức độ ưu tiên (priority) để hệ điều hành quyết định nên cấp CPU cho tiến trình nào trước.

Linux dùng nhiều chính sách lập lịch (scheduling policies):
- SCHED_OTHER (hay SCHED_NORMAL) – dùng cho tiến trình thông thường.
- SCHED_FIFO, SCHED_RR – dùng cho tiến trình real-time.

Chính sách SCHED_OTHER sử dụng thuật toán Completely Fair Scheduler (CFS) để chọn tiến trình tiếp theo.

CFS tổ chức tiến trình trong một cây nhị phân (binary tree) thay vì sd hàng đợi, binary tree có thứ tự:
- Thời gian CPU đã sử dụng (ít hơn -> được xử lý trước)
- Nice value (có thể thay đổi)

Các tiến trình SCHED_NORMAL có real-time priority là 0, tức là các tiến trình real-time luôn có độ ưu tiên cao hơn

Priority là mức độ quan trọng mà hệ điều hành dùng để quyết định tiến trình nào sẽ được CPU xử lý trước khi có nhiều tiến trình đang chờ cùng lúc

**Nice value**

Là giá trị ảnh hưởng đến độ ưu tiên của tiến trình trong CFS do người dùng thay đổi, ảnh hường gián tiếp đến priority của tiến trình trong hệ thống

Nằm trong khoảng -20 (độ ưu tiên cao nhất) -> 19 (độ ưu tiên thấp nhất), các tiến trình có Nice value mặc định là 0

Nice value không đảm bảo tuyệt đối CPU time

Chỉ user root mới có quyền thay đổi nice của các tiến trình và giảm nice (tăng độ ưu tiên).

User bình thường chỉ có thể tự giảm độ ưu tiên của các tiến trình user đó sở hữu

**Quan hệ giữa nice value và priority**

![nice-value](pic/nice-value.png)

Tiến trình real-time không sử dụng nice, tuy nhiên có priority cao nên được ưu tiên so với các tiến trình thông thường

Các tiến trình thường (ko phải real-time) mới cần sử dụng CFS để quyết định priority, các tiến trình này dựa trên:
- Nice value
- Tgian sử dụng CPU
- Loại tiến trình

Để xem các tiến trình cùng với Nice value và Priority, sử dungjL
- top - xem tiến trình, PR, NI, %CPU
- ps - xem thông tin tiến trình và các scheduling class (CLS)

```bash
# lệnh top
Tasks: 192 total,   1 running, 191 sleeping,   0 stopped,   0 zombie
%Cpu(s):  0.0 us,  1.6 sy,  0.0 ni, 96.9 id,  0.0 wa,  0.0 hi,  1.6 si,  0.0 st
MiB Mem :   5668.6 total,   4655.6 free,    470.1 used,    542.9 buff/cache
MiB Swap:      0.0 total,      0.0 free,      0.0 used.   4942.6 avail Mem

    PID USER      PR  NI    VIRT    RES    SHR S  %CPU  %MEM     TIME+ COMMAND
      1 root      20   0  172180  16232  10328 S   0.0   0.3   0:01.49 systemd
      2 root      20   0       0      0      0 S   0.0   0.0   0:00.01 kthreadd
      3 root       0 -20       0      0      0 I   0.0   0.0   0:00.00 rcu_gp
      4 root       0 -20       0      0      0 I   0.0   0.0   0:00.00 rcu_par_gp

# lệnh ps
[user@host ~]$ ps axo pid,comm,nice,cls --sort=-nice
  PID COMMAND          NI CLS
   33 khugepaged       19  TS
   32 ksmd              5  TS
  814 rtkit-daemon      1  TS
    1 systemd           0  TS
    2 kthreadd          0  TS
    5 kworker/0:0-cgr   0  TS
    7 kworker/0:1-rcu   0  TS
    8 kworker/u4:0-ev   0  TS
   15 migration/0       -  FF
...output omitted...
```

Các tiến trình mới được tạo thường sẽ kế thừa nice value từ tiến trình cha và thường bằng 0

```bash
# thêm nice -n <giá-trị> để thêm nice value cho tiến trình khi bắt đầu chạy
nice -n 15 sleep 60 &

# nếu ko chỉ định 1 giá trị, nice sẽ +10 vào nice value của lệnh
nice sleep 60 &

# thay đổi nice của tiến trình đang chạy với renice và PID
renice -n 19 <PID>
```

## Chapter 6: Quản lý bảo mật SELinux