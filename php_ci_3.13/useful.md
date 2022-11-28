# 자주 사용하는 코드 뭉치 PHP(Codeigniter 3.13)

### 캐쉬정리 
```php
function clear_cache(){ 
  $this->output->set_header("Cache-Control: no cache");
}
```
### 에러 redirect 처리 
```php
function error_redirect($error_msg, $go_url) {
  $data = [];
  $data['error_msg'] = $error_msg;
  $data['go_url'] = $go_url;
  

  $html = $this->load->view('errors/error_alert', $data, TRUE);
  echo $html;
  exit;
}
// view 
<?php
defined('BASEPATH') OR exit('No direct script access allowed');
//$error_msg : 뿌려줄 에러 메시지.
//줄바꿈 제거후 다시 script 에서 인식하는 \n 으로 변경
//preg_replace('/\r\n|\r|\n/','\n',$error_msg);

?>
<script type="text/javascript">
  <?php if(!empty($error_msg)): ?>
  alert("<?= preg_replace('/\r\n|\r|\n/','\n',$error_msg); ?>");
  <?php endif; ?>
  <?php if(isset($go_url)&& $go_url !== 'back'): ?>
  document.location.href="<?= $go_url ?>";
  <?php elseif ($go_url === 'back'): ?>
  history.back();
  <?php endif; ?>
</script>
</body>
</html>
```
### 페이지네이션 Pagination 
```php
function common_paging(&$data, $total_page){  
  if( empty($data['bottom_limit']) ){  //하단에 보여줄 페이징 갯수 
    $data['bottom_limit'] = 10;
  }
  if( (int)$data['nowPage'] > $total_page ){
    $data['nowPage'] = $total_page;
  }
  if( empty($data['nowPage']) ){ 
    $data['nowPage'] = 1;
  }
  $data['on_page'] = $data['nowPage'] % $data['bottom_limit']; //현재페이지 나누기 10의 나머지 
  if( $data['on_page'] == 0 ) {
    $data['on_page'] = 1;
  }; 
  $data['start'] = ((int)$data["nowPage"]-1) * (int)$data["perPage"];  //Limit 시작번호 
  $data['first_bottom_num'] = $data['nowPage'] - $data['on_page'] + 1; //하단부 첫번째 번호
  $data['last_bottom_num'] = $data['nowPage'] - $data['on_page'] + $data['bottom_limit']; //하단부 마지막 번호 
  if( $data['last_bottom_num'] > $total_page  ){  //마지막번호가 최대 페이지 넘기는것을 방지 
    $data['last_bottom_num'] = $total_page;
  }
}
```
### 아이피 주소 얻기 get ip address
```php
public function get_client_ip() {   
  $ipaddress = '';

  if (!empty($_SERVER['HTTP_CLIENT_IP'])) {
    $ipaddress = $_SERVER['HTTP_CLIENT_IP'];
  } else if(!empty($_SERVER['HTTP_X_FORWARDED_FOR'])) {
    $ipaddress = $_SERVER['HTTP_X_FORWARDED_FOR'];
  } else if(!empty($_SERVER['HTTP_X_FORWARDED'])) {
    $ipaddress = $_SERVER['HTTP_X_FORWARDED'];
  } else if(!empty($_SERVER['HTTP_FORWARDED_FOR'])) {
    $ipaddress = $_SERVER['HTTP_FORWARDED_FOR'];
  } else if(!empty($_SERVER['HTTP_FORWARDED'])) {
    $ipaddress = $_SERVER['HTTP_FORWARDED'];
  } else if(!empty($_SERVER['REMOTE_ADDR'])) {
    $ipaddress = $_SERVER['REMOTE_ADDR'];
  } else {
    $ipaddress = 'UNKNOWN';
  }

  return $ipaddress;
}
```
### PHP Mailer 설정 
```php
function mailer($fname, $fmail, $to, $subject, $content, $type=0, $file="", $cc="", $bcc=""){
  if ($type != 1) $content = nl2br($content); // type : text=0, html=1, text+html=2
  $mail = new PHPMailer();  //defaults to using php "mail()"
  try{
    // $mail->SMTPDebug = SMTP::DEBUG_SERVER;  디버깅 
    $mail->IsSMTP();
    $mail->Host = "smtp.gmail.com";
    $mail->SMTPAuth = true;
    $mail->Username = " ";
    $mail->Password = " ";
    $mail->SMTPSecure = "ssl";
    $mail->Port = 465;
    $mail->CharSet = 'UTF-8';
    $mail->setFrom($fmail, $fname);
    $mail->addAddress($to);  // 받는 이메일 
    if ($cc){
      $mail->addCC($cc);
    }
    if ($bcc){
      $mail->addBCC($bcc);
    }
    // $mail->AltBody = ""; // optional, comment out and test  
    if ( is_array($file) ) {
      foreach ($file as $f) {
        $mail->addAttachment($f['path'], $f['name']);
      }
    }
    $mail->isHTML(true);
    $mail->Subject = $subject;
    $mail->Body = $content;
    $mail->send();
    return true;
  } catch (Exception $e){
    log_message('error', $e);
    return false;
  }
}

#php 파일 맨 상단에 작성
use PHPMailer\PHPMailer\PHPMailer;
use PHPMailer\PHPMailer\SMTP;
// use PHPMailer\PHPMailer\Exception;
```
### 파일 넣기 (Controller) 다중파일 로직 
```php
if( !empty($_FILES['files']['name'] AND $_FILES['files']['name'] !== '') ){
  $files_name = $_FILES['files']['name'];  //input type file 
  $files = $_FILES['files'];
    if( count($files_name) > 0 ){
      foreach( $files_name as $key => $val ){
        $_FILES['files'] = array(
          'name'		=> $files['name'][$key],
          'type'		=> $files['type'][$key],
          'tmp_name'	=> $files['tmp_name'][$key],
          'error'		=> $files['error'][$key],
          'size'		=> $files['size'][$key]
        );
        #업로드 설정 초기화
        $this->upload_file_init('../test_uploads/', '*', $board_seq.'_'.$_FILES['files']['name']); #upload path, fileType, re_fileName
        #업로드 실행 후 결과 리턴
        $upload_result = $this->upload_file_save('files');  //input name 
        if( $upload_result["result"] == "Y" ){
          #업로드 파일정보 DB에 저장
          $this->Board_model->board_file_info_update(
            'insert',
            array(
              "target_seq"	 => $board_seq,
              "real_file_name" => $upload_result["full_path"],  // should be fullpath
              "save_file_name" => $_FILES['files']['name']
            )
          );
        }
      }
  }
}
```


