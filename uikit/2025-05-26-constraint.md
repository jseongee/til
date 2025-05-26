## Visual Indicator

- 파란색 라인: “I bar” 라고 부르며 뷰 간 제약 설정을 나타냄
- Equality Badge: 두 항목이 같음을 의미
- 제약 우선순위 (Priority)
    - 범위: `1 ~ 1000`
    - `1000`: 필수 제약 (`.required`)
    - `< 1000`: 옵션 제약
        - 상대적으로 높은 우선순위: `750` (`.defaultHigh`)
        - 상대적으로 낮은 우선순위: `250` (`.defaultLow`)
    - 코드에서 `.required`와 `.defaultHigh`를 토글하면 경고 발생 가능
    → `.defaultHigh`와 `.defaultLow` 사용 추천

<br />

## Constraint

- 기본 형태: `item1.attribute1 = multiplier × item2.attribute2 + constant`
- `item`: 제약 대상 (UIView, Safe Area 등)
- `attribute`: 제약 종류 (width, height, leading, trailing, NotAnAttribue 등)
- `multiplier`:  비율 (예: 0.5, 1.0, 2.0)
- `constant`: 설정값

<image src="https://github.com/user-attachments/assets/f5f4ae3e-9f36-43c7-8f12-9da63c56562c" width="400" />

( View.top = 1.0 x Superview.top + 118 )

- 최소한의 제약만 추가 권장
- 대부분의 경우 Safe Area 기준으로 설정

<br />

## Baseline

- First Baseline: 텍스트가 시작되는 기준선
- Last Baseline: 텍스트가 끝나는 기준선

<br />

## Intrinsic Size

- 뷰가 본질적으로 가지고 있는 크기

<br />

## Hugging & Compression Resistance

- 너비 또는 높이가 고정되지 않은 뷰 사이의 충돌 해결 기준
- Content Hugging Priority (CH)
    - 확장에 대한 저항력
    - 기본값: `250`
- Content Compression Resistance Priority (CR)
    - 축소에 대한 저항력
    - 기본값: `750`

<br />

## Adaptive Layout

- Size Class: 뷰의 배치 공간 크기
- Trait Collection 세부적인 실행환경 (예: 화면 방향, 다크모드 등)

<br />

## Size Class

<image src="https://github.com/user-attachments/assets/67038621-2bca-4d06-bc7c-b182360d721e" width="500" />

- `Regular`: 상대적으로 큰 사이즈
- `Compact`: 상대적으로 작은 사이즈
- Final Size Class (런타임 기준)
    - `Compact-Compact`
    - `Compact-Regular`
    - `Regular-Compact`
    - `Regular-Regular`
- Base Size Class (스토리보드 편집 기준)
    - `Compact-Any`
    - `Regular-Any`
    - `Any-Regular`
    - `Any-Compact`
    - `Any-Any`

<br />

## Traits

- iOS 기기의 환경 정보를 나타내는 값
- `UITraitCollection`: trait 모음
- `UITraitEnvironment`: trait를 인식하고 반응하는 객체 프로토콜
