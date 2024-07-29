<template>
  <div ref="container"></div>
</template>

<script setup lang="ts">
import * as THREE from "three";
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls";
import * as dat from "dat.gui";
import { onMounted, ref } from "vue";
import gsap from "gsap";

console.log(gsap);
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
      const z = array[i + 2];
      array[i + 2] = z + Math.random();
    }

    const colors = [];
    for (let i = 0; i < plane.geometry.attributes.position.count; i++) {
      colors.push(0, 0.19, 0.4);
    }
    plane.geometry.setAttribute(
      "color",
      new THREE.BufferAttribute(new Float32Array(colors), 3)
    );
  };

  const generateCamera = () => {
    camera.position.x = world.camera.x; // Set the camera position
    camera.position.y = world.camera.y; // Set the camera position
    camera.position.z = world.camera.z;
  };

  const world = {
    plane: {
      width: 25,
      height: 25,
      widthSegments: 25,
      heightSegments: 25,
    },
    camera: {
      x: 0,
      y: 0,
      z: 0,
    },
  };
  gui.add(world.camera, "x", -10, 20).onChange(generateCamera);
  gui.add(world.camera, "y", -10, 20).onChange(generateCamera);
  gui.add(world.camera, "z", -10, 20).onChange(generateCamera);
  gui.add(world.plane, "width", 1, 50).onChange(generatePlane);
  gui.add(world.plane, "height", 1, 50).onChange(generatePlane);
  gui.add(world.plane, "widthSegments", 1, 50).onChange(generatePlane);
  gui.add(world.plane, "heightSegments", 1, 50).onChange(generatePlane);

  const rayCaster = new THREE.Raycaster();
  const scene = new THREE.Scene();
  const camera = new THREE.PerspectiveCamera(
    65,
    window.innerWidth / window.innerHeight,
    0.1,
    1000
  );

  camera.position.z = 5; // Set the camera position
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

  const planeGeometry = new THREE.PlaneGeometry(
    world.plane.width,
    world.plane.height,
    world.plane.widthSegments,
    world.plane.heightSegments
  );
  const planeMaterial = new THREE.MeshPhongMaterial({
    side: THREE.DoubleSide,
    flatShading: true,
    vertexColors: true,
  });
  const plane = new THREE.Mesh(planeGeometry, planeMaterial);
  scene.add(plane);

  const { array } = plane.geometry.attributes.position;
  for (let i = 0; i < array.length; i += 3) {
    const x = array[i ];
    const z = array[i + 2];
    array[i + 2] = z + Math.random();
  }

  const colors = [];
  for (let i = 0; i < plane.geometry.attributes.position.count; i++) {
    colors.push(0, 0.19, 0.4);
  }
  plane.geometry.setAttribute(
    "color",
    new THREE.BufferAttribute(new Float32Array(colors), 3)
  );

  const light = new THREE.DirectionalLight(0xffffff, 2);
  light.position.set(2, 2, 1);
  scene.add(light);

  const backLight = new THREE.DirectionalLight(0x777777, 1);
  backLight.position.set(1, 1, -1);
  scene.add(backLight);

  if (container.value) {
    container.value.appendChild(renderer.domElement);
  }
  const mouse: any = {
    x: undefined,
    y: undefined,
  };

  // Animation loop
  const animate = () => {
    requestAnimationFrame(animate);
    renderer.render(scene, camera);
    rayCaster.setFromCamera(mouse, camera);

    const intersects = rayCaster.intersectObject(plane);

    if (intersects.length > 0) {
      console.log();
      const { color } = intersects[0].object.geometry.attributes;

      color.setX(intersects[0].face?.a, 0.1);
      color.setY(intersects[0].face?.a, 0.5);
      color.setZ(intersects[0].face?.a, 11);

      color.setX(intersects[0].face?.b, 0.1);
      color.setY(intersects[0].face?.b, 0.5);
      color.setZ(intersects[0].face?.b, 11);

      color.setX(intersects[0].face?.c, 0.1);
      color.setY(intersects[0].face?.c, 0.5);
      color.setZ(intersects[0].face?.c, 11);

      intersects[0].object.geometry.attributes.color.needsUpdate = true;

      const initialColor = {
        r: 0,
        g: 0.19,
        b: 0.4,
      };
      const hoverColor = {
        r: 0.1,
        g: 0.5,
        b: 1,
      };
      gsap.to(hoverColor, {
        r: initialColor.r,
        b: initialColor.b,
        g: initialColor.g,
        onUpdate: () => {
          color.setX(intersects[0].face?.a, hoverColor.r);
          color.setY(intersects[0].face?.a, hoverColor.g);
          color.setZ(intersects[0].face?.a, hoverColor.b);

          color.setX(intersects[0].face?.b, hoverColor.r);
          color.setY(intersects[0].face?.b, hoverColor.g);
          color.setZ(intersects[0].face?.b, hoverColor.b);

          color.setX(intersects[0].face?.c, hoverColor.r);
          color.setY(intersects[0].face?.c, hoverColor.g);
          color.setZ(intersects[0].face?.c, hoverColor.b);
        },
      });
    }
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

  window.addEventListener("mousemove", (event) => {
    mouse.x = (event.clientX / innerWidth) * 2 - 1;
    mouse.y = -(event.clientY / innerHeight) * 2 + 1;
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
