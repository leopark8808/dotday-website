# DotDay 공식 웹사이트

DotDay(바탕화면 월간 캘린더 위젯)의 공식 랜딩/소개 웹사이트. 앱 본체 repo(`dotday`)와 분리된 **공개 정적 사이트**다(한국어 단일 페이지).

- **앱 다운로드**: 주 배포는 [Microsoft Store](https://apps.microsoft.com/detail/9MWXX5BVG5JW). 보조로 **직접 다운로드 .exe** 채널(GitHub Releases `dotday-releases` 링크, NetSparkle 자동 업데이트).
- **배포**: Cloudflare Pages - 라이브 **`dotday-website.pages.dev`** (GitHub 네이티브 연결, `main`에 push하면 자동 배포, 빌드 30~60초). 구글 OAuth 검증/브랜딩 도메인.
- **스택**: 빌드 도구 없는 정적 HTML/CSS + 인라인 JS 약간.

## 구조

```
index.html        # 메인 (한국어)
privacy.html      # 개인정보처리방침
styles.css        # 공용 스타일 (?v=N 캐시버전으로 참조)
assets/           # 로고·스크린샷·OG 이미지
updater/          # exe 채널 자동 업데이트 appcast.xml + .signature (앱 repo build-update-manifest.ps1이 자동 복사)
version.json      # 푸터 버전 표기 소스 (JS fetch)
_headers          # HTML=no-cache, /assets/*·*.css=max-age 1년 immutable
```

## 캐시 규칙 (중요)

`_headers`로 CSS가 **immutable 장기 캐시**라 `index.html`에서 `styles.css?v=N`으로 참조한다.
**CSS를 수정하면 반드시 `?v=N`을 +1** 해야 기존 방문자에게 반영된다(현재 v=7).

## 다운로드 UI · 모바일 처리 (2026-07-05)

다운로드 버튼은 **히어로 + 하단 CTA 두 곳**(Store 버튼 + 직접 다운로드 exe 버튼 + SmartScreen 경고 안내). 모바일 노출은 `index.html` 하단 인라인 스크립트의 UA 감지가 담당한다:

- Android/iPhone/iPad UA 또는 "Mac UA + 멀티터치"(최신 아이패드)면 `<html>`에 `is-mobile` 클래스 부여 →
  - `.pc-only` 요소 숨김 = exe 직접 다운로드 버튼·SmartScreen 경고 안내 (Store 버튼은 모바일에도 노출)
  - `.mobile-only` 요소 노출 = "DotDay는 PC 전용 앱" 라운드 배지 안내(히어로·하단 CTA 각 1곳)
- **유지보수 규칙**: 설치 파일 다운로드 버튼·설치 안내를 새로 추가하면 `pc-only` 클래스를 같이 달아야 모바일에서 숨겨진다. FAQ 본문의 exe 텍스트 링크는 문장 중간이라 대상에서 제외.

## 출시 시 갱신처 (전부 6곳)

앱을 새 버전으로 출시하면 아래를 전부 갱신 후 push:

1. `version.json`의 `version` - 푸터 버전 실제 표시 소스
2. `index.html`·`privacy.html`의 `<span id="appVersion">` 정적 폴백 (2곳)
3. **직접 다운로드 링크 3곳** - `DotDay-Setup-<x.y.z.0>.exe` 파일명에 4자리 버전 하드코딩(히어로·FAQ 본문·하단 CTA). `grep DotDay-Setup`으로 전부 찾아 갱신. **잊으면 `/releases/latest/download/` 링크가 404 → 설치 불가**(1.3.5.0 때 실제 발생).

> 참고: Cloudflare Pages는 `/x.html`을 clean URL(`/x`)로 308 리다이렉트한다(정상 동작). curl 검증 시 `-L` 필요.
