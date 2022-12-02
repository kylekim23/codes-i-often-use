# JAVASCRIPT 자주 사용하는 함수모음 
### Empty Check 값이 비어있는지 체크 
```javascript
/**
 * 값이 비어있는지 확인한다.
 *
 * @example
 * empty();
 * empty(undefined);
 * empty(null);
 * empty(0);
 * empty('');
 * empty([]);
 * empty({});
 * @param  {*=} arg 값
 */
const empty = (arg) => {
    let result = [undefined, null, 0, ''].includes(arg);

    if(!result){
        if(Array.isArray(arg)){
            result = (arg.length == 0);
        }else if(typeof arg == 'object'){
            result = (Object.keys(arg).length == 0 && Object.keys(Object.getPrototypeOf(arg)).length == 0);
        }
    }
    return result;
};
```
### Number Check 정수 체크 
```javascript
/**
 * 값이 숫자인지 확인한다.
 *
 * @example
 * is_number(1);
 * is_number('1');
 * is_number('test');
 * is_number('1', true);
 * @param {*=} arg 확인할 값
 * @param {boolean} [strict=false] true일 경우 arg의 type도 확인
 */
const is_number = (arg, strict = false) => {
    let result = (!Number.isNaN(Number(arg)) && ['number', 'string'].includes(typeof arg));
    if(result && strict){
        result = (typeof arg == 'number');
    }

    return result;
};
```
### Object Check 객체인지 체크 
```javascript
/**
 * 해당 값이 객체인지 확인
 *
 * @example
 * is_object({});
 * is_object(new Date());
 * 
 * is_object();
 * is_object(undefined);
 * is_object(null);
 * is_object(0);
 * is_object('');
 * is_object([]);
 * @param  {*=} arg 값
 */
const is_object = (arg) => {
    return (typeof arg == 'object' && !Array.isArray(arg));
};
```
### Set Cookie 쿠키세팅 
```javascript
/**
 * @example 
 * set_cookie('test', 'test_cookie', 1);
 * 
 * @param {string} name
 * @param {any} value
 * @param {number} exp  1 = 1일, 2 = 2일
 * @param {string} path  [path='/']
 * @param {string} domain [domain=location.hostname]  
 */
function set_cookie(name, value, exp, path='/', domain = location.hostname ) {
  let date = new Date()
  date.setTime(date.getTime() + exp * 24 * 60 * 60 * 1000) //1일보관
  document.cookie = name + '=' + value + ';expires=' + date.toUTCString() + `;path=${path}` + `;domain=${domain}`;
}
```
### Get Cookie 쿠키 얻기 
```javascript
/**
 * @example 
 * getCookie('test');
 * 
 * @param {string} name
 * @return cookie_value 쿠키값 리턴 
 */
function getCookie(name) {
  var value = document.cookie.split('; ').find(row => row?.split('=')[0] === name)
  return value != undefined ? value.split('=')[1] : null
}
``` 
### Delete Cookie 쿠키 만료 
```javascript
/**
 * @exmplain
 * 쿠키는 삭제기능이 없기때문에 
 * 시간 세팅을 과거로 세팅해 만료시킨다.
 * 
 * @example 
 * deleteCookie('test');
 * 
 * @param {string} name
 */
function deleteCookie(name) {
  document.cookie = name + '=; expires=Thu, 01 Jan 1970 00:00:01 GMT;path=/'
}
```
### When use plural cookies 복수개 쿠키 사용법
```javascript
/**
 * @param {HTMLInputElement} node 
 * @expain : cookie value must be String type which means if you want to put 
 * something more than one, you should append  
 */ 
function visit_trace(node){   // visit site 
  let visit_cookie = getCookie('visit_trace');
  if( visit_cookie != null ){
    let visit_arr = visit_cookie.split(',');
    if( visit_arr.length != 0 ){
      visit_arr = visit_arr.filter((val, i) => { return val !== node.dataset.postid; });  //prevent duplicated
      if( visit_arr.length != 0){
        set_cookie('visit_trace', visit_arr.toString()+','+node.dataset.postid, 0.8);
      } else {
        set_cookie('visit_trace', node.dataset.postid, 0.8);
      }
    } 
  } else {
    set_cookie('visit_trace', node.dataset.postid, 0.8);
  }
  node.style.color = '#999999';
  set_cookie('backforward', node.dataset.postid, 0.8);
}
```  
### Check if the date is valid 유효한 날짜인지 확인
```javascript
/**
 * @example
 * checkdate(2022, 10, 28);
 * 
 * @param  {number} year 년
 * @param  {number} month 월
 * @param  {number} day 일
 * @return boolean
 */
const checkdate = (year, month, day) => {
    const date = new Date(year, (month-1), day);

    return (date.getFullYear() == year && (date.getMonth()+1) == month && date.getDate() == day);
};
```
### Check if the date matches  같은 날짜인지 비교
```javascript
/**
 * @example
 * const date_1 = new Date();
 * const date_2 = new Date();
 * 
 * equaldate(date_1);
 * equaldate(date_1, date_2);
 * date_1.setDate(date_1.getDate() + 1);
 * date_2.setDate(date_2.getDate() + 2);
 * equaldate(date_1);
 * equaldate(date_1, date_2);
 * 
 * @param  {Date} date_1 기준 날짜
 * @param  {Date} [date_2=new Date()] 비교할 날짜
 * @return boolean
 */
const equaldate = (date_1, date_2 = new Date()) => {
    return (strftime(date_1, '%Y-%m-%d') == strftime(date_2, '%Y-%m-%d'));
};
```
### Return Day 요일 반환 
```javascript
/**
 * Date객체에서 해당 하는 요일을 반환한다.
 * 
 * @example
 * const date = new Date(2022, 9, 27);
 * 
 * getWeek(date);
 * 
 * getWeek(date, false); 
 * @param {Date} date Date 객체
 * @param {boolean} [flag=true] 해당 요일의 약어반환 대한 구분 값 false일 경우 약어 반환 (기본값: true)
 * @return string 
 */
const getWeek = (date, flag = true) => {
    const week = ['일요일', '월요일', '화요일', '수요일', '목요일', '금요일', '토요일'],
    result = week[date.getDay()];

    return (flag)?result:result.replace(/^([ㄱ-ㅎㅏ-ㅣ가-힣]{1})[ㄱ-ㅎㅏ-ㅣ가-힣]+$/, '$1');
};
```
### Select all checkboxes  모든 체크박스 선택 
```javascript
/**
 * @example
 * <input type="checkbox" onclick="check_all(this, 'check_box')">
 * <input type="checkbox" data-name="check_box">
 * <input type="checkbox" data-name="check_box">
 * @param  {HTMLInputElement} node click 이벤트가 발생한 Node Object
 * @param  {string} name check 상태를 바꿀 체크박스들의 data-name의 값
 */
const check_all = (node, name) => {
    const el_list = document.querySelectorAll(`[data-name='${name}']`);

    for(const el of el_list){
        if(node.checked){
            el.checked = true;
        } else {
            el.checked = false;
        }
    }
};
```
### Remember ID 아이디 기억하기 
```javascript
/**
 * ! Onchange event 저장하기 체크 된 상태인지 확인 
 * @param {HTMLInputElement} node Onchage 이벤트가 발생한 Node Object 
 */
function set_remember(node) {
  const id = document.querySelector('input[name="id"]');
  if (node.checked) {
    set_cookie('remember', id.value, 7);  // 1주일 쿠키 세팅 
  } else {
    deleteCookie('remember');  // 쿠키 만료 
  }
}

/**
 * ! 이메일이 입력되어있는 상태일때 체크를 했을시 
 * @param {HTMLInputElement} node
 */
function set_remember2(node) {
  const id = document.querySelector('input[name="id"]');
  if( id.trim().value.length > 0  ){  // 공백이 아닐때 세팅 
    const rememberme = document.querySelector('input[name="rememberme"]');
    if (rememberme.checked) {
      set_cookie('remember', node.value, 7);
    }
  }
}

/**
 * ! 페이지 로드후 처리 
 */
window.onload = () =>{
  let remember = getCookie('remember');
  let id = document.querySelector('input[name="id"]');
  let rememberme = document.querySelector('input[name="rememberme"]');
  id.value = remember;

  if (id.value.trim() != '') {
    rememberme.checked = true // 로딩시 입력 칸에 저장된 email표시된 상태라면 email저장
  }
}
```
### Timer 타이머
```javascript
/**
 * ! timer
 */
hashcode : '',
timestart : '',
timer: function () {
  let time = 300; //기준시간 5분
  let min = ""; //분
  let sec = ""; //초
  this.time_start = setInterval(() => {
    min = parseInt(time / 60); //몫 계산
    sec = time % 60; //나머지 계산
    document.querySelector("#timer").innerText = min + "분" + sec + "초";
    time--;
    document.querySelector("#timer").style.color = "#1966c9";
    if (time < 0) {
      clearInterval(this.time_start); //setinterval() 실행 종료
      document.querySelector("#timer").innerText = "시간초과";
      document.querySelector("#timer").style.color = "#ff0000";
      node.dataset.check = "x";  //해당 element data-check 속성 x 초기화 
      this.hashcode = "";  // 암호코드 초기화
    }
  }, 1000);
}
```
###  Increase input efficiency 잦은 비동기 입력 효율 높이기
> 특정 Input 태그에 입력할 시 비동기 호출이 이뤄져야한다면 
타자를 치고있는 동안에는 호출을 보내지않고 타자를 멈췄을때 
호출을 보내는게 효율적이다 그것을 onkeydown과 onkeyup Event를 이용해 
작성해보자. 
```javascript
const test = {
  flag : true,  //test 지역변수 
  /**
   * ! key down
   * @param {HTMLInputElement} node 
   */
  on_key_down : (node) => {
    this.flag = false;
  },
  /**
   * ! key up
   * @param {HTMLInputElement} node 
   */
  on_key_up : async (node) => {
    this.flag = true;
    await setTimeout( () => {
      if( this.flag ) {
        const regExp = new RegExp(/^[0-9a-zA-Z]([-_.]?[0-9a-zA-Z])*@[0-9a-zA-Z]([-_.]?[0-9a-zA-Z])*.[a-zA-Z]{2,3}$/i);
        if( regExp.test(node.value.trim()) ){
          const data = new FormData();
          data.append('email', node.value);
          fetch("/auth/check_email", {
            method: "POST",
            mode: "cors",
            body: data,
            headers: {
              Accept: "application/json",
            },
          })
          .then( (res) => {
            return res.json();
          })
          .then( (json) => {
            console.log(json);
          })
          .catch( (error) => {
            console.log(error);
          })
        }  
        this.flag = false;
      }
    }, 1000);
  },
}
```  
### 최신 API INTL (Internationalization API) 국가별 세팅 
```javascript
/**
 * @explain : transformat the number by each country method    
 */
const number = 123456789;
const options = {
    notation : "compact"
}
console.log(new Intl.NumberFormat('ko-KR', options).format(number));
console.log(new Intl.NumberFormat('en-US', options).format(number));
//result1 : 1.2억 
//result2 : 123M

/**
 * @explain : appears each country timezone with their language   
 */
const options = {
    dateStyle : "medium",
    timeStyle : "full",
}
console.log(new Intl.DateTimeFormat('ko-KR', options).format(new Date));
// result : 2022. 12. 2. 오전 9시 32분 16초 한국 표준시
```
### check more than one inputs duplicated   
```javascript
/**
 * @param {String} type -> email or name
 * @explain is email or name duplicated?  
 */
async function duple_check(type){
  let selector = document.querySelector(`input[name="${type}"]`); 
  let flag = false;
  let value = selector.value.trim();
  let regExp = '';
  switch( type ){
    case 'email':
      regExp = /^[0-9a-zA-Z]([-_.]?[0-9a-zA-Z])*@[0-9a-zA-Z]([-_.]?[0-9a-zA-Z])*.[a-zA-Z]{2,3}$/i;
      flag = regExp.test(selector.value.trim()) ? true : false;
      break;
    case 'name':
      regExp = /^.{2,8}$/;
      flag = regExp.test(selector.value.trim()) ? true : false;
      break;
  }
  if( flag ){
    const data = new FormData();
    data.append('type', type);
    data.append('value', value);
    await fetch('url', {
      method: "POST",
      mode: "cors",
      body: data,
      headers: {
        Accept: "application/json",
      },
    })
    .then((res) => {
      return res.json();
    })
    .then((json) => {
      switch(json.flag){
        case 'o':
          if( json.result !== 'duplicated' ){
            selector.dataset.duple_check = 'o';
            alert('사용 가능합니다.');
          } else {
            selector.dataset.duple_check = 'x';
            alert('이미 사용중입니다.');
          }
          break;
        case 'no_permission':
          selector.dataset.duple_check = 'x';
          alert('권한이 없습니다.');
          break;
      }
    })
    .catch((e) => {
      alert('개발자에게 문의주세요. \n [abcd@gmail.com]');
      console.log(e);
    });
  } else {
    selector.dataset.duple_check = 'x';
    type = type === 'email' ? '이메일' : '이름';
    alert(`${type} 형식에 맞게 작성해주세요.`);
  }
}
```
### 통합형 Validation Check 
```javascript
function validation_control(input_name){
  const 
}
 
```