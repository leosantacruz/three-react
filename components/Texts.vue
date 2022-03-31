<template>
  <div>
    <input
      class="writeText"
      type="text"
      @change="textChange()"
      v-model="writeText"
      placeholder="write something here"
    />
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
  data() {
    return {
      writeText: "Hola Leo!",
      textMaterial: {},
      textMesh: {},
      scene: {},
      fontType: {},
    };
  },
  mounted() {
    const canvas = document.querySelector("canvas");
    this.scene = new THREE.Scene();

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

    this.scene.add(light_amb, light_point);

    //Textures
    const textureLoader = new THREE.TextureLoader();
    const texture_matcap = textureLoader.load("img/matcap.jpg");

    const cubeTextureLoader = new THREE.CubeTextureLoader();
    const enviorementTexture = cubeTextureLoader.load([
      "/img/px.jpg",
      "/img/nx.jpg",
      "/img/py.jpg",
      "/img/ny.jpg",
      "/img/pz.jpg",
      "/img/nz.jpg",
    ]);

    this.textMaterial = new THREE.MeshMatcapMaterial({
      matcap: texture_matcap,
    });

    //FONT

    const fontLoader = new FontLoader();
    fontLoader.load("fonts/helvetica.json", (font) => {
      this.fontType = font;
      this.addText("Hola");

      //Bounding box centering technique
      /* textGeometry.computeBoundingBox();
      textGeometry.translate(
        -textGeometry.boundingBox.max.x / 2,
        -textGeometry.boundingBox.max.y / 2,
        -textGeometry.boundingBox.max.z / 2
      );*/

      //Adding random donuts
      let donutGeometry = new THREE.TorusBufferGeometry(0.2, 0.1, 10, 25);

      for (let i = 0; i < 100; i++) {
        const donut = new THREE.Mesh(donutGeometry, this.textMaterial);
        donut.position.x = Math.random() * (4 + 4) - 4;
        donut.position.y = Math.random() * (4 + 4) - 4;
        donut.position.z = Math.random() * (4 + 4) - 4;
        donut.rotation.x = Math.random() * (4 + 4) - 4;
        donut.rotation.y = Math.random() * (4 + 4) - 4;
        this.scene.add(donut);
      }
    });

    //gui
    const gui = new dat.GUI();
    //MESH

    let sizes = {
      width: window.innerWidth,
      height: window.innerHeight,
    };

    //CAMERA
    const camera = new THREE.PerspectiveCamera(75, sizes.width / sizes.height);
    camera.position.z = 3;
    this.scene.add(camera);

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

      renderer.render(this.scene, camera);
      window.requestAnimationFrame(loop);
    };

    loop();
  },
  methods: {
    addText(text) {
      const textGeometry = new TextGeometry(this.writeText, {
        font: this.fontType,
        height: 0.2,
        size: 0.5,
        curveSegments: 12,
        bevelEnabled: true,
        bevelThickness: 0.03,
        bevelSize: 0,
        bevelOffset: 0,
        bevelSegments: 5,
      });

      textGeometry.center();

      this.textMesh = new THREE.Mesh(textGeometry, this.textMaterial);
      this.scene.add(this.textMesh);
    },
    textChange() {
      this.scene.remove(this.textMesh);
      this.addText();
    },
  },
};
</script>

<style>
body {
  margin: 0;
  padding: 0;
}
.writeText {
  position: absolute;
  top: 10;
  left: 10px;
}
</style>
