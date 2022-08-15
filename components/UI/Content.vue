<template>
  <div id="content">
    <transition name="list">
      <div id="bloque1" :key="cameraPosition">
        <h1>{{ content.titulo }}</h1>
        <p>{{ content.description1 }}</p>
        <p>
          {{ content.description2 }}
        </p>
        <div class="btn" @click="changeCamera()">Botón de acción</div>
      </div>
    </transition>
  </div>
</template>
<script>
export default {
  data() {
    return {
      cameraPosition: "top",
    };
  },
  methods: {
    changeCamera() {
      if (this.cameraPosition == "close") {
        this.$parent.$data.cameraAnimations.top();
        this.cameraPosition = "top";
      } else {
        this.$parent.$data.cameraAnimations.close();
        this.cameraPosition = "close";
      }
      //   this.$parent.data.cameraAnimations.top;
    },
  },
  computed: {
    content() {
      switch (this.cameraPosition) {
        case "top":
          return {
            titulo: "Cámara top",
            description1: "Desde esta vista se puede apreciar toda la escena",
            description2: "Es una forma vista tranquila, no?",
          };
        case "close":
          return {
            titulo: "Camera close-up",
            description1: "Un acercamiento al tren",
            description2:
              "Podemos ver todos los detalles, materiales y texturas",
          };
      }
    },
  },
};
</script>
<style scoped>
#content {
  /* Permalink - use to edit and share this gradient: https://colorzilla.com/gradient-editor/#001123+0,000000+100&0.65+0,0+100 */
  background: -moz-linear-gradient(
    left,
    rgba(0, 17, 35, 0.65) 0%,
    rgba(0, 0, 0, 0) 100%
  ); /* FF3.6-15 */
  background: -webkit-linear-gradient(
    left,
    rgba(0, 17, 35, 0.65) 0%,
    rgba(0, 0, 0, 0) 100%
  ); /* Chrome10-25,Safari5.1-6 */
  background: linear-gradient(
    to right,
    rgba(0, 17, 35, 0.65) 0%,
    rgba(0, 0, 0, 0) 100%
  ); /* W3C, IE10+, FF16+, Chrome26+, Opera12+, Safari7+ */
  filter: progid:DXImageTransform.Microsoft.gradient( startColorstr='#a6001123', endColorstr='#00000000',GradientType=1 ); /* IE6-9 */

  width: 100%;
  padding-left: 100px;
  height: 100vh;
  padding-top: 100px;
  display: flex;
  color: #fff;
}
h1 {
  font-size: 80px;
}
p {
  font-size: 20px;
}
.btn {
  display: inline-block;
  background-color: rgb(208, 36, 36);
  padding: 10px;
  border: 1px solid rgb(157, 29, 29);
  border-radius: 5px;
  text-align: center;
}
.btn:hover {
  background-color: rgb(242, 46, 46);
  cursor: pointer;
}

/* Enter and leave animations can use different */
/* durations and timing functions.              */
.list-move, /* apply transition to moving elements */
.list-enter-active,
.list-enter,
.list-enter-from,
.list-enter-to,
.list-leave-active {
  transition: all 1s ease;
}

.list-enter {
  opacity: 0;
}
.list-enter-from {
  opacity: 0;
}
.list-enter-to {
  opacity: 1;
}
.list-leave-to {
  opacity: 0;
  transform: scale(1.4);
  filter: blur(4px);
}

/* ensure leaving items are taken out of layout flow so that moving
   animations can be calculated correctly. */
.list-leave-active {
  position: absolute;
}
</style>