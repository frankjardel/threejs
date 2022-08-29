<template>
   <div id="app">
   </div>
</template>

<script>
import * as THREE from 'three';
import { GLTFLoader } from 'three/examples/jsm/loaders/GLTFLoader.js'
import {OrbitControls} from "three/examples/jsm/controls/OrbitControls"
import { FontLoader } from 'three/examples/jsm/loaders/FontLoader'
import { TextGeometry } from 'three/examples/jsm/geometries/TextGeometry'

export default {
   name: 'App',

   data() {
      return {
         text: 'SculpApp',
         trapeze: 'Trapeze'
      }
   },

   created() {
      const scene = new THREE.Scene()
      const camera = new THREE.PerspectiveCamera(60, window.innerWidth / window.innerHeight, 0.1, 1000)

      const renderer = new THREE.WebGLRenderer({antialias: true, alpha: true,})

      const controls = new OrbitControls(camera, renderer.domElement)

      let clock = new THREE.Clock()
      let mixer = null
      let action = null

      renderer.setSize(window.innerWidth, window.innerHeight)
      document.body.appendChild(renderer.domElement)

      camera.position.x = 0
      camera.position.y = 0.5
      camera.position.z = 2.5

      camera.lookAt(0,0,0)

      const light = new THREE.AmbientLight(0x404040) // soft white light
      scene.add(light)

      const directionalLight = new THREE.DirectionalLight(0xffffff, 1)
      scene.add(directionalLight)

      renderer.toneMapping = THREE.ACESFilmicToneMapping;
      renderer.outputEncoding = THREE.sRGBEncoding;

      // Import
      const loader = new GLTFLoader()
      loader.load('/models/Human.glb',(gltf) => {
         scene.add(gltf.scene)
         const animations = gltf.animations
         mixer = new THREE.AnimationMixer(gltf.scene)
         const clip = THREE.AnimationClip.findByName(animations, 'IDLE')
         action = mixer.clipAction(clip)

         gltf.scene.position.y = -1.15
      }, undefined, (error) => {
         console.error(error)
      } )


      // Texts
      const loaderFont = new FontLoader()

      loaderFont.load('fonts/Oswald_Regular.json', (font) => {
         const geometry = new TextGeometry(this.text, {
            font: font,
            size: 0.1,
            height: 0.01,
            curveSegments: 10,
            bevelEnabled: false,
            bevelThickness: 1,
            bevelSize: 1,
            bevelOffset: 0,
            bevelSegments: 1
         })
         const textMesh = new THREE.Mesh(geometry, [
            new THREE.MeshPhongMaterial({ color: 0x1a1a1a }),
            new THREE.MeshPhongMaterial({ color: 0x1a002f })
         ])
         scene.add(textMesh)
         textMesh.casShadow = true
         textMesh.position.x += 0.5
         textMesh.position.y += 0.75

         // Trapeze
         const trapeze = new TextGeometry(this.trapeze, {
            font: font,
            size: 0.1,
            height: 0.01,
            curveSegments: 10,
            bevelEnabled: false,
            bevelThickness: 1,
            bevelSize: 1,
            bevelOffset: 0,
            bevelSegments: 1
         })
         const trapezeText = new THREE.Mesh(trapeze, [
            new THREE.MeshPhongMaterial({ color: 0x1a1a1a }),
            new THREE.MeshPhongMaterial({ color: 0x1a002f })
         ])
         scene.add(trapezeText)

         trapezeText.casShadow = true
         trapezeText.position.x -= 1
         trapezeText.position.y += 0.75
      })


      function animate() {
         if (action != null){
            action.play()
         }
         
         let delta = clock.getDelta()

         if (mixer) {
            mixer.update(delta)
         }

         requestAnimationFrame( animate )
         renderer.render( scene, camera )
      }
      animate()
   }
}
</script>

<style>
* {
   padding: 0;
   margin:  0;
}
#app {
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #2c3e50;
}

canvas {
   background: lightgray;
}
</style>
