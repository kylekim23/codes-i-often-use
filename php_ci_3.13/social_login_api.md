# 소셜 로그인 REST API  PHP(Codeigniter 3.13) CURL 방식 
### 어떤 소셜인지 체크
```php
/**
 * @param (String) @type --> 'kakao' or 'naver' or 'google' or ....  
 */
public function social_login_check($type){   
  if( empty($_SESSION['m_seq']) ){
    switch( $type ){
      case 'kakao' : 
        redirect('https://kauth.kakao.com/oauth/authorize?response_type=code&client_id=[client_id]&redirect_uri=http://120.142.20.164:8084/login/rest_kakao_login_api', 'refresh');
        break;
      case 'naver' : 
        redirect('https://nid.naver.com/oauth2.0/authorize?response_type=code&client_id=[client_id]&redirect_uri=http://120.142.20.164:8084/login/rest_naver_login_api', 'refresh');         
        break; 
      case 'google' : 
        redirect('https://accounts.google.com/o/oauth2/v2/auth?response_type=code&client_id=[client_id].apps.googleusercontent.com&redirect_uri=http://localhost:8084/login/rest_google_login_api&scope=https://www.googleapis.com/auth/userinfo.email', 'refresh');  
    }
  } else {
    redirect($this->agent->referrer(), 'refresh');
  }
}
```
### 카카오 Kakao
```php
/**
 * Login 
 */
function rest_kakao_login_api(){     
  $access_token = $this->input->get('code');  //? token 

  $ch = curl_init('https://kauth.kakao.com/oauth/token');  //? token값이 맞는지 요처ㅇ 
  curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
  curl_setopt($ch,CURLOPT_HTTPHEADER, array(
    "Content-type: application/x-www-form-urlencoded;charset=utf-8",
    "Referer: http://120.142.20.164:8084"
  ));
  curl_setopt($ch, CURLOPT_POST, 1);
  curl_setopt($ch, CURLOPT_POSTFIELDS, http_build_query(array(
    'grant_type' => 'authorization_code',
    'client_id' => '[client_id]',
    'redirect_uri' => 'http://120.142.20.164:8084/login/rest_kakao_login_api',  //요청한곳이 동일한곳인 체크 
    'code' => $access_token,
    'client_secret' => '[client_secret]'
  )));

  $value = curl_exec($ch);
  curl_close($ch);
  $result = json_decode($value, true);

  $ch = curl_init('https://kapi.kakao.com/v2/user/me');   //토큰 매칭 완료되면 데이터 get 
  curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
  curl_setopt($ch,CURLOPT_HTTPHEADER, array(
    "Content-type: application/x-www-form-urlencoded;charset=utf-8",
    "Authorization: Bearer {$result['access_token']}"  
  ));

  $value = curl_exec($ch);
  curl_close($ch);
  $final_result = json_decode($value, true);  
  if( !empty($final_result['id']) ){
    $this->social_login(array('token' => $result['access_token'], 'id'=> $final_result['id'],
                                         'email' => $final_result['kakao_account']['email'], 'social_type' => 'K'));
  } else {
    echo '<script>alert("some error occurs \n 관리자에게 문의주세요.(orizl.kr@gmail.com)")</>';
    redirect('/login', 'refresh');
  }
}
/**
 * ! 탈퇴 처리 withdraw
 */
function unlink_kakao(){   
  $token = $_SESSION['m_token'];
  $ch_2 = curl_init('https://kapi.kakao.com/v1/user/unlink');
  curl_setopt($ch_2, CURLOPT_RETURNTRANSFER, true);
  curl_setopt($ch_2,CURLOPT_HTTPHEADER, array(
    "Authorization: Bearer {$token}"
  ));
  curl_exec($ch_2);
  curl_close($ch_2);
}
```
### 네이버 Naver
```php
/**
 * ! Login
 */
function rest_naver_login_api(){  //! Naver login   
  $access_token = $this->input->get('code');  //? token 
  $ch = curl_init('https://nid.naver.com/oauth2.0/token');  
  curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
  curl_setopt($ch,CURLOPT_HTTPHEADER, array(
    "Content-type: application/x-www-form-urlencoded;charset=utf-8",
    "Referer: http://120.142.20.164:8084"
  ));
  curl_setopt($ch, CURLOPT_POST, 1);
  curl_setopt($ch, CURLOPT_POSTFIELDS, http_build_query(array(
    'grant_type' => 'authorization_code',
    'client_id' => '[client_id]',
    'redirect_uri' => 'http://120.142.20.164:8084/login/rest_naver_login_api',  
    'code' => $access_token,
    'client_secret' => '[client_secret]'
  )));

  $value = curl_exec($ch);
  curl_close($ch);
  $result = json_decode($value, true);

  $ch = curl_init('https://openapi.naver.com/v1/nid/me');   
  curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
  curl_setopt($ch,CURLOPT_HTTPHEADER, array(
    "Content-type: application/x-www-form-urlencoded;charset=utf-8",
    "Authorization: Bearer {$result['access_token']}"  
  ));

  $value = curl_exec($ch);
  curl_close($ch);
  $final_result = json_decode($value, true);
  if( !empty($final_result['response']['id']) ){
    $this->social_login(array('token' => $result['access_token'], 'id'=> $final_result['response']['id'],
                                        'email' => $final_result['response']['email'] , 'social_type' => 'N'));
  } else {
    echo '<script>alert("some error occurs \n 관리자에게 문의주세요.(orizl.kr@gmail.com)")</>';
    redirect('/login', 'refresh');
  }
}
/**
 * ! 탈퇴처리 withdraw 
 */
function unlink_naver(){   
  $access_token = $_SESSION['m_token']; 
  log_message('error',$access_token);
  $ch = curl_init('https://nid.naver.com/oauth2.0/token');  
  curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
  curl_setopt($ch,CURLOPT_HTTPHEADER, array(
    "Content-type: application/x-www-form-urlencoded;charset=utf-8",
    "Referer: http://120.142.20.164:8084"
  ));
  curl_setopt($ch, CURLOPT_POST, 1);
  curl_setopt($ch, CURLOPT_POSTFIELDS, http_build_query(array(
    'grant_type' => 'delete',
    'client_id' => '[client_id]',
    'access_token' => $access_token,
    'client_secret' => '[client_secret]',
    'service_provider' => 'NAVER'
  )));
  curl_exec($ch);
  curl_close($ch);
}
```
### 구글 Google
```php
/**
 * ! Login
 */
function rest_google_login_api(){    
  $access_token = $this->input->get('code');  //? token 
  $ch = curl_init('https://oauth2.googleapis.com/token');
  curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
  curl_setopt($ch,CURLOPT_HTTPHEADER, array(
    "Content-type: application/x-www-form-urlencoded;charset=utf-8",
    "Referer: http://localhost:8084"
  ));
  curl_setopt($ch, CURLOPT_POST, 1);
  curl_setopt($ch, CURLOPT_POSTFIELDS, http_build_query(array(
    'grant_type' => 'authorization_code',
    'client_id' => '[client_id]',
    'redirect_uri' => 'http://localhost:8084/login/rest_google_login_api',   
    'code' => $access_token,
    'client_secret' => '[client_secret]'
  )));

  $value = curl_exec($ch);
  curl_close($ch);
  $result = json_decode($value, true);

  $ch = curl_init('https://www.googleapis.com/userinfo/v2/me');   
  curl_setopt($ch, CURLOPT_RETURNTRANSFER, true);
  curl_setopt($ch,CURLOPT_HTTPHEADER, array(
    "Content-type: application/json",
    "Authorization: Bearer {$result['access_token']}"  
  ));

  $value = curl_exec($ch);
  curl_close($ch);
  $final_result = json_decode($value, true);
  if( !empty($final_result['id']) ){
    $this->social_login(array('token' => $result['access_token'], 'id'=> $final_result['id'],
                                        'email' => $final_result['email'], 'social_type' => 'G'));
  } else {
    echo '<script>alert("some error occurs \n 관리자에게 문의주세요.(orizl.kr@gmail.com)")</>';
    redirect('/login', 'refresh');
  }
}
```

