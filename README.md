# [waple-app-web] 개발 가이드

[TOC]

## 1. 개발환경 구성

### 1.1. 이클립스

#### 1.1.1. 다운로드 및 설치

> 다운로드 : svn://scvm.kyowon.co.kr/epportal/tools/eclipse.zip

1. [eclipse.zip] 다운로드 
2. 압축해제
3. /eclipse/eclipse.exe 실행



### 1.1.2. lombok 설치

lombok 직접 설치 방법에 대해 가이드합니다.

1. /eclipse/lombok.js 실행
2. Specify location.. 선택
3. /eclipse/eclipse.exe 선택
4. Install / Update 실행



### 1.2. SVN 정보

#### 1.2.1. svn 접속 정보

> url : svn://scvm.kyowon.co.kr/epportal/source/waple-app-web
>
> id / pwd : 계정/계정!  ( ex> gs.itdev000/gs.itdev000! )



#### 1.2.2. 소스 체크아웃

1.  eclipse [new>Other..>SVN>Checkout Projects from SVN] 선택
2.  SVN 접속 정보 입력
3.  [svn://scvm.kyowon.co.kr/epportal/source/waple-app-web] 경로의 소스 체크아웃



### 1.3. node.js 설치

로컬 서버 및 빌드를 위해 Node.js  설치가 필요합니다.

> Node.js 다운로드 정보
>
> - https://nodejs.org/ko/



### 1.4. Vue cli 설치

#### 1.4.1. 설치

Vue.js를 빠르게 개발할 수 있도록 지원해주는 CLI(Command Line Interface)다.

본 문서를 Vue CLI 3 버전 기준으로 작성되었다.



**Vue CLI 3  설치**

> npm install @vue/cli -g



**(Optional) Vue CLI 2 버전도 사용할 수 있도록 설치**

> npm install -g vue-cli



### 1.5. 로컬서버 실행

Vue CLI로 생성된 프로젝트는 커맨드를 통해 프로잭트를 실행할 수 있다.



**[node_modules] install**

[waple-app-web] 소스를 체크아웃 받은 경로에서 명령어를 실행해야합니다.

> C:\develop\workspace-kyowon\waple-app-web> npm install



설치중 인증서 오류가 발생하면 네트워크를 무선(KYOWON)으로 연결 후 명령어를 실행하시면 됩니다.



**[Vue CLI 3] 개발 서버 실행** 

> npm run serve



### 1.6. 빌드

npm 명령을 통해서 프로젝트를 빌드하고,

빌드된 결과를 웹서버에 배포하여 실행할 수 있다.

(개발서버 배포 시 빌드 후 SVN에 커밋을 해야합니다.)

> npm run build



#### 1.6.1. Vue.js 개발 버전으로 빌드

devtools 사용을 위해 vue.js를 개발 버전으로 빌드를 하고 싶으면

package.json 파일을 수정해야 한다.

[참고]: https://cli.vuejs.org/guide/mode-and-env.html#modes



```js
-- package.json

"build": "vue-cli-service build --mode development",
```



### 1.7. 설정

#### 1.7.1. 정적 파일 참조 경로 변경

빌드 시 정적 파일 참조로 경로를 변경하는 방법을  가이드 한다.

> 프로젝트 root 폴더에 vue.config.js 파일 생성  후 publicPath 설정 추가

[참고]: https://m.blog.naver.com/mgveg/221980424679

```json
-- vue.config.js

module.exports = {
  publicPath: process.env.NODE_ENV === 'production'
    ? './'
    : './'
}
```



#### 1.7.2. API 서버 프록시 설정

vue.config.js 설정을 통해 devServer에서 API서버로 프로싱하도록 설정 가능하다.

> 샘플) /api/ 경로로 접근하는 호출은 프로싱 처리하여 API 서버로 전달 함

```js
    devServer: {  
        proxy : {
            '/api/' : {
                target : 'http://localhost:9080',
                changeOrigin: true            
            }  
        }
    },
```



## 2. 개발 가이드

### 2.1. 디렉토리 구조

```
ㄴ public : 정적자원(퍼블, favicon 등..)
ㄴ src : 컴파일되는 소스
  ㄴ api : api 서버 연동 모듈
    ㄴ api.js : export 객체
    ㄴ moudles : 연동 모듈 관리, API 서버 연동 시 해당 모듈 추가
      ㄴ core.js : 연동 코어
      ㄴ auth.js : 로그인 연동      
  ㄴ assets : 컴파일되는 정적 자원
  ㄴ components : 뷰 컴포넌트
    ㄴ base : 전역 컴포넌트(파일명을 Base로 시작해야 함)
  ㄴ router : 라우터 설정
  ㄴ store : vuex 설정
    ㄴ modules : vuex 저장 객체
    ㄴ store.js : export 객체
  ㄴ utils : 공통 유틸
    ㄴ modules : 유틸 객체
    ㄴ utils.js : export 객체
  ㄴ views : 라우터-뷰 파일
```



