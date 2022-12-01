<template>
  <div class="home">
    <div class="canvas-cotainer" ref="canvasDom">
    </div>
  </div>
  <!-- <HelloWorld msg="Vite + Vue" /> -->
</template>

<script setup>
 import * as THREE from "three"
 import gsap from "gsap"
//  import { OrbitControls }  from "three/examples/jsm/controls/OrbitControls"
 import { GLTFLoader } from "three/examples/jsm/loaders/GLTFLoader.js";
import { DRACOLoader } from "three/examples/jsm/loaders/DRACOLoader.js";
 import { onMounted, reactive, ref } from "vue";

 let canvasDom = ref(null)
 let pages = ref(null)

 onMounted(()=>{
    // 创建场景
    const scene = new THREE.Scene()
    // 创建相机
    const camera = new THREE.PerspectiveCamera(
       45,
       window.innerWidth / window.innerHeight,
       0.1,
       1000
    )
    camera.position.set(0, 0, 10)
    // 创建渲染器 antialias 抗锯齿
    const renderer = new THREE.WebGL1Renderer({
      // 抗锯齿
        antialias: true
    })
  
    renderer.setSize(window.innerWidth, window.innerHeight)
    // 将画布添加到页面
    canvasDom.value.appendChild(renderer.domElement)

    // 创建星空的背景
    
    let url = "./star-sky.jpg"
    let envTexture = new THREE.TextureLoader().load(url)
    envTexture.mapping = THREE.EquirectangularReflectionMapping
    scene.background = envTexture
    scene.environment = envTexture
    
    function render() {
      requestAnimationFrame(render);
      renderer.render(scene, camera)
    }
    render()
    // 添加灯光
    let light = new THREE.DirectionalLight(0xffffff, 1);
    light.position.set(0, 0, 1);
    scene.add(light)

    let light2 = new THREE.DirectionalLight(0xffffff, 0.5);
    light2.position.set(0, 0, -1);
    scene.add(light2)

    let light3 = new THREE.DirectionalLight(0xffffff, 0.5);
    light3.position.set(-1, 1, 1);
    scene.add(light3)

    let light4 = new THREE.DirectionalLight(0xffffff, 1);
    light4.position.set(0, 10, 0);
    scene.add(light4)

    // 设置解压缩加载器
    let dracoLoader = new DRACOLoader();
    dracoLoader.setDecoderPath("./draco/gltf/");
    dracoLoader.setDecoderConfig({type: "js"});
    let loader = new  GLTFLoader();
    loader.setDRACOLoader(loader);
    loader.load("./models/lancelot-class_light_cruiser.glb", (gltf) => {
      gltf.scene.scale.set(0.0001, 0.0001, 0.0001)
      gltf.scene.position.set(3, 0, 0)
      scene.add(gltf.scene)

      window.addEventListener("mousemove",(e) => {
        let x = (e.clientX / window.innerWidth) * 2 - 1;
        let y = (e.clientY / window.innerHeight) * 2 - 1;

        let timeline = gsap.timeline();
        timeline.to(gltf.scene.rotation, {
          duration: 0.5,
          x:y,
          y:x,
          duration: 1
        })
      })
    })

    loader.load("./models/spaceship_unitron.glb", (gltf) => {
      gltf.scene.scale.set(0.3, 0.3, 0.3)
      gltf.scene.position.set(3, -8, 0)
      scene.add(gltf.scene)

      window.addEventListener("mousemove",(e) => {
        let x = (e.clientX / window.innerWidth) * 2 - 1;
        let y = (e.clientY / window.innerHeight) * 2 - 1;

        let timeline = gsap.timeline();
        timeline.to(gltf.scene.rotation, {
          duration: 0.5,
          x:y,
          y: (Math.PI) + x,
          duration: 1
        })
      })
    })

    loader.load("./models/cyberpunk_hovercar.glb", (gltf) => {
      gltf.scene.scale.set(0.8, 0.8, 0.8)
      gltf.scene.position.set(3, -16, 0)
      scene.add(gltf.scene)

      window.addEventListener("mousemove",(e) => {
        let x = (e.clientX / window.innerWidth) * 2 - 1;
        let y = (e.clientY / window.innerHeight) * 2 - 1;

        let timeline = gsap.timeline();
        timeline.to(gltf.scene.rotation, {
          duration: 0.5,
          x:y,
          y:x,
          duration: 1
        })
      })
    })
   
    let page = 0;
   let timeline2 = gsap.timeline();
   window.addEventListener("mousewheel", (e) => {
        
        if(e.wheelDelta < 0) {
            page++;
            if(page > 2){
              page = 2
            }
        }

        if(e.wheelDelta > 0) {
          page--;
            if(page < 0){
              page = 0
            }
        }

        if(!timeline2.isActive()){
          timeline2.to(camera.position, {
              duration: 0.5,
              y: page * -8,
              duration: 1
          })
          console.log()
          gsap.to(pages.value,{
            duration: 1,
            y: -page * window.innerHeight,
            duration: 1
          })
        }

   })
    
   loader.load("./models/lowpoly.glb", (gltf)=>{
      let moon = gltf.scene.children[0];

      
      for(let j = 0; j < 10; j++){
          let moonInstance = new THREE.InstancedMesh(
            moon.geometry,
            moon.material,
            100
          );
     
      for(let i=0; i<100; i++){
        let x = Math.random() * 1000 -500;
        let y = Math.random() * 1000 -500;
        let z = Math.random() * 1000 -500;
        
        let matrix = new THREE.Matrix4();
        let size = Math.random() * 20 - 8;
        matrix.makeScale(size)
        matrix.makeTranslation(x, y, z);
        moonInstance.setMatrixAt(i, matrix)
      }
      gsap.to(moonInstance.position, {
        duration: Math.random() * 10 + 2,
        z: -1000,
        ease: "linear",
        repeat: -1
      })
      scene.add(moonInstance)
    }

   })

 })


</script>

<style scoped>
* {
  margin: 0;
  padding: 0;
}
body {
  background-color: #000;
}
.canvas-cotainer {
   width: 100vw;
   height: 100vh;
   z-index: -1;
   position: absolute
}
.home {
  width: 100vw;
  height: 100vh;
  transform-origin:0 0;
}
.header {
  position: fixed;
  top: 0;
  left: 0;
  color: #fff;
  display: flex;
  width: 100%;
  padding: 10px;
  align-items: center;
}
.menu {
   margin-left: 10%;
   
}
.menuItem {
   padding: 20px;
   font-size: 20px;
}
.pages {
   display: flex;
   flex-direction: column;
   position: fixed;
   top: 0;
   left: 0;
}
.pages .page {
  width: 100wh;
  height: 100vh;
  color: #fff;
  display: flex;
  flex-direction: column;
  justify-content: center;
  align-items: flex-start;
  color: #fff;
  padding: 250px;
  box-sizing: border-box;
}


</style>
