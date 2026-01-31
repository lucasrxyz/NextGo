<template>
  <div class="three-wrapper">
    <img src="../assets/background_1.png" class="background" />
    <div ref="sceneContainer" class="three-container"></div>
  </div>
</template>

<script>
import * as THREE from 'three'
import { GLTFLoader } from 'three/examples/jsm/loaders/GLTFLoader.js'

export default {
  mounted() {
    const container = this.$refs.sceneContainer
    const width = 1980
    const height = 1080

    const scene = new THREE.Scene()
    scene.background = null // fond transparent

    const camera = new THREE.PerspectiveCamera(45, width / height, 0.1, 1000)
    camera.position.set(-.1, 0.5, 2.4)

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
  }
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

</style>
