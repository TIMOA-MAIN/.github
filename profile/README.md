<h1 align="center">TIMOA</h1>
<p align="center">
  <b>Team Integrated Management Organization Assistance</b><br/>
  팀 단위 실시간 협업 플랫폼
</p>

<p align="center">
  <img src="https://img.shields.io/badge/Node.js-339933?style=flat-square&logo=nodedotjs&logoColor=white"/>
  <img src="https://img.shields.io/badge/Java-007396?style=flat-square&logo=openjdk&logoColor=white"/>
  <img src="https://img.shields.io/badge/Spring_Boot-6DB33F?style=flat-square&logo=springboot&logoColor=white"/>
  <img src="https://img.shields.io/badge/React-61DAFB?style=flat-square&logo=react&logoColor=black"/>
  <img src="https://img.shields.io/badge/Electron-47848F?style=flat-square&logo=electron&logoColor=white"/>
  <img src="https://img.shields.io/badge/TypeScript-3178C6?style=flat-square&logo=typescript&logoColor=white"/>
  <img src="https://img.shields.io/badge/AWS_S3-569A31?style=flat-square&logo=amazons3&logoColor=white"/>
</p>

<p align="center">
  <a href="https://www.timoa.app">🌐 www.timoa.app</a>
</p>

---

## 프로젝트 소개

TIMOA는 개발 팀을 위한 올인원 협업 환경입니다.  
**Visual Studio Live Share · Discord · Notion · Trello**의 핵심 기능을 하나의 데스크탑 애플리케이션으로 통합하여,  
팀원들이 창 전환 없이 코딩·소통·문서화·프로젝트 관리를 동시에 수행할 수 있습니다.

| 통합 대상 | TIMOA 제공 기능 |
|---|---|
| Visual Studio Live Share | Y.js CRDT 기반 실시간 코드 동시 편집 |
| Discord | 채널 채팅 + MediaSoup SFU 음성 통화 |
| Notion | Notionblock.js 기반 블록 문서 협업 |
| Trello / Jira | WBS · 칸반보드 대시보드 |

---

## 아키텍처