### 2.2. api 개발 가이드

#### 2.2.1. 연동 모듈 개발

modules 디렉토리에 js 파일 추가 

샘플)  ./modules/resource.js

```
// ./modules/resource.js
import core from '../modules/core';

export default {
  resource(parameters={}) {
    return core.call({
      url: '/api/v1/resource/sample',
      method: 'post',
      data: parameters,
      headers: {
        'Authorization': 'Bearer '+ store.getters.authToken,
        'deviceUuid': utils.nativeCom.getUserAgentValue("deviceUuid")
      }    
    });  
  }
}
```



core 객체

> import core from '../modules/core';
>
> core 객체는 axios 랩핑한 객체로 통신 모듈입니다.



요청 객체

> url : api url
>
> method : GET/POST
>
> data : 요청 객체
>
> headers 
>
> > Authorization : 인증이 필요한 경우 설정
> >
> > deviceUuid : 인증이 필요한 경우 설정 / 단말UUID 가 필요한 경우 설정



### 2.3. components 개발 가이드

뷰 컴포넌트는 ./components 하위에 위치해야 합니다.

전역 컴포넌트는 ./components/base 하위에 위치해야 하며, 파일명은 Base로 시작해야 합니다.



### 2.4. router 개발 가이드

#### 2.4.1. 라우터 뷰 개발

라우터 뷰 파일은 ./views 하위에 위치해야 합니다.

```
// ./views/Error.vue

<template>
  <div>
   오류 페이지
  </div>
</template>
<script>
export default {
  data() {
    return {
    };
  }
};
</script>
```



파일명

> 파일명은 대분자로 시작해야 합니다.



디렉토리 경로 정책

> url 패턴과 일치된 형태로 디렉토리를 정의해야 합니다.



#### 2.4.2. 라우터 뷰 설정

Vue-router개발 가이드는 [여기](./references/Vue-router.md)서 확인하세요.

라우터 뷰 개발이 끝나면 ./router/router.js에서 



라우터 뷰 Import

> import Error from '@/views/Error'



라우터 설정

> {
>   path: '/error',  // 라우터 패스
>   name: 'error', // 라우터 이름
>   component: Error, // 라우터 뷰 
>   meta: { auth: true } // 메타 정보
> }



메타 정보

> meta: { 
>
>   auth: true  // 로그인 체크 여부
>
> }



### 2.5. Vuex 설정

Vuex는 Vue.js 어플리케이션의 상태정보를 관리합니다.

Vuex 개발 가이드는 [여기](./references/Vuex.md)서 확인하세요.



### 2.6. Utils 개발 가이드

#### 2.6.1. 사용 가이드

Vue.js 에서 사용

```
// ./modules/utils.native.interface.js

this.$utils.nativeIf.outbound.close();
```



외부에서 사용

```
// ./modules/utils.native.interface.js

window.app.$root.$utils.nativeIf.inbound.sendChildData(child);
```



js 파일에서 사용

```
import utils from '@/utils/utils'

....
utils.nativeCom.getUserAgentValue("deviceUuid")
```



#### 2.6.2. 유틸 추가

js 파일 생성

> ./utils/modules/{유틸}.js
>
> 위 경로로 파일 추가



export 

```
// ./utils/utils.js

import nativeIf from './modules/utils.native.interface.js';
import nativeCom from './modules/utils.native.common.js';

export default {
  nativeIf,
  nativeCom
}
```



### 2.7. Vue.js 개발 가이드

Vue 개발 가이드는 [여기](./references/Vue.md)서 확인하세요.



## Appendix A. VS Code 설치

### A.1. vscode portable 다운로드

[다운로드](https://code.visualstudio.com/download)

> 설치 순서
>
> 1. .zip(64bit) 다운로드 
>
> 2. 압축 해제 후 폴더에 data 폴더 생성
>
>    ```
>    |- VSCode-win32-x64-1.25.0-insider
>    |   |- Code.exe (or code executable)
>    |   |- data
>    |   |- ...
>    ```
>
> 3. (optional) 마이그레이션
>
>    마이그레이션 순서
>
>    1) %APPDATA%\Code 폴더를 data 폴더에 복사 후 user-data로 이름 변경
>
>    2) %USERPROFILE%\.vscode\extensions 폴더를 data 폴더에 저장
>
>    ```
>    |- VSCode-win32-x64-1.25.0-insider
>    |   |- Code.exe (or code executable)
>    |   |- data
>    |   |   |- user-data
>    |   |   |   |- ...
>    |   |   |- extensions
>    |   |   |   |- ...
>    |   |- ...
>    ```
>
> 4. data 폴더에 tmp 폴더 생성



