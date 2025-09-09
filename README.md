<!DOCTYPE html>
<html lang="es">
<head>
<meta charset="UTF-8">
<title>Universo Animado - Cometa GERAL</title>
<meta name="viewport" content="width=device-width, initial-scale=1.0">
<style>
  body { margin:0; overflow:hidden; background:#000; }
  canvas { display:block; }
  .label {
    color: white;
    font-family: Arial, Helvetica, sans-serif;
    font-size: 12px;
    text-shadow: 0 0 5px black;
    pointer-events: none;
  }
</style>
</head>
<body>

<!-- Three.js y extensiones vía CDN -->
<script src="https://unpkg.com/three@0.161.0/build/three.min.js"></script>
<script src="https://unpkg.com/three@0.161.0/examples/js/controls/OrbitControls.js"></script>
<script src="https://unpkg.com/three@0.161.0/examples/js/renderers/CSS2DRenderer.js"></script>

<script>
// ===========================
// ESCENA, CÁMARA Y RENDERERS
// ===========================
const scene = new THREE.Scene();
const camera = new THREE.PerspectiveCamera(60, window.innerWidth/window.innerHeight, 0.1, 5000);
camera.position.set(0, 200, 500);

const renderer = new THREE.WebGLRenderer({antialias:true});
renderer.setPixelRatio(window.devicePixelRatio);
renderer.setSize(window.innerWidth, window.innerHeight);
renderer.shadowMap.enabled = true;
document.body.appendChild(renderer.domElement);

const labelRenderer = new THREE.CSS2DRenderer();
labelRenderer.setSize(window.innerWidth, window.innerHeight);
labelRenderer.domElement.style.position="absolute";
labelRenderer.domElement.style.top="0px";
document.body.appendChild(labelRenderer.domElement);

const controls = new THREE.OrbitControls(camera,labelRenderer.domElement);
controls.enableDamping = true;
controls.minDistance = 50;
controls.maxDistance = 1500;

// ===========================
// ESTRELLAS DE FONDO
// ===========================
function crearEstrellas(){
  const geo = new THREE.BufferGeometry();
  const count = 800; // Optimizado
  const positions = [];
  for(let i=0;i<count;i++){
    positions.push((Math.random()-0.5)*4000);
    positions.push((Math.random()-0.5)*4000);
    positions.push((Math.random()-0.5)*4000);
  }
  geo.setAttribute("position", new THREE.Float32BufferAttribute(positions,3));
  const mat = new THREE.PointsMaterial({color:0xffffff,size:1});
  return new THREE.Points(geo, mat);
}
scene.add(crearEstrellas());

// ===========================
// LUCES
// ===========================
const ambientLight = new THREE.AmbientLight(0xffffff,1.2);
scene.add(ambientLight);

const sunLight = new THREE.PointLight(0xffffff,3,5000);
sunLight.castShadow = true;
scene.add(sunLight);

// ===========================
// SOL
// ===========================
const sunGeo = new THREE.SphereGeometry(25,32,32);
const sunMat = new THREE.MeshBasicMaterial({color:0xffcc00});
const sun = new THREE.Mesh(sunGeo, sunMat);
scene.add(sun);

// Halo del Sol
const sunGlowGeo = new THREE.SphereGeometry(30,32,32);
const sunGlowMat = new THREE.MeshBasicMaterial({color:0xffdd55, transparent:true, opacity:0.3});
const sunGlow = new THREE.Mesh(sunGlowGeo, sunGlowMat);
scene.add(sunGlow);

// ===========================
// FUNCIÓN CREAR PLANETAS
// ===========================
function crearPlaneta(color, size, distancia, velocidad, rotacion, nombre){
  const orbit = new THREE.Object3D();
  scene.add(orbit);

  const geo = new THREE.SphereGeometry(size,32,32);
  const mat = new THREE.MeshPhongMaterial({color:color, shininess:50});
  const planet = new THREE.Mesh(geo,mat);
  planet.position.x = distancia;
  planet.rotation.y = Math.random()*Math.PI*2;
  orbit.add(planet);

  const div = document.createElement("div");
  div.className = "label";
  div.textContent = nombre;
  const label = new THREE.CSS2DObject(div);
  planet.add(label);

  return {orbit, planet, velocidad, rotacion};
}

// ===========================
// PLANETAS
// ===========================
const tierra = crearPlaneta(0x2233ff,6,70,0.002,0.01,"Tierra");
const marte = crearPlaneta(0xff3300,4,100,0.0016,0.008,"Marte");
const jupiter = crearPlaneta(0xcc8844,14,150,0.001,0.004,"Júpiter");
const saturno = crearPlaneta(0xffd27f,11,220,0.0008,0.003,"Saturno");

// Lunas de la Tierra
const moonGeo = new THREE.SphereGeometry(1.5,16,16);
const moonMat = new THREE.MeshPhongMaterial({color:0xaaaaaa});
const moon = new THREE.Mesh(moonGeo,moonMat);
moon.position.x = 10;
const moonOrbit = new THREE.Object3D();
moonOrbit.add(moon);
tierra.planet.add(moonOrbit);

// Lunas de Júpiter
for(let i=0;i<4;i++){
  const lunaGeo = new THREE.SphereGeometry(1.2,16,16);
  const lunaMat = new THREE.MeshPhongMaterial({color:0xbbbbbb});
  const luna = new THREE.Mesh(lunaGeo,lunaMat);
  luna.position.x = 15+i*3;
  const orb = new THREE.Object3D();
  orb.add(luna);
  jupiter.planet.add(orb);
}

// Anillos de Saturno
const ringGeo = new THREE.RingGeometry(15,22,64);
const ringMat = new THREE.MeshPhongMaterial({color:0xd2b48c, side:THREE.DoubleSide, transparent:true, opacity:0.6});
const ring = new THREE.Mesh(ringGeo,ringMat);
ring.rotation.x = Math.PI/2;
saturno.planet.add(ring);

// ===========================
// COMETA GERAL
// ===========================
const cometGeo = new THREE.SphereGeometry(3,16,16);
const cometMat = new THREE.MeshBasicMaterial({color:0xffffff});
const comet = new THREE.Mesh(cometGeo, cometMat);
scene.add(comet);

// Cola dinámica ligera
const tailGeo = new THREE.BufferGeometry();
const tailMat = new THREE.PointsMaterial({color:0x00ffff,size:1.2,transparent:true,opacity:0.7});
const tailCount = 200;
const tailPositions = new Float32Array(tailCount*3);
tailGeo.setAttribute("position", new THREE.BufferAttribute(tailPositions,3));
const tailPoints = new THREE.Points(tailGeo, tailMat);
scene.add(tailPoints);

// Label cometa
const cometLabelDiv = document.createElement("div");
cometLabelDiv.className = "label";
cometLabelDiv.textContent = "GERAL";
const cometLabel = new THREE.CSS2DObject(cometLabelDiv);
comet.add(cometLabel);

// Órbita elíptica del cometa
const a = 350, b = 150, inclinacion = 0.4;
let theta = 0;

// ===========================
// ANIMACIÓN
// ===========================
function animate(){
  requestAnimationFrame(animate);

  // Rotación de planetas sobre su eje
  [tierra, marte, jupiter, saturno].forEach(p=>{
    p.orbit.rotation.y += p.velocidad;
    p.planet.rotation.y += p.rotacion;
  });

  // Cometa
  theta += 0.008;
  comet.position.x = a*Math.cos(theta);
  comet.position.z = b*Math.sin(theta);
  comet.position.y = Math.sin(theta*1.5)*b*inclinacion;

  // Cola
  for(let i=tailCount-1;i>0;i--){
    tailPositions[i*3] = tailPositions[(i-1)*3];
    tailPositions[i*3+1] = tailPositions[(i-1)*3+1];
    tailPositions[i*3+2] = tailPositions[(i-1)*3+2];
  }
  tailPositions[0]=comet.position.x;
  tailPositions[1]=comet.position.y;
  tailPositions[2]=comet.position.z;
  tailGeo.attributes.position.needsUpdate=true;

  controls.update();
  renderer.render(scene,camera);
  labelRenderer.render(scene,camera);
}
animate();

// ===========================
// RESPONSIVO
// ===========================
window.addEventListener("resize",()=>{
  camera.aspect = window.innerWidth/window.innerHeight;
  camera.updateProjectionMatrix();
  renderer.setSize(window.innerWidth,window.innerHeight);
  labelRenderer.setSize(window.innerWidth,window.innerHeight);
});
</script>

</body>
</html>
