# inc03 섹션 Sticky Scroll 이슈 해결 문서

## 📋 문제 개요
**날짜**: 2025-01-14
**이슈**: inc03 섹션에서 이미지와 텍스트가 동기화되지 않고 개별 섹션으로 동작하는 문제

### 증상
- PC에서 이미지가 sticky로 고정되지 않음
- 텍스트와 이미지가 따로 스크롤됨
- GSAP ScrollTrigger가 제대로 작동하지 않음

## 🔍 문제 분석 과정

### 1단계: 초기 가설 - 모바일 CSS 간섭
**가설**: 모바일 반응형 CSS가 데스크톱 레이아웃에 영향
```css
@media (max-width: 768px) {
    #inc03 .img_inner {
        display: none !important;
    }
}
```
**결과**: 미디어 쿼리는 정상적으로 분리되어 있었음

### 2단계: GSAP 코드 검증
**원본 템플릿과 비교**:
```javascript
// 원본 템플릿 (template-original/index.html)
texts.forEach((text, i) => {
    ScrollTrigger.create({
        trigger: text,
        start: "top 40%",
        end: "bottom 0%",
        onEnterBack: () => { setActive(i); },
        onLeave: () => { if(i < lastIndex) setActive(i + 1); }
    });
});
```
**결과**: GSAP 코드는 원본과 100% 동일했음

### 3단계: CSS 충돌 확인
**불필요한 미디어 쿼리 발견**:
```css
/* 제거된 중복 코드 */
@media (min-width: 769px) {
    #inc03 .img_inner {
        position: sticky !important;
    }
}
```
**결과**: 중복 제거했지만 문제 지속

### 4단계: 근본 원인 발견 ✅
**body 태그의 overflow-x 속성이 sticky positioning을 방해**

## 💡 해결 방법

### 최종 해결책
```css
/* 문제가 된 코드 (user.css) */
body {
    overflow-x: hidden; /* 이것이 sticky를 방해 */
}

/* 수정 후 */
body {
    /* overflow-x 제거 */
}
```

## 📝 파일 변경 내역

### 1. `/us/css/user.css`
- **변경**: `overflow-x: hidden` 제거
- **이유**: sticky positioning과 충돌

### 2. `/us/css/image-gallery.css`
- **변경**: 불필요한 `@media (min-width: 769px)` 블록 제거
- **이유**: 기본 CSS가 이미 데스크톱용이므로 중복

### 3. `/us/html/index.html`
- **변경**: GSAP 코드를 원본과 동일하게 유지
- **확인**: HTML 구조와 스크립트 모두 원본과 일치

## 🎯 핵심 교훈

### Sticky Positioning 체크리스트
1. **부모 요소의 overflow 확인**
   - `overflow: hidden`, `overflow-x: hidden` 등이 sticky를 방해
   - body, html 태그 포함 모든 상위 요소 확인 필요

2. **높이(height) 설정 확인**
   - sticky 요소의 부모는 명확한 높이가 필요
   - `height: auto`나 미설정 시 문제 발생 가능

3. **z-index 충돌 확인**
   - 다른 요소와의 z-index 우선순위
   - position 속성이 있는 요소들 간의 관계

## ✅ 검증 결과

### PC (769px 이상)
- ✅ 이미지 sticky 고정 정상 작동
- ✅ 텍스트 스크롤 시 이미지 전환
- ✅ GSAP ScrollTrigger 정상 작동

### 모바일 (768px 이하)
- ✅ 서비스 카드 레이아웃 유지
- ✅ Sticky 애니메이션 비활성화
- ✅ 반응형 디자인 정상 작동

## 🔧 추가 권장사항

1. **CSS 구조 개선**
   - 데스크톱 스타일을 기본으로 설정
   - 모바일만 미디어 쿼리로 오버라이드
   - 불필요한 !important 제거

2. **디버깅 팁**
   - Chrome DevTools의 Computed 탭에서 실제 적용된 스타일 확인
   - ScrollTrigger.markers = true로 트리거 포인트 시각화
   - console.log로 GSAP 이벤트 발생 확인

## 📚 참고 자료
- [MDN - position: sticky](https://developer.mozilla.org/en-US/docs/Web/CSS/position#sticky)
- [GSAP ScrollTrigger Docs](https://greensock.com/docs/v3/Plugins/ScrollTrigger)
- 원본 템플릿: `/template-original/index.html`