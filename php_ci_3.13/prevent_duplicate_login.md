# 중복 로그인 방지 로직 PHP(Codeigniter 3.13)
### 1. Check Session Folder && File  ->
```php
if(!is_dir('./application/sessions')){ //check exist of sessions directory
  mkdir('./application/sessions', 0777);  //make a sessions directory  
}
$fp = fopen('./application/sessions/'.$id, 'w'); // 'w' -> ready for making a file 
$content = array(
  'session' => $this->session->session_id,  //session_id 
  'token' => $token    //소셜 로그인일때만 넣어주기
);
fwrite($fp, json_encode($content));  // operate it    
fclose($fp);  //close an open file pointer
```
### 2. Login Check 
```php
function login_check(){
  $result = false;
  if( !empty($_SESSION['m_id']) ){
    $id = $_SESSION['m_id'];
    $fp = fopen('./application/sessions/'.$id, 'r');  // ready for reading session_id and token 
    $content = fread($fp, filesize('./application/sessions/'.$id));  //get info 
    fclose($fp);
    $s_key = json_decode($content, true);  //got a session key 
    if($this->session->session_id != $s_key['session']){   //현재 내 세션아이디와 저장되어있는 세션아이디가 같은지 체크
      $this->session->sess_destroy();
      echo "<script>
              alert('다른 기기에서 로그인되었습니다.');
              location.href = `/main`;
            </script>";
    }else{
      $result = true;
    }
  }
  return $result;
}
```