```mermaid
flowchart LR
  subgraph CLIENT["🖥️ Desktop Client · Electron + React"]
    direction TB
    C_AUTH["<b>Auth</b><br/>로그인 · 회원가입 · 인증"]
    C_SRV["<b>Server</b><br/>팀 서버 생성/수정/삭제 · 멤버 · 초대 코드"]
    C_DASH["<b>Dashboard Channel</b><br/>WBS · Kanban"]
    C_DOC["<b>Document Channel</b><br/>블록 편집 · PDF 내보내기"]
    C_CHAT["<b>Chat Channel</b><br/>채팅 · 파일 업/다운로드 · 멘션"]
    C_VOICE["<b>Voice Channel</b><br/>mediasoup-client"]
    C_CODE["<b>Code-Editor Channel</b><br/>프로젝트/파일 CRUD · 휴지통 · 실시간 동기화"]
    C_IDE["<b>TIMOA IDE</b> · Eclipse Theia<br/>컴파일 · 인텔리센스 · 다이어그램 플러그인"]
  end

  subgraph PROXY["🔀 Proxy Server · api.timoa.app"]
    direction TB
    P_REST["<b>API Gateway</b><br/>/proxy/{service}"]
    P_SSE["<b>SSE Routing</b><br/>/proxy/sse-{service}"]
    P_WS["<b>WebSocket Gateway</b><br/>Upgrade 중계"]
    P_LB["헬스체크 5s · Round-robin LB<br/>/proxy/user 공개 · 나머지 인증 경로"]
  end

  subgraph SERVER["⚙️ Services"]
    direction TB
    S_USER["<b>User Server</b> · Spring Boot<br/>회원 · 팀 서버 · 멤버 · JWT 발급"]
    S_DASH["<b>Dashboard Server</b> · Spring Boot<br/>WBS · Kanban · Excel 내보내기"]
    S_CHAT["<b>Chat Server</b> · Spring Boot<br/>메시지 · 멘션 · 첨부파일"]
    S_DOC["<b>Document Server</b> · Node.js<br/>문서 채널 · 블록 CRDT (Y.js)"]
    S_CODE["<b>Code Server</b> · Node.js<br/>프로젝트/파일 · 휴지통 · CRDT (Y.js)"]
    S_VOICE["<b>Voice Server</b> · MediaSoup SFU<br/>팀 서버별 Router · 참여자 상태"]
    S_DIAG["<b>Diagram Server</b><br/>코드 → 클래스 다이어그램 생성"]
  end

  subgraph DATA["🗄️ Data"]
    direction TB
    DB[("<b>MS SQL</b> · 공유 DB<br/>사용자 · 팀 서버/멤버 · 채팅<br/>대시보드 · 문서 채널/블록 · 코드 프로젝트/파일")]
    S3[("<b>AWS S3</b><br/>프로필 이미지 · 채팅 첨부 · 대용량 파일")]
  end

  %% Client → Proxy
  CLIENT -->|REST| P_REST
  CLIENT -.->|SSE| P_SSE
  CLIENT ==>|WebSocket| P_WS
  C_CODE -.->|IPC 실행| C_IDE
  C_IDE -->|Plugin · /proxy/diagram| P_REST

  %% Proxy → Services
  P_REST --> S_USER & S_DASH & S_CHAT & S_DOC & S_CODE & S_DIAG
  P_SSE -.-> S_USER & S_DASH & S_CHAT & S_DOC & S_CODE
  P_WS ==>|"Y.js · /collaboration/:room"| S_CODE
  P_WS ==>|"Y.js · /document/..."| S_DOC
  P_WS ==>|"Socket.IO · /voice"| S_VOICE
  C_VOICE <-->|"WebRTC · UDP/TCP 10000-10100"| S_VOICE

  %% Services → Data
  S_USER & S_DASH & S_CHAT & S_DOC & S_CODE & S_VOICE --> DB
  S_USER & S_CHAT --> S3

  %% 0-1 REST/SSE, 2 WS, 3 IPC, 4 Plugin
  linkStyle 0 stroke:#e75480,stroke-width:2px
  linkStyle 1 stroke:#2e9e5b,stroke-width:2px
  linkStyle 2 stroke:#2f6fdf,stroke-width:3px
  linkStyle 3 stroke:#888,stroke-width:1px
  linkStyle 4 stroke:#8e44ad,stroke-width:2px
  %% 5-10 REST, 11-15 SSE, 16-18 WS, 19 WebRTC
  linkStyle 5,6,7,8,9,10 stroke:#e75480,stroke-width:2px
  linkStyle 11,12,13,14,15 stroke:#2e9e5b,stroke-width:2px
  linkStyle 16,17,18 stroke:#2f6fdf,stroke-width:3px
  linkStyle 19 stroke:#2f6fdf,stroke-width:2px,stroke-dasharray:4
  %% 20-25 SQL, 26-27 S3
  linkStyle 20,21,22,23,24,25 stroke:#e67e22,stroke-width:2px
  linkStyle 26,27 stroke:#c0392b,stroke-width:2px

  classDef planned stroke-dasharray:5 5
  class S_DIAG planned
```

