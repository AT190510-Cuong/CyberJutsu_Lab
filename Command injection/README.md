# Command injection

mình dùng docker để build những bài lap này

![image](https://hackmd.io/_uploads/S1xQM0eGA.png)

![image](https://hackmd.io/_uploads/HkKVGRlMA.png)

![image](https://hackmd.io/_uploads/HJ-AG0lM0.png)

# chúng ta cần tìm ra flag ở thư mục gốc /

## Phân tích chung

- với mỗi level lại có validate ở username khác nhau

```php

    switch($level){
        default:
        case 1:
            $input = addslashes($input);
            return $input;
        case 2:
            $input = substr($input,0,10);
            $input = addslashes($input);
            return $input;
        case 3:
            // Bad characters, please remove
            $input = preg_replace("/[\x{20}-\x{29}\x{2f}]/","",$input);
            $input = addslashes($input);
            return $input;
        case 4:
            // Bad characters (and more), please remove
            $input = preg_replace("/[\x{20}-\x{29}\x{2f}]/","",$input);
            $input = preg_replace("/[\x{3b}-\x{40}]/","",$input);
            $input = addslashes($input);
            return $input;
    }
}

$level      = $_GET['level'];
$type       = $_GET['type'];
$username   = $_GET['username'];
$length     = strlen($username);
$username   = validate_username($username,$level);
```

- cả 4 level đều dùng hàm addslashes() được sử dụng để thêm dấu gạch chéo ngay trước các ký tự đặc biệt trong một chuỗi, như các ký tự mà có thể gây ra các vấn đề về bảo mật khi được sử dụng trong các câu lệnh SQL. Cụ thể, các ký tự như dấu nháy đơn ('), dấu nháy kép ("), dấu gạch chéo (\), v.v., thường được thêm dấu gạch chéo vào trước đó khi sử dụng trong các truy vấn SQL để đảm bảo tính toàn vẹn của dữ liệu và tránh các cuộc tấn công thông qua SQL injection.

![image](https://hackmd.io/_uploads/SyWvtxWzC.png)

- cuối cùng hàm `passthru()` thực thi câu lệnh cowsay

- Hàm `passthru()` tương tự như hàm exec() ở chỗ nó thực thi một tệp command. Hàm này nên được sử dụng thay cho exec() hoặc system() khi đầu ra từ lệnh Unix là dữ liệu nhị phân cần được truyền trực tiếp trở lại trình duyệt.

## Level 1

### Phân tích

- đoạn code validate:

```php
case 1:
            $input = addslashes($input);
            return $input;
```

- đoạn code chỉ có cơ chế thêm dấu "\" trước các ký tự đặc biệt

- và chúng ta để ý echo có dùng dấu **"** và dấu **'** chúng có 1 số khác biệt
  - khi **'** coi các ký tự trong nó là 1 chuỗi thường
  - còn **"** có thể thực thi code được trong chuỗi này

![image](https://hackmd.io/_uploads/HysfumWGA.png)

- và dấu **\*\* trong các ký tự **"** và **'\*\* lại khác nhau

![image](https://hackmd.io/_uploads/Byw__7-fC.png)

- **"** thì coi **\"** là code để thể hiện chuỗi **"**
- còn **'** thì lại chỉ coi nó là ký tự bình thường trong chuỗi vậy là chúng ta có thể escape hàm **addslashes** với việc tận dụng tính năng này của **'**

### Khai thác

- mình đã thực thi được đoạn code mình muốn

![image](https://hackmd.io/_uploads/BybNYQ-zR.png)

- tiếp theo mình đọc flag ở thư mục /
- mình liệt kê các file và thấy được file secret.txt

![image](https://hackmd.io/_uploads/H1EptmWMC.png)

- và mình đọc được file này

![image](https://hackmd.io/_uploads/HkWW57Zz0.png)

![image](https://hackmd.io/_uploads/rke-3QbMR.png)

## Level 2

### Phân tích

- đoạn code validate:

```php
case 2:
            $input = substr($input,0,10);
            $input = addslashes($input);
            return $input;
```

- đoạn code tương tự level 1 nhưng có thêm ` $input = substr($input,0,10);` substr() để cắt chuỗi $input từ vị trí đầu tiên đến vị trí thứ 10

Kết quả của đoạn mã này là một chuỗi mới chỉ chứa 10 ký tự đầu tiên của chuỗi ban đầu $input.

### Khai thác

- tương tự bài trên nhưng chỉ nhận 10 ký tự đầu

![image](https://hackmd.io/_uploads/Hy3I67WM0.png)

- mình liệt kê các file
  ![image](https://hackmd.io/_uploads/SyD2CQZzA.png)

- Để đọc nội dung của tất cả các file trong thư mục hiện tại cùng một lúc, bạn có thể sử dụng lệnh cat kết hợp với một ký hiệu đại diện cho tất cả các file, chẳng hạn \_. Dưới đây là cách bạn có thể thực hiện điều này: `cat *`

![image](https://hackmd.io/_uploads/ByvslVWGC.png)

## Level 3

### Phân tích

- đoạn code validate:

```php
    case 3:
            // Bad characters, please remove
            $input = preg_replace("/[\x{20}-\x{29}\x{2f}]/","",$input);
            $input = addslashes($input);
            return $input;
```

- Chương trình thêm hàm `preg_replace("/[\x{20}-\x{29}\x{2f}]/","",$input)` loại bỏ các ký tự có mã Unicode từ \x20 đến \x29 và ký tự / từ chuỗi đầu vào

### Khai thác

## Level 4

### Phân tích

- đoạn code validate:

```php
 case 4:
            // Bad characters (and more), please remove
            $input = preg_replace("/[\x{20}-\x{29}\x{2f}]/","",$input);
            $input = preg_replace("/[\x{3b}-\x{40}]/","",$input);
            $input = addslashes($input);
            return $input;
```

### Khai thác
