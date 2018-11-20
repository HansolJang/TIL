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
- xml 형식의 언어인데 SSO를 구현하는 데에도 쓰인다
- 


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
