<template>
  <main>
    <canvas ref="canvas" class="webgl"></canvas>
  </main>
</template>

<script setup>
import * as THREE from "three";
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls.js";
import { ref, onMounted, onBeforeUnmount } from "vue";
import cubeVertexShader from "../assets/shaders/cube/vertex.glsl";
import cubeFragmentShader from "../assets/shaders/cube/fragment.glsl";
import { RoundedBoxGeometry } from "three/examples/jsm/geometries/RoundedBoxGeometry.js";

// Canvas
const canvas = ref(null);

// Sizes
const sizes = {
  width: window.innerWidth,
  height: window.innerHeight,
};

// Scene
const scene = new THREE.Scene();

// Camera
const camera = new THREE.PerspectiveCamera(
  75,
  sizes.width / sizes.height,
  0.1,
  1000
);
camera.position.set(0, 0, 8);
scene.add(camera);

// Renderer
let renderer = null;

// Controls
let controls = null;

// Objects
let cubeMesh = null;
let cubeMaterial = null;

// Clock
const clock = new THREE.Clock();

// Scene
const initScene = () => {
  // Lights
  const ambientLight = new THREE.AmbientLight(0xffffff, 2);
  scene.add(ambientLight);

  const directionalLight = new THREE.DirectionalLight(0x00ff00, 50);
  directionalLight.position.set(4, 3, 4);
  scene.add(directionalLight);

  const pointLight = new THREE.PointLight(0xff0000, 3.5, 40, 0);
  pointLight.position.set(-2, 2, 2);
  scene.add(pointLight);
  // const pointLightHelper = new THREE.PointLightHelper(pointLight);
  // scene.add(pointLightHelper);

  // const directionalLightHelper = new THREE.DirectionalLightHelper(
  //   directionalLight,
  // );
  // scene.add(directionalLightHelper);
};

// Renderer
const initRenderer = () => {
  if (!canvas.value) return;
  renderer = new THREE.WebGLRenderer({
    canvas: canvas.value,
    alpha: true,
    preserveDrawingBuffer: false,
  });
  renderer.setSize(sizes.width, sizes.height);
  renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2));
};

// Controls
const initControls = () => {
  if (!canvas.value) return;
  controls = new OrbitControls(camera, canvas.value);
  controls.enableDamping = true;
};

const onResize = () => {
  sizes.width = window.innerWidth;
  sizes.height = window.innerHeight;

  camera.aspect = sizes.width / sizes.height;
  camera.updateProjectionMatrix();

  if (renderer) {
    renderer.setSize(sizes.width, sizes.height);
    renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2));
  }
};

// Animation
let animationFrameId;

const tick = () => {
  const elapsedTime = clock.getElapsedTime();

  // Update material
  // material.uniforms.uTime.value = elapsedTime;

  // Rotate mesh
  // if (cubeMesh) cubeMesh.rotation.y = elapsedTime * 0.5;

  // Update controls
  if (controls) controls.update();

  // Render
  if (renderer) renderer.render(scene, camera);

  animationFrameId = window.requestAnimationFrame(tick);
};

// Mounted
onMounted(() => {
  initScene();

  // Material & Mesh
  const materialParameters = { color: "#18e77c" };

  // material = new THREE.ShaderMaterial({
  //   vertexShader: cubeVertexShader,
  //   fragmentShader: cubeFragmentShader,
  //   uniforms: {
  //     uTime: new THREE.Uniform(0),
  //     uColor: new THREE.Uniform(new THREE.Color(materialParameters.color)),
  //   },
  //   side: THREE.DoubleSide,
  //   depthWrite: false,
  //   transparent: true,
  //   blending: THREE.AdditiveBlending,
  // });

  const planeMaterial = new THREE.MeshBasicMaterial({
    color: "#f3f5e1",
  });
  const floorMesh = new THREE.Mesh(
    new THREE.PlaneGeometry(12, 12),
    planeMaterial
  );
  floorMesh.rotation.x = -Math.PI * 0.5;
  floorMesh.position.y = -3;
  floorMesh.position.z = 3;
  scene.add(floorMesh);

  const wallMesh = new THREE.Mesh(
    new THREE.PlaneGeometry(12, 12),
    planeMaterial
  );
  // wallMesh.rotation.x = -Math.PI * 0.5;
  wallMesh.position.y = 3;
  wallMesh.position.z = -3;
  scene.add(wallMesh);

  cubeMaterial = new THREE.MeshPhysicalMaterial({
    side: THREE.DoubleSide,
    transparent: true,
    transmission: 0.99,
    roughness: 0,
    color: new THREE.Color("#9e90f5"),
  });

  cubeMesh = new THREE.Mesh(
    new RoundedBoxGeometry(3, 3, 3, 5, 0.2),
    cubeMaterial
  );
  // mesh = new THREE.Mesh(new THREE.SphereGeometry(3), material);
  // mesh = new THREE.Mesh(new THREE.BoxGeometry(3, 3, 3), material);
  // cubeMesh.rotation.x = Math.PI * 0.25;
  // cubeMesh.rotation.y = Math.PI * 0.25;
  cubeMesh.rotation.y = Math.PI * 0.25;
  scene.add(cubeMesh);

  initRenderer();
  initControls();

  window.addEventListener("resize", onResize);
  tick();
});

// Cleanup
onBeforeUnmount(() => {
  if (animationFrameId) cancelAnimationFrame(animationFrameId);
  window.removeEventListener("resize", onResize);
  renderer?.dispose();
});
</script>

<style scoped>
.webgl {
  position: fixed;
  top: 0;
  left: 0;
  outline: none;
  background: transparent;
}
</style>
