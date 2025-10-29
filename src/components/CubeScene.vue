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
const camera = new THREE.PerspectiveCamera(75, sizes.width / sizes.height, 0.1, 1000);
camera.position.set(0, 0, 8);
scene.add(camera);

// Renderer
let renderer = null;

// Controls
let controls = null;

// Objects
let mesh = null;

// Clock
const clock = new THREE.Clock();

// Scene
const initScene = () => {
  // Lights
  const ambientLight = new THREE.AmbientLight(0xffffff, 2);
  scene.add(ambientLight);

  const directionalLight = new THREE.DirectionalLight(0xeeff00, 2);
  directionalLight.position.set(0, 0.5, 1);
  scene.add(directionalLight);
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

  // Rotate mesh
  // if (mesh) mesh.rotation.y = elapsedTime * 0.5;

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
  const material = new THREE.ShaderMaterial({
    vertexShader: cubeVertexShader,
    fragmentShader: cubeFragmentShader,
    side: THREE.DoubleSide,
    transparent: true
  });
  mesh = new THREE.Mesh(new THREE.BoxGeometry(3, 3, 3), material);
  scene.add(mesh);

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
