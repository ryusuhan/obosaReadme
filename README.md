# Project

SSAFY Final Project 개인 중고 물품 경매 사이트 'OBOSA'



## Project Spec

WEB FrontEnd : Vue.js, Vuetify, Vuex

BackEnd : Java Springboot 2.1.9 RELEASE
	자바 버전 :  JAVA 8
	ORM 기술 : Hibernate JPA



## 서버 정보

|             |               역할                | host | Domain |
| :---------: | :-------------------------------: | :--: | :----: |
| **AWS EC2** |      Web Server & Reverse Proxy(Nginx)     |   ubuntu@13.124.253.169   |    obosa.ssafy.io    |
|             |      API 어플리케이션 서버 1      |   ubuntu@13.125.166.156	   |        |
|             | API 어플리케이션 서버 2 |   ubuntu@52.78.137.53   |        |
|             | Redis Server |   ubuntu@15.164.234.157   |        |
| **AWS RDS** |          MySQL DB Master Server          |   obosa-master.cpcceaqwm3sy.ap-northeast-2.rds.amazonaws.com   |        |
|             |          MySQL DB Slave Server1          |   obosa-slave1.cpcceaqwm3sy.ap-northeast-2.rds.amazonaws.com   |        |
| **AWS S3**  |       Remote 파일 스토리지        |   https://s3.console.aws.amazon.com/s3/buckets/obosa/   |        |

 

## 배포 단계(1차 스펙)

#### 1. ec2 접속

```cmd
ssh -i "keyfile" ubuntu@13.124.253.169 
```



#### 2. 파일전송

```cmd
scp -i "keyfile" "전송할 파일" ubuntu@13.124.253.169 :~/
```



#### 3. jar 실행

```cmd
java -jar "파일명"
```



#### 4. ec2에서 실시간 로그 보기

~~~cmd
tail -f "filename"
~~~



## 패키지 설명

|           패키지명           |                         간략한 설명                          |
| :--------------------------: | :----------------------------------------------------------: |
|  com.ssafy.obosa.annotation  |                   Custom Annotation을 저장                   |
|     com.ssafy.obosa.aop      | 어노테이션의 포인트컷 적용 시점(CRUD 중 R을 뺀 나머지에 적용하기 위해 사용) <br/>관점 지향 프로그래밍 도입 |
|    com.ssafy.obosa.config    |       AWS S3, Redis, Swagger, Jpa, Mvc 관련 설정 파일        |
|  com.ssafy.obosa.controller  |                     컨트롤러 관련 클래스                     |
| com.ssafy.obosa.enumeration  | Status Code, Response Message, Jwt 만료 기간 관련 Enum 클래스 |
|    com.ssafy.obosa.model     |                   dto, 도메인 관련 클래스                    |
| com.ssafy.obosa.registration |                       email 인증 관련                        |
|  com.ssafy.obosa.repository  |                  jpa repository 관련 클래스                  |
|   com.ssafy.obosa.service    |              Business Logic 관련 Service 클래스              |
|     com.ssafy.obosa.util     | 단방향/양방향 암호화를 위한 Utility 클래스 <br />AWS S3관련 Utility 클래스<br /> ImgHandler Component 클래스 |
|  com.ssafy.obosa.validation  |               Backend 유효성 검사 관련 클래스                |




---



## JWT

WEB/Mobile APP 개발을 하면 로그인 과정에서 반드시 **Cookie-Session**개념을 마주치게 된다.

이는 사용자 인증과 관련하여 기존의 방식들이다.

두가지 방식은 모두 널리 알려진 문제점이 존재하는데 대표적인 문제를 설명해보자면

1.  서버 확장시 **세션 정보의 동기화 문제**가 있다. 세션 정보가 동기화되지 않기 때문에 사용자는 인증을 진행하고도 인증되지 않았다는 응답을 받을 수 있다.

2.  **서버 / 세션 저장소의 부하**이다.
세선을 각 서버에 저장하지 않고 세션 전용 서버, DB에 저장한다고 해도 문제가 발생할 수 있다.
모든 요청 시에 해당 서버에 조회해야 한다. **DB에 부하**가 발생할 수 있다.

