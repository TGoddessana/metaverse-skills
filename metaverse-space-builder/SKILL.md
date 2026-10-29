---
name: "metaverse-space-builder"
description: "실제 장소(카페, 사무실, 매장, 전시장, 캠퍼스 등)를 걸어 다닐 수 있는 3D 메타버스 공간(three.js 웹페이지)으로 만들어 달라는 요청에 사용."
---

# 메타버스 공간 만들기

"OO 공간을 메타버스로 구현해줘" 같은 요청을 받으면, 장소 정보를 모으고, 자료를 찾아 공간 설계도(spec)를 만든 뒤, three.js로 직접 걸어 다닐 수 있는 3D 페이지를 만들어 Artifact로 올린다.

전체 흐름: 1) 정보 묻기 → 2) 자료 찾기 → 3) 설계도 확인 → 4) 3D 페이지 만들기 → 5) 점검 → 6) 올리고 다듬기

작업이 여러 단계이므로 시작할 때 TaskCreate로 위 6단계를 할 일 목록으로 만들고, 끝날 때마다 체크한다.

---

## 1단계. 정보 묻기

요청을 받으면 바로 AskUserQuestion으로 한 번에 묻는다. 이미 요청에 들어 있는 정보는 다시 묻지 않는다.

꼭 물을 것:
- **장소 이름** (가게 이름, 건물 이름)
- **주소** (최소한 동/역 수준)
- **사진이 있는지** → 있으면 지금 첨부해 달라고 한다. 사용자가 찍은 사진은 실제 질감(텍스처)으로 쓸 수 있어서 결과가 훨씬 좋아진다.

있으면 좋은 것 (선택지로 제시):
- 범위: 실내만 / 실외(건물 외관)까지 / 둘 다
- 용도: 홍보용 둘러보기 / 행사·전시장 / 사무실 가상 출근 / 그냥 재미
- 꼭 넣고 싶은 요소 (간판, 특정 가구, 작품, 메뉴판 등)

사용자가 답하지 않거나 자리에 없으면: 실내 중심, 홍보용 둘러보기로 정하고 맨 위에 그렇게 정했다고 적고 진행한다.

주의: 개인 집처럼 사적인 공간인데 사용자 본인 공간이 아닌 것으로 보이면(남의 집 주소 등) 진행하지 않는다. 공개된 가게, 공공시설, 사용자 본인 공간은 괜찮다.

---

## 2단계. 자료 찾기

여러 번 검색해서 공간의 모양을 최대한 구체적으로 파악한다. 검색은 한 턴에 여러 개를 같이 보낸다.

검색할 것:
- WebSearch: `장소이름 주소`, `장소이름 내부`, `장소이름 후기`, `장소이름 평수` 또는 `면적`, 공식 홈페이지/인스타그램
- WebFetch: 공식 홈페이지, 지도 장소 페이지, 자세한 방문 후기(블로그)
- mcp__claude_ai__image_search: `장소이름 인테리어`, `장소이름 외관`, `장소이름 좌석` 등으로 사진 확인

사진과 글에서 뽑아낼 것:
- 층수, 대략적인 크기(가로 x 세로 x 천장 높이, 미터). 정보가 없으면 좌석 수나 사진 속 테이블 개수로 추정하고 추정이라고 표시
- 공간 구조: 입구 위치, 카운터, 창문, 계단, 방 구분
- 바닥/벽/천장 재질과 색 (예: 노출 콘크리트, 원목 바닥, 흰 벽)
- 주요 가구와 물건, 배치
- 조명 분위기 (따뜻한 노란 조명, 밝은 자연광 등)
- 간판, 로고 문구 (글자로만 기록)

**사진 사용 규칙**
- 웹에서 찾은 사진은 **참고용으로만** 쓴다. 모양, 색, 배치를 보고 직접 3D로 다시 만든다. 웹 사진을 내려받아 페이지에 붙이거나 링크로 넣지 않는다.
- 사용자가 직접 첨부한 사진만 텍스처(벽 액자, 간판, 바닥 무늬 등)로 쓸 수 있다.
- 사람 얼굴이 나온 사진은 텍스처로 쓰지 않는다.
- 작업 공간의 터미널에서는 일반 웹사이트 접속이 막혀 있으니 curl 등으로 사진을 받으려 하지 않는다.

---

## 3단계. 설계도 만들고 확인받기

찾은 내용을 `space-spec.json`으로 정리한다. 단위는 미터, 원점(0,0,0)은 바닥 한가운데, y가 위쪽.

