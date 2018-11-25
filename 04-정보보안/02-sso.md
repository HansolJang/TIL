# Single Sign-On(SSO)

### 인증
- ***시스템이 사용자를 식별하는 것*** (WHO)
- 예: 사용자가 ID/PW를 입력함으로써 주인이라는 것을 식별

### 인가
- ***무엇을 해도 좋은지 체크하는 것*** (WHAT)

## SSO
- 한번의 **인증**으로 여러 서비스를 이용할 수 있는 것
- 인증 정보 관리 시스템 일원화
- 인증 시스템의 보안 확보가 더 중요해짐    

|  | 인증 | 인가 | 표준화 단체 |
|:-- |:--- |:-- |:------- |
|SAML| O | O | OASIS |
|OpenID| O | X | IETF |
|OAuth | X | O | IETF |
+ 주로 인터넷 서비스에서 사용하는 규격
+ 관리 가능한 네트워크에선 Active Directory를 주로 사용   

### SAML (Security Assertion Markup Language)
- identity provider와 service provider 간의 인증, 인가 데이터를 교환하기 위한 XML 기반의 데이터 포맷
- identity provider(IdP): 사용자 인증 정보 제공자
- service provider(SP): 서비스 제공자

![](https://docs.citrix.com/en-us/netscaler/12/media/SAML-flow.png)


### OpenID
- 새로운 웹사이트(relying party)에 가입하는 것이 아니라 OpenID를 제공하는 사이트(ID 제공자)의 아이디(openID)와 비밀번호를 입력하면 사용자가 인증됨
- 예: 롯데시네마에 로그인 하기 위해 롯데멤버십 ID를 OpenID로 허용하고 로그인한다

- end user: 자신의 identity를 어떤 사이트에 밝히고자 하는 사람.
- ID (identifier): 최종 사용자가 그들의 OpenID 아이디로 선택한 URL 이나 XRI
- ID 제공자 (identity provider): OpenID URL 또는 XRI 등록을 제공하고 OpenID 인증 (추가적으로 다른 identity 서비스도) 을 제공하는 서비스 제공자
- relying party: 최종 사용자의 ID를 검증하고자 하는 사이트
- server or server-agent: 최종 사용자의 ID를 검증해주는 서버로, 최종 사용자의 자체 서버 (블로그 등) 또는 ID 제공자에 의해 운영되는 서버가 될 수 있다
- user-agent: 최종 사용자가 ID 제공자나 relying party에 접속할 때 사용하는 (브라우저 등) 프로그램.

![](http://assets.devx.com/devx/0804rgf3.jpg)

1. 사용자가 사이트A에 접근
2. 사용자가 openID로 로그인 요청
3. 사이트A에서 사용자를 OpenID 제공자 사이트로 리다이렉트 시킴
4. 사용자가 openID 제공자 사이트에서 로그인
5. openID 제공자가 사용자를 식별하고 사용자 정보 전달
6. 사용자가 사이트A 접근 (필요하면 추가 정보 입력을 요청)

###OAuth
- 인터넷 사용자들이 비밀번호를 제공하지 않고 다른 웹사이트 상의 자신들의 정보에 대해 웹사이트나 애플리케이션의 접근 권한을 부여하는 수단으로 쓰는 표준
- OAuth는 인증 프로토콜이 아닌, 인가 프로토콜이다.

![](https://upload.wikimedia.org/wikipedia/commons/thumb/3/32/OpenIDvs.Pseudo-AuthenticationusingOAuth.svg/512px-OpenIDvs.Pseudo-AuthenticationusingOAuth.svg.png)

1. **Resource Owner**   
	접근 권한을 가진 개체. id/pw를 알고 있는 사용자.
2. **Resource Server**   
	자원을 가진 서버. 접근 토큰을 이용하면 필요한 자원을 응답.
3. **Client**   
	권한을 갖고 resource server에 자원을 요청하는 애플리케이션. (웹, 모바일 등)
4. **Authorization Server**   
	resource owner를 인증하고 client에게 accessToken를 부여하는 서버.

~~~
     +--------+                               +---------------+    
     |        |--(A)- Authorization Request ->|   Resource    |    
     |        |                               |     Owner     |    
     |        |<-(B)-- Authorization Grant ---|               |     
     |        |                               +---------------+    
     |        |    
     |        |                               +---------------+    
     |        |--(C)-- Authorization Grant -->| Authorization |    
     | Client |                               |     Server    |    
     |        |<-(D)----- Access Token -------|               |    
     |        |                               +---------------+    
     |        |    
     |        |                               +---------------+    
     |        |--(E)----- Access Token ------>|    Resource   |    
     |        |                               |     Server    |    
     |        |<-(F)--- Protected Resource ---|               |    
     +--------+                               +---------------+    
~~~

권한을 인가하는 방법에는 4가지가 있다.

1. **Authorization Code**  
	- (A) 단계를 거치지 않고 Resource Owner가 직접 Authorization Server에게 인가하여 Client 에게 Authorization Code를 응답하는 방식.   
	- Client가 사용자 정보를 전혀 모르기 때문에 보안성이 높지만 개발이 복잡하다.   
	- Client는 사용자에게 받은 Authorization Code를 이용하여 토큰을 발급받는다.  
2. **Implicit**
	- Authorization Code에서 간소화된 방식
	- Authorization Code를 받는게 아니라 바로 토큰을 응답받는다.
	- 사용자가 권한 서버에게 토큰을 받아 Client에게 전달하는 방식.
	- 권한 서버가 Client를 인증하지 않기 때문에, 떄로는 redirection_uri를 통해 client id를 확인하기도 한다.
3. **Resource Owner Password Credentials**
	- 사용자의 id/pw를 사용해 token을 발급
	- 사용자가 client를 신뢰하는 경우에만 사용
	- 토큰을 발급받은 후에 사용자 정보를 저장하면 안됨
4. **Client Credentials**
	- Client = Resource Owner일 경우 사용
	- 클라이언트가 자신을 입증하는 id/secret을 제출하면 토큰을 발급받는다
	- 자신의 자원을 가진 서비스/서버에서 요청하는 경우
