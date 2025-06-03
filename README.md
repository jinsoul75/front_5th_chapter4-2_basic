# 바닐라 JS 프로젝트 성능 개선

- url: https://frontend-1902a.web.app/

## 최초 성능 개선 보고서

1. Lighthouse 성능 측정 결과

   ![Image](https://github.com/user-attachments/assets/c63c09d3-6bae-4abe-bb60-eb994a513329)

2. PageSpeed Insights 성능 측정 결과

- https://pagespeed.web.dev/analysis/https-frontend-1902a-web-app/59o21hvf5w?form_factor=desktop

  ![Image](https://github.com/user-attachments/assets/98dd2b02-0141-4e5b-916c-223abae18f46)

  ![Image](https://github.com/user-attachments/assets/c5543313-6dd1-404d-aa3f-1753480e3996)

## 🔧 개선 방법

### 1. 대규모 레이아웃 변경 피하기

![Image](https://github.com/user-attachments/assets/49ee3485-1fea-4eec-a0c5-d324c55294f8)

| 항목            | 내용                                                           |
| --------------- | -------------------------------------------------------------- |
| 문제명          | Cumulative Layout Shift (CLS)                                  |
| 주요 발생 위치  | `<section class="best-sellers">` 내 이미지                     |
| 주요 원인       | 명시적 크기가 없는 `<img>` 요소 및 웹폰트 렌더링 지연          |
| 영향            | 페이지 콘텐츠가 예기치 않게 이동하여 사용자 혼란 유발, UX 저하 |
| 측정된 CLS 점수 | **최대 0.490** (권장 기준: ≤ 0.1)                              |

https://pagespeed.web.dev/analysis/https-frontend-1902a-web-app/5n4ypbww6g?form_factor=desktop
| 문제 요소 | 개선 방법 | 설명 |
|------------|-----------|------|
| `<img class="desktop" src="images/Hero_Desktop.jpg">` | `width`와 `height` 명시 또는 `aspect-ratio` 적용 | 예측 가능한 공간 확보로 렌더링 안정화 |
| `<img src="images/vr1.jpg" alt="...">` | `width`와 `height` 속성 추가 | 제품 이미지 크기를 명확히 지정하여 shift 방지 |
| 공통 | 레이아웃 요소의 크기 사전 예약 | Skeleton UI, CSS aspect-ratio 또는 min-height 등 활용 |

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

성능 60 → 83

2. 차세대 형식을 사용해 이미지 제공하기

![Image](https://github.com/user-attachments/assets/b828469c-d7ef-41b0-8ef6-cdc5412fd11e)

![Image](https://github.com/user-attachments/assets/331111c5-5a23-49aa-9cea-a805f305e84c)

각 이미지에 srcset과 sizes 속성을 추가하여, 브라우저가 화면 크기에 맞는 최적의 이미지를 선택하도록 했습니다.
모든 이미지에 loading="lazy" 속성을 추가해, 사용자가 이미지를 볼 때까지 로딩을 지연시켜 초기 로딩 속도를 개선했습니다.
WebP 포맷을 사용하도록 했으며, 고해상도(2x) 이미지를 위한 예시도 추가했습니다.

> LCP 2.6초 -> 1.9초

3. 이미지 크기 적절하게 설정하기
   ![Image](https://github.com/user-attachments/assets/850f5b06-70dc-4310-9e57-95a79a8074de)

4. 렌더링 차단 리소스 제거하기
   아래와 같이 코드를 수정하면 렌더링 차단 리소스를 줄이고, 페이지 로딩 성능을 개선할 수 있습니다.

---

### 1. **Google Fonts 최적화**

- `preconnect`을 추가하여 폰트 서버와 미리 연결합니다.
- `rel="stylesheet"`에 `media="print"`와 `onload`를 활용해 비동기 로딩을 적용합니다.

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

- 동일하게 `media="print"`와 `onload`를 사용합니다.

```html
<link rel="stylesheet" href="/css/styles.css" media="print" onload="this.media='all'" />
<noscript>
  <link rel="stylesheet" href="/css/styles.css" />
</noscript>
```

---

### 3. **Cookie Consent JS 지연 로딩**

- 렌더링이 끝난 후(예: DOMContentLoaded) 동적으로 삽입합니다.

```html
<!-- 기존 코드 삭제 -->
<!--
<script
  type="text/javascript"
  src="//www.freeprivacypolicy.com/public/cookie-consent/4.1.0/cookie-consent.js"
  charset="UTF-8"
></script>
<script type="text/javascript" charset="UTF-8">
  cookieconsent.run({ ... });
</script>
-->

<!-- 아래처럼 변경 -->
<script>
  window.addEventListener('DOMContentLoaded', function () {
    var s = document.createElement('script');
    s.src = '//www.freeprivacypolicy.com/public/cookie-consent/4.1.0/cookie-consent.js';
    s.async = true;
    s.onload = function () {
      cookieconsent.run({
        notice_banner_type: 'simple',
        consent_type: 'express',
        palette: 'light',
        language: 'en',
        page_load_consent_levels: ['strictly-necessary'],
        notice_banner_reject_button_hide: false,
        preferences_center_close_button_hide: false,
        page_refresh_confirmation_buttons: false,
        website_name: 'Performance Course',
      });
    };
    document.body.appendChild(s);
  });
</script>
```

---

### 4. **JS 파일 defer 적용**

- `<script src="/js/main.js" defer></script>`
- `<script src="/js/products.js" defer></script>`

---

## **정리된 head 예시**

```html
<head>
  <meta charset="UTF-8" />
  <meta name="viewport" content="width=device-width, initial-scale=1.0" />
  <title>Home - Tech Shop</title>

  <!-- Google Fonts 최적화 -->
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

  <!-- 메인 CSS 비동기 로딩 -->
  <link rel="stylesheet" href="/css/styles.css" media="print" onload="this.media='all'" />
  <noscript>
    <link rel="stylesheet" href="/css/styles.css" />
  </noscript>
</head>
```

그리고 `<body>` 끝부분에 Cookie Consent JS 지연 로딩 스크립트를 추가하세요.

---

이렇게 변경하면 렌더링 차단 리소스 경고가 크게 줄어듭니다!  
적용이 필요하다면 바로 코드로 반영해드릴 수 있습니다.
