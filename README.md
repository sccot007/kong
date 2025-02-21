# kong 인증에 관해

> Kong API Gateway는 OAuth 2.0, JWT, Basic Auth, Key Authentication 등 다양한 인증 방식을 지원합니다.
클라이언트가 API 요청을 보낼 때, Kong이 인증 절차를 수행하는 과정을 __JWT__ 토큰 기반 인증을 예제

## kong에 JWT 플러그인 활성화
<pre>
POST http://localhost:8001/services/user-service/plugins
Content-Type: application/json
<code>
{
  "name": "jwt",
  "config": {
    "key_claim_name": "iss",
    "secret_is_base64": false
  }
}
</code>
</pre>

## 1. 인증 과정
```plaintext

1. 인증요청
+-----------+        +-----------------+        +-----------------+
|  Client   | -----> |      Kong       | -----> |    Auth Server  |
+-----------+        +-----------------+        +-----------------+
                 |                        |              |
                 v                        v              v
            사용자 요청            인증 정보 요청      access_token 발급
          (ID/PWD 로그인)

2. 이후 API 호출 >  JWT 플러그인에서 인증수행 > 인증 성공이면 kong에서 Backend Service 호출
                     +--------------------------------------------+
+-----------+        |                          +-----------------+        +-------------------+
|  Client   | -----> |      Kong  ------------> |    JWT 플러그인  | -----> |  Backend Service  |
+-----------+        |                          +-----------------+        +-------------------+
                     +--------------------------------------------+
                 |                                        |
                 v                                        v
            발급받은 access_token 적용            1. JWT 토큰 Base64 디코딩 2. 서명검증
          "Authorization" 헤더에 포             3. 유효기간(exp)검증 4.발급자(lss) 검증 5.Audience(aud) 검증

3. Backend Service 결과값을 kong을 거처 Client로 반환
```

## 2. OAuth2.0을 이용한 인증 예시(쇼핑몰 홈페이지)
```plaintext

1. 회원가입 (예. /member/oauth2)
+-----------+        +--------------------+             +-----------------+             +-----------------+
|  Client   | -----> |        Kong        | ----------> | 쇼핑몰-회원가입  | ----------> |    OAuth 2.0    |
+-----------+        +--------------------+             +-----------------+             +-----------------+
                |                                                               |                |
                v                                                               v                v
          사용자 가입 요청                                              가입 후 OAuth       client_id, client_secret
                                                                       자격증명요청       생성 후 DB 저장
          

2. 로그인 시 Authorization Code 발급 (예. /oauth2/authorize)
                              +--------------------------------------------------------------------+
                              |                                                                    |
+-----------+        +--------------------+             +-----------------+                +-----------------+
|  Client   | -----> |        Kong        | ----------> | 쇼핑몰 로그인    | -------------> |    OAuth 2.0    |
+-----------+        +--------------------+             +-----------------+                +-----------------+
                 |                                                               |                  |
                 v                                                               v                  v
              로그인                                                       id/pwd 확인 후        확인후 승인하면 
                                                                          client_id인증 요청     Authorization Code 발급.

3. Access Token 발급 (예. /oauth2/token)
+-----------+        +--------------------+             +-----------------+
|  Client   | -----> |        Kong        | ----------> |  OAuth 2.0      |
+-----------+        +--------------------+             +-----------------+
                 |                               |                  |
                 v                               v                  v
              로그인                    Authorization Code로    확인후 승인하면 
                                          Access Token 요청   Access_token 발급.

4. 이후 API 호출 >  OAuth2.0 플러그인에서 인증수행 > 인증 성공이면 kong에서 Backend Service 호출
```