### 파일업로드 설정
```php
/**
 * @author Kim Min Woo <alsdl6106@dherald.com>
 * @param (string) $save_path --> 저장된 파일경로 
 * @param (string) $allowed_type --> 허용 파일타입
 * @param (string) $re_filename --> 덮어씌울 파일 이름 
 */
function upload_file_init($save_path, $allowed_type, $re_filename=''){
  if(!is_dir( $save_path )){  //경로가 없으면 생성
    mkdir( $save_path , 0777, TRUE);
  }
  $config['upload_path'] = $save_path;
  $config['allowed_types'] = $allowed_type;//'gif|jpg|png|bmp';
  $config['max_size'] = '10240';//파일 사이즈.0이면 크기제한 없슴. php.ini 설정도 확인해봐야함.  10240 == 10m 메가바이트 
  $config['file_ext_tolower'] = TRUE;//확장자 무조건 소문자로 저장.
  $config['overwrite'] = FALSE;//파일 덮어씀.false 로 설정되면, 파일명에 숫자가 추가.
  if(!empty($re_filename)){
    $config['file_name'] = $re_filename;
  }
  $this->load->library('upload', $config);
  $this->upload->initialize($config);
}
```
### 파일업로드 저장 
```php
function upload_file_save($file_input_name){
  if(!$this->upload->do_upload($file_input_name)){
    //저장실패
    $error = array('error' => $this->upload->display_errors());
    $result["error_msg"] = $error;
    $result["result"] = 'N';
  }else{
    $result["result"] = "Y"; //저장 성공
    $result["file_name"] = $this->upload->data("file_name"); // mypic.jpg
    $result["file_type"] = $this->upload->data("file_type"); //image/jpeg
    $result["file_path"] = $this->upload->data("file_path"); ///path/to/your/upload/
    $result["full_path"] = $this->upload->data("full_path"); ///path/to/your/upload/jpg.jpg
    $result["raw_name"] = $this->upload->data("raw_name"); //mypic
    $result["orig_name"] = $this->upload->data("orig_name"); //mypic.jpg
    $result["client_name"] = $this->upload->data("client_name"); //mypic.jpg  
    $result["file_ext"] = $this->upload->data("file_ext"); //.jpg
    $result["file_size"] = $this->upload->data("file_size"); //22.2
    $result["is_image"] = $this->upload->data("is_image"); //1
    $result["image_width"] = $this->upload->data("image_width"); //800
    $result["image_height"] = $this->upload->data("image_height"); //600
    $result["image_type"] = $this->upload->data("image_type"); //jpeg
    $result["image_size_str"] = $this->upload->data("image_size_str"); //width="800" height="200"
  }
  return $result;
}
```
### 파일 다운로드 
```php
public function downloadIE(){
  $this->load->helper('download');
  $file_path = $this->input->get('file_path');
  $file_path = str_replace(' ','_',$file_path);
  $file_name = $this->input->get('file_name');
  
  if(empty($file_name)){//저장할 파일명 파라메터가 없다면 경로에서 파일명을 생성.
    $file_name = preg_replace('/^.*\//','',$file_path);
  }
  $file_data = file_get_contents($file_path);  //파일 fullpath 
  force_download(rawurlencode($file_name), $file_data);
}
```


