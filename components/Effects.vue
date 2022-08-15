<template>
  <div>
    <div id="page">
      <UIContent></UIContent>
    </div>
    <canvas></canvas>
  </div>
</template>

<script>
import * as THREE from "three";
import dat from "dat.gui";
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls.js";
import { FontLoader } from "three/examples/jsm/loaders/FontLoader.js";
import { GLTFLoader } from "three/examples/jsm/loaders/GLTFLoader.js";

import { TextGeometry } from "three/examples/jsm/geometries/TextGeometry.js";
import { DragControls } from "three/examples/jsm/controls/DragControls";
import { TransformControls } from "three/examples/jsm/controls/TransformControls.js";
import { RenderPass } from "three/examples/jsm/postprocessing/RenderPass.js";
import { ShaderPass } from "three/examples/jsm/postprocessing/ShaderPass.js";
import { EffectComposer } from "three/examples/jsm/postprocessing/EffectComposer.js";
import { GlitchPass } from "three/examples/jsm/postprocessing/GlitchPass.js";
import { LuminosityShader } from "three/examples/jsm/shaders/LuminosityShader.js";

import gsap from "gsap";
import { gunzip } from "zlib";
export default {
  name: "Viewer",
  head: {
    meta: [
      { name: "viewport", content: "width=device-width, initial-scale=1" },
    ],
  },
  data() {
    return {
      writeText: "Move it",
      textMaterial: {},
      textMesh: {},
      scene: {},
      fontType: {},
      selectedObject: {},
      objects: [],
      composer: {},
      cameraAnimations: {},
    };
  },
  mounted() {
    const canvas = document.querySelector("canvas");
    let mesh = { rotation: { y: 0 }, position: { z: 0 } };

    const axesHelper = new THREE.AxesHelper(1);
    // this.scene.add(axesHelper);

    this.scene = new THREE.Scene();

    this.scene.background = new THREE.Color("#000000");
    //Basic parameters
    this.textMaterial = new THREE.MeshStandardMaterial({
      color: new THREE.Color("red"),
    });
    const parameters = {
      flatShading: false,
    };

    const loader = new GLTFLoader();
    let boxHelper = {};
    loader.load("models/tren/proyectotrencito_001.gltf", (gltf) => {
      mesh = gltf.scene;
      mesh.scale.set(0.1, 0.1, 0.1);
      mesh.position.y = -0.13;
      this.scene.add(mesh);
      boxHelper = new THREE.BoxHelper(mesh, 0xffff00);

      // this.scene.add(boxHelper);
      mesh.traverse((o) => {
        if (o.isMesh) {
          o.castShadow = true;
          o.material.roughness = 0.8;
          o.material.metalness = 0.1;
        }
      });

      // gui.add(mesh.position, "x").min(-3).max(3);
      // gui.add(mesh.position, "y").min(-3).max(3).step(0.01);

      return;
      const box1 = new THREE.Box3().setFromObject(mesh);
      const center1 = box1.getCenter(new THREE.Vector3());

      mesh.position.x += mesh.position.x - center1.x;
      mesh.position.y += mesh.position.y - center1.y;
      mesh.position.z += mesh.position.z - center1.z;
      // mesh.translate(0, 0, 0);

      this.scene.add(mesh);
      let grupito = new THREE.Group();

      mesh.traverse((o) => {
        if (o.isMesh) {
          o.material.roughness = 0.2;
          o.material.metalness = 0.5;
          grupito.attach(o);
        }
      });

      this.scene.add(grupito);

      moveControls.attach(grupito);
      grupito.rotation.x = Math.PI * -2.5;

      return;
      mesh.position.set(0, 0, 0);

      var box = new THREE.Box3().setFromObject(mesh);
      var center = new THREE.Vector3();
      box.getCenter(center);
      mesh.position.sub(center); // center the model
      mesh.rotation.y = Math.PI; // rotate the model
      this.scene.add(mesh);
      return;

      mesh.scale.set(0.1, 0.1, 0.1);
      mesh.position.set(0, 0, 0);
      console.log(mesh);

      var newMaterial = new THREE.MeshStandardMaterial({ color: "#FFFFFF" });
      mesh.traverse((o) => {
        if (o.isMesh && o.material.ccc) {
          o.material = newMaterial;
          o.material.roughness = 0.2;
          o.material.metalness = 0.7;
          o.material.envMap = enviorementTexture;
        }
      });

      mesh.position.set(0, 0, 0);
      group.add(mesh);
      this.scene.add(group);
    });

    //Lights
    //best performance : ambient and hemisphere lights type
    //Always change the mapsize, far and near values
    const light_amb = new THREE.AmbientLight("#FFFFFF", 0.5);

    const light_point1 = new THREE.PointLight("#FFFFFF", 0.5);
    light_point1.position.set(0, 1, 0.5);

    const directional_light = new THREE.DirectionalLight("#ffffff", 0.4);

    directional_light.castShadow = true;
    directional_light.shadow.mapSize.width = 1024;
    directional_light.shadow.mapSize.height = 1024;
    directional_light.shadow.camera.far = 6;
    directional_light.shadow.radius = 10;
    directional_light.position.z = -1.5;
    directional_light.position.y = 3;

    const spotLight = new THREE.SpotLight("#FFFFFF", 1, 0, Math.PI * 0.3);
    spotLight.position.set(-1.2, 1.2, -1.6);
    spotLight.shadow.camera.near = 1;
    spotLight.shadow.camera.far = 6;
    spotLight.shadow.mapSize.width = 1024;
    spotLight.shadow.mapSize.height = 1024;
    spotLight.shadow.camera.fov = 30;
    spotLight.penumbra = 0.05;
    spotLight.castShadow = true;
    // //gui
    // const gui = new dat.GUI();
    // gui.add(directional_light.position, "x").min(-5).max(5);
    // gui.add(directional_light.position, "y").min(-5).max(5);
    // gui.add(directional_light.position, "z").min(-5).max(5);

    this.scene.add(spotLight.target);
    spotLight.target.position.set(0, -5, 0);

    this.scene.add(light_point1, spotLight);

    //Textures
    const textureLoader = new THREE.TextureLoader();
    const texture_matcap = textureLoader.load("img/matcap.jpg");
    const texture_floor = textureLoader.load("img/1K-tiling_31_basecolor.jpg");
    const texture_floor_roughness = textureLoader.load(
      "img/1K-1K-tiling_31_roughness.jpg"
    );
    let repeatWrapping = THREE.RepeatWrapping;

    texture_floor.wrapS = repeatWrapping;
    texture_floor.wrapT = repeatWrapping;
    texture_floor.repeat.set(30, 30);
    texture_floor_roughness.wrapS = repeatWrapping;
    texture_floor_roughness.wrapT = repeatWrapping;
    texture_floor_roughness.repeat.set(30, 30);
    const cubeTextureLoader = new THREE.CubeTextureLoader();
    const enviorementTexture = cubeTextureLoader.load([
      "/img/px.jpg",
      "/img/nx.jpg",
      "/img/py.jpg",
      "/img/ny.jpg",
      "/img/pz.jpg",
      "/img/nz.jpg",
    ]);

    // Adding a plane
    let plane = new THREE.Mesh(
      new THREE.PlaneBufferGeometry(30, 30),
      //  new THREE.ShadowMaterial({ opacity: 0.5 })
      new THREE.MeshStandardMaterial({
        // color: new THREE.Color("#FFF"),
        map: texture_floor,
        roughnessMap: texture_floor_roughness,
        displacementMap: texture_floor_roughness,
      })
    );
    plane.rotation.x = Math.PI * -0.5;
    plane.receiveShadow = true;
    this.scene.add(plane);
    let sizes = {
      width: window.innerWidth,
      height: window.innerHeight,
    };

    //Add the text
    const fontLoader = new FontLoader();
    fontLoader.load("fonts/helvetica.json", (font) => {
      const textGeometry = new TextGeometry("el tren", {
        font: font,
        height: 0.01,
        size: 0.2,
        curveSegments: 12,
        bevelEnabled: true,
        bevelThickness: 0.03,
        bevelSize: 0,
        bevelOffset: 0,
        bevelSegments: 5,
      });
      this.textMesh = new THREE.Mesh(
        textGeometry,
        new THREE.MeshStandardMaterial()
      );
      this.textMesh.castShadow = true;
      //this.scene.add(this.textMesh);
      this.textMesh.position.x = -1.1;

      // const gui = new dat.GUI();
      // gui.add(this.textMesh.position, "x").min(-5).max(5);
      // gui.add(this.textMesh.position, "y").min(-5).max(5);
      // gui.add(this.textMesh.position, "z").min(-5).max(5);
    });

    //CAMERA
    const camera = new THREE.PerspectiveCamera(75, sizes.width / sizes.height);
    camera.position.y = 1;
    camera.position.x = -1;
    camera.position.z = 1;

    this.scene.add(camera);

    // const gui = new dat.GUI();
    // gui.add(camera.position, "y").min(-5).max(5);
    // gui.add(camera.position, "x").min(-5).max(5);
    // gui.add(camera.position, "z").min(-5).max(5);
    this.cameraAnimations = {
      top: () => {
        gsap.to(camera.position, { y: 3.8, duration: 2 });
      },
      close: () => {
        gsap.to(camera.position, { y: 0.5, x: -0.3, z: 0.4, duration: 1 });
      },
    };
    this.cameraAnimations.top();
    // gui.add(cameraAnimations, "top");
    // gui.add(cameraAnimations, "close");
    window.addEventListener("resize", () => {
      sizes = {
        width: window.innerWidth,
        height: window.innerHeight,
      };
      camera.aspect = sizes.width / sizes.height;
      camera.updateProjectionMatrix();

      renderer.setSize(window.innerWidth, window.innerHeight);
      renderer.physicallyCorrectLights = true;
      renderer.setPixelRatio(Math.min(window.devicePixelRatio, 2)); //set a better pixel ratio depending on the device
    });

    const orbitControls = new OrbitControls(camera, canvas);
    orbitControls.target.y = 0;
    orbitControls.update();
    orbitControls.enableDamping = true; //smooth movment in the camera
    //RENDERER
    const renderer = new THREE.WebGLRenderer({
      canvas,
    });
    this.composer = new EffectComposer(renderer);
    this.composer.addPass(new RenderPass(this.scene, camera));
    // this.composer.addPass(new GlitchPass());
    // this.composer.addPass(new ShaderPass(LuminosityShader));
    renderer.setSize(sizes.width, sizes.height);

    renderer.shadowMap.enabled = true;
    renderer.shadowMap.type = THREE.PCFSoftShadowMap;

    document.body.appendChild(renderer.domElement);
    let miniBoxGeometry = new THREE.BoxBufferGeometry(0.2, 0.2, 0.2);
    //Change the pivot
    miniBoxGeometry.applyMatrix(new THREE.Matrix4().makeTranslation(0, 0.1, 0));

    const miniBox = new THREE.Mesh(
      miniBoxGeometry,
      new THREE.MeshStandardMaterial({ color: "red", wireframe: false })
    );
    miniBox.position.y = 0;
    miniBox.position.z = 0.2;
    miniBox.castShadow = true;

    // this.scene.add(miniBox);
    // this.objects.push(miniBox);
    // miniBox.cursor = "pointer";

    document.onkeypress = (e) => {
      e = e || window.event;
      //FORWARD
      if (e.keyCode == 119) {
        // mesh.translateZ(0.2);
        // gsap.to(mesh.position, { z: mesh.position.z + 0.2, duration: 0.2 });
      }
      //BACKWARD
      if (e.keyCode == 115) {
        // mesh.translateZ(-0.2);
        // gsap.to(mesh.position, { z: mesh.position.z - 0.2, duration: 0.2 });
      }
      //ROTATE LEFT
      if (e.keyCode == 97) {
        gsap.to(mesh.rotation, { y: mesh.rotation.y + 0.5, duration: 0.2 });
        mesh.updateMatrix();
        mesh.updateMatrixWorld();
      }
      //ROTATE RIGHT
      if (e.keyCode == 100) {
        gsap.to(mesh.rotation, { y: mesh.rotation.y - 0.5, duration: 0.2 });
      }
      if (e.keyCode == 99) {
        let newBox = this.selectedObject.clone();
        newBox.cursor = "pointer";

        newBox.position.x += 0.2;
        this.scene.add(newBox);

        moveControls.attach(newBox);
        this.objects.push(newBox);
        this.selectedObject = newBox;
      }
    };

    //BEST WAY TO MOVE AN OBJECT
    const moveControls = new TransformControls(camera, renderer.domElement);
    moveControls.addEventListener("dragging-changed", function (event) {
      orbitControls.enabled = !event.value;
    });

    this.scene.add(moveControls);
    moveControls.mode = "translate";
    moveControls.showY = true;
    moveControls.translationSnap = 0.2;

    //SECOND WAY TO MOVE AN OBJECT
    // const moveControls = new DragControls(
    //   [sphere],
    //   camera,
    //   renderer.domElement
    // );
    // moveControls.addEventListener("drag", function (event) {
    //   event.object.material.opacity = 0.33;
    //   event.object.position.y = 0.2;
    // });
    document.addEventListener("click", (event) => {
      console.log("Clicked");
      event.preventDefault();
      var mouse3D = new THREE.Vector3(
        (event.clientX / window.innerWidth) * 2 - 1,
        -(event.clientY / window.innerHeight) * 2 + 1,
        0.5
      );
      var raycaster = new THREE.Raycaster();
      raycaster.setFromCamera(mouse3D, camera);
      var intersects = raycaster.intersectObjects(this.objects);
      console.log(intersects);
      if (intersects.length > 0) {
        this.selectedObject = intersects[0].object;
        //Change random color
        // this.selectedObject.material.color.setHex(Math.random() * 0xffffff);
        // moveControls.attach(this.selectedObject);
      }
    });
    let count = 0;
    const loop = () => {
      renderer.render(this.scene, camera);
      texture_floor.offset.y -= 0.02;
      if (this.textMesh.position) {
        count = count + 0.04;
        this.textMesh.position.y = Math.abs(Math.cos(count)) * 0.3;
      }
      if (mesh.translateZ) {
        // mesh.translateZ(0.01);
        let limit = 1.3;
        if (
          mesh.position.z > limit ||
          mesh.position.z < -limit ||
          mesh.position.x < -limit ||
          mesh.position.x > limit
        ) {
          // mesh.translateZ(-0.01);
        }
      }
      orbitControls.update(); //In order to the dumping to be applied
      this.composer.render();
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
      this.textMesh.position.y = 0.3;
      this.textMesh.castShadow = true;

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
@import url("https://fonts.googleapis.com/css2?family=Lora&display=swap");
body {
  font-family: "Lora", sans-serif;
  margin: 0;
  padding: 0;
}
.writeText {
  position: absolute;
  top: 10;
  left: 10px;
}
#page {
  position: fixed;
  width: 100%;
  z-index: 10;
  height: 100vh;
  box-sizing: border-box;
}
</style>
