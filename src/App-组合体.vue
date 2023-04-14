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
      collisionFilterGroup: GROUP1,
      collisionFilterMask:  GROUP2
    });
    // planeBody.quaternion.setFromAxisAngle(new CANNON.Vec3(1, 0, 0), 0.1);
    world.addBody(planeBody);
   
    // 创建body
    const boxMateralCon = new CANNON.Material("boxMaterial");
    boxMateralCon.friction = 0
    boxMateralCon.restitution = 0.14
    const capsuleBody = new CANNON.Body({
      mass: 1,
      position: new CANNON.Vec3(0, 3, 0),
      material: boxMateralCon,
      collisionFilterGroup: GROUP2,
      collisionFilterMask: GROUP1
    })
   // 球形几何体
   const sphereShape = new CANNON.Sphere(0.5);  
   // 创建圆柱几何体
  const cylinderShape = new CANNON.Cylinder(0.5, 0.5, 1.5, 20);
  capsuleBody.addShape(sphereShape, new CANNON.Vec3(0, 0.75, 0))
  capsuleBody.addShape(cylinderShape, new CANNON.Vec3(0, 0, 0))
  capsuleBody.addShape(sphereShape, new CANNON.Vec3(0, -0.75, 0))
  // 添加到物理世界
  world.addBody(capsuleBody);
  phyMeshes.push(capsuleBody)

  const geometry1 = new THREE.SphereGeometry( 0.5, 32, 32 );
  const material1 = new THREE.MeshBasicMaterial( { color: 0xffff00 } );
  const sphere1 = new THREE.Mesh( geometry1, material1 );
  sphere1.position.set(0, 0.75, 0)

  const geometry2 = new THREE.SphereGeometry( 0.5, 32, 32 );
  const material2 = new THREE.MeshBasicMaterial( { color: 0xffff00 } );
  const sphere2 = new THREE.Mesh( geometry2, material2 );
  sphere2.position.set(0, -0.75, 0)

  const geometry3 = new THREE.CylinderGeometry( 0.5, 0.5, 1.5, 32 );
  const material3 = new THREE.MeshBasicMaterial( {color: 0xffff00} );
  const cylinder = new THREE.Mesh( geometry3, material3 );
  let meshbody = new THREE.Group();
  meshbody.add(sphere1)
  meshbody.add(sphere2)
  meshbody.add(cylinder)
  scene.add( meshbody );
  meshes.push( meshbody );
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
