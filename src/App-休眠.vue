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
    // plane.rotation.x = 0.1;
    scene.add(plane)

    // 创建物理世界
    const world = new CANNON.World();
    // 设置物理世界休眠
    world.allowSleep = true;
    // 设置物理世界重力
    world.gravity.set(0, -8.5, 0);
    // 开启物理环境debug
    const cannonDebugger = new CannonDebugger(scene, world, {})
    // 物理世界物体数组
    let phyMeshes = [];
    let meshes = [];
    // 碰撞租
    const GROUP1 = 1;
    const GROUP2 = 2;
    const GROUP3 = 4;
    const GROUP4 = 8;
    // 创建地面
    const planeShape = new CANNON.Box(new CANNON.Vec3(5, 0.1, 5));
    const planeShapeMaterial = new CANNON.Material("boxSlipperyMaterial");
    planeShapeMaterial.friction = 0;
    planeShapeMaterial.restitution =  1;

    const planeBody = new CANNON.Body({
      shape: planeShape,
      mass: 0,    // 当质量为0时，使得物体保持不动
      position: new CANNON.Vec3(0, -0.1, 0),
      type: CANNON.Body.STATIC,
      material: planeShapeMaterial,
      collisionFilterGroup: GROUP4,
      collisionFilterMask: GROUP1 | GROUP2 | GROUP3
    });
    // planeBody.quaternion.setFromAxisAngle(new CANNON.Vec3(1, 0, 0), 0.1);
    world.addBody(planeBody);
   
    // 创建物理世界物体
    const boxShape = new CANNON.Box(new CANNON.Vec3(0.5, 0.5, 0.5));
    const boxMateralCon = new CANNON.Material("boxMaterial");
    boxMateralCon.friction = 0
    boxMateralCon.restitution = 1
    const boxBody = new CANNON.Body({
      shape: boxShape,
      position: new CANNON.Vec3(-2, 5, 0),
      mass: 1,
      material: boxMateralCon,
      collisionFilterGroup: GROUP1,
      collisionFilterMask: GROUP2 | GROUP3 | GROUP4
    })
    // 设置正方体允许休眠
    boxBody.allowSleep = true;
    // 设置正方体如果速度小于0.1,允许休眠
    boxBody.sleepSpeedLimit = 0.1;
    // 设置正方体如果速度小于0.1s,1秒之后进入休眠
    boxBody.sleepTimeLimit = 1
    world.addBody(boxBody);
    phyMeshes.push(boxBody)
    boxBody.addEventListener("collide", (e) => {
      console.log("碰撞了", e)
      let height =  e.contact.getImpactVelocityAlongNormal();
      console.log("height", height)
    })
    boxBody.addEventListener("sleepy", (e) => {
      console.log("即将进入休眠", e)
    })
    boxBody.addEventListener("sleep", (e) => {
      console.log("进入休眠", e)
    })
    // 创建第二物理环境个圆球
    const sphereShape = new CANNON.Sphere(0.5);
    // 创建一个刚体
    const sphereBody =  new CANNON.Body({
      shape: sphereShape,
      position: new CANNON.Vec3(0, 0.5, 0),
      mass: 1,
      material: boxMateralCon,
      collisionFilterGroup: GROUP2,
      collisionFilterMask: GROUP1 | GROUP4
    });
    world.addBody(sphereBody);
    phyMeshes.push(sphereBody);

    // 创建第二物理环境个圆球
    const cylinderShape = new CANNON.Cylinder(0.5, 0.5);
    // 创建一个刚体
    const cylinderBody =  new CANNON.Body({
      shape: cylinderShape,
      position: new CANNON.Vec3(2, 0.5, 0),
      mass: 1,
      material: boxMateralCon,
      collisionFilterGroup: GROUP3,
      collisionFilterMask: GROUP1 | GROUP4
    });
    world.addBody(cylinderBody);
    phyMeshes.push(cylinderBody);


    // 创建正方形
    const geometry = new THREE.BoxGeometry( 1, 1, 1 );
    const material = new THREE.MeshBasicMaterial( {color: 0xffff00} );
    const cube = new THREE.Mesh( geometry, material );
    scene.add( cube );
    meshes.push( cube );
   
    // 第二个立方体
    const geometry2 = new THREE.SphereGeometry( 0.5, 32, 32 );
    const material2 = new THREE.MeshBasicMaterial( { color: 0xffff00 } );
    const sphere = new THREE.Mesh( geometry2, material2 );
    scene.add( sphere );
    meshes.push( sphere );

    // 第三个物体
    const geometry3 = new THREE.CylinderGeometry( 0.5, 0.5, 1, 32 );
    const material3 = new THREE.MeshBasicMaterial( {color: 0xffff00} );
    const cylinder = new THREE.Mesh( geometry3, material3 );
    scene.add( cylinder );
    meshes.push( cylinder );
    // 设置boxbody移动速度
    // boxBody.velocity.set(2, 0, 0)


    const clock = new THREE.Clock();
   
    function animate() {
      let delta = clock.getDelta();
      world.step(1 / 60, delta);
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