| 선 | 프로토콜 | 사용처 |
|---|---|---|
| 🩷 실선 | **REST** | 모든 CRUD 요청 (`/proxy/{service}`), 파일 업로드 (`/chat-upload`, `/user-upload`) |
| 💚 점선 | **SSE** | 서버 → 클라이언트 변경 알림 (User · Dashboard · Chat · Document · Code) |
| 💙 굵은 선 | **WebSocket** | Y.js CRDT 동기화 (Code · Document), Socket.IO 시그널링 (Voice) |
| 💙 점선 | **WebRTC** | 음성 미디어. Proxy를 거치지 않고 SFU로 직접 전송 |
| 🧡 | **SQL** | 단일 MS SQL 공유 DB ([ADR 0003](https://github.com/TIMOA-MAIN/TIMOA_Docs/blob/main/decisions/architecture/0003-shared-db.md)) |
| ❤️ | **S3** | User(프로필 이미지 · 파일), Chat(첨부파일) |
| 💜 | **Plugin** | TIMOA IDE 플러그인 → Diagram Server (`/proxy/diagram`) |

> Diagram Server는 Proxy에 라우팅(`/diagram`)만 등록되어 있고 이 조직에 레포가 없어 점선 테두리로 표시했습니다.

MSA 지향 구조로 서비스별 독립 기술 스택을 선택했습니다.  
각 기술 결정의 근거는 **[TIMOA_Docs](https://github.com/TIMOA-MAIN/TIMOA_Docs)** 에 ADR 형식으로 기록되어 있습니다.

---

## 주요 기술 결정

| 결정 | 선택 | 핵심 이유 |
|---|---|---|
| 실시간 동시 편집 | Y.js CRDT | 충돌 없는 병합 수학적 보장 |
| 음성 통화 | MediaSoup SFU | 다중 사용자 확장성·방화벽 안정성 |
| 채팅·대시보드 실시간 | REST + SSE | 단방향 푸시에 최적, WebSocket 대비 서버 부하 낮음 |
| Reverse Proxy | Node.js (자체 구현) | 헬스체크 기반 런타임 동적 라우팅 |
| 코드 에디터 | Eclipse Theia | VS Code Extension 생태계 활용, 언어 제약 없음 |
| 파일 스토리지 | AWS S3 | 정형/비정형 데이터 분리, DB 용량 최적화 |

---

## 서비스 구성

| 레포지토리 | 언어 | 역할 |
|---|---|---|
| [Proxy_Server](https://github.com/TIMOA-MAIN/Proxy_Server) | Node.js | 헬스체크·동적 라우팅·WebSocket 중계 |
| [Code_Server](https://github.com/TIMOA-MAIN/Code_Server) | Node.js | Eclipse Theia + Y.js CRDT 코드 편집 |
| [Document_Server](https://github.com/TIMOA-MAIN/Document_Server) | Node.js | Notionblock.js + Y.js CRDT 문서 편집 |
| [Voice_Server](https://github.com/TIMOA-MAIN/Voice_Server) | Node.js | MediaSoup SFU 음성 통화 |
| [Chat_Server](https://github.com/TIMOA-MAIN/Chat_Server) | Java | REST + SSE 채팅, AWS S3 파일 |
| [Dashboard_Server](https://github.com/TIMOA-MAIN/Dashboard_Server) | Java | WBS · 칸반보드, REST + SSE 동기화 |
| [User_Server](https://github.com/TIMOA-MAIN/User_Server) | Java | 사용자 인증·계정 관리 |
| [Desktop_Client](https://github.com/TIMOA-MAIN/Desktop_Client) | TypeScript | Electron 데스크탑 클라이언트 |
| [TIMOA_Docs](https://github.com/TIMOA-MAIN/TIMOA_Docs) | — | 아키텍처 결정 기록 (ADR) |

---

## 기술 스택

<table>
<tr>
  <td><b>클라이언트</b></td>
  <td>Electron · React · TypeScript · Vite</td>
</tr>
<tr>
  <td><b>Node.js 서버</b></td>
  <td>Node.js · Y.js CRDT · MediaSoup · http-proxy-middleware</td>
</tr>
<tr>
  <td><b>Java 서버</b></td>
  <td>Java · Spring Boot · JPA · SSE</td>
</tr>
<tr>
  <td><b>실시간 동기화</b></td>
  <td>Y.js CRDT · WebSocket · SSE</td>
</tr>
<tr>
  <td><b>음성 통화</b></td>
  <td>WebRTC · MediaSoup (SFU) · OPUS 48kHz</td>
</tr>
<tr>
  <td><b>코드 에디터</b></td>
  <td>Eclipse Theia · Monaco Editor · VS Code Extension API</td>
</tr>
<tr>
  <td><b>문서 에디터</b></td>
  <td>Notionblock.js</td>
</tr>
<tr>
  <td><b>DB / 스토리지</b></td>
  <td>MS SQL Server · AWS S3</td>
</tr>
</table>

---

<p align="center">
  📄 <a href="https://github.com/TIMOA-MAIN/TIMOA_Docs">아키텍처 문서 (ADR)</a> &nbsp;|&nbsp;
  🌐 <a href="https://www.timoa.app">www.timoa.app</a>
</p>
