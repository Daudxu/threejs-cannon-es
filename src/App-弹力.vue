<script setup>
import * as THREE from 'three'
import  Stats  from "three/examples/jsm/libs/stats.module"
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls.js"
import * as CANNON from "cannon-es"
import CannonDebugger from 'cannon-es-debugger'
import { onMounted } from "vue"

onMounted (()=>{

    // 场景初始化
    const scene = new THREE.Scene();
    scene.background = new THREE.Color(0x88ccee);
    // scene.fog = new THREE.Fog(0x88ccee, 0, 50);
    // 创建相机
    const camera = new THREE.PerspectiveCamera(80, window.innerWidth / window.innerHeight, 0.1, 1000);
    camera.position.set(0, 3, 8)
    // 添加环境光
    const light = new THREE.AmbientLight( 0x404040 ); // soft white light
    scene.add( light );
    let pointLightHelper = new THREE.PointLightHelper( light, 1 );
    scene.add( pointLightHelper );
    // 辅助坐标轴
    const axesHelper = new THREE.AxesHelper( 10 );
    scene.add( axesHelper );
    // 渲染
    const container = document.getElementById("container");
    const renderer = new THREE.WebGLRenderer({ antialias: true })
    renderer.setPixelRatio(window.devicePixelRatio);
    renderer.setSize(window.innerWidth, window.innerHeight);
    renderer.shadowMap.enabled = true;
    renderer.shadowMap.type = THREE.VSMShadowMap;
    renderer.outputEncoding = THREE.sRGBEncoding;
    renderer.toneMapping = THREE.ACESFilmicToneMapping;
    container.appendChild(renderer.domElement)
    // 性能监控
    const stats = new Stats();
    stats.domElement.style.position = "absolute";
    stats.domElement.style.top = '0px';
    container.appendChild(stats.domElement);

    // 轨道控制
    const controls = new OrbitControls(camera, renderer.domElement);
    controls.target.set(0, 0, 0);


    // 创建一个平面
    const planeGeometry = new THREE.BoxGeometry(10, 0.2 ,10);
    const planMaterial = new THREE.MeshBasicMaterial({
      color: 0xffffff,
      side: THREE.DoubleSide
    });
    const plane = new THREE.Mesh(planeGeometry, planMaterial);
    plane.receiveShadow = true;
    plane.rotation.x = 0.1;
    scene.add(plane)

    // 创建物理世界
    const world = new CANNON.World();
    // 设置物理世界重力
    world.gravity.set(0, -8.5, 0);
    // 开启物理环境debug
    const cannonDebugger = new CannonDebugger(scene, world, {})
    // 物理世界物体数组
    let phyMeshes = [];
    let meshes = [];
   
    // 创建地面
    const planeShape = new CANNON.Box(new CANNON.Vec3(5, 0.1, 5));
    const planeShapeMaterial = new CANNON.Material("boxSlipperyMaterial");
    planeShapeMaterial.friction = 0.7;
    planeShapeMaterial.restitution = 0.2;

    const planeBody = new CANNON.Body({
      shape: planeShape,
      mass: 0,    // 当质量为0时，使得物体保持不动
      position: new CANNON.Vec3(0, 0, 0),
      type: CANNON.Body.STATIC,
      material: planeShapeMaterial
    });
    planeBody.quaternion.setFromAxisAngle(new CANNON.Vec3(1, 0, 0), 0.1);
    world.addBody(planeBody);
   
    // 创建物理世界物体
    const boxShape = new CANNON.Box(new CANNON.Vec3(0.5, 0.5, 0.5));
    const boxMateralCon = new CANNON.Material("boxMaterial");
    // boxMateralCon.friction = 100
    const boxBody = new CANNON.Body({
      shape: boxShape,
      position: new CANNON.Vec3(0, 5, 0),
      mass: 1,
      material: boxMateralCon,
    })
    world.addBody(boxBody);
    phyMeshes.push(boxBody)

    // 创建第二物理环境个立方体
    const boxSlipperyMaterial = new CANNON.Material("boxSlipperyMaterial");
    boxSlipperyMaterial.friction = 0;
    const boxBody2 = new CANNON.Body({
      shape: boxShape,
      position: new CANNON.Vec3(2, 5, 0),
      mass: 1,
      material: boxSlipperyMaterial,
    })
    world.addBody(boxBody2);
    phyMeshes.push(boxBody2)

    // 创建第三物理环境个立方体
    const boxBouncyMaterial = new CANNON.Material("boxBouncyMaterial");
    boxBouncyMaterial.friction = 0.1;
    boxBouncyMaterial.restitution = 3;
    const boxBody3 = new CANNON.Body({
      shape: boxShape,
      position: new CANNON.Vec3(-2, 5, ),
      mass: 1,
      material: boxBouncyMaterial,
    })
    world.addBody(boxBody3);
    phyMeshes.push(boxBody3)

    // 创建正方形
    const geometry = new THREE.BoxGeometry( 1, 1, 1 );
    const material = new THREE.MeshBasicMaterial( {color: 0x00ff00} );
    const cube = new THREE.Mesh( geometry, material );
    scene.add( cube );
    meshes.push( cube )
   
    // 第二个立方体
    const cube2 = new THREE.Mesh( geometry, material );
    scene.add( cube2 );
    meshes.push( cube2 )

    // 第三个立方体
    const cube3 = new THREE.Mesh( geometry, material );
    scene.add( cube3 );
    meshes.push( cube3 )

    const clock = new THREE.Clock();

    function animate() {
      let delta = clock.getDelta();
      world.step(1 / 60, delta, 10);
      for(let i = 0; i< phyMeshes.length; i++) {
        meshes[i].position.copy(phyMeshes[i].position)
        meshes[i].quaternion.copy(phyMeshes[i].quaternion)
      }

      stats.update();
      controls.update();
      renderer.render(scene, camera);
      cannonDebugger.update()
      requestAnimationFrame(animate);
    }
    animate();
});


</script>

<template>
  <div id="container">
  </div>
</template>

<style scoped>
/* .logo {
  height: 6em;
  padding: 1.5em;
  will-change: filter;
  transition: filter 300ms;
}
.logo:hover {
  filter: drop-shadow(0 0 2em #646cffaa);
}
.logo.vue:hover {
  filter: drop-shadow(0 0 2em #42b883aa);
} */
</style>
