<template>
    <div class="home">
        <div class="view-container" ref="threeDBox"></div>
  </div>
</template>

<script setup>
import { onMounted, ref } from '@vue/runtime-core'
import livingImg from '../assets/img/pano/P20220721174057493.jpg'
import kitchenImg from '../assets/img/pano/P20220720105535682.jpg'
import tipImg from '../assets/img/pano/point.png'
import * as THREE from 'three'
import { TWEEN } from 'three/examples/jsm/libs/tween.module.min.js';
import {OrbitControls} from 'three/examples/jsm/controls/OrbitControls'
import gsap from 'gsap'

let renderer, camera, mesh, scene, controller, texture = [], currentSceneIndex = 0
let threeDBox = ref(null)
let spriteList = []
let dataList = [
  {
    image: livingImg,
    position: { x: 0, y: 0, z: 0 },
    tip:{
        position: { x: -200, y:0, z: -145 },
        image: 1,
      }
  },
  {
    image: kitchenImg,
    position: { x: 100, y: 0, z: 0 },
    tip: {
        position: { x: -199, y: -24, z: 145 },
        image: 0,
      }
  }
];
 onMounted(()=>{
    init()
    render()
 })
function init(){
    initRenderer()
    initCamera()
    initScene()
    initMesh(currentSceneIndex);
    initController()
    addTipsSprite()
    changeScene()
    
}
// 初始化渲染器
function initRenderer() {
  renderer = new THREE.WebGLRenderer({
    antialias: true
  })
  renderer.setPixelRatio(window.devicePixelRatio)
  renderer.setSize(window.innerWidth, window.innerHeight)
  threeDBox.value.appendChild(renderer.domElement)
}

// 初始化镜头
function initCamera() {
  camera = new THREE.PerspectiveCamera(60, window.innerWidth / window.innerHeight, 1, 1000)
  camera.position.z = 0.0001
}
// 初始化场景
function initScene(){
  scene = new THREE.Scene()
}
// 初始化网格，  网格单元需要几何体和材质
function initMesh(index){
  const geomatry = new THREE.SphereGeometry(16,50,50)
  texture = dataList.map(item => new THREE.TextureLoader().load(item.image))
  
  const material = new THREE.MeshBasicMaterial({map:texture[index], side: THREE.DoubleSide})
  mesh = new THREE.Mesh(geomatry, material)
  mesh.geometry.scale(16,16,-16)
  scene.add(mesh)
}
// 控制器
function initController(){
  controller = new OrbitControls(camera,renderer.domElement)
  controller.enableDamping = true
}
// 渲染函数
function render(){
  controller.update();
  TWEEN && TWEEN.update();
  renderer.render(scene, camera)
  requestAnimationFrame(render)
}
// 在场景中添加精灵
function addTipsSprite(index=0){
  let map = new THREE.TextureLoader().load(tipImg)
  let material = new THREE.SpriteMaterial({map:map})
  let sprite = new THREE.Sprite(material)

  let tip = dataList[index].tip
  sprite.scale.set(10,10,10)
  sprite.position.set(tip.position.x,tip.position.y,tip.position.z)
  sprite.image = tip.image
  spriteList.push(sprite)
  scene.add(sprite)
}

// 切换场景,根据index切换不同场景
function changeScene(index=0){
  //将场景中的精灵图清除，留下基础元素
  scene.children = scene.children.filter(item=>item.type!=='Sprite')

  // 创建新的材质
  // const newMaterial = new THREE.MeshBasicMaterial({
  //   map: texture[index],
  //   transparent: true,
  //   opacity: 0
  // });

  // 将新材质赋值给网格的 material 属性
  // mesh.material = newMaterial;

  // 使用 Tween.js 或 gsap 实现渐变过渡
  // gsap.to(newMaterial, { duration: 0.5, opacity: 1 });

  mesh.material.map = texture[index]
      
  // let texture = new THREE.TextureLoader().load(dataList[index].image)
  // let sphereMaterial = new THREE.MeshBasicMaterial({map:texture})
  // mesh.material = sphereMaterial
  // gsap.to(sphereMaterial,{transparent:true,opacity:1,duration:1})
  // controller.update()
  // camera.updateProjectionMatrix();
  addTipsSprite(index)
}

//页面大小发生变化
window.addEventListener("resize",()=>{
  camera.aspect = window.innerWidth/window.innerHeight
  camera.updateProjectionMatrix()
  renderer.setSize(window.innerWidth,window.innerHeight)
})
window.addEventListener("click",function (e){
  e.preventDefault();
  //这里通过raycaster光线投影抓取点击目标
  let raycaster = new THREE.Raycaster()
  let mouse = new THREE.Vector2()
  mouse.x = (e.clientX / window.innerWidth) * 2 - 1;
  mouse.y = -(e.clientY / window.innerHeight) * 2 + 1;
  raycaster.setFromCamera(mouse, camera);

  let  intersects = raycaster.intersectObjects(scene.children)
  if (intersects.length > 0 && intersects[0].object.type === 'Sprite') {
      changeScene(intersects[0].object.image);
  }
},false)
</script>

<style lang="scss">
.home {
  position: relative;
  width: 100%;
  height: 100%;
  overflow: hidden;

  .view-container {
    width: 100%;
    height: 100%;
    overflow: hidden;
  }
}
</style>