```json
{
  "name": "장소 이름",
  "address": "주소",
  "floors": [
    {
      "level": 0,
      "size": [14, 3.2, 10],
      "floor": { "color": "#b08a5a", "pattern": "wood" },
      "walls": { "color": "#f2efe9" },
      "openings": [
        { "type": "door", "wall": "south", "at": 0, "width": 1.8 },
        { "type": "window", "wall": "east", "at": 2, "width": 4, "height": 2 }
      ],
      "objects": [
        { "kind": "counter", "pos": [-5, 0, -3], "size": [3, 1.1, 0.8], "color": "#3b3b3b" },
        { "kind": "table", "pos": [2, 0, 1], "size": [1.2, 0.75, 0.8], "color": "#d8c3a5", "repeat": [3, 2], "gap": [2.2, 2.0] },
        { "kind": "sign", "pos": [0, 2.4, -4.95], "text": "간판 문구" }
      ]
    }
  ],
  "lighting": { "mood": "warm", "sun": true },
  "spawn": [0, 1.6, 4],
  "hotspots": [ { "pos": [-5, 1.2, -3], "title": "카운터", "text": "주문하는 곳" } ],
  "estimates": ["천장 높이는 사진으로 추정"],
  "sources": [ { "title": "공식 홈페이지", "url": "..." } ]
}
```

사용자가 대화에 있으면, SendUserMessage로 설계도 요약을 짧게 보여주고 바로 다음 단계로 넘어간다(승인을 기다리며 멈추지 않는다). 요약 예:
- 크기: 약 14m x 10m, 1층
- 주요 요소: 입구 남쪽, 왼쪽 안쪽 카운터, 4인 테이블 6개, 동쪽 큰 창
- 추정한 부분: 천장 높이, 뒤쪽 공간 배치

---

## 4단계. 3D 페이지 만들기

먼저 Skill 도구로 `artifact-design` 스킬을 불러온다. 그 다음 하나의 HTML 파일(`metaverse.html`)로 만든다. spec은 파일 안에 그대로 넣는다.

### 기본 원칙
- three.js는 importmap으로 jsdelivr에서 불러온다 (`three@0.160.0`).
- 3D 모델 파일(glTF 등)을 외부에서 가져오지 않는다. 모든 것은 기본 도형(Box, Cylinder, Plane)과 CanvasTexture로 직접 만든다.
- 재질 무늬(원목, 타일, 콘크리트)는 canvas에 그려서 텍스처로 쓴다.
- 사용자가 첨부한 사진은 Bash로 줄여서(긴 변 1024px 이하, JPEG) base64로 넣는다. 전체 페이지는 16MB 이하.
- **휴대폰에서도 반드시 움직일 수 있어야 한다.** 화면 왼쪽 아래 가상 조이스틱으로 이동, 화면 나머지 부분을 끌어서 시점 회전.
- PC: WASD/방향키 이동, 클릭하면 마우스로 시점 회전(PointerLock), Shift 달리기.
- 벽과 가구를 뚫고 지나가지 않게 간단한 충돌 처리(Box3 와 플레이어 원 비교, x축 z축 따로 검사).
- 계단/층이 있으면 층 이동 버튼 또는 계단 위치에 닿으면 높이 바꾸기.
- hotspot은 떠 있는 표지로 보여주고, 가까이 가거나 누르면 설명 창.
- 화면 구석에 작은 미니맵(위에서 본 2D canvas)과 장소 이름.
- 처음 화면에 "조작 방법" 안내를 띄우고, 누르면 시작.

### 뼈대 코드

