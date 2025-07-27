# Hehe<!DOCTYPE html>
<html lang="es">
<head>
<meta charset="UTF-8" />
<meta name="viewport" content="width=device-width, initial-scale=1" />
<title>Moisés 3D - Demo</title>
<style>
  body { margin: 0; overflow: hidden; background-color: #202533; }
  canvas { display: block; }
</style>
</head>
<body>
<script src="https://cdn.jsdelivr.net/npm/three@0.152.2/build/three.min.js"></script>
<script src="https://cdn.jsdelivr.net/npm/three@0.152.2/examples/js/controls/OrbitControls.js"></script>

<script>
// --- Setup básico ---
const scene = new THREE.Scene();
scene.background = new THREE.Color(0x202533);

const camera = new THREE.PerspectiveCamera(45, window.innerWidth / window.innerHeight, 0.1, 1000);
camera.position.set(0, 2.5, 6);

const renderer = new THREE.WebGLRenderer({ antialias: true, toneMapping: THREE.ACESFilmicToneMapping });
renderer.outputEncoding = THREE.sRGBEncoding;
renderer.setSize(window.innerWidth, window.innerHeight);
renderer.shadowMap.enabled = true;
renderer.shadowMap.type = THREE.PCFSoftShadowMap;
document.body.appendChild(renderer.domElement);

const controls = new THREE.OrbitControls(camera, renderer.domElement);
controls.target.set(0, 1.2, 0);
controls.enableDamping = true;
controls.dampingFactor = 0.05;
controls.update();

// --- Iluminación ---
const hemiLight = new THREE.HemisphereLight(0xffffff, 0x444466, 1.2);
hemiLight.position.set(0, 20, 0);
scene.add(hemiLight);

const dirLight = new THREE.DirectionalLight(0xffffff, 1);
dirLight.position.set(5, 10, 7);
dirLight.castShadow = true;
dirLight.shadow.mapSize.width = 2048;
dirLight.shadow.mapSize.height = 2048;
dirLight.shadow.camera.left = -5;
dirLight.shadow.camera.right = 5;
dirLight.shadow.camera.top = 5;
dirLight.shadow.camera.bottom = -5;
scene.add(dirLight);

const fillLight = new THREE.AmbientLight(0x404050, 0.3);
scene.add(fillLight);

// --- Suelo ---
const ground = new THREE.Mesh(
  new THREE.PlaneGeometry(30, 30),
  new THREE.MeshStandardMaterial({ color: 0x222222, roughness: 0.8, metalness: 0.1 })
);
ground.rotation.x = -Math.PI / 2;
ground.receiveShadow = true;
scene.add(ground);

// --- Textura simulada para la túnica (arrugas) ---
const tunicTextureSize = 128;
const tunicCanvas = document.createElement('canvas');
tunicCanvas.width = tunicCanvas.height = tunicTextureSize;
const ctx = tunicCanvas.getContext('2d');

// Fondo dorado base
ctx.fillStyle = '#d4af37';
ctx.fillRect(0, 0, tunicTextureSize, tunicTextureSize);

// Dibujar líneas para simular arrugas
ctx.strokeStyle = 'rgba(160,130,30,0.4)';
ctx.lineWidth = 2;
for (let i = 0; i < 15; i++) {
  ctx.beginPath();
  ctx.moveTo(i * 10 + 5, 0);
  ctx.bezierCurveTo(i * 10 + 10, 20, i * 10, 50, i * 10 + 5, tunicTextureSize);
  ctx.stroke();
}
const tunicTexture = new THREE.CanvasTexture(tunicCanvas);
tunicTexture.wrapS = tunicTexture.wrapT = THREE.RepeatWrapping;
tunicTexture.repeat.set(2, 4);

// Material dorado pulido con textura de arrugas
const goldMaterial = new THREE.MeshStandardMaterial({
  map: tunicTexture,
  color: 0xd4af37,
  metalness: 1,
  roughness: 0.25,
  emissive: 0x2a2100,
  emissiveIntensity: 0.2,
});

// Material piel
const skinMaterial = new THREE.MeshStandardMaterial({
  color: 0xffdbac,
  roughness: 0.7,
  metalness: 0,
  clearcoat: 0.3,
  clearcoatRoughness: 0.1,
});

// Material interno túnica oscuro
const innerTunicMaterial = new THREE.MeshStandardMaterial({
  color: 0x3c2f1f,
  roughness: 0.9,
  metalness: 0,
});

// --- Barba con textura simulada ---
const beardCanvas = document.createElement('canvas');
beardCanvas.width = beardCanvas.height = 64;
const bctx = beardCanvas.getContext('2d');
bctx.fillStyle = '#4b2e05';
bctx.fillRect(0, 0, 64, 64);
for (let i = 0; i < 20; i++) {
  bctx.strokeStyle = `rgba(75,46,5,${Math.random() * 0.7 + 0.3})`;
  bctx.beginPath();
  const x = Math.random() * 64;
  bctx.moveTo(x, 0);
  bctx.lineTo(x + (Math.random() * 10 - 5), 64);
  bctx.stroke();
}
const beardTexture = new THREE.CanvasTexture(beardCanvas);
beardTexture.wrapS = beardTexture.wrapT = THREE.RepeatWrapping;
beardTexture.repeat.set(1, 1);

const beardMaterial = new THREE.MeshStandardMaterial({
  map: beardTexture,
  roughness: 0.7,
  metalness: 0,
  transparent: true,
  opacity: 0.95,
});

// --- Material vara ---
const staffMaterial = new THREE.MeshStandardMaterial({
  color: 0x6b3e10,
  roughness: 0.4,
  metalness: 0,
});

// --- Cuerpo y partes ---
const bodyGroup = new THREE.Group();

// Túnica exterior con textura
const tunicGeometry = new THREE.CylinderGeometry(0.45, 0.5, 1.7, 24, 1, true);
const tunicMesh = new THREE.Mesh(tunicGeometry, goldMaterial);
tunicMesh.castShadow = true;
tunicMesh.position.y = 0.85;
bodyGroup.add(tunicMesh);

// Túnica interna
const innerTunicGeometry = new THREE.CylinderGeometry(0.38, 0.4, 1.65, 24);
const innerTunicMesh = new THREE.Mesh(innerTunicGeometry, innerTunicMaterial);
innerTunicMesh.position.y = 0.85;
innerTunicMesh.castShadow = true;
bodyGroup.add(innerTunicMesh);

// Cabeza
const headGeometry = new THREE.SphereGeometry(0.38, 32, 32);
const head = new THREE.Mesh(headGeometry, skinMaterial);
head.castShadow = true;
head.position.set(0, 2.05, 0);
bodyGroup.add(head);

// Ojos blancos
const eyeWhiteGeometry = new THREE.SphereGeometry(0.08, 16, 16);
const eyeWhiteMaterial = new THREE.MeshStandardMaterial({ color: 0xffffff, roughness: 0.2 });
const leftEyeWhite = new THREE.Mesh(eyeWhiteGeometry, eyeWhiteMaterial);
const rightEyeWhite = leftEyeWhite.clone();
leftEyeWhite.position.set(-0.13, 2.15, 0.33);
rightEyeWhite.position.set(0.13, 2.15, 0.33);
leftEyeWhite.castShadow = true;
rightEyeWhite.castShadow = true;
bodyGroup.add(leftEyeWhite);
bodyGroup.add(rightEyeWhite);

// Pupilas negras
const pupilGeometry = new THREE.SphereGeometry(0.04, 12, 12);
const pupilMaterial = new THREE.MeshStandardMaterial({ color: 0x000000, roughness: 0.3 });
const leftPupil = new THREE.Mesh(pupilGeometry, pupilMaterial);
const rightPupil = leftPupil.clone();
leftPupil.position.set(-0.13, 2.15, 0.4);
rightPupil.position.set(0.13, 2.15, 0.4);
leftPupil.castShadow = true;
rightPupil.castShadow = true;
bodyGroup.add(leftPupil);
bodyGroup.add(rightPupil);

// Barba con textura
const beardGeometry = new THREE.CylinderGeometry(0.18, 0.25, 0.3, 32, 1);
const beard = new THREE.Mesh(beardGeometry, beardMaterial);
beard.position.set(0, 1.75, 0.32);
beard.rotation.x = Math.PI / 2.5;
beard.castShadow = true;
bodyGroup.add(beard);

// Vara de Moisés
const staffGeometry = new THREE.CylinderGeometry(0.04, 0.04, 2.2, 20);
const staff = new THREE.Mesh(staffGeometry, staffMaterial);
staff.position.set(0.65, 1.1, 0);
staff.rotation.z = Math.PI / 14;
staff.castShadow = true;
bodyGroup.add(staff);

// Brazos simples animados
const armMaterial = new THREE.MeshStandardMaterial({
  color: 0xd4af37,
  metalness: 1,
  roughness: 0.25,
  emissive: 0x2a2100,
  emissiveIntensity: 0.2,
});

const leftArmGeometry = new THREE.CylinderGeometry(0.12, 0.12, 1.1, 20);
const rightArmGeometry = leftArmGeometry.clone();

const leftArm = new THREE.Mesh(leftArmGeometry, armMaterial);
leftArm.position.set(-0.6, 1.3, 0);
leftArm.castShadow = true;
bodyGroup.add(leftArm);

const rightArm = new THREE.Mesh(rightArmGeometry, armMaterial);
rightArm.position.set(0.6, 1.3, 0);
rightArm.castShadow = true;
bodyGroup.add(rightArm);

scene.add(bodyGroup);

// --- Animación ---
const clock = new THREE.Clock();

function animate() {
  requestAnimationFrame(animate);

  const elapsed = clock.getElapsedTime();

  // Movimiento de cabeza y ojos
  head.rotation.y = 0.15 * Math.sin(elapsed * 1.2);
  leftEyeWhite.rotation.y = 0.1 * Math.sin(elapsed * 1.2);
  rightEyeWhite.rotation.y = 0.1 * Math.sin(elapsed * 1.2);
  leftPupil.position.x = -0.13 + 0.02 * Math.sin(elapsed * 3);
  rightPupil.position.x = 0.13 + 0.02 * Math.sin(elapsed * 3);

  // Parpadeo
  const blink = (Math.sin(elapsed * 8) > 0.95) ? 0.05 : 1;
  leftEyeWhite.scale.y = blink;
  rightEyeWhite.scale.y = blink;
  leftPupil.scale.y = blink;
  rightPupil.scale.y = blink;

  // Vara se mueve con la cabeza
  staff.rotation.z = Math.PI / 14 + 0.06 * Math.sin(elapsed * 1.2);

  // Brazos balanceo suave
  leftArm.rotation.x = 0.2 * Math.sin(elapsed * 1.5);
  rightArm.rotation.x = -0.2 * Math.sin(elapsed * 1.5);

  // Túnica se mueve suavemente (ondulación)
  // NOTA: Para evitar errores, comentamos esta parte:
  // const positions = tunicMesh.geometry.attributes.position.array;
  // for (let i = 0; i < positions.length; i += 3) {
  //   positions[i + 2] = 0.05 * Math.sin(elapsed * 2 + i);
  // }
  // tunicMesh.geometry.attributes.position.needsUpdate = true;

  controls.update();
  renderer.render(scene, camera);
}

animate();

window.addEventListener('resize', () => {
  camera.aspect = window.innerWidth / window.innerHeight;
  camera.updateProjectionMatrix();
  renderer.setSize(window.innerWidth, window.innerHeight);
});
</script>
</body>
</html>