3.  **WEB/APP간의 Cookie-Session 처리 로직의 차이**가 있을 수 있다는 것이다.
기존의 Client는 WEB Browser만 있었지만 이제는 Mobile도 고려해야한다.
WEB과 Mobile의 처리 방식이 다르고 또 다른 Client가 생겨나면 또 다른 처리 방안을 고려해야한다.

이러한 문제점을 해결할 수 있는 최선의 방안으로 찾은 것이 **Token 기반의 통신**이다.

토큰은 서버의 상태를 저장하지 않고 Self-Contained이기 때문에 별도의 인증서버가 필요하지 않다. 또한 JSON 포맷으로 통신하기 때문에 Client 별 처리 방식과 별개로 Data 통신에 사용할 수 있다.



#### JWT's Architecture

```
AAA(Header).BBB(Payload).CCC(Signature)
```

의 구성으로 이루어져 있으며 각각을 설명하면

**Header**에는 토큰의 타입과 해싱 알고리즘을 포함한다.

**Payload**에는 실제 토큰으로 사용하려는 데이터가 담긴다. 각 데이터를 Claim이라고 한다.

**Signature**에는 Header와 Payload를 합친후 Secret키와 함께 Header의 해싱 알고리즘으로 해싱한 결과를 담는다.

각각의 JSON형태의 데이터를 **base64 인코딩** 후 .을 이용하여 합치면 하나의 JWT Token이 완성된다.



#### JWT 통신 과정

