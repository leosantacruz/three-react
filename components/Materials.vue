<template>
  <div>
    <canvas></canvas>
  </div>
</template>

<script>
import * as THREE from "three";
import dat from "dat.gui";
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls.js";
import { FontLoader } from "three/examples/jsm/loaders/FontLoader.js";
import { TextGeometry } from "three/examples/jsm/geometries/TextGeometry.js";
import gsap from "gsap";
import { gunzip } from "zlib";
export default {
  name: "Viewer",
  mounted() {
    const canvas = document.querySelector("canvas");
    const scene = new THREE.Scene();

    //Basic parameters

    const parameters = {
      flatShading: false,
    };

    //Lights
    const light_amb = new THREE.AmbientLight("#FFFFFF", 0.5);
    const light_point = new THREE.PointLight("#FFFFFF", 0.5);
    light_point.position.x = 3;
    light_point.position.y = 1;
    light_point.position.z = 2;

    scene.add(light_amb, light_point);

    //Textures
    const textureLoader = new THREE.TextureLoader();
    const texture_color = textureLoader.load("img/base.jpg");
    const texture_alpha = textureLoader.load("img/opacity.jpg");
    const texture_ambient = textureLoader.load("img/ambient.jpg");
    const texture_roughness = textureLoader.load("img/roughness.jpg");
    const texture_normal = textureLoader.load("img/normal.jpg");
    const texture_matcap = textureLoader.load("img/platinium.jpg");

    const cubeTextureLoader = new THREE.CubeTextureLoader();
    const enviorementTexture = cubeTextureLoader.load([
      "/img/px.jpg",
      "/img/nx.jpg",
      "/img/py.jpg",
      "/img/ny.jpg",
      "/img/pz.jpg",
      "/img/nz.jpg",
    ]);

    //FONT

    const fontLoader = new FontLoader();
    fontLoader.load("fonts/helvetica.json", (font) => {
      const textGeometry = new TextGeometry("Hola Leo", {
        font: font,
        height: 0.2,
        size: 0.5,
        curveSegments: 12,
        bevelEnable: true,
        bevelThickness: 0.03,
        bevelSize: 0.02,
        bevelOffset: 0,
        bevelSegments: 5,
      });
      const textMaterial = new THREE.MeshBasicMaterial({
        color: new THREE.Color("red"),
      });
      const textMesh = new THREE.Mesh(textGeometry, textMaterial);
      textMesh.position.y = 0.5;
      textMesh.position.x = -1;
      scene.add(textMesh);
    });
    //Material
    /* let material = new THREE.MeshMatcapMaterial({
      matcap: texture_matcap,
    });*/

    const material = new THREE.MeshStandardMaterial();
    material.envMap = enviorementTexture;
    material.roughness = 0.2;
    material.metalness = 0.7;
    //material.map = texture_color;

    //gui
    const gui = new dat.GUI();
    gui.add(material, "metalness").min(0).max(1).name("Metalness").step(0.1);
    gui.add(material, "roughness").min(0).max(1).name("Roughness").step(0.1);

    //MESH
    let box = new THREE.Mesh(new THREE.BoxBufferGeometry(1, 1, 1), material);
    let sphere = new THREE.Mesh(
      new THREE.SphereBufferGeometry(0.5, 16, 16),
      material
    );
    let plane = new THREE.Mesh(new THREE.PlaneBufferGeometry(1, 1), material);
    let torus = new THREE.Mesh(
      new THREE.TorusBufferGeometry(0.7, 0.1, 10, 50),
      material
    );

    box.position.x = 1.5;
    sphere.position.x = -1.5;
    torus.position.y = 1.2;
    plane.position.y = -1.2;

    scene.add(box, sphere, plane, torus);

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

    //RENDER
    renderer.setSize(sizes.width, sizes.height);

    const loop = () => {
      controls.update(); //In order to the dumping to be applied
      torus.rotation.y += 0.01;
      box.rotation.x += 0.01;

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
