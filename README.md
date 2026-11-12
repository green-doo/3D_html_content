# Three.js 3D 디지털 트윈 웹 애플리케이션 분석 보고서

본 문서는 첨부된 두 개의 웹 디지털 트윈 프로젝트인 **스마트 교실(`smart_classroom_camera_stable.html`)**과 **한국형 전기 시내버스 조립공장(`ThreeJS_korean_bus.html`)** 소스코드를 분석하여 구동 방법과 코드의 전체적인 문법/아키텍처 구조를 상세히 설명합니다.

---

## 1. 프로젝트 개요

두 파일은 모두 별도의 백엔드 서버나 외부 3D 모델 파일(GLTF, OBJ 등) 없이, **단일 HTML 파일 내에서 HTML5, CSS3, JavaScript(ES Module) 및 Three.js CDN 라이브러리만을 활용해 완전한 3D 인터랙티브 공간 및 기계 설비를 절차적(Procedural)으로 생성**하는 웹 애플리케이션입니다.

* **Smart Classroom (`smart_classroom_camera_stable.html`)**: 교실 공간, 학생/교사 가구, IoT 센서 및 환경 공조(온도, CO₂, 조도, 재실 인원) 제어 시뮬레이션 및 데이터 내보내기(JSON, CSV) 제공.
* **Korean Electric Bus Factory (`ThreeJS_korean_bus.html`)**: 8단계 버스 조립 공정을 자동/수동으로 시뮬레이션하고, 부품 Raycasting 선택, Explode(분해) 뷰, X-Ray 모드, 노선별 도장 색상 변경 기능을 제공.

---

## 2. 실행 방법 (How to Run)

### 2.1 기본 실행 조건
1. **인터넷 연결 필수**: 외부 3D 모델 파일은 없으나, Three.js 엔진 모듈을 CDN(`cdn.jsdelivr.net`)에서 ES Module 형식으로 불러오므로 최초 로딩 시 인터넷 연결이 필요합니다.
2. **최신 브라우저 권장**: WebGL 2.0 및 ES6+ 모듈 시스템을 지원하는 Chrome, Edge, Safari, Firefox 등.

### 2.2 권장 실행 방식 (로컬 웹 서버 사용)
ES Module 기반의 script import 및 Canvas 데이터 추출(`toDataURL`, `toBlob`)이 동작하므로 **로컬 웹 서버 환경에서 실행하는 것을 권장**합니다.

#### 방법 A: VS Code Live Server 확장 프로그램 (가장 간단한 방법)
1. VS Code에서 해당 HTML 파일이 위치한 폴더를 열어줍니다.
2. `Live Server` 확장을 설치한 후, HTML 파일 우클릭 → `Open with Live Server` 선택.

#### 방법 B: Python 내장 서버 활용 (터미널)
HTML 파일이 위치한 디렉토리에서 터미널을 열고 명령어를 실행합니다.
```bash
# Python 3.x
python -m http.server 8000
```
이후 웹 브라우저 주소창에 `http://localhost:8000/smart_classroom_camera_stable.html` 또는 `http://localhost:8000/ThreeJS_korean_bus.html`을 입력합니다.

#### 방법 C: Node.js `serve` 패키지
```bash
npx serve .
```

### 2.3 네트워크/오류 발생 시 대처 기능
두 소스코드 모두 로딩 및 WebGL 초기화 실패 시 화면이 먹통이 되는 것을 방지하는 예외 처리 루틴이 포함되어 있습니다.
* **스마트 교실**: ES Module 로딩 에러 또는 14초 이상 응답이 없으면 3D 뷰 대신 경고 안내 UI와 함께 `새로고침` 버튼을 띄웁니다.
* **시내버스 공장**: Three.js 로딩 실패 시 SVG 기반 2D 버스 참고도(Fallback)로 자동 전환되는 안심 구조가 구현되어 있습니다.

---

## 3. 전체적인 문법 구조 및 아키텍처 (Code Architecture)

두 파일은 거의 동일한 현대적 웹 프론트엔드 모듈 아키텍처 패턴을 공유하고 있습니다.

```
[ HTML5 Structure ]
 ├── <head> 
 │    ├── Dynamic Viewport & Meta Tags
 │    ├── CSS3 Styling (Variables, Flex/Grid, Glassmorphic CSS)
 │    └── Import Map (<script type="importmap">) -> Three.js ES Modules CDN
 ├── <body>
 │    ├── Viewport DOM Canvas Container (`#stage` or `#world`)
 │    ├── Floating Overlay Controls / Inspection Sidebar UI
 │    ├── Loading Spinner & Fallback Error Display
 │    └── Main Script Module (<script type="module">)
 │         ├── 1. Initial Scene Setup (Renderer, Camera, Fog, Lights)
 │         ├── 2. Procedural Geometry & Material Helper Functions
 │         ├── 3. 3D Object Group Construction (Architecture, Machinery, Parts)
 │         ├── 4. Interactive State Management & Data Handlers
 │         ├── 5. Raycasting / Event Listeners Binding
 │         └── 6. Animation Rendering Loop & Dynamic Telemetry Update
