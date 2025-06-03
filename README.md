# 🚀 성능 개선 보고서

## 📍 프로젝트 개요

- **URL**: https://frontend-1902a.web.app/
- **분석 도구**: [Google PageSpeed Insights](https://pagespeed.web.dev/)
- **분석 대상**: 데스크톱
- **개선 링크**:
  - [개선 전 결과](https://pagespeed.web.dev/analysis/https-frontend-1902a-web-app/59o21hvf5w?form_factor=desktop)
  - [개선 후 결과](https://pagespeed.web.dev/analysis/https-frontend-1902a-web-app/ey5tekd7u0?form_factor=desktop)

## 1. 성능 개선 전후 비교

| 항목                           | 개선 전 | 개선 후 | 변화폭   |
| ------------------------------ | ------- | ------- | -------- |
| Performance                    | 60      | 98      | ▲ +34    |
| CLS (Cumulative Layout Shift)  | 0.513   | 0.016   | ▼ -0.497 |
| LCP (Largest Contentful Paint) | 2.9s    | 1.0s    | ▼ -1.9s  |
| TBT (Total Blocking Time)      | 130ms   | 120ms   | ▼ 감소   |

## 2. 개선 이유

| 문제      | 원인                                 | 영향                |
| --------- | ------------------------------------ | ------------------- |
| CLS 높음  | 이미지 크기 미지정, 웹폰트 지연 로딩 | 레이아웃 흔들림     |
| LCP 지연  | Hero 이미지 비최적화                 | 첫 콘텐츠 노출 지연 |
| TBT 높음  | 동기 JS 로딩                         | 스레드 블로킹       |
| 번들 과다 | 사용되지 않는 CSS/JS 포함            | 로딩 시간 증가      |

## 3. 개선 방법

### 1. 대규모 레이아웃 변경 피하기

![Image](https://github.com/user-attachments/assets/49ee3485-1fea-4eec-a0c5-d324c55294f8)

| 항목            | 내용                                                           |
| --------------- | -------------------------------------------------------------- |
| 문제명          | Cumulative Layout Shift (CLS)                                  |
| 주요 발생 위치  | `<section class="best-sellers">` 내 이미지                     |
| 주요 원인       | 명시적 크기가 없는 `<img>` 요소 및 웹폰트 렌더링 지연          |
| 영향            | 페이지 콘텐츠가 예기치 않게 이동하여 사용자 혼란 유발, UX 저하 |
| 측정된 CLS 점수 | **최대 0.490** (권장 기준: ≤ 0.1)                              |

| 문제 요소 | 개선 방법 | 설명 |
|------------|-----------|------|
| `<img class="desktop" src="images/Hero_Desktop.jpg">` | `width`와 `height` 명시 또는 `aspect-ratio` 적용 | 예측 가능한 공간 확보로 렌더링 안정화 |
| `<img src="images/vr1.jpg" alt="...">` | `width`와 `height` 속성 추가 | 제품 이미지 크기를 명확히 지정하여 shift 방지 |

### 예시 코드 (개선 전 → 개선 후)

**개선 전**

```html
<img class="desktop" src="images/Hero_Desktop.jpg" />
```

**개선 후**

```html
<img
  class="desktop"
  src="images/Hero_Desktop.jpg"
  width="1440"
  height="600"
  alt="Hero Image"
/>
```

![Image](https://github.com/user-attachments/assets/b14e91c3-69d6-479c-b8e4-0d7e4385041e)

> CLS 개선(0.49 → 0.01)만으로도 Performance 점수가 60점 → 83점까지 상승

### 2. 차세대 형식을 사용해 이미지 제공하기

![Image](https://github.com/user-attachments/assets/b828469c-d7ef-41b0-8ef6-cdc5412fd11e)

![Image](https://github.com/user-attachments/assets/331111c5-5a23-49aa-9cea-a805f305e84c)

- 각 이미지에 srcset과 sizes 속성을 추가하여, 브라우저가 화면 크기에 맞는 최적의 이미지를 선택
- 모든 이미지에 loading="lazy" 속성을 추가해, 사용자가 이미지를 볼 때까지 로딩을 지연시켜 초기 로딩 속도를 개선
- WebP 포맷을 사용
  - Webp : 구글(Google)에서 개발한 차세대 이미지 포맷으로, 기존의 JPEG, PNG, GIF보다 더 효율적인 압축률을 제공

> LCP 개선: 2.6초 → 1.9초

### 3. 이미지 크기 적절하게 설정하기
   ![Image](https://github.com/user-attachments/assets/850f5b06-70dc-4310-9e57-95a79a8074de)

### 4. 렌더링 차단 리소스 제거하기
   ### 1. **Google Fonts 최적화**
   
   - `preconnect`을 추가하여 폰트 서버와 미리 연결
   - `rel="stylesheet"`에 `media="print"`와 `onload`를 활용해 비동기 로딩 적용
   
   ```html
   <!-- Google Fonts preconnect & async load -->
   <link rel="preconnect" href="https://fonts.googleapis.com" />
   <link rel="preconnect" href="https://fonts.gstatic.com" crossorigin />
   <link
     href="https://fonts.googleapis.com/css?family=Heebo:300,400,600,700&display=swap"
     rel="stylesheet"
     media="print"
     onload="this.media='all'"
   />
   <noscript>
     <link
       href="https://fonts.googleapis.com/css?family=Heebo:300,400,600,700&display=swap"
       rel="stylesheet"
     />
   </noscript>
   ```
   
   ---
   
   ### 2. **메인 CSS 비동기 로딩**
   
   - `media="print"`와 `onload`를 사용
   
   ```html
   <link rel="stylesheet" href="/css/styles.css" media="print" onload="this.media='all'" />
   <noscript>
     <link rel="stylesheet" href="/css/styles.css" />
   </noscript>
   ```
   
   ---
   
   ### 3. **JS 파일 defer 적용**
   
   - `<script src="/js/main.js" defer></script>`
   - `<script src="/js/products.js" defer></script>`
   
   ---
   
   ![스크린샷 2025-06-03 오후 9 22 27](https://github.com/user-attachments/assets/d5843681-8bf9-4ae2-825c-1195ebc7270e)
