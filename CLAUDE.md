# CLAUDE.md

This file provides guidance to Claude Code (claude.ai/code) when working with code in this repository.

## Project Overview

**Secure Notes App** - 보안 로컬 메모 앱 (비밀번호 관리)

> "대충 입력해도 알아서 정리해드려요"

- 100% 로컬 저장 + DB 레벨 암호화 (SQLCipher)
- AI는 조언자 역할만 수행, 규칙 엔진 우선
- 클라우드는 백업/기기이동 목적의 옵션 (MVP 이후)

## Directory Structure

- `docs/prd/` - Product Requirements Documents
- `docs/plans/` - Project planning documents
- `docs/works/` - Work-in-progress documentation

## File Naming Convention

docs/plans/ 및 docs/works/ 폴더의 파일은 다음 형식을 따름:
```
{번호}_{파일내용}.md
예: 001_quick_add_architecture.md
```
- 3자리 넘버링 (001, 002, ...)
- 순차적으로 증가

## Tech Stack (MVP)

| 영역 | 기술 |
|------|------|
| Framework | Flutter |
| Navigation | go_router |
| State | Riverpod |
| Local DB | sqflite_sqlcipher |
| Secure Storage | flutter_secure_storage |
| 생체인증 | local_auth |
| Backend | Supabase (Edge Functions) |
| AI | Gemini Flash |

## Development Commands

*To be added once the project framework is established.*

## Screen Flow

```
잠금화면 → 노트북 목록 → 노트북 상세
(PIN/생체)    (홈)      (검색+내용)
                │
                └→ 설정 (Import)
```

## Input Methods

| 모드 | AI 사용 | 데이터 전송 | 용도 |
|------|---------|------------|------|
| Manual | 없음 | 없음 | 직접 입력 |
| Quick Add | 특성 기반 | feature만 전송 | 일상 빠른 추가 |
| Import | 전체 분석 | 전체 텍스트 (동의 필수) | 마이그레이션 |

## Architecture

### Quick Add AI 시스템
- **규칙 엔진**: 1차 분류 (우선순위 높음)
- **AI**: 불확실한 경우만 호출 (confidence < 0.7)
- **분류 필드**: service, id, password
- **처리 흐름**: 토큰화 → 특성 추출 → 규칙 분류 → AI 보조 → 사용자 확인

### 보안 원칙
- 모든 데이터는 SQLCipher로 DB 레벨 암호화
- 암호화 키는 flutter_secure_storage에 보관 (iOS Keychain / Android Keystore)
- Quick Add: 실제 문자열 값은 AI에 전송 금지, feature만 전송
- Import: 사용자 명시적 동의 후에만 전체 텍스트 전송
- AI 실패 시 규칙 엔진 결과만 사용 (Graceful Degradation)

### 데이터 모델
- **notebooks 테이블**: id, title, icon, content (JSON), order_index, timestamps
- **content 구조**: `{ "items": [{ "id", "service", "username", "password", "memo" }] }`

## MVP Scope (5~6주)

**포함:**
- PIN/생체 인증, 자동 잠금
- 노트북 CRUD, 기본 3종 (아이디/비밀번호, 은행계좌, 전화번호)
- Manual / Quick Add / Import
- 노트 내 검색 (실시간 + 스크롤 이동)
- 비밀번호 마스킹/복사

**제외 (v2 이후):**
- 클라우드 백업/동기화
- 태그/즐겨찾기, 비밀번호 생성기
- 검색 하이라이트, 클립보드 자동 삭제
