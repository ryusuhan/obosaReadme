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
<br />

## 최종 인프라 다이어그램 모델 구상도

![obosa architecture]( https://obosa.s3.ap-northeast-2.amazonaws.com/obosa/obosa+Structure.png)
