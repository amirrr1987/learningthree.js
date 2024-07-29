<template>
  <div ref="container"></div>
</template>

<script setup lang="ts">
import * as THREE from "three";
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls";
console.log("🚀 ~ OrbitControls:", OrbitControls);
import * as dat from "dat.gui";
import { onMounted, ref } from "vue";

// Create a reference to the container div
const container = ref<HTMLDivElement | null>(null);

onMounted(() => {
  if (!container.value) return;

  const gui = new dat.GUI();

  const generatePlane = () => {
    plane.geometry.dispose();
    plane.geometry = new THREE.PlaneGeometry(
      world.plane.width,
      world.plane.height,
      world.plane.widthSegments,
      world.plane.heightSegments
    );

    const { array } = plane.geometry.attributes.position;
    for (let i = 0; i < array.length; i += 3) {
      const x = array[i];
      const y = array[i + 1];
      const z = array[i + 2];
      array[i + 2] = z + Math.random();
    }
  };

  const world = {
    plane: {
      width: 10,
      height: 10,
      widthSegments: 10,
      heightSegments: 10,
    },
  };
  gui.add(world.plane, "width", 1, 20).onChange(generatePlane);
  gui.add(world.plane, "height", 1, 20).onChange(generatePlane);
  gui.add(world.plane, "widthSegments", 1, 20).onChange(generatePlane);
  gui.add(world.plane, "heightSegments", 1, 20).onChange(generatePlane);

  const scene = new THREE.Scene();
  const camera = new THREE.PerspectiveCamera(
    75,
    window.innerWidth / window.innerHeight,
    0.1,
    1000
  );

  camera.position.z = 10; // Set the camera position
  camera.position.x = 1; // Set the camera position
  camera.position.y = 1; // Set the camera position

  const renderer = new THREE.WebGLRenderer();
  renderer.setSize(window.innerWidth, window.innerHeight);
  renderer.setPixelRatio(devicePixelRatio);

  new OrbitControls(camera, renderer.domElement);

  //   const cubeGeometry = new THREE.BoxGeometry(1, 1, 1);
  //   const cubeMaterial = new THREE.MeshBasicMaterial({ color: 0x00ff00 });
  //   const cube = new THREE.Mesh(cubeGeometry, cubeMaterial);
  //   scene.add(cube);

  const planeGeometry = new THREE.PlaneGeometry(10, 10, 10, 10);
  const planeMaterial = new THREE.MeshPhongMaterial({
    color: 0x0000ff,
    side: THREE.DoubleSide,
    flatShading: true,
  });
  const plane = new THREE.Mesh(planeGeometry, planeMaterial);
  scene.add(plane);

  const { array } = plane.geometry.attributes.position;
  for (let i = 0; i < array.length; i += 3) {
    const x = array[i];
    const y = array[i + 1];
    const z = array[i + 2];
    array[i + 2] = z + Math.random();
  }

  const light = new THREE.DirectionalLight(0xffffff, 1);
  light.position.set(1, 1, 1);
  scene.add(light);

  const backLight = new THREE.DirectionalLight(0x777777, 1);
  backLight.position.set(1, 1, -1);
  scene.add(backLight);

  if (container.value) {
    container.value.appendChild(renderer.domElement);
  }

  // Animation loop
  const animate = () => {
    requestAnimationFrame(animate);
    renderer.render(scene, camera);
    // cube.rotation.x += 0.01;
    // cube.rotation.y += 0.01;

    // plane.rotation.x += 0.01;
    // plane.rotation.y += 0.01;
  };

  animate();

  // Handle window resize
  window.addEventListener("resize", () => {
    camera.aspect = window.innerWidth / window.innerHeight;
    camera.updateProjectionMatrix();
    renderer.setSize(window.innerWidth, window.innerHeight);
  });
});
</script>

<style scoped>
div {
  width: 100%;
  height: 100vh;
  overflow: hidden;
}
</style>
