---
title: Creating static site with Jekyll
date: 2024-09-09
category: [ctf]
tag: [web, crypto, forensic]
---

# Web 

## Empty Execution

### Tổng quan 

Khi mở bài tập ta thấy website cho ta phần src code. Từ đây ta phân tích có đường dẫn `/run_command` nhận request `POST` để nhận một data dưới dạng `json`. Ngoài ra web cũng sẽ check xem đầu vào có chứa `..` hay `/` không.

![](../assets/Cookie%20Arena/img/Overview_Empty_Execution.png)


### Ý tưởng

Ta nghĩ ngay đến `OS command injection`. Tuy nhiên câu lệnh không được ngắn hơn 5 và cần phải qua `os.access(executable_to_run, os.X_OK)`.  Chúng ta dùng 1 trick như sau:

- Lệnh command.split()[0]: tách chuỗi command thành một danh sách các từ

        Đối với chuỗi . ; cat flag.txt, kết quả sẽ là ['.', ';', 'cat', 'flag.txt']. command.split()[0] sẽ trả về phần tử đầu tiên của danh sách, trong trường hợp này là '.'

- `os.access('.', os.X_OK)` trả về True vì trong Unix/Linux, thư mục hiện tại (được đại diện bởi dấu chấm) cần quyền thực thi để người dùng có thể thay đổi vào thư mục. Quyền thực thi trên thư mục cho phép bạn truy cập vào các file và thư mục bên trong nó.

        Đối với thư mục, quyền thực thi không giống như quyền thực thi trên file thực thi. Quyền thực thi trên thư mục cho phép người dùng liệt kê các file và truy cập vào chúng

### Get Flag

Bởi vì `/` ở trong blacklist nên ta không thể nào đọc trực tiếp file `/flag.txt` bằng lệnh `cat`. Ta nghĩ đến 1 cách đó là encode thành `base64` rồi dùng lệnh echo để decode.

Payload: {"command":". ;cat `echo 'L2ZsYWcudHh0'|base64 -d`; "}

### Knowledge

- OS command injection

- Python

## Nslookup

### Tổng quan 

Bài này hiển thị ra một trang web yêu cầu nhập domain. Khi nhập ta thấy nó sẽ thực hiện một lệnh `nslookup` tới một trang web. Cụ thể câu lệnh sẽ là `$ nslookup '{input}'`.

### Ý tưởng

Ta nghĩ ngay đến `OS command injection`. Ta chèn `google.com'; echo 'a` để test xem nó có thể thực hiện được nhiều lệch cùng một lúc không. Câu trả lời là có.

Khi này ta thử tiến hành các lệnh như `ls, cat, head` thì thấy nó đã bị filter --> Màn hình hiển thị `Hack detected`

Ta tìm cách bypass bằng những cách sau như:

    l''s
    l``s
    l\s

Các cách web đều không báo thông tin gì cả. Rất có thể hệ thống đã không tồn tại các câu lệnh cơ bản này. Khi này ta bắt đầu tìm cách khác để tìm file trên hệ thống như là `dir, echo *, locate, find`. Rất may là lệnh `dir` đã hoạt động và đưa ra file `index.php`. 

Tuy nhiên lại không dùng được lệnh `cat` (chắc là hệ thống không tồn tại câu lệnh này). Ta phải đi tìm các cách khác nhưng hệ thống đều không có trừ `grep`

Ta đọc file index.php bằng input `google.com'; gr''ep '' index.php; echo 'a`

![index.php](../assets/Cookie%20Arena/img/index.png)

### Get Flag

Dùng lệnh `dir /` để tìm file flag -> Payload để đọc file flag: `google.com'; gr''ep '' /flagXXXX.txt ; echo 'a`

### Knowledge

- OS command injection

# Crypto