```html
<script type="importmap">
{ "imports": {
  "three": "https://cdn.jsdelivr.net/npm/three@0.160.0/build/three.module.js",
  "three/addons/": "https://cdn.jsdelivr.net/npm/three@0.160.0/examples/jsm/"
} }
</script>
<script type="module">
import * as THREE from 'three';
import { PointerLockControls } from 'three/addons/controls/PointerLockControls.js';

const SPEC = /* space-spec.json 내용 */ {};

const renderer = new THREE.WebGLRenderer({ antialias: true });
renderer.setPixelRatio(Math.min(devicePixelRatio, 2));
renderer.setSize(innerWidth, innerHeight);
renderer.shadowMap.enabled = true;
document.body.appendChild(renderer.domElement);

const scene = new THREE.Scene();
const camera = new THREE.PerspectiveCamera(70, innerWidth / innerHeight, 0.1, 500);
camera.position.set(...SPEC.spawn);

scene.add(new THREE.HemisphereLight(0xffffff, 0x777777, 0.8));
const sun = new THREE.DirectionalLight(0xfff2e0, 1.1);
sun.position.set(8, 15, 6); sun.castShadow = true; scene.add(sun);

const colliders = [];
function box(size, pos, mat, solid = true) {
  const [w, h, d] = size;
  const m = new THREE.Mesh(new THREE.BoxGeometry(w, h, d), mat);
  m.position.set(pos[0], pos[1] + h / 2, pos[2]);
  m.castShadow = m.receiveShadow = true;
  scene.add(m);
  if (solid) colliders.push(new THREE.Box3().setFromObject(m));
  return m;
}

function canvasTex(draw, size = 512, repeat = [1, 1]) {
  const c = document.createElement('canvas'); c.width = c.height = size;
  draw(c.getContext('2d'), size);
  const t = new THREE.CanvasTexture(c);
  t.wrapS = t.wrapT = THREE.RepeatWrapping; t.repeat.set(...repeat);
  t.colorSpace = THREE.SRGBColorSpace;
  return t;
}

// 이동: 키보드 + 터치 조이스틱 값을 move.x, move.z 에 모은다
const move = { x: 0, z: 0 }, R = 0.3;
function tryMove(dx, dz) {
  const p = camera.position;
  const hit = (x, z) => colliders.some(b =>
    x + R > b.min.x && x - R < b.max.x && z + R > b.min.z && z - R < b.max.z &&
    p.y - 1.5 < b.max.y && p.y > b.min.y);
  if (!hit(p.x + dx, p.z)) p.x += dx;
  if (!hit(p.x, p.z + dz)) p.z += dz;
}

const clock = new THREE.Clock();
renderer.setAnimationLoop(() => {
  const dt = Math.min(clock.getDelta(), 0.05);
  // 카메라가 보는 방향 기준으로 move 값을 월드 좌표로 바꿔 tryMove 호출
  renderer.render(scene, camera);
});
addEventListener('resize', () => {
  camera.aspect = innerWidth / innerHeight; camera.updateProjectionMatrix();
  renderer.setSize(innerWidth, innerHeight);
});
</script>
```

나머지(벽과 문·창 구멍 만들기, 가구 종류별 모양, 간판 글자 텍스처, 터치 조이스틱, 미니맵, hotspot)는 spec을 읽어서 만드는 함수로 짠다. 가구는 `kind`별로 작은 함수 하나씩(table, chair, counter, shelf, sofa, plant, sign, frame) 만들고 `repeat`가 있으면 격자로 복제한다.

### 분위기 살리기
- 조명 mood가 warm이면 노란 빛 PointLight 몇 개를 천장 아래에, bright면 창 쪽 강한 빛.
- 창 밖은 단색 하늘 + 흐릿한 건물 실루엣 정도로 처리.
- 바닥/벽 텍스처는 단색보다 canvas 무늬를 쓰면 훨씬 그럴듯해진다.
- 성능: 그림자 맵 1024, 같은 가구는 InstancedMesh, 휴대폰에서 60fps 근처가 목표.

---

## 5단계. 점검

- Bash로 Playwright(없으면 `pip install playwright --break-system-packages && playwright install chromium`)를 써서 페이지를 열고, 콘솔 오류가 없는지, 첫 화면 스크린샷과 몇 걸음 이동한 뒤 스크린샷을 찍어 Read로 직접 본다. CDN 접속이 막혀 있으면 importmap 경로를 node_modules의 three로 바꾼 사본으로 점검한다.
- 확인할 것: 시작 위치가 벽 안이 아닌지, 벽을 뚫고 나가지 않는지, 가구 배치가 설계도와 맞는지, 휴대폰 크기(390x844)에서 조이스틱이 보이는지.

---

## 6단계. 올리고 다듬기

- Artifact 도구로 `metaverse.html`을 올린다 (icon: "map").
- 답장은 짧게: 무엇을 만들었는지 한두 줄, 추정한 부분, 다음에 하면 좋을 것 하나(예: "실제 사진을 보내주시면 벽 액자와 간판에 넣을게요"), 그리고 참고한 자료 목록(Sources).
- 수정 요청이 오면 spec을 먼저 고치고 같은 파일을 다시 올려 같은 주소를 유지한다.

## 하지 않는 것
- 웹에서 찾은 사진, 로고 이미지, 다른 회사의 3D 모델을 페이지에 넣기
- 유명 캐릭터나 상표 캐릭터를 공간 장식으로 그려 넣기
- 사람 얼굴 사진을 텍스처로 쓰기
- 남의 개인 주거 공간을 본인 동의 없이 재현하기