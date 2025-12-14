# Phase 3: Input & Quick Add (입력 기능)

## 목표

Manual 입력 및 Quick Add 기능 구현 (규칙 엔진 + AI 보조)

---

## 태스크 목록

### 3.1 Manual 입력
- [ ] Manual 입력 화면
- [ ] 입력 폼 구현
  - 서비스명 (필수)
  - 아이디 (필수)
  - 비밀번호 (필수)
  - 메모 (선택)
- [ ] 폼 검증
- [ ] 저장 로직

### 3.2 Quick Add - 토큰화 (Tokenizer)
- [ ] 분리 기준 구현
  - 공백, 탭, `/` → 항상 분리
  - `-` → 조건부 분리
  - `:` → 분리 + 레이블 제거
- [ ] 레이블 제거 (아이디:, pw:, 비밀번호: 등)
- [ ] 연속 구분자 병합

### 3.3 Quick Add - 특성 추출 (Feature Extractor)
- [ ] FeatureSet 모델 구현
  ```dart
  class FeatureSet {
    final int length;
    final bool hasKorean;
    final bool hasAlpha;
    final bool hasDigit;
    final bool hasSpecial;
    final bool isEmail;
    final bool isURL;
    final bool isNumeric;
    final bool isPhoneLike;
  }
  ```
- [ ] 정규식 기반 특성 추출

### 3.4 Quick Add - 규칙 엔진 (Rule Engine)
- [ ] 규칙 우선순위 구현
  | 순위 | 규칙 | 분류 |
  |------|------|------|
  | 0 | 첫 번째 + 한글 | 서비스명 |
  | 1 | 이메일 형식 | ID |
  | 2 | 특수문자 + 길이6+ | PW |
  | 3 | 영문+숫자 혼합 | ID 후보 |
  | 4 | 한글 + 짧음 | 서비스명 |
- [ ] 신뢰도(confidence) 계산
- [ ] consumed 플래그로 중복 방지

### 3.5 Quick Add - UI
- [ ] Quick Add 입력 화면
- [ ] 한 줄 입력 TextField
- [ ] 힌트 텍스트 ("대충 공백으로 구분해서 입력해도...")
- [ ] 분류 결과 미리보기 UI
- [ ] 신뢰도별 UI 표시
  - high: 표시 없음
  - medium: ⚠️ + 확인 요청
  - low: ⚠️ + 수정 권유
- [ ] 버튼: [이대로 저장] [수정하기] [취소]

### 3.6 AI 연동 (Supabase Edge Function)
- [ ] Supabase 프로젝트 설정
- [ ] Edge Function 생성 (classify-tokens)
- [ ] Gemini API 키 설정 (환경변수)
- [ ] 프롬프트 구현 (PRD D.13 참고)
- [ ] Rate Limit 설정

### 3.7 AI 호출 로직 (클라이언트)
- [ ] AI 호출 조건 (confidence < 0.7)
- [ ] Feature Payload 생성 (값 제외, 특성만)
- [ ] AI 응답 파싱
- [ ] AI 응답 검증
  - JSON 파싱 가능 여부
  - 필수 필드 존재 여부
  - index 범위 검증
  - 중복 index 검증
- [ ] 규칙 엔진 vs AI 충돌 처리
- [ ] AI 사용량 카운트 (일 10회 제한)

### 3.8 폴백 처리
- [ ] AI timeout (5초)
- [ ] AI error → 규칙 기반 결과 사용
- [ ] AI 응답 전부 null → 전체 메모 저장 제안
- [ ] AI 한도 초과 시 안내 문구

---

## 기술 고려사항

### 토큰화 예시
```dart
// 입력: "네이버 hong123 pass123!"
// 결과: ["네이버", "hong123", "pass123!"]

// 입력: "구글 아이디: test@gmail.com 비번: secret!"
// 결과: ["구글", "test@gmail.com", "secret!"]
```

### Feature Payload (AI 전송용)
```json
{
  "task": "classify_tokens",
  "tokens": [
    { "index": 0, "features": ["korean", "len3"] },
    { "index": 1, "features": ["alpha", "digit", "len7"] },
    { "index": 2, "features": ["special", "digit", "len8"] }
  ]
}
```

### AI 응답
```json
{
  "service": 0,
  "id": 1,
  "password": 2
}
```

### 신뢰도 계산
```dart
// 규칙 매칭 시 confidence 증가
// 이메일 → +0.3
// 특수문자+길이6+ → +0.3
// 영문+숫자 → +0.2
// 한글 서비스명 → +0.25

// 필수 필드 누락 시 감소
// service 없음 → -0.2
// id 없음 → -0.2
// password 없음 → -0.1
```

### Edge Function 예시
```typescript
// supabase/functions/classify-tokens/index.ts
import { GoogleGenerativeAI } from "@google/generative-ai";

Deno.serve(async (req) => {
  const { tokens } = await req.json();

  const genAI = new GoogleGenerativeAI(Deno.env.get("GEMINI_API_KEY"));
  const model = genAI.getGenerativeModel({ model: "gemini-1.5-flash" });

  const result = await model.generateContent(buildPrompt(tokens));
  return new Response(result.response.text());
});
```

---

## 의존성/선행 조건

- Phase 2 완료 (노트북, 항목 저장)
- Supabase 프로젝트 생성
- Gemini API 키 발급

---

## 완료 기준 (Definition of Done)

- [ ] Manual 입력으로 항목을 저장할 수 있다
- [ ] Quick Add 입력 시 자동 분류가 동작한다
- [ ] 규칙 엔진이 우선 적용된다
- [ ] 신뢰도가 낮을 때만 AI가 호출된다
- [ ] AI에 실제 값이 전송되지 않는다 (feature만)
- [ ] AI 실패 시 규칙 기반 결과로 폴백된다
- [ ] 일 10회 AI 호출 제한이 동작한다

---

## 예상 소요 시간

약 1 ~ 1.5주