```

---

### 3.1 CSS 및 UI 디자인 패턴

두 애플리케이션 모두 다크 모드 기반의 **디지털 트윈/스마트팩토리 관제 센터 테마**를 적용하고 있습니다.

* **CSS 변수(`:root`) 활용**:
  ```css
  :root {
    --bg: #08131f; 
    --panel: rgba(12, 29, 39, .87); 
    --line: rgba(211, 241, 238, .19); 
    --teal: #72dfcb;
  }
  ```
* **유리 질감(Glassmorphism)**: `backdrop-filter: blur(...)`를 적용하여 3D 화면 위에 투명 패널을 플로팅 배치.
* **반응형 레이아웃**:
  * `smart_classroom`: Absolute/Fixed 위치 기반의 햄버거 메뉴 및 하단 가로 배치 패널 (`@media(max-width: 780px)` 대응).
  * `ThreeJS_korean_bus`: CSS Grid 3열 구조(`sidebar`, `viewport`, `inspect`) 및 모바일 Slide-out 드로어 메뉴 구현.

---

### 3.2 Three.js 모듈 로드 방식 (`importmap`)

전통적인 `<script src="...">` 대신 Modern JavaScript 표준인 **Import Map**을 사용하여 모듈 식별자를 CDN URL로 매핑합니다.

```html
<script type="importmap">
{
  "imports": {
    "three": "https://cdn.jsdelivr.net/npm/three@0.180.0/build/three.module.js",
    "three/addons/": "https://cdn.jsdelivr.net/npm/three@0.180.0/examples/jsm/"
  }
}
</script>
<script type="module">
  import * as THREE from 'three';
  import { OrbitControls } from 'three/addons/controls/OrbitControls.js';
</script>
```

---

### 3.3 절차적 3D 지오메트리 빌드 (Procedural Modeling)

외부 `.gltf` / `.obj` 3D 모델 파일을 다운로드받지 않고, 소스코드 내에서 기본 도형(Box, Cylinder, Sphere, Tube, Torus)을 엮어 complex 오브젝트 그룹을 생성합니다.

#### 헬퍼 함수 패턴 (코드 간결화 및 재사용성 향상)
```javascript
// 버스 프로젝트의 헬퍼 예시
const G = (geo, mat, x=0, y=0, z=0) => {
  const o = new THREE.Mesh(geo, mat);
  o.position.set(x, y, z);
  o.castShadow = true;
  o.receiveShadow = true;
  return o;
};

function bx(parent, w, h, d, mat, x=0, y=0, z=0) {
  const m = G(new THREE.BoxGeometry(w, h, d), mat, x, y, z);
  parent.add(m);
  return m;
}
```

#### 재질(Material) 및 PBR 렌더링 설정
* **Standard Material (`MeshStandardMaterial`)**: 금속성(`metalness`), 거칠기(`roughness`)를 조정하여 철재 프레임, 우드 바닥, 고무 타이어 등의 질감 구현.
* **Physical Material (`MeshPhysicalMaterial`)**: 자동차 도장(`clearcoat`), 유리의 광학 굴절 및 투명도(`transmission`, `opacity`, `transparent`) 구현.

---

### 3.4 2D Canvas를 이용한 동적 텍스처 (Dynamic Canvas Texture & Sprite Tags)

외부 이미지 파일 없이 **HTML Canvas 2D API**로 즉석에서 글자 및 차트를 그린 후 Three.js 텍스처로 전환하는 기법이 두 프로젝트 모두 사용되었습니다.

* **전자칠판 & 표지판 텍스처**:
  ```javascript
  const canvas = document.createElement('canvas');
  const ctx = canvas.getContext('2d');
  // 2D 캔버스 텍스트/도형 그리기...
  const texture = new THREE.CanvasTexture(canvas);
  texture.colorSpace = THREE.SRGBColorSpace;
  ```
* **3D 씬 내 데이터 라벨 스프라이트 (`THREE.Sprite`)**:
  카메라 시점에 맞추어 항상 정면을 바라보는 `THREE.SpriteMaterial`과 `CanvasTexture`를 조합하여 3D 공간 상에 센서 데이터 태그를 실시간 갱신 표시.

---

### 3.5 애니메이션 및 상태 제어 시스템

#### 애니메이션 루프 구조
* `smart_classroom`: 표준 `requestAnimationFrame(animate)` 방식 사용.
* `ThreeJS_korean_bus`: Three.js 권장 표준인 `renderer.setAnimationLoop(...)` 사용.

#### 시간 간격 및 선형 보간 (Lerp / Delta Time)
프레임 레이트에 의존하지 않고 시뮬레이션 속도를 일정하게 유지하기 위해 `THREE.Clock`의 `dt` (Delta Time) 값을 계산하고, `Vector3.lerp()` 또는 `MathUtils.lerp()`를 사용하여 카메라 이동, 문 열림, 파트 조립 이동을 부드럽게 연출합니다.

```javascript
// 버스 분해/조립 위치 보간 예시
const pos = state.explode ? p.explode : v3(0,0,0);
p.group.position.lerp(pos, Math.min(1, dt * 6.5));
```

---

### 3.6 주요 고급 인터랙션 기능 구현 원리

#### 1) 3D 마우스 클릭 부품 탐색 (Raycasting)
사용자가 3D 화면을 클릭했을 때 마우스 2D 좌표를 3D 공간 광선으로 투사하여 클릭된 객체를 식별합니다.
```javascript
const raycaster = new THREE.Raycaster();
const pointer = new THREE.Vector2();

