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
    camera.position.set(0, 10, 15)
    // 添加环境光
    const light = new THREE.AmbientLight( 0x404040 ); // soft white light
    scene.add( light );
    let pointLightHelper = new THREE.PointLightHelper( light, 1 );
    scene.add( pointLightHelper );
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

    // 创建胶囊几何体
    const geometry = new THREE.SphereGeometry( 0.5, 32, 32 );
    const material = new THREE.MeshBasicMaterial({
      color: 0xff0000,
      side: THREE.DoubleSide
    } );
    const sphere = new THREE.Mesh( geometry, material );
    scene.add( sphere );

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
   
    // 创建物理小球形状
    const sphereShape = new CANNON.Sphere(0.5);
 
    // 创建物理世界的物体
    const sphereBody = new CANNON.Body({
        shape: sphereShape,
        // 位置
        position: new CANNON.Vec3(0, 5, 0),
        // 小球质量
        mass: 1,
    });

    // sphereBody.position.y = 150
    // 将物体添加至物理世界
    world.addBody(sphereBody);

    // 创建地面
    const planeShape = new CANNON.Box(new CANNON.Vec3(5, 0.1, 5));
    const planeBody = new CANNON.Body({
      shape: planeShape,
      mass: 0,    // 当质量为0时，使得物体保持不动
      position: new CANNON.Vec3(0, 0, 0),
      type: CANNON.Body.STATIC,
    });
    planeBody.quaternion.setFromAxisAngle(new CANNON.Vec3(1, 0, 0), 0.1);
    world.addBody(planeBody);


    const clock = new THREE.Clock();

    function animate() {
      let delta = clock.getDelta();
      world.step(1 / 60, delta, 10);
      // floorBody.position.set(plane.position)
      // sphereBody.position.x += delta * 1
      sphere.position.copy(sphereBody.position)
      sphere.quaternion.copy(sphereBody.quaternion)
     
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