![obosa aesEncryption](https://obosa.s3.ap-northeast-2.amazonaws.com/obosa/readme/jwt.png)

**JWT의 기본적인 통신 과정**이다.

우리는 여기서 한단계 더 보안에 신경쓰기 위하여 만료 기한이 길지만 자원 접근 권한이 없는**Refresh Token**과 만료 기간이 짧지만 자원 접근 권한을 가지는 **AccessToken**을 함께 활용하는 방식을 사용하였다.



#### JWT의 단점 & 도입시 고려사항

1. Self-Contained : 토큰이 자체적으로 정보를 가진다는 것 자체로 문제가 발생할 수 있다.
   1-1. 토큰 자체 payload에 Claim Set을 저장하기 때문에 정보가 많아지면 길이가 늘어나 **네트워크 부하**를 초래할 수 있다.
   1-2. payload 부분에 정보를 넣는데 이는 암호화되지 않고 base64로 인코딩했을 뿐인 데이터이기 때문에 JWT를 탈취하게 되면 안에 있는 정보를 가져갈 수 있다.
   --> 이를 방지하기 위해 민감하지 않은 정보만 담아서 통신하도록 개발하였다.
2. 토큰은 Client Side에서 관리해야하기 때문에 **토큰을 저장해야 한다.**




---




## Encryption

#### - Symmetric Key Encryption(AES256)

**AES256** 암호화는 **Advanced Ecryption Standard**의 약자로 대칭키를 사용하는 블럭 암호이며 **Rijndael(레인달) 알고리즘**을 가지고 만들어졌다.

**높은 안전성**과 **속도**로 인기를 얻어 많이 사용되고 있는 암호화로 미국 정보가 기밀정보를 암호화할 때 사용했다고 하는 신뢰성 높은 알고리즘이다. 

해당 프로젝트에서는 비밀번호를 제외한 사용자들의 민감한 개인정보를 암호화하기 위해 도입되었으며, 암호화된 Cipher가 DB에 저장되고 필요한 시기에 복호화하여 Plain Text를 제공한다.

![obosa aesEncryption](https://obosa.s3.ap-northeast-2.amazonaws.com/obosa/readme/aes-encryption.PNG)

Address와 Name은 사용자의 민감한 개인정보로 판단되어 암호화를 통해 저장했다.

Email은 사용자의 id를 대체하여 사용되기 때문에 따로 암호화하지 않았다.

![obosa aesDecryption](https://obosa.s3.ap-northeast-2.amazonaws.com/obosa/readme/aes-decryption.PNG)

정보를 조회하는 경우에는 이렇게 복호화를 통해 제공한다.

#### - One-Way Hashing Encryption(SHA256)

**SHA256**는 단방향 암호화로 해시값을 생성하여 생성된 해시값은 겹치지 않는 해당 Text의 **식별값**이 된다.

하지만 단순히 Plain Text를 해싱하는 방식으로는 Brute Force나 Rainbow Table 공격에 **취약**할 수 있기 때문에 해당 프로젝트에서는 **Salt**를 도입하였다.

랜덤으로 생성된 **Salt**를 암호화할 Password뒤에 붙이고 함께 Hashing한다.

같은 비밀번호라도 Salt값이 다를 경우 **전혀 다른 값**으로 나오게 되므로 이를 통해 비밀번호를 식별한다.

단방향 해싱 암호화의 경우 **처리 속도가 굉장히 빠르기** 때문에 이후에 Salt값을 합쳐서 비밀번호를 확인하는 것에 성능상에 문제를 일으키지 않고 처리될 수 있다.  



---


## DB Replication

DB에 요청되는 Query에는 **Read**가 Write/Delete에 비해 상당히 높은 비율로 발생하게 된다.

과도한 Select Query에 의한 부하를 방지하기 위해 해당 프로젝트에서는 **DB Replication**을 통해 **Master DB**와 여러 개의 Slave DB를 통해 부하를 방지하고자 설계하였다.

Master와 Slave DB들은 동일한 데이터를 저장하고 있으며 Read가 발생할 경우 Slave를 통해 Write/Delete가 발생할 경우에는 Master를 통해 Query를 처리하도록 개발하였다.

현재는 Slave DB가 하나만 존재하고 있지만 확장성을 위해 **CircularList**를 만들어 Slave DB가 늘어났을 경우의 확장성을 고려하여 개발하였다.

```java
public class CircularList<T>
{
    private List<T> list;
    private AtomicInteger counter = new AtomicInteger(0);

    public CircularList(List<T> list)
    {
        this.list = list;
    }

    public T getOne()
    {
        if(counter.get() + 1 >= list.size())
        {
            counter.set(-1);
        }
        return list.get(counter.addAndGet(1));
    }
}
```

위의 CircularList는 List에 Slave DB를 저장하고 counter를 **AtomicInteger**를 사용하여 **병렬 처리**가 발생해야할 경우를 대비하여 개발하였다.

-**어려웠던 점**

**@Transaction(readonly = true) 일 경우에 Slave DB로** 요청을 진행하도록 구성하였고, 이외의 요청들은 Master DB를 통해 처리되도록 구현하였기 때문에 요청에 따라 다른 DB 커넥션이 요구된다. 

Springboot의 구성에서 **DB Configuration을 자동으로 잡아주는 것**이 @SpringBootApplication 어노테이션에 포함되어 있기 때문에 DB 커넥션을 분산하는 것에 어려움을 겪었으나

```java
@SpringBootApplication(exclude = {DataSourceAutoConfiguration.class, HibernateJpaAutoConfiguration.class})
public class ObosaApplication {

    public static void main(String[] args)
    {
            SpringApplication.run(ObosaApplication.class, args);
    }

}
```

위처럼 **exclude**를 통해 자동 Configuration을 제외하고 직접 Connection을 이어주는 로직을 작성하여 해결할 수 있었다.



---



## 아임포트 결제 연동

하나 이상의 PG사와 결제 연동을 시킬 경우 각각 다른 연동 가이드를 참고해 구현해야 하는 어려움이 있습니다. 아임포트 결제 연동 서비스를 사용해서 최소한의 노력으로 결제 연동을 구현할 수 있었습니다.

![Imgur](https://i.imgur.com/RMBTXR6.png)

OBOSA 서비스에서 결제가 진행되는 흐름은 다음과 같습니다.
1. 클라이언트에서 아임포트로 결제 요청
2. 아임포트에서 PG사에 결제 요청
3. PG사(카카오페이)에서 사용자의 카카오톡으로 결제 요청 발송
4. 사용자가 카카오톡에서 카카오페이 결제 버튼 클릭
5. PG사 에서 결제 승인 정보를 아임포트에 응답
6. 아임포트에서 클라이언트에 결제 승인 정보를 확인할 수 있는 토큰을 응답
7. 클라이언트는 서버에 결제 승인 정보를 확인할 수 있는 토큰을 담아 결제 완료 요청 발송
8. 서버는 아임포트에게 토큰을 담아 결제 승인 정보 요청
9. 아임포트가 서버에 토큰에 해당하는 결제 승인 정보 응답
10. 서버는 승인 정보를 확인하고, 결제 완료 상태를 DB(RDS)에 반영 요청
11. DB는 결제 완료 상태 반영 후 서버에 응답
12. 결제 완료 상태 변경이 성공적으로 수행되었다고 클라이언트에 응답

## 이메일 인증

회원 가입 시 사용자에게 받는 개인 정보중 서비스에서 가장 활용도가 높은 항목은 이메일이다. 서버 내부에서 사용자를 유일하게 식별해내는 데 사용 되기도 하고 ( e.g., `findUserByEmail()` ) 각종 인증 및 정보 메일을 발송하는 데도 사용된다. 이렇게 중요한 이메일 정보는 다음과 같은 인증 과정을 거친다.

1. **클라이언트**에서의 **유효성** 검사
2. **서버**에서의 **유효성** 검사
3. **인증 메일 발송** 후 사용자의 **링크** 클릭으로 실사용중인 이메일인지 검사

1은 사용자의 실수로 잘못된 이메일이 담긴 회원가입 요청이 불필요하게 네트워크를 낭비하는 것을 방지하는 것이고,
2는 클라이언트를 통하지 않은 다른 툴로 서버에 직접 HTTP 요청을 날릴 때 잘못된 형식의 이메일이 들어오는 것을 방지한다.

1, 2는 이메일의 형식이 올바른지를 검사해 주는 부분이다. 서비스가 필요로 하는 것은 사용자와 소통이 가능한 살아있는 이메일이기 때문에 1, 2를 통과하지 못했더라도 3을 통과하는 것이 중요하다.

링크를 통한 인증 과정은 다음과 같다.

1. 클라이언트에서 회원 가입 요청 (input 에 입력된 email 형식 검사)
2. 서버에서 SignupDTO를 통해 사용자 정보를 받는다.
3. 서버 측에서 email 형식 검사
4. SignupDTO로 받은 정보를 토대로 User 객체 구성
5. User 의 field 중 이메일 인증 여부를 나타내는 state를 false로 초기화
6. DB에 User 객체 저장
7. 회원 가입이 완료 되었음을 알리는 RegistrationCompleteEvent를 publish 한다.
8. RegistrationCompleteEvent 를 listening하는 RegistrationListener에서 24시간의 유효기간을 가지는 VerificationToken을 생성한다.
9. 생성한 토큰으로 인증 링크를 생성해서 사용자의 이메일로 인증 메일을 발송

![image](uploads/238b117d6233685b5b8ecdfe2ef57f1f/image.png)

위와 같은 메일이 온 것을 확인할 수 있다.

![image](uploads/e9c04a3f3078b19df2a2467668c2474b/image.png)

메일 본문은 인증링크를 포함하고 있다.

10. 사용자가 인증링크를 클릭
11. 서버에서 링크에 포함된 토큰이 유효한지를 검사한다
12. 토큰의 만료기간이 지나지 않았다면 토큰이 담고 있는 이메일 정보로 User 테이블에서 User를 찾은 뒤 state를 true 로 설정한다. 



---



## Logback

**Logback**은 "자바 오픈소스 로깅 프레임워크"로 SLF4J의 구현체이자 **스프링 부트의 기본 로그** 객체다.

log4j, log4j2, JUL(java.util.logging)과 성능을 비교했을 때 logback은 훌륭한 성능을 보여준다.

그리고 결정적으로 자바 프로그램에서 로그를 사용할 때 가장 많이 사용되고 있기 때문에 알아두어야 한다.

출처: https://jeong-pro.tistory.com/154 [기본기를 쌓는 정아마추어 코딩블로그]



```xml
<?xml version="1.0" encoding="UTF-8"?>
<configuration>
    <include resource="org/springframework/boot/logging/logback/base.xml"/>

    <appender name="dailyRollingFileAppender" class="ch.qos.logback.core.rolling.RollingFileAppender">
        <prudent>true</prudent>
        <rollingPolicy class="ch.qos.logback.core.rolling.TimeBasedRollingPolicy">
            <fileNamePattern>./log/dowadog.%d{yyyy-MM-dd}.log</fileNamePattern>
            <maxHistory>30</maxHistory>
        </rollingPolicy>
        <filter class="ch.qos.logback.classic.filter.ThresholdFilter">
            <level>DEBUG</level>
        </filter>

        <encoder>
            <pattern>%d{yyyy:MM:dd HH:mm:ss.SSS} %-5level --- [%thread] %logger{35} : %msg %n</pattern>
        </encoder>
    </appender>

    <logger name="org.springframework.web" level="INFO"/>
    <logger name="com.sopt.dowadog" level="INFO"/>
    <logger name="org.hibernate.sql" level="ERROR"/>

    <root level="INFO">
        <appender-ref ref="dailyRollingFileAppender" />
    </root>
</configuration>
```

* maxHistory : 30일
* Logging 수준 : Debug 이상만 세팅(전역), 각 패키지별 Logging 수준 별도 세팅




---



## Nginx
#### -Port Forwarding & Https 적용
**letsencrypt**를 통하여 편리하게 발급받은 무료 SSL인증서를 **Nginx에 적용**시키고 이를 통해 Https가 적용되도록 설정하였다.
server Block내에 80포트를 listen하고 있을 때 들어오는 요청을 443포트로 포워딩하여 https로 이동하도록 구성하였고, 내부의 location 설정을 통하여 **/로 이동할 경우** Frontend 서버가 구성되어 있는 8111포트로 이동하도록 포워딩하고, **uri가 /api를 포함하여 구성되는 경우** 비지니스 로직을 가진 API 서버로 이동할 수 있도록 load Balancing을 적용하였다.

**https**를 적용함으로서 WEB에서 기본적으로 챙겨야하는 보안성을 구성하였다.

#### -load-balancing
Nginx는 road-balancing을 간단한 방법으로 제공해주는데 upstream이라는 옵션을 config파일 안에 넣어주는 것으로 설정할 수 있다. 
이러한 방식은 **default로 Round-Robin 방식**을 사용하여 load-balancing해주게 된다. 
upstream에 우리가 가지고 있는 분산할 서버의 도메인 두개를 넣어주고 각각 마다 weight나 기타 balancing 설정들을 넣어줄 수 있다. least_conn이라는 설정을 넣어줄 경우 현재 트래픽이 가장 적은 서버로 매핑해주는 것으로 보이는데 우리는 **least connection방식**을 사용하였고, main, sub서버가 구분되어 있는 것이 아니기 때문에 따로 가중치를 추가적으로 주지는 않았다.
ip_hash 설정을 추가하여 **Session Stikiness**를 통해 동일한 IP의 경우 통신했던 서버와 계속 통신할 수 있도록 설정하였다.  
upstream에 위의 설정을 잡아준 뒤 config 파일내의 server block에서 proxy_pass로 upstream 옵션이름을 지정해주는 것으로 설정을 마무리했고, load-balancing을 적용할 수 있었다.

#### 로드밸런싱을 했을 때 얻을 수 있는 장점
1. 이렇게 서비스를 할 때 로드 밸런서를 두고 서버를 두개 놓고 서비스를 하게 되면 얻을 수 있는 가장 큰 이득은 **중단 없는 재배포**가 가능하다는 점이다.
   2대의 서버중 1대가 작동을 멈추어도 **서비스를 온전히 계속 제공**할 수 있으므로 재배포나 유지 보수부분에서 얻을 수 있는 것이 많다.
2. 저가의 서버를 여러대 이용해서 고가의 서버를 사용하는 것처럼 사용할 수 있다는 것도 장점이 될 수 있을 것이다.
3. 또한 혹여 추후 사용량이 많아서 서버를 확장해야하는 경우가 발생해도 서비스를 중단하지 않고 확장할 수 있는 **유연한 확장성**을 얻는 다는 것이다.


---



## 최종 인프라 다이어그램 모델 구상도

![obosa architecture]( https://obosa.s3.ap-northeast-2.amazonaws.com/obosa/obosa+Structure.png)
