# heart
<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8">
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<title>Matte Red Heart From Darkness</title>

<style>
* {
  margin: 0;
  padding: 0;
  box-sizing: border-box;
}

body {
  background: #000;
  overflow: hidden;
}

canvas {
  display: block;
}

#label {
  position: fixed;
  bottom: 35px;
  width: 100%;
  text-align: center;
  color: #e00020;
  font: 11px Arial, sans-serif;
  letter-spacing: 6px;
  pointer-events: none;
}
</style>
</head>

<body>
<div id="label">BORN FROM DARKNESS</div>

<script type="module">
import * as THREE from
'https://cdn.jsdelivr.net/npm/three@0.170.0/build/three.module.js';

// SCENE
const scene = new THREE.Scene();
scene.background = new THREE.Color(0x000000);

const camera = new THREE.PerspectiveCamera(
  45, innerWidth / innerHeight, 0.1, 100
);
camera.position.set(0, 0, 12);

const renderer = new THREE.WebGLRenderer({
  antialias: true
});

renderer.setSize(innerWidth, innerHeight);
renderer.setPixelRatio(Math.min(devicePixelRatio, 2));
renderer.outputColorSpace = THREE.SRGBColorSpace;
renderer.toneMapping = THREE.NoToneMapping;

document.body.appendChild(renderer.domElement);

// SOFT LIGHTING
scene.add(new THREE.AmbientLight(0xffffff, 1.0));

function addLight(color, intensity, x, y, z) {
  const light = new THREE.PointLight(color, intensity, 35);
  light.position.set(x, y, z);
  scene.add(light);
}

addLight(0xff2233, 45, -4, 3, 5);
addLight(0xff3344, 35, 4, 2, 4);
addLight(0x990015, 25, 0, -3, 4);
addLight(0x55000a, 20, 0, 4, -2);

// HEART SHAPE
const shape = new THREE.Shape();

shape.moveTo(0, -1.8);
shape.bezierCurveTo(-0.3, -1.1, -2.4, 0.3, -2.4, 1.35);
shape.bezierCurveTo(-2.4, 2.5, -0.8, 2.7, 0, 1.45);
shape.bezierCurveTo(0.8, 2.7, 2.4, 2.5, 2.4, 1.35);
shape.bezierCurveTo(2.4, 0.3, 0.3, -1.1, 0, -1.8);

const heartGeometry = new THREE.ExtrudeGeometry(shape, {
  depth: 0.65,
  bevelEnabled: true,
  bevelSegments: 5,
  steps: 1,
  bevelSize: 0.12,
  bevelThickness: 0.12,
  curveSegments: 32
});

heartGeometry.computeVertexNormals();

// HEART GROUP
const heart = new THREE.Group();
scene.add(heart);

// MATTE RED MATERIAL: NO METALLIC OR GLOSSY FINISH
const heartMaterial = new THREE.MeshStandardMaterial({
  color: 0xe00020,
  metalness: 0.0,
  roughness: 1.0,
  emissive: 0x300006,
  emissiveIntensity: 0.35,
  flatShading: false
});

const heartBody = new THREE.Mesh(
  heartGeometry,
  heartMaterial
);

heartBody.position.z = -0.3;
heart.add(heartBody);

// PARTICLES THAT FORM THE HEART
const particleCount = 8000;

const particlePositions = new Float32Array(particleCount * 3);
const targetPositions = new Float32Array(particleCount * 3);
const startPositions = new Float32Array(particleCount * 3);
const particleDelays = new Float32Array(particleCount);

function getHeartPoint(t) {
  const x = 16 * Math.pow(Math.sin(t), 3);

  const y =
    13 * Math.cos(t)
    - 5 * Math.cos(2 * t)
    - 2 * Math.cos(3 * t)
    - Math.cos(4 * t);

  return [x / 6.8, y / 6.8 - 0.25];
}

for (let i = 0; i < particleCount; i++) {
  const t = Math.random() * Math.PI * 2;
  const point = getHeartPoint(t);

  // Fill the heart with particles
  const fill = Math.sqrt(Math.random());

  targetPositions[i * 3] = point[0] * fill;
  targetPositions[i * 3 + 1] = point[1] * fill;
  targetPositions[i * 3 + 2] = (Math.random() - 0.5) * 0.7;

  // Start outside the heart
  const side = Math.random() < 0.5 ? -1 : 1;

  startPositions[i * 3] = side * (4 + Math.random() * 7);
  startPositions[i * 3 + 1] = (Math.random() - 0.5) * 10;
  startPositions[i * 3 + 2] = (Math.random() - 0.5) * 5;

  particlePositions[i * 3] = startPositions[i * 3];
  particlePositions[i * 3 + 1] = startPositions[i * 3 + 1];
  particlePositions[i * 3 + 2] = startPositions[i * 3 + 2];

  particleDelays[i] = Math.random() * 0.4;
}

const particleGeometry = new THREE.BufferGeometry();

particleGeometry.setAttribute(
  'position',
  new THREE.BufferAttribute(particlePositions, 3)
);

const particleMaterial = new THREE.PointsMaterial({
  color: 0xff243b,
  size: 0.025,
  transparent: true,
  opacity: 0.95,
  blending: THREE.AdditiveBlending,
  depthWrite: false
});

const heartParticles = new THREE.Points(
  particleGeometry,
  particleMaterial
);

heart.add(heartParticles);

// FLOATING RED FRAGMENTS
const fragmentCount = 1500;
const fragmentPositions = new Float32Array(fragmentCount * 3);
const fragmentSpeeds = new Float32Array(fragmentCount);

