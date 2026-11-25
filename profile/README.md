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

```
Desktop Client (Electron · React · TypeScript)
        │
        ▼
   Proxy Server  (Node.js · 헬스체크 기반 동적 라우팅 · LB)
        │
  ┌─────┼──────────┬─────────────────┬──────────┐
  ▼     ▼          ▼                 ▼          ▼
Chat  Dashboard  User           Code / Doc    Voice
(Java · SSE)    (Java)     (Node.js · Y.js CRDT) (MediaSoup SFU)
  │      │        │                 │
  └──────┴────────┘            MS SQL + AWS S3
         │
      MS SQL
```

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
