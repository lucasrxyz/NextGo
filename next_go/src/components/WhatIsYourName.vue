<template>
  <div class="three-wrapper">
    <img src="../assets/background_1.png" class="background" />
    <div ref="sceneContainer" class="three-container"></div>
  </div>

  <div
    class="dialog-wrapper"
    :class="{ show: dialogVisible }"
    @click="nextDialog"
  >
    <img
      src="../assets/dialogs_containers/dialog1_blue.png"
      class="dialog"
    />
    <span class="dialog-text">
      {{ displayedText }}
      <span class="dots" v-if="!isTyping">{{ dots }}</span>
    </span>
  </div>
    
</template>

<script>
import * as THREE from 'three'
import { GLTFLoader } from 'three/examples/jsm/loaders/GLTFLoader.js'

export default {
  data() {
    return {
      dialogVisible: false,
      currentDialog: 0,
      displayedText: '',
      isTyping: false,

      dots: '',
      dotsInterval: null,

      typingSpeed: 35,
      typingInterval: null,

      kims_bot_dial: [
        { value: 1, dialog: 'Hi!' },
        { value: 2, dialog: "I'm Kims, a flying bot." },
        { value: 3, dialog: "I've got a few questions." },
        { value: 4, dialog: "Have you played parkour games before?" },
        { value: 5, dialog: "Okay!" },
        { value: 6, dialog: "How should I call you?" }
      ]
    }
  },
  mounted() {
    const container = this.$refs.sceneContainer
    const width = 1980
    const height = 1080

    const scene = new THREE.Scene()
    scene.background = null // fond transparent

    const camera = new THREE.PerspectiveCamera(45, width / height, 0.1, 1000)
    camera.position.set(-.1, 0.3, 2.4)

    const renderer = new THREE.WebGLRenderer({ antialias: true, alpha: true })
    renderer.setSize(width, height)
    renderer.setClearColor(0x000000, 0)
    container.appendChild(renderer.domElement)

    // Lumière
    const directionalLight = new THREE.DirectionalLight(0xffffff, 3)
    directionalLight.position.set(5, 5, 7.5)
    scene.add(directionalLight)

    const ambientLight = new THREE.AmbientLight(0x404040, 30)
    scene.add(ambientLight)

    // GLTF Loader avec timeout
    setTimeout(() => {
      const loader = new GLTFLoader()
      loader.load(
        '/3D_Characters/KimsBOT_what_is_your_name.gltf',
        (gltf) => {
          const model = gltf.scene
          model.position.x += -0.1
          model.rotation.y = Math.PI / 2 // rotate 90° vers la droite
          scene.add(model)

          // Outline blanc
          model.traverse((child) => {
            if (child.isMesh && !child.userData.hasOutline) {
              const outlineMaterial = new THREE.MeshBasicMaterial({
                color: 0xffffff,
                side: THREE.BackSide
              })
              const outlineMesh = new THREE.Mesh(child.geometry, outlineMaterial)
              outlineMesh.scale.multiplyScalar(1.1)
              outlineMesh.userData.hasOutline = true
              child.add(outlineMesh)
            }
          })

          // Animations
          let mixer = null
          if (gltf.animations && gltf.animations.length) {
            mixer = new THREE.AnimationMixer(model)
            gltf.animations.forEach((clip) => mixer.clipAction(clip).play())
          }

          const clock = new THREE.Clock()
          const animate = () => {
            requestAnimationFrame(animate)
            if (mixer) mixer.update(clock.getDelta())
            renderer.render(scene, camera)
          }
          animate()
        },
        (xhr) => console.log((xhr.loaded / xhr.total) * 100 + '% chargé'),
        (error) => console.error('Erreur de chargement : ', error)
      )
    }, 200) // <-- délai de 200ms avant d'afficher le robot

    setTimeout(() => {
      this.dialogVisible = true
    }, 2200)
    setTimeout(() => {
      this.startTyping()
    }, 2700)
  },
  methods: {
    
    startTyping() {
      this.stopDots()

      this.displayedText = ''
      this.isTyping = true

      const fullText = this.kims_bot_dial[this.currentDialog].dialog
      let index = 0

      this.typingInterval = setInterval(() => {
        this.displayedText += fullText[index]
        index++
      
        if (index >= fullText.length) {
          clearInterval(this.typingInterval)
          this.isTyping = false
          this.startDots() // 👈 points quand le texte est fini
        }
      }, this.typingSpeed)
    },


    nextDialog() {
      if (this.isTyping) {
        clearInterval(this.typingInterval)
        this.displayedText = this.kims_bot_dial[this.currentDialog].dialog
        this.isTyping = false
        this.startDots()
        return
      }
    
      if (this.currentDialog < this.kims_bot_dial.length - 1) {
        this.stopDots()
        this.currentDialog++
        this.startTyping()
      }
    },

    startDots() {
      this.dots = ''
      let count = 0

      this.dotsInterval = setInterval(() => {
        count = (count + 1) % 4
        this.dots = '.'.repeat(count)
      }, 400)
    },

    stopDots() {
      clearInterval(this.dotsInterval)
      this.dots = ''
    },
  },
}
</script>



<style>
/* Fullscreen & no scroll */
html, body, #app {
  margin: 0;
  padding: 0;
  width: 100%;
  height: 100%;
  overflow: hidden;
  background: transparent; /* important */
}

.three-wrapper {
  position: relative;
  width: 100%;
  height: 100%;
  overflow: hidden;
}

.background {
  position: absolute;
  width: 100%;
  height: 100%;
  object-fit: contain;
  top: 0;
  left: 0;
  z-index: 0;
  /* Animation de rotation */
  animation: rotateBackground 20s linear infinite;
  transform-origin: center center !important;
}

/* Définition de l’animation */
@keyframes rotateBackground {
  from {
    transform: rotate(0deg) scale(2.2);
  }
  to {
    transform: rotate(360deg) scale(2.2);
  }
}


.three-container {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: 100%;
  z-index: 1;
  background: transparent; /* important */
}

.dialog {
  position: absolute;
  bottom: 50px;
  left: 50%;
  transform: translateX(-50%);
  z-index: 10;
  filter: drop-shadow(5px 5px 10px rgba(0, 174, 255, 0.7));
  width: 600px;

  
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  image-rendering: auto;
}
.dialog-wrapper {
  position: absolute;
  bottom: 0px;
  left: 50%;
  transform: translateX(-50%) scale(0);
  opacity: 0;
  z-index: 10;
  width: 400px;
}
.dialog-wrapper.show {
  animation: dialogZoomIn 0.6s cubic-bezier(0.22, 1, 0.36, 1) forwards;
}

@keyframes dialogZoomIn {
  from {
    transform: translateX(-50%) scale(0) rotate(-15deg);
    opacity: 0;
  }
  to {
    transform: translateX(-50%) scale(1) rotate(0deg);
    opacity: 1;
  }
}

.dialog-text {
  position: absolute;
  top: -185px;
  left: -20px;
  color: #00D9FF;
  font-size: 28px;
  line-height: 1.4;
  text-shadow: 2px 2px 0px rgba(0, 99, 248, 0.8);
  pointer-events: none;
  z-index:20;
}
.dots {
  margin-left: 4px;
  letter-spacing: 2px;
  opacity: 0.8;
}

</style>
