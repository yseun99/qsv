# Sphere SDK - Web

Sphere Analytics 웹 SDK 연동 가이드입니다.

* [웹 SDK 설치](#웹-SDK-설치)
* [기본 연동](#기본-연동)
  * [네이티브 SDK 연동](#네이티브-SDK-연동)
  * [웹 SDK API](#웹-SDK-API)
* [커스텀 이벤트 사용하기](#커스텀-이벤트-사용하기)
* [사용자 속성 사용하기](#사용자-속성-사용하기)
  * [사용자 아이디 설정](#사용자-아이디-설정)
  * [사용자 정보 설정](#사용자-정보-설정)
  * [커스텀 사용자 속성 설정](#커스텀-사용자-속성-설정)

## 웹 SDK 설치

* **1 단계**: 전달한 웹 SDK 파일(sphere.min.js)을 서비스 도메인에서 다운로드 가능하도록 웹 서버에 복사해 주세요.  
   **(확인)** '*https://[서비스-도메인]/sphere.min.js*' 형식의 HTTPS URL로 웹 SDK 파일 다운로드가 가능한지 확인해 주세요.

* **2 단계**: 아래 `<script>` 태그를 `<head>`에 추가해 주세요. 아래 2가지 중 원하시는 형태를 사용하시면 됩니다.
    ```html
    <script>
        var _sphereJsUrl = "https://[서비스-도메인]/sphere.min.js";    
        !function(e,t,r,s,n){e.SphereAnalytics=e.SphereAnalytics||{_q:[]};for(var i=0;i<s.length;i++)n(e.SphereAnalytics,s[i]);o=t.createElement("script");o.async=!0,o.src=r;a=t.getElementsByTagName("script")[0];a.parentNode.insertBefore(o,a)}(window,document,_sphereJsUrl,["init","setUserId","setGrade","setGender","setBirthYear","setPhone","setEmail","setRemainingPoint","setTotalEarnedPoint","setTotalUsedPoint","setUserProperty","logEvent","requestUpload","setLogLevel"],function(e,t){e[t]=function(){return e._q.push([t,arguments]),e}});
    </script>
    ```
    또는
    ```html
    <script src="https://[서비스-도메인]/sphere.min.js" async></script>
    <script>
        !function(e,t,n,s){e.SphereAnalytics=e.SphereAnalytics||{_q:[]};for(var r=0;r<n.length;r++)s(e.SphereAnalytics,n[r])}(window,document,["init","setUserId","setGrade","setGender","setBirthYear","setPhone","setEmail","setRemainingPoint","setTotalEarnedPoint","setTotalUsedPoint","setUserProperty","logEvent","requestUpload","setLogLevel"],function(e,t){e[t]=function(){return e._q.push([t,arguments]),e}});
    </script>
    ```
* **3 단계**: **1 단계**에서 생성한 다운로드 URL을 위 **2 단계**의 `_sphereUrl` 변수 또는 `<script src>`속성값으로 할당해 주십시오.
  - 웹 SDK는 비동기(async) 방식으로 다운로드되며, 다운로드가 지연되더라도 서비스 페이지 로딩에 지연을 발생시키지는 않습니다.
  - 웹 SDK 다운로드 완료전에도 웹 SDK API 호출이 가능하며, 다운로드 완료된 시점에 호출했던 API들이 일괄 지연 처리됩니다.

* **4 단계(선택사항)**:  
  - **(웹 용으로 사용한다면)** 전달받은 `[Web Key]`를 초기화 때, 반드시 웹 SDK에 설정해 주세요.  
  - **(웹 뷰 전용으로 사용한다면)** 생략 가능합니다.
    ```html
    <script>
        SphereAnalytics.init({
            token: '[Web Key]'
        });
    </script>
    ```

### 웹 SDK 연동 테스트 설정
> 웹 SDK를 웹 용으로 사용한다면, 아래 테스트 설정을 통해 다양한 테스트용 이벤트를 발송할 수 있습니다.  

* **(웹 용으로 사용한다면)** 연동 테스트를 위해서, 위 **4 단계** 코드를 아래로 교체한 후 진행해 주세요.
    ```html
    <script>
        SphereAnalytics.init({
            token: "[Web Key]",
            test: true, // Communicate with the test server.
            logLevel: 'verbose', // default: "error" ["none", "error", "info", "verbose"]
            trackAnonymous: false,
        });
    </script>
    ```
    - `SphereAnalytics.init()` 메소드를 통해 설정 가능한 옵션은 아래와 같습니다.
        + **token** : (웹 용으로 사용할 때 필수) 웹 용으로 데이타 집계하기 위한 키 값입니다.
            + 자료형: `string`
        - **test** : 테스트 서버 연동 (기본값: false)
            + 자료형: `boolean`
            + 기본값: `false`
            + 테스트 서버에 전송한 데이타는 상용 서비스에 영향을 끼치지 않습니다.
        - **logLvel** : (웹 용으로 사용할 때) 설정한 로그 레벨에 따라, 브라우저의 개발자 console에 디버깅 로그를 출력합니다.
            + 자료형: `string` - 다음 값 중 하나를 설정해야 합니다. [`'none'`,`'error'`,`'info'`,`'verbose'`]
            + 기본값: `'error'`
        - **trackAnonymous** : 기본적으로 로그인 사용자의 이벤트만 수집합니다. 이 값을 true로 설정하면, 익명 사용자의 이벤트도 함께 수집할 수 있습니다.
            + 자료형: `boolean`
            + 기본값: `false`

## 기본 연동

> 웹 뷰 환경에서는 네이티브 SDK를 통해 이벤트를 수집하고, 일반 브라우저 환경에서는 웹 SDK 단독으로 이벤트를 수집합니다.

* 웹 용으로 사용될 때는 웹 SDK에 설정한 `token` 값을 사용해서, 웹 사용 이력이 집계됩니다.
* 웹 뷰용으로 사용될 때는 네이티브 SDK를 통해 처리되며, 네이티트 SDK에 설정된 앱 키를 사용해서 앱 사용 이력으로 집계됩니다.

### 네이티브 SDK 연동
> 네이티브 SDK 설치는 **기본 연동**과 **웹뷰 설정**이 필수로 완료되어야 합니다.

* [Android SDK 연동가이드](https://github.com/tand-git/android-sdk) : [기본 연동](https://github.com/tand-git/android-sdk#기본-연동), [웹뷰 설정](https://github.com/tand-git/android-sdk#웹뷰-설정)
* [iOS SDK 연동가이드](https://github.com/tand-git/ios-sdk) : [기본 연동](https://github.com/tand-git/ios-sdk#기본-연동), [웹뷰 설정](https://github.com/tand-git/ios-sdk#웹뷰-설정)
* [SDK 연동 검증 가이드](https://github.com/tand-git/sphere-sdk/blob/master/guide/SDK_Inspection.md) : 기본 연동이 완료되었다면 SDK 연동 검증 가이드에 따라 SDK 동작 상태를 확인할 수 있습니다.

### 웹 SDK API

[웹 SDK를 설치](#웹-SDK-설치)한 후, 페이지 로딩 또는 이벤트 발생 시점에 자바스크립트 API 함수를 호출합니다.

`<sphereAnalytics.js>` Sphere 웹 SDK API
> [web/sphereAnalytics.js](web/sphereAnalytics.js) 파일 참조

`<index.html>` 웹페이지 사용 예제
> [web/index.html](web/index.html) 파일 참조

## 커스텀 이벤트 사용하기

> 이벤트는 가장 기본이 되는 수집 정보이며 이벤트는 이벤트명과 파라미터들로 구성이 됩니다.

SDK가 초기화 되었다면 `logEvent` 함수를 이용하여 커스텀 이벤트를 설정할 수 있으며, 한 이벤트는 최대 25개의 파라미터를 설정할 수 있습니다.
파라미터는 파라미터명과 파라미터값의 쌍으로 구성되며 JSON 타입을 통해 설정이 가능합니다.

이벤트명은 필수이며 파라미터는 없는 경우 `null`로 설정 가능합니다. 이벤트명과 파라미터에 관한 규칙은 다음과 같습니다.

1. 이벤트명
    * 최대 40자  
    * 영문 대소문자, 숫자, 특수문자 중 ‘_’ 만 허용  
    * 첫 글자는 영문 대소문자만 허용

2. 파라미터명
    * 최대 40자  
    * 영문 대소문자, 숫자, 특수 문자 중 ‘_’ 만 허용  
    * 첫 글자는 영문 대소문자만 허용

3. 파라미터값
    * 지원 타입 : String(최대 100자), Number

```js
// 이벤트 및 파라미터 기록. 파라미터 형식: JSON 타입 { name : value, ... }
var params = { item: "notebook", price: 9.9, quantity: 1 };
SphereAnalytics.logEvent("purchase", params);

// 파라미터가 없는 이벤트 기록
SphereAnalytics.logEvent("purchase_clicked", null);
```

## 사용자 속성 사용하기

> 사용자 속성을 사용할 경우 수집된 이벤트들을 세분화하여 더욱 자세한 분석 정보를 얻을 수 있으며 개인 정보들은 암호화되어 서버에 저장됩니다. 사용자 속성들은 한번 설정되면 이후 재설정 또는 초기화될 때까지 설정된 값으로 유지됩니다.

사용자 속성 연동 시 고려해야 할 사항은 다음과 같으며 해당되는 모든 시점에 사용자 속성들을 설정해야 정확한 분석이 가능합니다.

1. 자동 로그인 사용 : 로그인 상태 및 사용자 정보를 알 수 있는 가장 빠른 시점에 로그온 또는 로그오프 상태에 따라 사용자 아이디 및 정보를 설정 또는 초기화
2. 자동 로그인 미사용 : 로그인 또는 로그아웃 시 해당 상태에 따라 해당 사용자 아이디 및 정보를 설정 또는 초기화

### 사용자 아이디 설정

고유한 사용자를 구분하기 위한 사용자 아이디로서 설정 여부에 따라 로그인 여부를 판단합니다.  
해당 정보는 유저를 구분하기 위한 용도로만 사용되므로 사용자를 구분하는 어떠한 식별 아이디도 사용 가능합니다.  
사용자 아이디는 최대 256자까지 설정가능하고 `null`로 설정 시 사용자 아이디 정보는 초기화되고 로그아웃 상태로 설정됩니다.  

```js
if (isLogIn) { // 로그인: ON 상태
    // 사용자 아이디 설정 - 로그인: ON 상태
    SphereAnalytics.setUserId("[USER ID]");
} else { // 로그아웃: OFF 상태
    // 사용자 아이디 초기화 - 로그아웃: OFF 상태
    SphereAnalytics.setUserId(null);
}
```

### 사용자 정보 설정

추가적인 사용자 정보(보유 포인트, 등급, 성별, 출생년도, 전화번호, 이메일)를 설정합니다.  
로그아웃 상태 시 다음과 같이 설정된 사용자 정보들을 초기화해야 합니다.

1. 문자형(등급, 성별, 전화번호, 이메일) 초기화 : `null`로 설정
2. 숫자형(보유 포인트) 초기화 : `resetPoints` 함수 호출
3. 숫자형(출생년도) 초기화 : `0`으로 설정

```js
if (isLogIn) { // 로그인: ON 상태

    // 사용자 아이디 설정
    SphereAnalytics.setUserId("[USER ID]");

    // 보유 포인트 설정
    SphereAnalytics.setRemainingPoint(1000);
    // 등급 설정
    SphereAnalytics.setGrade("vip");
    // 성별 설정
    SphereAnalytics.setGender("m"); // 남성일 경우: "m", 여성일 경우: "f"
    // 출생년도 설정
    SphereAnalytics.setBirthYear(1995); // 출생년도
    // 이메일 설정
    SphereAnalytics.setEmail("xxxx@xxxx.com");
    // 전화번호 설정
    SphereAnalytics.setPhoneNumber("821011112222");

} else { // 로그아웃: OFF 상태

    // 사용자 아이디 초기화
    SphereAnalytics.setUserId(null);

    // 보유 포인트 초기화
    SphereAnalytics.resetPoints();
    // 등급 초기화
    SphereAnalytics.setGrade(null);
    // 성별 초기화
    SphereAnalytics.setGender(null);
    // 출생년도 초기화
    SphereAnalytics.setBirthYear(0);
    // 이메일 초기화
    SphereAnalytics.setEmail(null);
    // 전화번호 초기화
    SphereAnalytics.setPhoneNumber(null);

}
```

### 커스텀 사용자 속성 설정

미리 정의되지 않은 사용자 속성 정보를 사용 시 `setUserProperty` 함수를 이용하여 커스텀 사용자 속성을 설정할 수 있습니다.  
사용자 속성은 속성명과 속성값의 쌍으로 구성되며 속성값을 `null`로 설정 시 해당 속성은 초기화 됩니다.

사용자 속성에 관한 규칙은 다음과 같습니다.

1. 사용자 속성명
    * 최대 40자
    * 영문 대소문자, 숫자, 특수문자 중 ‘_’ 만 허용
    * 첫 글자는 영문 대소문자만 허용
    * "sap"으로 시작되는 속성명은 사전 정의된 속성명으로 사용 불가

2. 사용자 속성값
    * 최대 100자
    * 지원 타입 : String

```js
// 커스텀 사용자 속성 설정
SphereAnalytics.setUserProperty("user_property_name", "user_property_value");
// 커스텀 사용자 속성 초기화
SphereAnalytics.setUserProperty("user_property_name", null);
```
