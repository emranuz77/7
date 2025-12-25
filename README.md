<!DOCTYPE html>
<html lang="en">
<head>
<meta charset="UTF-8" />
<title>PYTHONCHI BOLA — GODMODE</title>
<meta name="viewport" content="width=device-width, initial-scale=1.0" />
<style>
*{margin:0;padding:0;box-sizing:border-box}
html,body{width:100%;height:100%;overflow:hidden;background:#000;font-family:system-ui}
#hint{position:fixed;bottom:15px;width:100%;text-align:center;color:#fff;opacity:.35;font-size:.7rem;z-index:10}
</style>
</head>
<body>
<div id="hint">3D · MUSIC · PHYSICS · REAL-TIME</div>

<script src="https://cdn.jsdelivr.net/npm/three@0.158.0/build/three.min.js"></script>
<script>
/* ================================
   PYTHONCHI BOLA — GODMODE ENGINE
   ================================ */

// SCENE
const scene = new THREE.Scene();
const camera = new THREE.PerspectiveCamera(75, innerWidth/innerHeight, 0.1, 1000);
camera.position.z = 6;

const renderer = new THREE.WebGLRenderer({ antialias:true });
renderer.setSize(innerWidth, innerHeight);
document.body.appendChild(renderer.domElement);

addEventListener('resize',()=>{
  camera.aspect = innerWidth/innerHeight;
  camera.updateProjectionMatrix();
  renderer.setSize(innerWidth, innerHeight);
});

// LIGHTS
scene.add(new THREE.AmbientLight(0xffffff, .6));
const light = new THREE.PointLight(0x00ffff, 2, 50);
light.position.set(5,5,5);
scene.add(light);

// CORE OBJECT (IMPOSSIBLE SHAPE)
const geo = new THREE.IcosahedronGeometry(2, 4);
const mat = new THREE.MeshStandardMaterial({
  color:0x00ffff,
  wireframe:true
});
const core = new THREE.Mesh(geo, mat);
scene.add(core);

// PARTICLES
const pCount = 4000;
const pGeo = new THREE.BufferGeometry();
const pPos = new Float32Array(pCount * 3);
for(let i=0;i<pCount*3;i++) pPos[i] = (Math.random() - .5) * 20;
pGeo.setAttribute('position', new THREE.BufferAttribute(pPos, 3));
const pMat = new THREE.PointsMaterial({ color:0xffffff, size:0.03 });
const particles = new THREE.Points(pGeo, pMat);
scene.add(particles);

// AUDIO
let analyser, dataArray;
navigator.mediaDevices.getUserMedia({ audio:true }).then(stream=>{
  const ctx = new AudioContext();
  const src = ctx.createMediaStreamSource(stream);
  analyser = ctx.createAnalyser();
  analyser.fftSize = 256;
  src.connect(analyser);
  dataArray = new Uint8Array(analyser.frequencyBinCount);
});

// MOUSE
let mx=0,my=0;
addEventListener('mousemove',e=>{
  mx = (e.clientX/innerWidth - .5) * 2;
  my = (e.clientY/innerHeight - .5) * 2;
});

// LOOP
function animate(t){
  requestAnimationFrame(animate);

  let audio = 0;
  if(analyser){
    analyser.getByteFrequencyData(dataArray);
    audio = dataArray.reduce((a,b)=>a+b,0)/dataArray.length/255;
  }

  core.rotation.x += 0.005 + audio * 0.1;
  core.rotation.y += 0.008 + audio * 0.1;
  core.scale.setScalar(1 + audio * 0.8);

  particles.rotation.y += 0.0005 + audio * 0.01;

  camera.position.x += (mx*2 - camera.position.x) * 0.05;
  camera.position.y += (-my*2 - camera.position.y) * 0.05;

  light.intensity = 1 + audio * 3;

  renderer.render(scene, camera);
}
animate();
</script>
</body>
</html>
