<template>
  <div>
    <canvas></canvas>
  </div>
</template>

<script>
import * as THREE from "three";
import dat from "dat.gui";
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls.js";
import gsap from "gsap";
export default {
  name: "Viewer",
  mounted() {
    const canvas = document.querySelector("canvas");

    //TEXTURES
    const loadingManager = new THREE.LoadingManager();
    loadingManager.onStart = () => {
      console.log("Textures began loading...");
    };
    loadingManager.onLoaded = () => {
      console.log("Textures are loaded");
    };
    loadingManager.onProgress = () => {
      console.log("Texture being loading...");
    };
    const textureLoader = new THREE.TextureLoader(loadingManager);
    const texture_base = textureLoader.load("img/diamond_ore.png");
    const texture_alpha = textureLoader.load("img/opacity.jpg");
    const texture_ambient = textureLoader.load("img/ambient.jpg");
    const texture_roughness = textureLoader.load("img/roughness.jpg");
    const texture_normal = textureLoader.load("img/normal.jpg");
    const texture_height = textureLoader.load("img/height.jpg");
    const texture_metalness = textureLoader.load("img/metalness.jpg");
    const texture_mapcap = textureLoader.load("img/46PS9.jpg");

    //Filtro para amejorar que no tienen buena calidad
    //TinyPNG
    texture_base.magFilter = THREE.NearestFilter;

    let parameters = {
      color: "#ffffff",
      spin: () => {
        gsap.to(mesh.rotation, { y: mesh.rotation.y + 3, duration: 1 });
      },
    };
    const scene = new THREE.Scene();
    const geometry = new THREE.BoxBufferGeometry(1, 1, 1, 10, 10, 10);
    const material = new THREE.MeshBasicMaterial({
      color: parameters.color,
      map: texture_base,
      wireframe: false,
    });
    const mesh = new THREE.Mesh(geometry, material);
    scene.add(mesh);

    let sizes = {
      width: window.innerWidth,
      height: window.innerHeight,
    };

    //CAMERA
    const camera = new THREE.PerspectiveCamera(75, sizes.width / sizes.height);
    camera.position.z = 3;
    scene.add(camera);

    window.addEventListener("resize", () => {
      sizes = {
        width: window.innerWidth,
        height: window.innerHeight,
      };
      camera.aspect = sizes.width / sizes.height;
      camera.updateProjectionMatrix();

      renderer.setSize(window.innerWidth, window.innerHeight);
      renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2)); //set a better pixel ratio depending on the device
    });

    const controls = new OrbitControls(camera, canvas);
    controls.target.y = 0;
    controls.update();
    controls.enableDamping = true; //smooth movment in the camera
    const renderer = new THREE.WebGLRenderer({
      canvas,
    });

    //DAT GUI
    const gui = new dat.GUI();
    gui.add(mesh.position, "y").min(-3).max(3).name("Position").step(0.01);
    gui.add(mesh.scale, "y").min(-3).max(3).name("Scale").step(0.01);

    gui.add(mesh, "visible");
    gui.add(mesh.material, "wireframe");
    gui.addColor(parameters, "color").onChange(() => {
      material.color.set(parameters.color);
    });
    gui.add(parameters, "spin").name("Crazy spin");

    //RENDER
    renderer.setSize(sizes.width, sizes.height);

    const loop = () => {
      controls.update(); //In order to the dumping to be applied
      renderer.render(scene, camera);
      window.requestAnimationFrame(loop);
    };

    loop();
  },
};
</script>

<style>
body {
  margin: 0;
  padding: 0;
}
</style>