for (let i = 0; i < fragmentCount; i++) {
  fragmentPositions[i * 3] = (Math.random() - 0.5) * 16;
  fragmentPositions[i * 3 + 1] = (Math.random() - 0.5) * 12;
  fragmentPositions[i * 3 + 2] = (Math.random() - 0.5) * 5;

  fragmentSpeeds[i] = 0.005 + Math.random() * 0.018;
}

const fragmentGeometry = new THREE.BufferGeometry();

fragmentGeometry.setAttribute(
  'position',
  new THREE.BufferAttribute(fragmentPositions, 3)
);

const fragmentMaterial = new THREE.PointsMaterial({
  color: 0x9e2438,
  size: 0.035,
  transparent: true,
  opacity: 0.7,
  blending: THREE.AdditiveBlending,
  depthWrite: false
});

const fragments = new THREE.Points(
  fragmentGeometry,
  fragmentMaterial
);

scene.add(fragments);

// SMOKE-LIKE PARTICLE STREAMS
const smokeCount = 1800;
const smokePositions = new Float32Array(smokeCount * 3);
const smokeData = [];

for (let i = 0; i < smokeCount; i++) {
  smokeData.push({
    side: Math.random() < 0.5 ? -1 : 1,
    progress: Math.random(),
    y: (Math.random() - 0.5) * 6,
    z: (Math.random() - 0.5) * 2,
    speed: 0.002 + Math.random() * 0.005
  });
}

const smokeGeometry = new THREE.BufferGeometry();

smokeGeometry.setAttribute(
  'position',
  new THREE.BufferAttribute(smokePositions, 3)
);

const smokeMaterial = new THREE.PointsMaterial({
  color: 0x7d2638,
  size: 0.045,
  transparent: true,
  opacity: 0.5,
  blending: THREE.AdditiveBlending,
  depthWrite: false
});

const smoke = new THREE.Points(
  smokeGeometry,
  smokeMaterial
);

scene.add(smoke);

// ANIMATION
const clock = new THREE.Clock();

function animate() {
  requestAnimationFrame(animate);

  const time = clock.getElapsedTime();

  // 12-second loop: build, hold, dissolve, rebuild
  const cycle = (time % 12) / 12;
  let reveal;

  if (cycle < 0.38) {
    reveal = cycle / 0.38;
  } else if (cycle < 0.72) {
    reveal = 1;
  } else {
    reveal = 1 - (cycle - 0.72) / 0.28;
  }

  reveal = THREE.MathUtils.clamp(reveal, 0, 1);
  reveal = reveal * reveal * (3 - 2 * reveal);

  // Assemble particles from both sides
  for (let i = 0; i < particleCount; i++) {
    const delay = particleDelays[i] * 0.5;

    const progress = THREE.MathUtils.clamp(
      (reveal - delay) / (1 - delay),
      0, 1
    );

    const scatter = 1 - progress;
    const k = i * 3;

    particlePositions[k] =
      THREE.MathUtils.lerp(
        startPositions[k],
        targetPositions[k],
        progress
      ) + Math.sin(time * 2 + i) * 0.04 * scatter;

    particlePositions[k + 1] =
      THREE.MathUtils.lerp(
        startPositions[k + 1],
        targetPositions[k + 1],
        progress
      ) + Math.cos(time * 1.8 + i * 0.2) * 0.04 * scatter;

    particlePositions[k + 2] =
      THREE.MathUtils.lerp(
        startPositions[k + 2],
        targetPositions[k + 2],
        progress
      );
  }

  particleGeometry.attributes.position.needsUpdate = true;

  // Gentle heartbeat
  const beat = 1 + 0.035 * Math.pow(
    Math.max(0, Math.sin(time * 3.2)), 16
  );

  heart.scale.setScalar(0.88 * beat);
  heart.rotation.y = Math.sin(time * 0.35) * 0.16;
  heart.rotation.x = Math.sin(time * 0.25) * 0.045;

  // Fade the matte heart in and out
  heartBody.visible = false;

  heartMaterial.opacity = THREE.MathUtils.clamp(
    (reveal - 0.4) * 2.5, 0, 1
  );

  heartMaterial.transparent = true;
  particleMaterial.opacity = 0.95 * reveal;

  // Floating fragments
  for (let i = 0; i < fragmentCount; i++) {
    const k = i * 3;

    fragmentPositions[k] += Math.sin(time + i) * 0.0015;
    fragmentPositions[k + 1] += fragmentSpeeds[i];

    if (fragmentPositions[k + 1] > 6) {
      fragmentPositions[k + 1] = -6;
    }
  }

  fragmentGeometry.attributes.position.needsUpdate = true;

  // Smoke-like streams flow inward
  for (let i = 0; i < smokeCount; i++) {
    const s = smokeData[i];

    s.progress += s.speed;

    if (s.progress > 1) s.progress = 0;

    const k = i * 3;

    smokePositions[k] = s.side * (6 - s.progress * 4);

    smokePositions[k + 1] =
      s.y + Math.sin(time * 1.5 + i) * 0.22;

    smokePositions[k + 2] =
      s.z + Math.cos(time + i) * 0.15;
  }

  smokeGeometry.attributes.position.needsUpdate = true;
  smokeMaterial.opacity = 0.5 * (1 - reveal * 0.3);

  renderer.render(scene, camera);
}

// START ANIMATION
animate();

// RESPONSIVE SCREEN
addEventListener('resize', () => {
  camera.aspect = innerWidth / innerHeight;
  camera.updateProjectionMatrix();

  renderer.setSize(innerWidth, innerHeight);
});
</script>
</body>
</html>

