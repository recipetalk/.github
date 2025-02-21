# 레시피톡 자세히 보기

## 결과물

[앱 빌드 파일](https://www.notion.so/430d0fdc6a08437fb28a8db813ca6881?pvs=21)

- Github
    - backend Code : https://github.com/recipetalk/back-end
    - frontend Code : https://github.com/recipetalk/front-end

## 기술 스택

> 프론트엔드 : React Native CLI, FCM, redux(toolkit), encrypted-storage 

백엔드 :  corretoJDK 17.0.3 + Spring Framework(With Spring Boot 3.0.1), ThymeLeaf, JPA, Hibernate, QueryDSL 5.0, FCM

데이터베이스 : MySQL

클라우드 : AWS (s3, EC2, CodeDeploy, RDS)

CI/CD : Git Action, Jenkins

형상관리 : Github
> 

## 아키텍처

![스크린샷 2023-10-05 오후 8.13.08.png](https://github.com/user-attachments/assets/c7492a25-cd46-473f-8659-4d316b9da8ed)

## ERD

![Image](https://github.com/user-attachments/assets/ec2a57d5-c9e7-4de1-ad91-a500a9f0fe16)

## 디자인

<table>
  <tr>
    <th>코드 업데이트 화면</th>
    <th>레시피 상세 조회 및 타이머 동작 화면</th>
    <th>레시피 게시판 상세 조회 화면</th>
    <th>검색 화면</th>
  </tr>
  <tr>
    <td><img src="https://github.com/user-attachments/assets/7048c35d-9867-4276-a5c3-d09a4f5f58fe"></td>
    <td><img src="https://github.com/user-attachments/assets/28a36ccd-9942-4de6-a60e-9adba6adc680"></td>
    <td><img src="https://github.com/user-attachments/assets/57529ca8-c9cc-4463-b10c-8e03912084d0"></td>
    <td><img src="https://github.com/user-attachments/assets/a17a0679-d95d-46c0-835a-a9be2bff626e"></td>
  </tr>
    <tr>
    <th>로그인 화면</th>
    <th>레시피 리스트 조회 화면</th>
    <th>식재료 관리 화면</th>
    <th>전체 검색 화면</th>
  </tr>
  <tr>
    <td><img src="https://github.com/user-attachments/assets/c61d7742-85c9-4fa5-868a-78101487f490"></td>
    <td><img src="https://github.com/user-attachments/assets/8f19aa71-9b0e-476c-9355-1af9ea5888f4"></td>
    <td><img src="https://github.com/user-attachments/assets/0b087257-5ba6-4ae3-99f6-2ef399948632"</td>
    <td><img src="https://github.com/user-attachments/assets/d8da5f4c-d365-423e-a6db-904c2c2aedb6"></td>
    
  </tr>
</table>
  
## 맡은 업무

# 서비스 소개

이 프로젝트는 만개의 레시피를 모티브로 한 레시피 제공 서비스에 식재료 관리 기능을 추가한 애플리케이션입니다.

# 문제 개선

## **팔로우 및 팔로잉 유저 조회 TPS 속도 5배 개선**

### 개요 & 원인

- DB : Mysql 8.0을 사용중
- Entity
    
    <img width="198" alt="Image" src="https://github.com/user-attachments/assets/ab4bd49f-0f25-4d4b-90f1-7d54b2beb0d4" />
    
    - 복합키로 구성된 엔티티
        - user_id : 팔로우한 사용자의 아이디
        - following_id : 팔로잉한 사용자의 아이디

- 조회 쿼리

```sql
SELECT u.username, u.nickname, u.profile_image_uri
				FROM user_follow uf JOIN user_detail u ON ub.following_id = u.user_detail_id
				WHERE uf.user_id = 1 limit 0, 20;
```

- 쿼리 실행 규칙 파악 결과
    
    <img width="1087" alt="Image" src="https://github.com/user-attachments/assets/1265899d-82f3-4065-9adc-6e2bd4f1a55a" />
    
- possible_keys : 탐색에 사용 가능한 key의 리스트
- key : 쿼리에 사용된 키
- key_len : 키의 길이
- Extra : 비고와 같은 칸으로, 메인 쿼리에서 Using where, Using index를 사용하는 것을 볼 수 있습니다.

### 접근 방법

- 복합 키의 쿼리 탐색 방식에 대해 알아야 합니다.

### 해결 과정

레퍼런스에서 복합 키 조회 방식에 대한 참고

[MySQL :: MySQL 8.0 Reference Manual :: 8.3.6 Multiple-Column Indexes](https://dev.mysql.com/doc/refman/8.0/en/multiple-column-indexes.html)

복합키 1번 PK와 2번 PK가 있다고 하면, 

조회 시 1번 PK를 제외하고 2번 PK에 해당하는 조건문으로만 조회시 Full Index Scan 발생하여 성능이 떨어지는 문제였습니다.

복합키로 구성된 테이블을 Integer 대리키로 설정하고 복합키를 FK로 설정함으로서 문제를 해결했습니다.

### 결론

복합키를 구성할 때는, 정말 필요한 이유가 있지 않는 한 득보단 실이 더 크다는 것을 알게 되었습니다.

### **추가**

> **Real MYSQL 8.0에서…**
MySQL에서 Index Page size 는 16KB 가 기본 값 입니다. 그리고 12Byte는 자식 노드 주소를 가리킵니다. 
복합키로 구성된 key_len은 16Byte이므로  28Byte가 한 인덱스 노드에 사용된다는 의미이고
16 * 1024 / (16 + 12) = 585.xxx 즉, level당 585개의 인덱스를 담을 수 있습니다.
MySQL은 B+Tree를 사용하기 때문에, 3단계의 depth를 지난다면,
585 * 585 * 585는 약 2억개의 데이터가 사용가능 하다는 것이고, Following 같은 경우는 수가 많아지는 특성이 있는 엔티티이므로 복합키는 적합하지 않습니다.
> 

## **Axios interceptor를 활용한 JWT토큰 만기 시 토큰 업데이트 후 요청하도록 구현**

### 개요 & 원인

API 기능을 모두 작성하고 나서, 자동 로그인 기능을 구현해야 하는 일이 있었습니다.

현재 프론트엔드 코드에 작성된 API 기능만 80개이고 모두 같은 내용의 코드들이 중복될 것임이 예상됩니다.

### 접근 방법

먼저 현재 서버와 프론트엔드 사이에 통신으로 사용되고 있는 Axios 라이브러리의 레퍼런스를 찾아보기로 합니다.

[인터셉터 | Axios Docs](https://axios-http.com/kr/docs/interceptors)

**예제코드**

```jsx
// 요청 인터셉터 추가하기
axios.interceptors.**request**.use(function (config) {
    // 요청이 전달되기 전에 작업 수행
    return config;
  }, function (error) {
    // 요청 오류가 있는 작업 수행
    return Promise.reject(error);
  });

// 응답 인터셉터 추가하기
axios.interceptors.**response**.use(function (response) {
    // 2xx 범위에 있는 상태 코드는 이 함수를 트리거 합니다.
    // 응답 데이터가 있는 작업 수행
    return response;
  }, function (error) {
    // 2xx 외의 범위에 있는 상태 코드는 이 함수를 트리거 합니다.
    // 응답 오류가 있는 작업 수행
    return Promise.reject(error);
  });
```

interceptors.**request** : 요청이 서버에 전달되기 전 (전송하기 전) 작업을 수행하는 요청 인터셉터

interceptors.**response** : 응답이 됬을 때, 특정 응답인 경우 작업을 수행하는 응답 인터셉터

### 해결 과정

여기서 저희가 필요한 것은  토큰이 만료되었을 때 401(unauthorized)이 반환되므로, 토큰을 재발급해서 재요청 후 결과를 받아와야 하는 interceptors.**response** 임을 알 수 있습니다.

시퀀스 다이어그램을 작성해서 한눈에 이해하기 쉽게 해보겠습니다.

<img width="446" alt="Image" src="https://github.com/user-attachments/assets/cf125b7b-3d12-4055-a6af-92177cf47c23" />

- 여기서 **주황색 화살표** 부분이 인터셉터로 처리가 가능한 공통 부분입니다.
- 따라서, 토큰 재발급 하는 코드를 아래와 같이 작성합니다.

```jsx
jsonAPI.interceptors.response.use(
  res => {
    return res;
  },
  async err => {
    const {
      config,
      response: {status},
    } = err;
    if (config?.headers?.authorization && status === 401) {
      const originalRequest = config;
      console.log('repeat Request');
      const {username, password} = await loadLoginFromStorage();
      config.headers.authorization = await jsonLogin(username, password);
      return axios(originalRequest);
    }
    return Promise.reject(err);
  },
);
```

위 코드에서 status 코드가 401이면 다시 로그인 요청을 하여 토큰을 받아오고 다시 요청하는 것을 확인 할 수 있습니다.

### 결론

- 기존 프론트코드에서 JWT Access Token 체킹 중복 코드 발생
- Axios Interceptor를 사용하여 체킹 후 요청하도록 변경, 중복 코드 제거

## **푸시** **알림 기능 구현**

### 개요 & 원인

백엔드에서 앱에게 푸시 알림을 전송해야 하는 기능을 작성하게 되었습니다.

### 접근 방법

특정 이벤트가 발생하면, 서버에서 타겟에게 보내야 하는 형태의 알림입니다.

따라서, 앱에게 푸쉬 알림을 전송할 수 있는 방식은 아래 4가지 입니다.

1. **Polling** 
    
    → Http 요청을 지속적으로 보내서 보낼꺼 있는지 확인하는 방식
    

1. **Long Polling**
    
    → Pooling에서, 보낼 꺼 있는지 request를 날리고 response를 기다리는 방식
    

[RFC 6202: Known Issues and Best Practices for the Use of Long Polling and Streaming in Bidirectional HTTP](https://www.rfc-editor.org/rfc/rfc6202#page-4)

1. **webSocket** 
    
    → 서버와 연결하여, 상시 통신하는 방식
    
    [RFC 6455: The WebSocket Protocol](https://datatracker.ietf.org/doc/html/rfc6455)
    

1. **SSE 방식**

→ Server Sent Event의 약자로, 서버에서 이벤트를 보내는 방식.

1. **Firebase를 사용한 FCM 방식**
    
    → 앱에서 FCM 토큰을 발급받고 서버에서 FCM 토큰으로 Firebase에 알림을 보내면, Firebase가 FCM 토큰의 사용자에게 알림을 보내주는 방식
    

여기서 저는 4번 **Firebase를 사용한 FCM 방식**을 사용하려고 합니다.

Polling 방식은 HTTP 기반으로 동작하며, 서버가 한 커넥션마다 일정 시간마다 Connection 체킹을 해줘야 합니다.

Long Polling 방식은 HTTP 기반으로 동작하며, 악의 사용자가 다수의 Long Polling을 걸어버리면 서버는 트래픽 부하 상태에 빠질 수 있습니다. 대신 빠른 실시간성을 보장할 수 있습니다. 또한 푸시 알림은 그렇게 실시간성이 필요로 하는 기능은 아닙니다.

webSocket 방식은 Long Pollling보단 나으나, webSocketSession을 관리해야 하는 단점이 있으며 Auto scaling이 일어 났을 시 세션을 찾아야 하는 번거러움이 존재합니다. 또한 푸시 알림은 그렇게 실시간성이 필요로 되는 기능은 아닙니다.

SSE또한 클라이언트가 접속했는지 안했는지 SseEmitter 객체 관리를 해주어야 한다는 단점이 존재합니다.

FCM은 1,500,000/min 메시지가 프로젝트마다 제한이 되어있으며 이는 차 후 잡큐를 활용하여 제한하면 커버가 어느정도 가능할 것이라 예상됩니다. 

기능적으로, 사용자가 앱의 Terminate 상태에서도 앱을 수신하게 하려면 앱을 켜서 Connection을 맺어야 하는 방식이 아닌 FCM 방식이 맞다고 생각했습니다.

### 해결 과정

```java
@EventListener
    public void handleNotificationEvent(final NotificationVo notificationVo){
        try {
            firebaseCloudMessageService.sendMessageTo(notificationVo.toMessage());
        } catch (RuntimeException | FirebaseMessagingException e) {
            if(notificationVO.getFcmtoken() != null){
                fcmTokenRepository.delete(notificationVo.getFcmtoken());
            }
        } finally {
            notificationRepository.save(notificationVO.toEntity());
        }
    }
```

추상화된 NotificationVO 인터페이스의 객체를 EventListener 방식으로 받아와 중복 코드를 줄였으며, 

사용자의 FCM 토큰이 문제가 있는 경우 해당 토큰 사용자는 로그아웃 하였거나 앱을 삭제한 경우로 판단되어 삭제하는 방식을 취하였습니다.

### 결론

- 메모리 소비 문제와 세션 관리의 단점을 FCM 사용으로 해결

## **페이지 렌더링 시 이미지 계속 깜빡임 메모이제이션으로 해결**

### 개요 & 원인

<img src="https://github.com/user-attachments/assets/2bad336f-3bbd-47dc-8f52-08bbdfb2123e" width="300">

위 스크린에서 사용되는 이미지 컴포넌트가 많습니다.

처음 포커스에 맞춰졌을때 렌더링이 끝날 때까지 이미지가 깜빡이는 문제가 있었습니다.

무한스크롤로 구현되어있어 아래로 내리면 다시 렌더링이 끝날 때까지 이미지가 깜빡이는 문제가 있었습니다.

### 접근 방법

리액트 네이티브의 동작방식에 대해 알아보았습니다.

[React Fundamentals · React Native](https://reactnative.dev/docs/intro-react)

React Native는 JavaScript로 사용자 인터페이스를 구축하기 위한 인기 있는 오픈 소스 라이브러리인 React 에서 실행된다고 합니다.

따라서, React에 대한 동작방식을 캐치해야 React native를 한 단계 더 이해할 수 있을 것이라 판단됩니다.

리액트에 대한 설명입니다.

[Describing the UI – React](https://react.dev/learn/describing-the-ui)

React는 “사용자 인터페이스(UI)를 렌더링하기 위한 JavaScript 라이브러리” 라고 표현합니다.

즉, React는 렌더링에 포커스를 준 라이브러리라고 판단할 수 있으며, 렌더링이 어떠한 경우에 대해 일어나는지 숙지해보겠습니다.

[Render and Commit – React](https://react.dev/learn/render-and-commit)

“렌더링”은 React가 구성요소를 호출하는 것임을 알 수 있으며, 

초기 렌더링 시 React는 루트 구성 요소를 호출하고

후속 렌더링의 경우 React는 상태 업데이트가 렌더링을 트리거한 함수 구성 요소를 호출합니다.

렌더링은 재귀적입니다 : 업데이트된 구성 요소가 다른 구성 요소를 반환하면 React는 해당 구성 요소를 다음에 렌더링하고 해당 구성 요소도 무언가를 반환하면 해당 구성 요소를 다음에 렌더링 하는 식입니다.

부모 컴포넌트가 변경되면 자식 컴포넌트도 전부 렌더링 된다는 의미를 지니고 있습니다.

따라서, 이미지 플리커가 발생하는 이유는 부모 상태가 변경되면서 재귀 호출된 자식 컴포넌트인 이미지가 리렌더링되어 깜빡이는 현상을 볼 수 있다고 할 수 있었습니다.

### 해결 과정

이를 막기 위해, 먼저 렌더링을 그만두게 해야 하는 방식이 있나 알아볼 필요가 있습니다.

[memo – React](https://react.dev/reference/react/memo)

memo는 컴포넌트가 변경되지 않은 경우 구성 요소를 다시 렌더링하는 것을 건너 뛸 수 있다고 합니다.

```jsx
function RecipeComponent({navigation, item}) {...}

export default React.memo(RecipeComponent);
```

위와 같이 코드를 작성함으로써, 이 컴포넌트의 상태가 더이상 변경되지 않으니 그 이후로 다시 렌더링하지 않으며 이미지가 깜빡이지 않는 모습을 확인할 수 있었습니다.

### 결론

- 부모가 렌더링 되는 경우 해당 자식 컴포넌트가 계속 리렌더링되어 깜빡임 현상 문제 발생
    - 메모이제이션 방식으로 해당 자식 컴포넌트들을 메모리에 저장하여 깜빡임 현상 및 버벅임 문제 해결

# API 명세

<a href="https://khj745700.notion.site/04561b1c96e7455d8037818deb67bc48?v=8c466a55b30a49b2ac5aa619234b5f7a&pvs=4">API 명세 보기</a>

---

# 에러 코드 명세

<a href="https://khj745700.notion.site/71d9ab15cbf04f3cab76b60d36af0265?v=fc5838a3a874438592691d81e8224ba3&pvs=4">에러코드 명세 보기</a>

---
