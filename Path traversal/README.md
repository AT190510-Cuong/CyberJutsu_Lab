# Path traversal

mình dùng docker để build những bài lap này

![image](https://hackmd.io/_uploads/BkhHvKef0.png)

![image](https://hackmd.io/_uploads/H1tDvFefR.png)

# nhiệm vụ của chúng ta qua các bài lab là RCE và đọc flag

## Level 1

![image](https://hackmd.io/_uploads/HkTjOFgGC.png)

### Phân tích

- đọc source code mình được

```php
<?php
$file_name = $_GET['file_name'];
$file_path = '/var/www/html/images/' . $file_name;
var_dump($file_path); //them de debug
if (file_exists($file_path)) {
    header('Content-Type: image/png');
    readfile($file_path);
}
else { // Image file not found
    echo " 404 Not Found";
}
```

Đoạn mã PHP này làm các công việc sau:

1. Lấy giá trị của tham số "file_name" từ URL sử dụng biến siêu toàn cục $\_GET.
2. Xây dựng đường dẫn tới tệp ảnh dựa trên giá trị của tham số "file_name".
3. Kiểm tra xem tệp ảnh có tồn tại tại đường dẫn đã xây dựng hay không bằng cách sử dụng hàm file_exists.
4. Nếu tệp ảnh tồn tại, nó sẽ trả về nội dung của tệp ảnh với kiểu MIME là "image/png" bằng cách sử dụng hàm header để gửi tiêu đề HTTP và hàm readfile để đọc và ghi nội dung của tệp ảnh ra output.
5. Nếu tệp ảnh không tồn tại, nó sẽ trả về mã lỗi "404 Not Found".
6. Thêm câu lệnh `var_dump($file_path);` là để kiểm tra giá trị của biến $file_path để debug khi cần thiết.

- vì chương trình không validate tên file mà ghéo chuỗi thẳng tên file do người dùng nhập vào nên mình có thể thêm các ký tự "../" để di chuyền đến các thư mục khác trong hệ thống và đọc file nhạy cảm

### Khai thác

- mình lên thư mục root và đọc file /etc.passwd trong này có chứa flag

![image](https://hackmd.io/_uploads/Bk_8tFlGA.png)

## Level 2

![image](https://hackmd.io/_uploads/BJjJjYxMA.png)

### Phân tích

- đọc source code mình được

```php
<?php
$file = $_GET['file'];
if (strpos($file, "..") !== false)
    die("Hack detected");
if (file_exists($file)) {
    header('Content-Type: image/png');
    readfile($file);
}
else { // Image file not found
    echo " 404 Not Found";
}?>
```

![image](https://hackmd.io/_uploads/HJSvoteMC.png)

- chức năng của bài tương tự với level 1 nhưng có thêm đoạn code phát hiện phần ".."
- nếu có thì sẽ hiện ra **Hack detected**
- ngược lại chúng ta đọc được file

![image](https://hackmd.io/_uploads/SyPE3FxzA.png)

giống như chúng ta đã phân tích chương trình phát hiện file có ".."

### Khai thác

- mình thử truy cập bằng đường dẫn tuyệt đối và đọc được file /etc/passwd có chứa flag

![image](https://hackmd.io/_uploads/BkcuntgGA.png)

## Level 3

### Phân tích

- đọc source code mình được

```php
<?php

    // Create store place for each user (we place this in /var/www/html/upload for easily handle)
    session_start();
    if (!isset($_SESSION['dir'])) {

        $_SESSION['dir'] = '/var/www/html/upload/' . bin2hex(random_bytes(16));
    }
    $dir = $_SESSION['dir'];

    if ( !file_exists($dir) )
        mkdir($dir);

    if(isset($_FILES["files"]) && $_POST['album'] !="" ) {
        try {

            //Create Album
            $album = $dir . "/" . strtolower($_POST['album']);
            if ( !file_exists($album))
                mkdir($album);

            //Count Files
            $files = $_FILES['files'];
            $count = count($files["name"]);

            // Save files to user's directory
            for ($i = 0; $i < $count; $i++) {

                $newFile = $album . "/" . $files["name"][$i];

                move_uploaded_file($files["tmp_name"][$i], $newFile);
            }

       } catch(Exception $e) {
            $error = $e->getMessage();
         }
    }
?>
```

Đoạn mã PHP này thực hiện các công việc sau:

- Bắt đầu phiên làm việc với session_start(). Nếu không có thư mục lưu trữ nào được tạo cho phiên này ($_SESSION['dir'] không được đặt), một thư mục mới được tạo bằng cách sử dụng bin2hex(random_bytes(16)) để tạo một chuỗi hex ngẫu nhiên. Đường dẫn thư mục được lưu trong ```$\_SESSION['dir']```.
- Kiểm tra xem thư mục được lưu trữ trong `$_SESSION['dir']` có tồn tại không. Nếu không tồn tại, thư mục sẽ được tạo mới bằng cách sử dụng mkdir.
- Kiểm tra xem có dữ liệu tệp được gửi lên không `(isset($_FILES["files"]))` và tên album không rỗng `($_POST['album'] !="")`. Nếu có dữ liệu tệp và tên album không rỗng, quá trình lưu trữ tệp sẽ được thực hiện.
- Thư mục album được tạo bên trong thư mục của người dùng (được lưu trong `$_SESSION['dir'])`. Tên thư mục album được chuyển thành chữ thường (strtolower) và nối với đường dẫn của thư mục người dùng.
- Số lượng tệp được gửi lên được đếm bằng cách sử dụng `count($_FILES['files']["name"])`.
- Mỗi tệp được di chuyển từ vị trí tạm thời (tmp_name) đến thư mục album được tạo. Mỗi tệp được đổi tên thành tên gốc của nó trong máy chủ.
- Bất kỳ lỗi nào xảy ra trong quá trình này sẽ được bắt và ghi lại vào biến $error.
- Đoạn mã này cho phép người dùng tải lên các tệp và lưu chúng vào thư mục của họ trong máy chủ.

bài có vẻ khá giống với lỗ hổng file uplaod của mình tại https://hackmd.io/@monstercuong7/SJWB9t9Z0

## Level 4

![image](https://hackmd.io/_uploads/Hyj8O5lMC.png)

### Phân tích

- đọc source code mình được

```php
<?php
    include './db.php';

    // if is not login
    if (!isset($_SESSION['name'])) {
        header('Location: /register.php');
        die();
    }

    $arr = $db->select_all_users_with_point();
?>
```

- đầu tiên kiểm tra xem người dùng đã đăng nhập hay chưa bằng cách kiểm tra xem biến `$_SESSION['name']` có tồn tại hay không. Nếu không tồn tại, nó chuyển hướng người dùng đến trang đăng ký (register.php) bằng cách sử dụng `header('Location: /register.php')` và kết thúc kịch bản bằng `die()`.
- Sau khi kiểm tra đăng nhập thành công, nó gọi hàm select_all_users_with_point() từ đối tượng $db để lấy danh sách tất cả người dùng và điểm của họ. Kết quả này được lưu trữ trong một mảng $arr.
- trong db.php có các phương thức để thực hiện các thao tác cơ sở dữ liệu như `select_all_users_with_point(), select_user_by_username(), create_user(), và update_point()`.

file register:

```php
<?php
  include './db.php';

  if(isset($_GET["debug"])) die(highlight_file(__FILE__));

  // If is login
  if (isset($_SESSION['name'])) {
    header('Location: /');
    die();
  }
  $error = '';
  if (isset($_POST["name"])) {
    $name = strval($_POST["name"]);
    if (!preg_match('/^[a-z0-9]+$/', $name)) { // -> ../ or / deu khong duoc
      $error = 'Name must be [a-z0-9]+';
    } else {
      $user = $db->select_user_by_username($name);
      if (!!$user) {
        $error = 'Name already exist';
      } else {
        $_SESSION["name"] = $name;
        $db->create_user($name);
        // Create place for upload
        $dir = '/var/www/html/upload/' . $name;
        if ( !file_exists($dir) )
          mkdir($dir);
        die(header('Location: /')); #tao xong file dir thi quay ve trang chu khong thuc thi gi nua
      }
    }
  }
```

có `if(isset($_GET["debug"])) die(highlight_file(__FILE__));` để chúng ta đọc source code

![image](https://hackmd.io/_uploads/Hy3AW5xfR.png)

- đoạn code của db.php dùng preparestatement nên chúng ta không thể sql injection
- tên đăng ký ở đây đã lọc "../"

- khi đăng kí xong mình có dao diện chứa các tragn có chức năng khác nhau

![image](https://hackmd.io/_uploads/ryh17cgzA.png)

![image](https://hackmd.io/_uploads/HyzJ7cgG0.png)

- để ý profile có chức năng upload file

![image](https://hackmd.io/_uploads/ryA77qxMC.png)

```php
<?php
  // error_reporting(0);
  include './db.php';

  // if is not login
  if (!isset($_SESSION['name'])) {
    header('Location: /register.php');
    die();
  }

  $response = "";
  if (isset($_FILES["fileUpload"])) {
    // Always store as avatar.jpg
    //upload code php duoc nhung bi doi ten thanh avarta.jpg
    move_uploaded_file($_FILES["fileUpload"]["tmp_name"], "/var/www/html/upload/" . $_SESSION["name"] . "/avatar.jpg");
    $response = "Success";
  }
?>
```

- đoạn code đã chặn lỗ hổng file upload bằng việc đổi extention nhưng có vẻ ở đây không lọc dấu "../" như chức năng register trước và tên file được nối thẳng vào đường dẫn file mà không có validate

và ở trang index.php sẽ đến đường dẫn file mình vừa upload để đọc file hiển thị ra avatar

![image](https://hackmd.io/_uploads/ryOtHcxfA.png)

nhưng mà các file được tải lên đề được đổi tên thành "avatar.jpg"

![image](https://hackmd.io/_uploads/HJS38cgM0.png)

- nên mình không thể thực hiện file path travesal ở đây

![image](https://hackmd.io/_uploads/H1IQI9gGA.png)

- vậy mình không thể khai thác ở trang profile.php
- mình chuyển sang file game.php

```php
<?php
    include './db.php';


    // if is not login
    if (!isset($_SESSION['name'])) {
        header('Location: /register.php');
        die();
    }
    if (isset($_POST["points"])) { //unstrdata -> 0 co gi
        $points = intval($_POST["points"]);
        $db->update_point($_SESSION['name'], $points);
        header('Content-Type: application/json');
        echo "Successfully update points";
        die();
    }

    if (!isset($_GET['game'])) { //unstrdata
        header('Location: /game.php?game=fatty-bird-1.html');
        die();
    }
    $game = $_GET['game']; //-> game thanh unstrdata
?>
```

![image](https://hackmd.io/_uploads/ByYVPqxMC.png)

- chương trình lấy tên file game trên url và nó sẽ include vào trang game.php và tên game này mình có thể kiểm soát và thay đổi và chương trình không validate tên file game mình nhập vào nên mình có thể thêm các ký tự "../" để di chuyển lên các thư mục

![image](https://hackmd.io/_uploads/HJ4mDqeGC.png)

- khi mình thay đổi tên file thì nó đã hiện ra thông báo lỗi như chúng ta phân tích

![image](https://hackmd.io/_uploads/H1S2vclGA.png)

- vậy bài có thể khai thác thông qua lỗ hổng local file inclusion

### Khai thác

- mình thêm các ký tự "../" và đọc được file /etc/passwd thành công

![image](https://hackmd.io/_uploads/rkxr_qeG0.png)

- tiếp theo mình đọc flag trong file /secret.txt

![image](https://hackmd.io/_uploads/Sk8EtqgfC.png)

- và mình lấy được flag

## Level 5

![image](https://hackmd.io/_uploads/rJOH_2xGC.png)

### Phân tích

- đọc source code mình được

```php
<?php
    // error_reporting(0);
    if (!isset($_GET['game'])) {
        header('Location: /?game=fatty-bird-1.html');
    }
    $game = $_GET['game'];
?>

<!DOCTYPE html>
<html lang="en">
    <head>
        <?php include './views/header.html'; ?>
    </head>

    <body>
        <br><br>
        <p class="display-5 text-center">Goal: RCE me</p>

        <br>
        <div style="background-color: white; padding: 20px;">
            <?php include './views/' . $game; ?>
        </div>

    </body>

    <?php include './views/footer.html' ?>
</html>
```

Đoạn mã trên thực hiện các công việc sau:

- Kiểm tra xem có tham số "game" được truyền trong URL không bằng cách sử dụng `isset($_GET['game'])`. Nếu không có, nó sẽ chuyển hướng người dùng đến trang chính với tham số "game" mặc định là "fatty-bird-1.html" bằng cách sử dụng `header('Location: /?game=fatty-bird-1.html')`.
- Nếu có tham số "game" được truyền, nó sẽ lấy giá trị của tham số "game" từ URL và gán cho biến $game.
- Tiếp theo, nó bắt đầu một tài liệu HTML.
- Trong phần `<head>`, nó bao gồm nội dung của tệp header.html từ thư mục views.
- Trong phần `<body>`, nó hiển thị một thông điệp "Goal: RCE me" và một div có nền màu trắng và padding. Trong div này, nó bao gồm nội dung của tệp có tên được chỉ định bởi biến $game.
- Cuối cùng, trong phần cuối cùng của tài liệu HTML, nó bao gồm nội dung của tệp footer.html từ thư mục views.

### Khai thác

tương tự bài trước mình có thể thực hiện khai thác lỗ hổng qua local file inclusion

- mình thêm các ký tự "../" và đọc được file /etc/passwd thành công

![image](https://hackmd.io/_uploads/BkP5u3lGA.png)

- tiếp theo mình đọc flag trong file /secret.txt

![image](https://hackmd.io/_uploads/rk3h_nefR.png)

- và mình lấy được flag

- nhưng mục đích của chúng ta là RCE
- bài không có chức năng upload để mình có thể upload webshell để RCE

- mình thấy có 2 file `/var/log/apache2/access.log /var/log/apache2/error.log` mình có toàn quyền đọc, ghi và thực thi

![image](https://hackmd.io/_uploads/BksP86xMA.png)

![image](https://hackmd.io/_uploads/ByTQSTlfC.png)

![image](https://hackmd.io/_uploads/H1QZVeWMR.png)

- 2 file `/var/log/apache2/access.log /var/log/apache2/error.log` mình có quyền ghi vào và nó hiện log của các request của user gửi đến gồm có thông tin của trường **User-Agent** và trường này mình có thể thay đổi được
- mình đã chèn vào trường này đoạn script php và nó sẽ được include vào trang php chính và chạy đoạn code php này

![image](https://hackmd.io/_uploads/rkasVeZf0.png)

- và đoạn code php này đã được thực hiện vậy mình có thể RCE đến hệ thống website như <a href="https://hackmd.io/@monstercuong7/B1WSanCp6">bài này</a> mình đã làm

## Level 6

![image](https://hackmd.io/_uploads/H1C9wpefA.png)

### Phân tích

- đọc source code mình được

```php
<?php

    // Create store place for each user (we place this in /var/www/html/upload for easily handle)
    session_start();
    if (!isset($_SESSION['dir'])) {

        $_SESSION['dir'] = '/var/www/html/upload/' . bin2hex(random_bytes(16));
    }
    $dir = $_SESSION['dir'];

    if ( !file_exists($dir) )
        mkdir($dir);

    if(isset($_FILES["file"]) ) {
        try {

          $file_name = $_FILES["file"]["name"];
          if(substr($file_name,-4,4) == ".zip")
          {
            $result = _unzip_file_ziparchive($_FILES["file"]["tmp_name"],$dir);
          }
          else
          {
            $newFile = $dir . "/" . $file_name;
            move_uploaded_file($_FILES["file"]["tmp_name"], $newFile);
          }

       } catch(Exception $e) {
            $error = $e->getMessage();
         }
    }

    function _unzip_file_ziparchive($file, $to)
    {
        $z = new ZipArchive();
        $zopen = $z->open( $file, ZipArchive::CHECKCONS);
        if ( true !== $zopen )
          return false;
        for ( $i = 0; $i < $z->numFiles; $i++ ) {

            if ( ! $info = $z->statIndex($i) )
              return false; //Could not retrieve file from archive.

            if ( '/' == substr($info['name'], -1) ) // directory
              continue;

            $contents = $z->getFromIndex($i);
            if ( false === $contents )
              return false; //Could not extract file from archive.

            if(file_exists(dirname($to . "/" . $info['name']))){ // directory exists
                file_put_contents($to . "/" . $info['name'], $contents);
            }
          }

        $z->close();
        return true;
    }
?>
```

Đoạn mã trên thực hiện các công việc sau:

- Bắt đầu một phiên làm việc với `session_start()`.
- Nếu không có thư mục được lưu trữ trong `$_SESSION['dir']`, một thư mục mới sẽ được tạo bằng cách sử dụng bin2hex(random_bytes(16)) để tạo một chuỗi hex ngẫu nhiên. Đường dẫn thư mục được lưu trong `$_SESSION['dir']`.
- Kiểm tra xem thư mục đã được tạo trong `$_SESSION['dir']` có tồn tại không. Nếu không tồn tại, thư mục sẽ được tạo mới bằng cách sử dụng mkdir.
- Nếu có dữ liệu tệp được gửi lên `(isset($_FILES["file"]))`, quá trình lưu trữ tệp sẽ được thực hiện. Đoạn mã kiểm tra loại tệp được gửi lên và xử lý tương ứng. Nếu tệp là một file zip (được nhận diện bằng phần mở rộng ".zip"), nó sẽ sử dụng hàm `_unzip_file_ziparchive()` để giải nén tệp zip và lưu nội dung vào thư mục người dùng. Nếu không, nó sẽ di chuyển tệp đến thư mục người dùng.
- Nếu có lỗi xảy ra trong quá trình này, nó sẽ được gán vào biến $error.
  - Hàm `_unzip_file_ziparchive()` được định nghĩa để giải nén tệp zip bằng cách sử dụng lớp ZipArchive. Hàm này sẽ duyệt qua các tệp trong tệp zip, giải nén và lưu chúng vào thư mục chỉ định.

<img  src="https://3198551054-files.gitbook.io/~/files/v0/b/gitbook-x-prod.appspot.com/o/spaces%2FVvHHLY2mrxd5y4e2vVYL%2Fuploads%2FF8DJirSFlv1Un7WBmtvu%2Fcomplete.gif?alt=media&token=045fd197-4004-49f4-a8ed-ee28e197008f">