### A.2. Extentions 툴 설치

> vetur : Vue.js 편의 도구
>
> Live Server :  로컬 서버
>
> vscode-icons : 선택
>
> vue : Vue.js 문법 검사기
>
> Vue VSCode Snippets : Vue 문법



### A.3. 디버깅

[참고]: https://kr.vuejs.org/v2/cookbook/debugging-in-vscode.html



extendtion : Debugger for Chrome 설치 필요 



> Vue CLI 3 버전 사용 필요 함

[vue.config.js]

```json
// vue.config.js
module.exports = {
  configureWebpack: {
    devtool: 'source-map'
  }
}
```



> vscode 디버거 태스크 추가

```json
{
  "version": "0.2.0",
  "configurations": [
    {
      "type": "chrome",
      "request": "launch",
      "name": "vuejs: chrome",
      "url": "http://localhost:8080",
      "webRoot": "${workspaceFolder}/src",
      "breakOnLoad": true,
      "sourceMapPathOverrides": {
        "webpack:///./src/*": "${webRoot}/*"
      }
    },
    {
      "type": "firefox",
      "request": "launch",
      "name": "vuejs: firefox",
      "url": "http://localhost:8080",
      "webRoot": "${workspaceFolder}/src",
      "pathMappings": [{ "url": "webpack:///src/", "path": "${webRoot}/" }]
    }
  ]
}
```



## Appendix B. 크롬 개발도구  설정

### B.1. Vue.js devtools (Chrome Extention)

Vue.js는 크롬 확장도구로 devtools를 제공한다.

> devtools을 사용하기 위해서는 Vue.js를 production 버전이 아닌 development 버전을 사용해야 한다.

[설치]: https://chrome.google.com/webstore/detail/vuejs-devtools/nhdogjmejiglipccpnnnanhbledajbpd



### B.2. Emuldated Devices 설정

#### B.2.1. 디바이스 추가

1. 개발자 도구 열기 (F12)
2. Toggle device toolbar(Ctrl + Shift + M)
3. Edit 클릭

![](.\references\assets\chrome-emulate-1.PNG)

4. Add custom device.. 클릭

5. User agent string 입력

   > Mozilla/5.0 (Linux; Android 10; SM-G977N Build/QP1A.190711.020; wv) AppleWebKit/537.36 (KHTML, like Gecko) Version/4.0 Chrome/86.0.4240.99 Mobile Safari/537.36 platform=PC  deviceUuid=DEICE-ID-1111-2222-3333-4444-5555-6666-7777 client=ZXBwb3J0YWxhcHA6ZXBwb3J0YWxhcHAh pushToken=2aK9KHmw8E:APA91bF7MY9bNnvGAXgbHN58lyDxc9KnuXNXwsqUs4uV4GyeF06HM1hMm-etu63S_4C-GnEtHAxJPJJC4H__VcIk90A69qQz65toFejxyncceg0_j5xwoFWvPQ5pzKo69rUnuCl1GSSv1

   

6. 저장



### B.3. ModHeader(Chrome Extention) 설치

다운로드는 [여기](https://chrome.google.com/webstore/detail/modheader/idgpnmonknjnojddfkpgkljpfnnfcklj?utm_source=chrome-ntp-icon)에서 하세요.



User-Agent 설정

![](.\references\assets\chrome-extention-modheader.PNG)



> Mozilla/5.0 (Windows NT 10.0; Win64; x64) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/89.0.4389.128 Safari/537.36 platform=PC  deviceUuid=DEICE-ID-1111-2222-3333-4444-5555-6666-7777



## Appendix C. User-Agent 

### C.1. platform (필수)

플랫폼은 접속 단말을 의미합니다.

안드로이드/아이폰 구분은 앱과 연동 시 사용되며, 로컬(PC) 환경에서는 정상적으로 동작하지 않습니다.

| 구분       | 값   |
| ---------- | ---- |
| 로컬(PC)   | PC   |
| 안드로이드 | AND  |
| 아이폰     | IOS  |



### C.2. deviceUuid (필수)

단말UUID는 인증토큰 발급 및 검증에 사용합니다.

로컬환경에서 테스트 시 개발자별 중복하지 않는 값을 사용하는 것을 추천합니다.

(로그인 시 단말 중복체크 기능 존재 함)



### C.3. client (필수)

인증(로그인) 시 사용된 client secret 정보입니다.

인증 유형별 고정된 값을 설정합니다.

고정값 : ZXBwb3J0YWxhcHA6ZXBwb3J0YWxhcHAh



### C.4. pushToken (선택)

FCM에서 발급받은 푸시 토큰입니다.

