# Frontend CI/CD

React · GitHub Actions · AWS EC2 + Nginx 기반 프론트엔드 CI/CD 자동 배포 실습 저장소입니다.  
Spring Boot 백엔드와 연동하는 인증 흐름(로그인·회원가입·권한 분기)을 구현하고,  
main 브랜치 push 시 GitHub Actions가 자동으로 빌드 → EC2 업로드 → Nginx 재가동까지 처리합니다.

---

## 기술 스택

![React](https://img.shields.io/badge/React_18-61DAFB?style=flat&logo=react&logoColor=black)
![React Router](https://img.shields.io/badge/React_Router_v6-CA4245?style=flat&logo=reactrouter&logoColor=white)
![Axios](https://img.shields.io/badge/Axios-5A29E4?style=flat&logo=axios&logoColor=white)
![JavaScript](https://img.shields.io/badge/JavaScript-F7DF1E?style=flat&logo=javascript&logoColor=black)
![Node.js](https://img.shields.io/badge/Node.js_22-339933?style=flat&logo=nodedotjs&logoColor=white)
![GitHub Actions](https://img.shields.io/badge/GitHub_Actions-2088FF?style=flat&logo=githubactions&logoColor=white)
![Nginx](https://img.shields.io/badge/Nginx-009639?style=flat&logo=nginx&logoColor=white)
![AWS EC2](https://img.shields.io/badge/AWS_EC2-FF9900?style=flat&logo=amazonec2&logoColor=white)

---

## 프로젝트 구조

```
project-cicd-frontend/
├── .github/
│   └── workflows/
│       └── frontend.yml     # GitHub Actions CI/CD 파이프라인
├── public/
│   ├── index.html           # 단일 페이지 진입점
│   └── ...
└── src/
    ├── App.js               # 최상위 컴포넌트 (레이아웃 프레임)
    ├── index.js             # BrowserRouter 래핑 · 렌더링 진입점
    └── components/
        ├── Basic.jsx        # 전체 Routes/Route 등록
        ├── Login.jsx        # 로그인 폼 — 인증 없이 접근 가능
        ├── Signup.jsx       # 회원가입 폼 — 인증 없이 접근 가능
        ├── Home.jsx         # 홈 — 로그인 후 이메일·권한 표시
        ├── MyInfo.jsx       # 마이페이지 — 사용자 상세 정보 조회
        └── Admin.jsx        # 관리자 페이지 — ADMIN 권한 확인 후 진입
```

---

## 실행 방법

```bash
# 의존성 설치
npm install

# 개발 서버 실행 (http://localhost:3000)
npm start
```

> 백엔드(Spring Boot)가 **포트 8080**에서 먼저 실행 중이어야 합니다.  
> `package.json`의 `"proxy": "http://localhost:8080"` 설정으로 API 요청을 자동으로 백엔드로 전달합니다.

### 환경 변수 설정

프로덕션 배포 시 `.env` 파일에 백엔드 API 주소를 설정합니다.

```
REACT_APP_API_URL=http://<EC2_PUBLIC_IP>:8080
```

---

## CI/CD 파이프라인

`main` 브랜치에 push 또는 PR이 생성되면 아래 순서로 자동 배포됩니다.

```
push / PR to main
    │
    ▼
[GitHub Actions]
    ├── 1. Checkout 소스
    ├── 2. .env 환경 변수 동적 생성 (Secrets.CLIENT_ENV)
    ├── 3. Node.js 22 설치
    ├── 4. npm install
    ├── 5. npm run build  →  ./build 생성
    ├── 6. SCP → EC2 /home/ubuntu/client 업로드
    └── 7. SSH 접속
            ├── /var/www/html/* 삭제
            ├── build/* → /var/www/html/ 이동
            └── nginx 재가동
```

### GitHub Secrets 필요 항목

| Secret 키 | 설명 |
|-----------|------|
| `EC2_HOST` | EC2 퍼블릭 IP 또는 도메인 |
| `EC2_USER` | EC2 SSH 접속 사용자 (예: `ubuntu`) |
| `EC2_KEY` | EC2 SSH 개인 키 (PEM 파일 내용) |
| `CLIENT_ENV` | 프론트엔드 `.env` 파일 내용 전체 |

---

## 학습 내용

| 구분 | 주요 학습 내용 |
|------|---------------|
| **React 기본** | 함수형 컴포넌트, useState · useEffect Hook, JSX 작성 |
| **라우팅** | React Router v6 — `<BrowserRouter>`, `<Routes>/<Route>`, `useNavigate`, `useLocation` |
| **HTTP 통신** | Axios 인스턴스 설정, `baseURL` 환경변수 주입, `withCredentials`로 세션 쿠키 전송 |
| **인증 흐름** | 로그인 → 세션 쿠키 발급 → 보호 라우트 진입 → 로그아웃(세션 삭제) |
| **권한 분기** | ADMIN 권한 여부를 서버 응답으로 확인 후 UI 조건부 렌더링 |
| **쿠키 처리** | `js-cookie` / `react-cookie` 모듈을 통한 클라이언트 쿠키 관리 |
| **상태 전달** | `navigate('/home', { state: {...} })` — 페이지 이동 시 데이터 전달 |
| **CI/CD** | GitHub Actions로 빌드 자동화, SCP로 EC2 파일 전송, SSH로 Nginx 재가동 |
| **Nginx 서빙** | 정적 빌드 파일(`build/*`)을 `/var/www/html/`에 배포하여 Nginx로 서빙 |
| **환경 분리** | 개발 환경 proxy 설정 vs 프로덕션 `.env` 기반 API URL 분리 |

---

## 개발 환경

- **Node.js 22** 이상 — [다운로드](https://nodejs.org/)
- **npm** (Node.js 설치 시 포함)
- **VS Code** (추천) / WebStorm

### 백엔드 연동 전제 조건

- Spring Boot 백엔드 서버 실행 중 (포트 8080)
- 백엔드에 CORS 설정 또는 `withCredentials` 대응 세션 처리 구현 필요
- Docker로 MySQL 컨테이너 실행 (백엔드 연동 시)

```bash
docker run -d -p 3306:3306 --name mysql \
  --env MYSQL_USER=ai \
  --env MYSQL_PASSWORD=1234 \
  --env MYSQL_ROOT_PASSWORD=1234 \
  mysql
```
