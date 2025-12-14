# Phase 1: Foundation (기반 구축)

## 목표

프로젝트 기반 구축 및 보안 인증 시스템 완성

---

## 태스크 목록

### 1.1 프로젝트 세팅
- [ ] Flutter 프로젝트 생성
- [ ] go_router 설정
- [ ] Riverpod 설정
- [ ] 폴더 구조 설계
  ```
  lib/
  ├── core/           # 핵심 유틸, 상수, 테마
  ├── features/       # 기능별 모듈
  │   ├── auth/       # 인증 (PIN, 생체)
  │   ├── notebook/   # 노트북
  │   └── settings/   # 설정
  └── shared/         # 공용 위젯, 모델
  ```

### 1.2 데이터베이스 설정
- [ ] sqflite_sqlcipher 패키지 추가
- [ ] flutter_secure_storage 패키지 추가
- [ ] 암호화 키 생성 및 저장 로직
- [ ] DatabaseHelper 클래스 구현
- [ ] 앱 최초 실행 시 DB 초기화

### 1.3 PIN 인증
- [ ] PIN 설정 화면 (최초 실행 시)
- [ ] PIN 입력 화면 (잠금 해제)
- [ ] PIN 검증 로직
- [ ] PIN 해시 처리 (SHA-256)
- [ ] PIN 변경 기능

### 1.4 생체 인증
- [ ] local_auth 패키지 추가
- [ ] 생체 인증 가능 여부 확인
- [ ] 생체 인증 활성화 설정
- [ ] 생체 인증 → PIN 폴백

### 1.5 자동 잠금
- [ ] 자동 잠금 타이머 설정 (즉시/1분/5분)
- [ ] 앱 lifecycle 감지 (WidgetsBindingObserver)
- [ ] 백그라운드 전환 시 잠금 처리

### 1.6 인증 실패 처리
- [ ] 실패 횟수 카운트
- [ ] 5회 실패 → 30초 대기
- [ ] 10회 실패 → 5분 대기
- [ ] 대기 시간 UI 표시

---

## 기술 고려사항

### SQLCipher 암호화
```dart
// 암호화 키 생성 (최초 1회)
final key = generateSecureRandomKey(32);
await secureStorage.write(key: 'db_key', value: key);

// DB 열기
final db = await openDatabase(
  path,
  password: await secureStorage.read(key: 'db_key'),
);
```

### PIN 해시 처리
```dart
import 'package:crypto/crypto.dart';

String hashPin(String pin) {
  final bytes = utf8.encode(pin);
  final digest = sha256.convert(bytes);
  return digest.toString();
}
```

### 앱 Lifecycle 감지
```dart
class _AppState extends State<App> with WidgetsBindingObserver {
  @override
  void didChangeAppLifecycleState(AppLifecycleState state) {
    if (state == AppLifecycleState.paused) {
      // 백그라운드 진입 시간 기록
    }
    if (state == AppLifecycleState.resumed) {
      // 잠금 여부 확인
    }
  }
}
```

---

## 의존성/선행 조건

- 없음 (첫 번째 Phase)

---

## 완료 기준 (Definition of Done)

- [ ] PIN 설정 및 인증이 정상 동작한다
- [ ] 생체 인증이 동작한다 (선택적 활성화)
- [ ] 자동 잠금이 설정에 따라 동작한다
- [ ] 인증 실패 시 대기 시간이 적용된다
- [ ] DB가 SQLCipher로 암호화되어 있다 (검증 완료)
- [ ] 암호화 키가 Secure Storage에 안전하게 저장된다

---

## 예상 소요 시간

약 1.5 ~ 2주
