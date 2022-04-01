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
import { DragControls } from "three/examples/jsm/controls/DragControls";
import { TransformControls } from "three/examples/jsm/controls/TransformControls.js";

import gsap from "gsap";
import { gunzip } from "zlib";
export default {
  name: "Viewer",
  data() {
    return {
      writeText: "Move it",
      textMaterial: {},
      textMesh: {},
      scene: {},
      fontType: {},
      selectedObject: {},
      objects: [],
    };
  },
  mounted() {
    const canvas = document.querySelector("canvas");

    //gui
    const gui = new dat.GUI();
    this.scene = new THREE.Scene();

    //Basic parameters

    const parameters = {
      flatShading: false,
    };

    //Lights
    //best performance : ambient and hemisphere lights type
    //Always change the mapsize, far and near values
    const light_amb = new THREE.AmbientLight("#FFFFFF", 0.01);
    const directional_light = new THREE.DirectionalLight("#ffffff", 0.1);
    const directionalLightHelper = new THREE.DirectionalLightHelper(
      directional_light
    );
    directional_light.castShadow = true;
    directional_light.shadow.mapSize.width = 1024;
    directional_light.shadow.mapSize.height = 1024;
    directional_light.shadow.camera.far = 6;
    directional_light.shadow.radius = 10;
    directional_light.position.z = -1.5;
    // this.scene.add(directionalLightHelper);
    this.scene.add(directional_light);
    gui.add(directional_light.position, "x").min(-4).max(4).step(0.01);
    gui.add(directional_light.position, "z").min(-4).max(4).step(0.01);

    const hemis_light = new THREE.HemisphereLight("#1a2238", "#bf6d02", 0.5);

    const light_point = new THREE.PointLight("#FFFFFF", 0.5);
    light_point.position.x = 3;
    light_point.position.y = 1;
    light_point.position.z = 2;
    this.scene.add(hemis_light, light_point, light_amb);

    const rectLight = new THREE.RectAreaLight("orange", 2, 2, 1); //only works with mesh physical/standar materials
    gui.add(rectLight.position, "x").min(-4).max(4).step(0.01);
    gui.add(rectLight.position, "z").min(-4).max(4).step(0.01);
    rectLight.position.z = -0.2;

    this.scene.add(rectLight);

    const spotLight = new THREE.SpotLight("#FFFFFF", 0.4, 10, Math.PI * 0.3);
    spotLight.castShadow = true;
    spotLight.position.set(0, 2, 2);
    spotLight.shadow.camera.near = 1;
    spotLight.shadow.camera.far = 6;
    spotLight.shadow.mapSize.width = 1024;
    spotLight.shadow.mapSize.height = 1024;
    spotLight.shadow.camera.fov = 30;
    this.scene.add(spotLight);
    this.scene.add(spotLight.target);

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

    this.textMaterial = new THREE.MeshStandardMaterial({
      color: new THREE.Color("#FFFFFF"),
    });

    //FONT

    const fontLoader = new FontLoader();
    fontLoader.load("fonts/helvetica.json", (font) => {
      this.fontType = font;
      this.addText("Hola");
    });

    // Adding a plane
    let plane = new THREE.Mesh(
      new THREE.PlaneBufferGeometry(3, 3),
      this.textMaterial
    );
    plane.rotation.x = Math.PI * -0.5;
    plane.receiveShadow = true;
    this.scene.add(plane);
    let sizes = {
      width: window.innerWidth,
      height: window.innerHeight,
    };

    //CAMERA
    const camera = new THREE.PerspectiveCamera(75, sizes.width / sizes.height);
    camera.position.y = 1;
    camera.position.x = 1;
    camera.position.z = 2;
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

    const orbitControls = new OrbitControls(camera, canvas);
    orbitControls.target.y = 0;
    orbitControls.update();
    orbitControls.enableDamping = true; //smooth movment in the camera
    //RENDERER
    const renderer = new THREE.WebGLRenderer({
      canvas,
    });

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

    this.scene.add(miniBox);
    this.objects.push(miniBox);
    miniBox.cursor = "pointer";

    document.onkeypress = (e) => {
      e = e || window.event;
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
        moveControls.attach(this.selectedObject);
      }
    });

    const loop = () => {
      renderer.render(this.scene, camera);
      orbitControls.update(); //In order to the dumping to be applied
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
