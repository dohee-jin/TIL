## TIL - 2025.12.04

### 🔍 오늘 배운 내용

#### AWS 클라우드 수동 배포 - 프론트엔드

#### 환경 변수 파일 분리
.env : npm run dev 실행 시 사용, Vite 프록시를 통해 로컬 백엔드와 통신
.env.production:  npm run build 실행 시 사용, 운영용 환경 변수 파일
```
# =================================
# API 서버 설정 (배포 환경)
# =================================

VITE_API_BASE_URL=https://api.dialogym.site


# =================================
# WebSocket URL (대화 기능용)
# =================================

VITE_WS_URL=wss://api.dialogym.site/ws

VITE_APP_ENV=production
VITE_USE_DYNAMIC_HOST=false
```

#### apiClient.jsx    
axios 설정 부분 수정   
import.meta.env.VITE_API_/...  
env파일로부터 읽어오도록 수정  

#### 배포용 프론트엔드 파일 생성  
```
npm run build 
```  
dist 파일 생성되었는지 확인   
dist > assest > index~~~.js 들어가서 내 도메인 검색   
도메인이 제대로(=env가 제대로 읽혔는지) 반영되었는지 확인   

#### s3 버킷 생성

기존 dialogym 프로젝트 배포 시 OpenAI API 키 노출 되는 문제 해결
프론트엔드 ci/cd 워크플로우 내 깃허브 시크릿으로 AI 키 주입받는 부분 삭제
프론트엔드 레포 > 세팅 > 깃허브 시크릿에 등록되어 있는 OpenAI API 키 삭제


ec2 다시 실행 후 도메인 접속 안되는 문제 발생
탄력적 IP 할당받아서 문제 해결해 보려고함


