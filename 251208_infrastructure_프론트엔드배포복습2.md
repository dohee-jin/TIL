## TIL - 2025.12.08

### 🔍 오늘 배운 내용

#### AWS 클라우드 수동 배포 - 프론트엔드

#### s3 버킷 
버킷이름은 보통 배포용 도메인 이름으로 생성한다.
![alt text](image-22.png)

버킷 생성 화면에서 버킷의 퍼블릭 엑세스는 차단으로 설정한다(기본 설정, 수정하지 않아도 됨)
![alt text](image-23.png)
해당 설정은 cloudFront로 우회할 예정이라 기본 설정으로 두면 된다.

버킷은 이름만 설정하고 기본설정 변경없이 생성하면 된다.

![alt text](image-24.png)   
![alt text](image-25.png)   
![alt text](image-33.png)
업로드를 누르고 프론트 npm run build 해서 빌드한 dist 안의 assets, index.html, vite.svg 파일을 업로드 한다.

버킷 > 속성 > 정적 웹사이트 호스팅 > 활성화
![alt text](image-34.png)  
![alt text](image-35.png)   
![alt text](image-36.png)
s3 설정 끝

#### https 연결, cloudFront용 ssl/tls 인증서 발급(프론트용 인증서 발급)
![alt text](image-37.png)
couldFront용 인증서 발급할 때는 리전이 버지니아 리전이어야 함   
![alt text](image-38.png)   

![alt text](image-39.png)
이번 인증서 도메인 이름은 *.~~~ 이 아닌 실제 사용할 메인 도메인 이름으로 받는다.
추가 도메인은 *.~~~ 으로 설정
나머지 설정은 기본으로 두고 인증서 요청을 한다.

![alt text](image-40.png)   
인증서를 발급받았으면 route53에서 도메인 소유권을 검증한다.
![alt text](image-41.png)   
이때는 route53에서 레코드 생성하고 넘어간 페이지에 아무것도 뜨지 않는다. 화면 확인 후 다시 원래 페이지도 이동한다.  

![alt text](image-42.png)
원래 페이지에서 인증서 상태가 발급됨 으로 변경되었는지 확인한다.


#### couldFront 배포  
![alt text](image-43.png)  
![alt text](image-44.png)
specify origin    
![alt text](image-45.png)  
소스를 어디서 가져오는지 설정 > s3로 설정   
![alt text](image-46.png)   
자동으로 설정되어 있지만, Browse S3 눌러서 확인한다.
다른 옵션들은 기본으로 설정하고 다음으로 이동

waf 설정   
![alt text](image-47.png)
waf 설정(방화벽 설정) 은 비활성화로 선택   
원래는 활성화 하는 것이 보안적 측면에서 좋다.

인증서 선택   
![alt text](image-48.png)
버지니아에 만들어 둔 인증서가 있기 때문에 바로 선택되어있다.

![alt text](image-49.png)
배포 설정이 완료되면 이상한 배포 도메인이 뜬다. 
![alt text](image-50.png) 
해당 도메인에 접속하면 사이트에 연결 할 수 없다고 뜬다.

사용자 정의 오류 응답 설정   
![alt text](image-51.png)
오류 페이지로 이동해서 사용자 정의 오류 응답 설정을 해야한다.  
![alt text](image-52.png)
403을 선택하고 customize error response > /index.html   

![alt text](image-53.png)
s3에 호스팅을 설정하면서 퍼블릭 엑세스를 차단하고 getObject 정책 설정도 하지 않아서 발생한 문제로 s3 버킷 엔드포인드에 접속하면 403 에러가 뜬다.   

사용자 정의 오류 설정을 통해 클라우드프론트가 
403 뜨면 강제로 index.html을 열어주도록 설정하는 작업이다.   

설정을 완료하고 CloudFront 엔드포인트로 접속하면 다음과 같은 프론트엔드 화면을 확인할 수 있다.
![alt text](image-54.png) 

마지막으로 route53에서 도메인과 cloudFront 연결 설정을 완료하면 프론트엔드 + 백엔드 배포가 완료된다.

#### route53에서 cloudFront + 도메인 연결 설정  
![alt text](image-55.png)
새 레코드를 생성한다.
![alt text](image-56.png)
여기서는 서브도메인을 사용하지 않고 메인 도메인 dialogym.site로 작성한다.   

alias 설정
![alt text](image-57.png)
Alias to CloudFront distribution  
CloudFront 배포에 대한 별칭을 선택한다.   

![alt text](image-58.png)
레코드 생성이 완료되면 프론트엔드 배포가 완료된다.   

도메인으로 접속되는 것을 확인한 후 API 테스트를 진행한다.   
![alt text](image-59.png) 
CRUD도 문제없이 작동하여 프론트엔드 + 백엔드 수동배포를 완료했다.   

배포 아키텍처는 아래와 같다.
```mermaid
graph TB
    Users[사용자]
    
    subgraph CDN["CloudFront (CDN)"]
        CF[CloudFront Distribution]
    end
    
    subgraph Storage["S3"]
        S3[S3 Bucket<br/>정적 파일 저장소<br/>HTML/CSS/JS/Images]
    end
    
    subgraph LoadBalancer["Application Load Balancer"]
        ALB[ALB<br/>트래픽 분산<br/>Health Check<br/>SSL/TLS 종료]
    end
    
    subgraph EC2Cluster["EC2 인스턴스"]
        EC2_1[EC2 Instance 1<br/>백엔드 API]
        EC2_2[EC2 Instance 2<br/>백엔드 API]
        EC2_N[EC2 Instance N<br/>백엔드 API]
    end
    
    Users -->|HTTPS 요청| CF
    CF -->|정적 콘텐츠| S3
    CF -->|API 요청| ALB
    ALB --> EC2_1
    ALB --> EC2_2
    ALB --> EC2_N
```
#### 수정해야 할 문제
1. OAUTH URI 경로 변경
2. gpt realtime-preview 지원 중단으로 모델명(gpt-realtime-preview -> gpt-realtime) 변경 필요