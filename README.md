# FFacio Build

FFacio APK 배포용 공개 저장소입니다. **소스는 없습니다.** 빌드 워크플로와 릴리스 산출물만 있습니다.

소스는 비공개 저장소 [`sampple-korea/FFacio`](https://github.com/sampple-korea/FFacio)에 있습니다. 이 저장소의 워크플로가 그 소스를 체크아웃해서 빌드하고, 결과 APK를 이곳 릴리스로 발행합니다. 공개 저장소는 GitHub Actions 사용 시간이 무료이므로 비공개 저장소의 Actions 한도를 쓰지 않습니다.

## 다운로드

[Releases](https://github.com/sampple-korea/FFacio-Build/releases)에서 최신 태그를 받으세요. 릴리스마다 아래 파일이 모두 들어 있습니다.

| 파일 | 패키지 | 역할 |
| --- | --- | --- |
| `FFacio-release.apk` | `com.ffacio.mobile` | 출입 인증 본체 |
| `ITSOKEY-Runtime-release.apk` | `io.ffacio.itsokeyruntime` | 도어락 세션·제어 Runtime |
| `FFacio-Face-Runtime-release.apk` | `com.kbyai.faceattribute` | 얼굴 엔진 Runtime (런처 없음) |
| `FFacio-Runtime-Demo-release.apk` | `io.ffacio.demo` | Runtime 점검용 데모 |
| `*-debug.apk` | — | 같은 네 앱의 debug 빌드 |
| `FFacio-Runtime-client-release.aar` | `io.ffacio.client` | Face Runtime Binder 클라이언트 |
| `ITSOKEY-Runtime-client-release.aar` | `io.ffacio.itsokeyruntime.client` | ITSOKEY Runtime Binder 클라이언트 |
| `*-signature.txt`, `*-badging.txt` | — | 서명·패키지 검증 리포트 |
| `SHA256SUMS.txt` | — | 모든 산출물의 SHA-256 |

## 설치 순서

1. `FFacio-Face-Runtime-release.apk`
2. `ITSOKEY-Runtime-release.apk`
3. `FFacio-release.apk`

세 앱은 `signature` 보호 수준의 Binder 권한으로 연결되므로 **같은 인증서로 서명된 같은 릴리스의 APK를 함께 설치해야 합니다.** 서로 다른 릴리스나 debug/release를 섞으면 Binder 연결이 실패합니다.

Face Runtime은 런처 아이콘이 없습니다. 설치 후 홈 화면에 나타나지 않는 것이 정상입니다.

## 빌드 실행 방법

### 자동

비공개 저장소의 `master`에 push하면 빌드가, `v*` 태그를 push하면 빌드와 릴리스 발행이 자동으로 실행됩니다.

### 수동

Actions 탭에서 **Build and release FFacio APKs**를 `Run workflow`로 실행합니다.

| 입력 | 설명 |
| --- | --- |
| `ref` | 빌드할 소스 ref. 브랜치, 태그, 커밋 SHA 모두 가능합니다. 기본값 `master` |
| `release_tag` | 릴리스로 발행할 태그. 비우면 워크플로 아티팩트만 남기고 릴리스는 만들지 않습니다 |

## 워크플로가 하는 일

통합 이전 세 저장소의 워크플로(`build-app-apk.yml`, `build-runtime-apk.yml`, `android.yml`, `android-ci.yml`)가 하던 검사를 하나로 합쳐 모두 수행합니다.

1. 비공개 소스 저장소 체크아웃 (`FFACIO_SOURCE_TOKEN`)
2. Gradle wrapper 검증
3. 소스 계약 검증 4종 — FFacio 정적 감사, ITSOKEY 연동, ITSOKEY 프로토콜 재구성, Runtime Demo 정렬
4. 단위 테스트 — FFacio 본체와 ITSOKEY Runtime
5. debug APK 4종 빌드
6. 서명된 release APK 4종과 클라이언트 AAR 2종 빌드
7. 패키지 역할 검증 — 런처 Activity 유무까지 확인
8. 네 APK의 서명 인증서 SHA-256이 서로 같고 기대값과 일치하는지 검증
9. 산출물 수집, `SHA256SUMS.txt` 생성, 아티팩트 업로드
10. 태그가 있으면 릴리스 생성 또는 갱신

소스 ZIP은 만들지 않습니다. 이 저장소는 공개이고 소스는 비공개이기 때문입니다.

## 필요한 Secrets

| 이름 | 용도 |
| --- | --- |
| `FFACIO_SOURCE_TOKEN` | 비공개 소스 저장소 체크아웃용 PAT (Contents: read) |
| `FFACIO_KEYSTORE_BASE64` | PKCS#12 서명키를 base64로 인코딩한 값 |
| `FFACIO_KEYSTORE_PASSWORD` | 키스토어 비밀번호 |
| `FFACIO_KEY_ALIAS` | 키 별칭 |
| `FFACIO_KEY_PASSWORD` | 키 비밀번호 |

서명키 원문은 저장소와 릴리스 산출물 어디에도 포함되지 않습니다. 워크플로는 러너의 임시 디렉터리에만 키를 복원하고, 기대 인증서 지문과 일치하는지만 확인합니다.