pointer.set((e.clientX - rect.left) / rect.width * 2 - 1, -(e.clientY - rect.top) / rect.height * 2 + 1);
raycaster.setFromCamera(pointer, camera);
const hit = raycaster.intersectObjects(validObjects, false);
if (hit.length > 0) {
  selectPart(hit[0].object.userData.partId);
}
```

#### 2) 스마트 교실 - 카메라인 경계 자동 절개 (Wall Cutaway)
카메라가 교실 외부에 위치할 때 시야를 가리는 앞벽이나 좌측 창문 벽을 자동으로 안 보이게 처리하여 내부 인테리어가 깔끔하게 노출되도록 구현되었습니다.
```javascript
function updateWallCutaway() {
  const p = camera.position;
  if (p.z < -4.38) wallCutaway.frontHidden = true;
  else if (p.z > -3.90) wallCutaway.frontHidden = false;
  
  frontWall.visible = !wallCutaway.frontHidden;
}
```

#### 3) 데이터 내보내기 (Data Export)
* **PNG 이미지 저장**: `renderer.render(scene, camera)` 호출 후 `canvas.toDataURL()` 또는 `canvas.toBlob()`을 통해 이미지 파일 다운로드.
* **JSON / CSV 덤프**: `smart_classroom`에서 씬 전체 객체의 좌표 스냅샷을 JSON으로 보관(Blender 파이프라인 변환용)하고, 센서 수치 이력을 CSV 스트림 문장으로 생성하여 텍스트 Blob 형태로 출력.

---

## 4. 두 파일 간 핵심 기능 비교 요약

| 항목 | 스마트 교실 (`smart_classroom_camera_stable.html`) | 전기버스 공장 (`ThreeJS_korean_bus.html`) |
| :--- | :--- | :--- |
| **주요 목적** | 실내 환경 IoT 센서 관제 및 조명/공조 시뮬레이션 | 8단계 시내버스 조립/제조 공정 시뮬레이션 |
| **Three.js 버전** | r164.1 | r180.0 |
| **핵심 인터랙션** | 슬라이더 기반 데이터 조절, 씬 스냅샷(JSON/CSV) Export | 단계별 자동/수동 조립, X-Ray, Explode, 도어 열기, 도장 변경 |
| **카메라 제어** | OrbitControls (범위 Clamp 및 자동 벽 절개 조율) | OrbitControls + 프리셋 타겟 뷰 이동 보간 |
| **렌더링 루프** | `requestAnimationFrame` | `renderer.setAnimationLoop` |
| **오류 대응** | WebGL Context Lost / Restored 이벤트 핸들러 적용 | 네트워크 미연결 시 SVG 2D 백업 화면 표시 |

---

## 5. 결론 및 응용 가이드

두 파일은 웹 3D 프론트엔드 개발 시 참조할 수 있는 아주 우수한 **독립형(Self-contained) 디지털 트윈 참조 소스코드**입니다.

1. **새로운 3D 장치 및 공간 추가**: 파일 내부의 `part()` 또는 `group()` 헬퍼 구조를 참고하여 원하는 형태의 3D 객체를 코드로 추가할 수 있습니다.
2. **실제 데이터 연동 (IoT / MES)**: `sim` 혹은 `state` 객체의 값을 주기적인 REST API 또는 WebSocket/MQTT 수신 데이터로 변경하면 즉시 실제 현장 모니터링 시스템으로 변환 가능합니다.
```

---

제안해주신 두 개 파일의 분석, 실행 가이드, 그리고 코드 아키텍처 문법 구조를 정리한 **`threejs_projects_analysis.md`** 마크다운 파일 생성을 완료했습니다.

### 주요 포함 내용
1. **실행 환경 및 방법**: 로컬 웹 서버(VS Code Live Server, Python 등)를 활용한 실행법과 네트워크/WebGL 예외 처리 구조 설명.
2. **코드 문법 및 아키텍처**:
   * Modern Import Map을 활용한 ES Module 연결 방식.
   * 절차적 지오메트리(Procedural Modeling) 구축 헬퍼 함수 구조.
   * 2D Canvas를 이용한 동적 텍스처 및 3D Sprite 라벨 구현.
   * Raycasting 부품 선택, 선형 보간(Lerp) 애니메이션, 벽 자동 절개(Cutaway) 및 데이터 Export(PNG, JSON, CSV) 로직 분석.
3. **두 프로젝트 기능 비교 표**: 각 파일별 Three.js 버전, 인터랙션 특성, 렌더링 루프 방식 차이 요약.

추가로 코드 수정이나 특정 기능 확장이 필요하시면 언제든 말씀해 주세요!