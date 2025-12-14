# Phase 4: Import & Search (Import 및 검색)

## 목표

Import 기능으로 기존 메모 일괄 가져오기, 노트 내 검색 기능 구현

---

## 태스크 목록

### 4.1 설정 화면
- [ ] 설정 화면 구현
- [ ] 메뉴 항목
  - 📋 다른 앱에서 가져오기 (Import)
  - 🔐 보안 설정
  - ℹ️ 앱 정보
- [ ] 보안 설정 화면
  - PIN 변경
  - 생체 인증 on/off
  - 자동 잠금 시간

### 4.2 Import - 텍스트 입력
- [ ] Import 진입 화면
- [ ] 대용량 텍스트 입력 (TextField, multiline)
- [ ] 힌트 텍스트 ("기존 메모를 붙여넣으세요")
- [ ] 안내 문구 ("어떤 형식이든 괜찮아요. AI가 자동으로 분석합니다.")
- [ ] [다음] 버튼

### 4.3 Import - 동의 화면
- [ ] AI 분석 안내 화면
- [ ] 데이터 처리 방식 설명
  - 텍스트가 외부 AI 서버로 전송됩니다
  - 분석 완료 후 즉시 삭제됩니다
  - 서버에 저장되지 않습니다
- [ ] 동의 체크박스 ("위 내용을 이해했습니다")
- [ ] [취소] [AI로 분석하기] 버튼

### 4.4 Import - AI 분석
- [ ] Edge Function 생성 (import-analyze)
- [ ] Import용 프롬프트 구현
- [ ] 분석 중 로딩 화면
- [ ] 타임아웃 처리 (30초)
- [ ] 에러 처리

### 4.5 Import - 결과 화면
- [ ] 분석 결과 목록 UI
- [ ] 헤더: "N개 항목을 찾았습니다"
- [ ] 노트북 선택 드롭다운
- [ ] 항목별 카드
  - 체크박스 (선택/해제)
  - 서비스명, ID, PW 표시
  - [수정] 버튼
  - 불확실 항목 표시 (⚠️)
  - 안내 메시지 ("확인이 필요해 보입니다")
- [ ] 개별 항목 수정 모달
- [ ] [선택 항목 저장] [전체 저장] 버튼

### 4.6 Import - 저장
- [ ] 선택된 항목만 저장
- [ ] 노트북에 일괄 추가
- [ ] 저장 완료 후 노트북 상세로 이동

### 4.7 검색 - UI
- [ ] 검색 Input (상단 고정)
- [ ] 검색 아이콘 (🔍)
- [ ] 결과 탐색 버튼 ([▼] [▲])
- [ ] 매칭 카운트 표시 ("3개 중 1번째")
- [ ] 검색어 지우기 버튼

### 4.8 검색 - 로직
- [ ] 실시간 검색 매칭 (debounce 300ms)
- [ ] 검색 대상: 서비스명, 아이디, 메모
- [ ] 대소문자 무시
- [ ] 매칭 항목 인덱스 수집

### 4.9 검색 - 스크롤
- [ ] ScrollController 설정
- [ ] GlobalKey로 항목 위치 추적
- [ ] 첫 번째 매칭으로 자동 스크롤
- [ ] [▼] [▲] 버튼으로 다음/이전 이동
- [ ] 스크롤 애니메이션

---

## 기술 고려사항

### Import AI 응답 형식
```json
{
  "items": [
    {
      "service": "디브로스 구글 계정",
      "id": "dbros.m@dbros.co.kr",
      "password": "elqmfhtm1!",
      "confidence": "high"
    },
    {
      "service": "야후 (flickr)",
      "id": "augustkim05",
      "password": "시작대문자",
      "confidence": "low",
      "note": "비밀번호가 설명처럼 보입니다"
    }
  ]
}
```

### Import 프롬프트 (예시)
```
You are a data extraction engine.
Extract account information from the following text.

For each account found, extract:
- service: The service/website name
- id: The username, email, or ID
- password: The password

If uncertain about any field, set confidence to "low" and add a note.

Return JSON array only.
```

### 검색 스크롤 구현
```dart
final itemKeys = <String, GlobalKey>{};
final scrollController = ScrollController();

void scrollToItem(String itemId) {
  final key = itemKeys[itemId];
  if (key?.currentContext != null) {
    Scrollable.ensureVisible(
      key!.currentContext!,
      duration: Duration(milliseconds: 300),
      curve: Curves.easeInOut,
    );
  }
}
```

### Debounce 검색
```dart
Timer? _debounce;

void onSearchChanged(String query) {
  _debounce?.cancel();
  _debounce = Timer(Duration(milliseconds: 300), () {
    performSearch(query);
  });
}
```

---

## 의존성/선행 조건

- Phase 3 완료 (AI 연동, Edge Function)
- Supabase Edge Function 추가 (import-analyze)

---

## 완료 기준 (Definition of Done)

- [ ] Import로 텍스트를 붙여넣을 수 있다
- [ ] AI 분석 전 동의 화면이 표시된다
- [ ] Import 결과에서 항목을 선택/수정할 수 있다
- [ ] 불확실 항목에 ⚠️ 표시가 된다
- [ ] 선택한 항목을 노트북에 저장할 수 있다
- [ ] 검색 시 실시간으로 매칭된다
- [ ] 매칭 항목으로 자동 스크롤된다
- [ ] [▼][▲] 버튼으로 결과를 탐색할 수 있다

---

## 예상 소요 시간

약 1주
