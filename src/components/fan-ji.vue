<template>
  <div class="main">
    <div id="container"></div>
    <div class="btn-group">
      <div
        class="btn-item start-stop"
        :class="[isStart ? 'active' : '']"
        @click="start"
      >
        风机开启
      </div>
      <div
        class="btn-item fan-view"
        :class="[isShowFan ? 'active' : '']"
        @click="showFan"
      >
        数字风机
      </div>
      <div
        class="btn-item detail-view"
        :class="[isShowDetail ? 'active' : '']"
        @click="showDetail"
      >
        舱体视角
      </div>
      <div
        class="btn-item split-fan"
        :class="[isPlit ? 'active' : '']"
        @click="splitFan"
      >
        爆炸分裂
      </div>
    </div>
  </div>
</template>

<script setup>
import { onMounted, ref } from "vue";
import * as THREE from "three";
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls";
import { GLTFLoader } from "three/examples/jsm/loaders/GLTFLoader";
import { DRACOLoader } from "three/examples/jsm/loaders/DRACOLoader";
import gsap from "gsap";

let scene = new THREE.Scene();
let camera = new THREE.PerspectiveCamera(
  45,
  window.innerWidth / window.innerHeight,
  0.1,
  10000
);
let loader = new GLTFLoader();
let arrTweenAn = [];
let fan = new THREE.Object3D();
let land1, land2, land3;
let sceneOne = new THREE.Object3D();
let sceneTwo = new THREE.Object3D();
// 流光风机
let flowTrunk = new THREE.Object3D();
let flowFan = new THREE.Object3D();

// 按钮点中状态
let isStart = ref(true);
let isShowFan = ref(false);
let isShowDetail = ref(false);
let isPlit = ref(false);
let loading = ref(false);

function init() {
  camera.position.set(10, 7, 20);
  scene.add(camera);

  let light = new THREE.DirectionalLight(0xffffff);
  light.position.set(20, 20, 20);
  scene.add(light);

  let renderer = new THREE.WebGLRenderer({
    alpha: true,
    antialias: true,
    logarithmicDepthBuffer: true,
  });

  renderer.setSize(window.innerWidth, window.innerHeight);
  renderer.setPixelRatio(2);

  document.getElementById("container")?.appendChild(renderer.domElement);
  const controls = new OrbitControls(camera, renderer.domElement);
  controls.enableDamping = true;
  controls.maxDistance = 1000;
  controls.minPolarAngle = 0;
  controls.maxPolarAngle = Math.PI / 2;

  function tick() {
    controls.update();
    renderer.render(scene, camera);
    requestAnimationFrame(tick);
  }
  tick();
}

function cloneFan(x, y, z) {
  let fan_clone = fan.clone();
  fan_clone.position.set(x, y, z);
  // let an = TweenMax.to(fan_clone.children[0].rotation, 5, {
  //   x: fan_clone.children[0].rotation.x + Math.PI * 2,
  //   repeat: -1,
  //   ease: Linear.easeOut,
  // });
  let an = gsap.to(fan_clone.children[0].rotation, {
    duration: 5,
    x: fan_clone.children[0].rotation.x + Math.PI * 2,
    repeat: -1,
    ease: "none",
  });
  arrTweenAn.push(an);
  sceneOne.add(fan_clone);
}

function cloneLand1(x, y, z) {
  let land1_clone = land1.clone();
  land1_clone.position.set(x, y, z);
  sceneOne.add(land1_clone);
}

function cloneLand2(x, y, z) {
  let land2_clone = land2.clone();
  land2_clone.position.set(x, y, z);
  sceneOne.add(land2_clone);
}

function resetName(obj, name) {
  let i = 0;
  obj.traverse((item) => {
    if (item.isMesh) {
      item.name = `${name}_${i}`;
      ++i;
    }
  });
}

function addModel() {
  //加载叶片模型
  let p1 = new Promise((resolve) => {
    loader
      .setPath("./model/")
      .setDRACOLoader(new DRACOLoader().setDecoderPath("./js/draco/gltf/"))
      .load("GLTF/yepian.glb", (gltf) => {
        // let an = gsap.to(gltf.scene.rotation, 5, {
        //   x: gltf.scene.rotation.x + Math.PI * 2,
        //   repeat: -1,
        //   ease: Linear.easeOut,
        // });
        let an = gsap.to(gltf.scene.rotation, {
          duration: 5,
          x: gltf.scene.rotation.x + Math.PI * 2,
          repeat: -1,
          ease: "none",
        });

        arrTweenAn.push(an);
        gltf.scene.position.set(-2.22, 99.65, 0);
        resolve(gltf.scene);
      });
  });

  // 加载风机柱子
  let p2 = new Promise((resolve) => {
    loader
      .setPath("./model/")
      .setDRACOLoader(new DRACOLoader().setDecoderPath("./js/draco/gltf/"))
      .load("GLTF/zhuji2.glb", (gltf) => {
        gltf.scene.position.set(17.3, 2.73, -1.85);
        resolve(gltf.scene);
      });
  });

  // 加载地形1
  let p3 = new Promise((resolve) => {
    loader
      .setPath("./model/")
      .setDRACOLoader(new DRACOLoader().setDecoderPath("./js/draco/gltf/"))
      .load("GLTF/dixing1.glb", (gltf) => {
        land1 = gltf.scene;
        gltf.scene.traverse((child) => {
          if (child.isMesh) {
            child.material.side = 2;
            if (child.name.includes("地形") || child.name.includes("山脉")) {
              child.material.transparent = true;
              child.material.metalness = 0.8;
              child.material.roughness = 0.71;
              child.material.opacity = 1;
              child.material.color = new THREE.Color(0x132441);
              child.material.emissive = new THREE.Color(0x132441);
            }
          }
        });
        resolve(gltf.scene);
      });
  });

  // 加载地形2
  let p4 = new Promise((resolve) => {
    loader
      .setPath("./model/")
      .setDRACOLoader(new DRACOLoader().setDecoderPath("./js/draco/gltf/"))
      .load("GLTF/dixing2.glb", (gltf) => {
        land2 = gltf.scene;
        gltf.scene.traverse((child) => {
          if (child.isMesh) {
            child.material.side = 2;
            if (child.name.includes("地形") || child.name.includes("山脉")) {
              child.material.transparent = true;
              child.material.metalness = 0.8;
              child.material.roughness = 0.71;
              child.material.opacity = 1;
              child.material.color = new THREE.Color(0x132441);
              child.material.emissive = new THREE.Color(0x132441);
            }
          }
        });
        resolve(gltf.scene);
      });
  });

  // 加载地形3
  let p5 = new Promise((resolve) => {
    loader
      .setPath("./model/")
      .setDRACOLoader(new DRACOLoader().setDecoderPath("./js/draco/gltf/"))
      .load("GLTF/dixing3.glb", (gltf) => {
        land3 = gltf.scene;
        gltf.scene.traverse((child) => {
          if (child.isMesh) {
            child.material.side = 2;
            if (child.name.includes("地形") || child.name.includes("山脉")) {
              child.material.transparent = true;
              child.material.metalness = 0.8;
              child.material.roughness = 0.71;
              child.material.opacity = 1;
              child.material.color = new THREE.Color(0x132441);
              child.material.emissive = new THREE.Color(0x132441);
            }
          }
        });
        resolve(gltf.scene);
      });
  });

  // 流光风叶
  let p6 = new Promise((resolve) => {
    loader
      .setPath("./model/")
      .setDRACOLoader(new DRACOLoader().setDecoderPath("./js/draco/gltf/"))
      .load("GLTF/liuguang.glb", (gltf) => {
        let an = gsap.to(gltf.scene.rotation, {
          duration: 5,
          x: gltf.scene.rotation.x + Math.PI * 2,
          repeat: -1,
          ease: "none",
        });

        arrTweenAn.push(an);
        resolve(gltf.scene);
      });
  });

  // 风机底盘
  let p7 = new Promise((resolve) => {
    loader
      .setPath("./model/")
      .setDRACOLoader(new DRACOLoader().setDecoderPath("./js/draco/gltf/"))
      .load("GLTF/dipan2.glb", (gltf) => {
        gltf.scene.scale.set(0.04, 0.04, 0.04);
        gltf.scene.position.set(0, -5, -0.7);
        gltf.scene.rotation.set(0, 90.43445, 0);
        resolve(gltf.scene);
      });
  });

  // 风叶中电子元件
  let p8 = new Promise((resolve) => {
    loader
      .setPath("./model/")
      .setDRACOLoader(new DRACOLoader().setDecoderPath("./js/draco/gltf/"))
      .load("GLTF/toubu.glb", (gltf) => {
        gltf.scene.position.set(3.5, 0, 0);
        gltf.scene.rotation.set(10, 0, 0);
        resolve(gltf.scene);
      });
  });

  // 尾部电子元件
  let p9 = new Promise((resolve) => {
    loader
      .setPath("./model/")
      .setDRACOLoader(new DRACOLoader().setDecoderPath("./js/draco/gltf/"))
      .load("GLTF/zhuzhou.glb", (gltf) => {
        gltf.scene.position.set(14.1, 100, -1.8);
        resolve(gltf.scene);
      });
  });

  // 舱体底部元件2
  let p10 = new Promise((resolve) => {
    loader
      .setPath("./model/")
      .setDRACOLoader(new DRACOLoader().setDecoderPath("./js/draco/gltf/"))
      .load("GLTF/dizuo_2.glb", (gltf) => {
        gltf.scene.position.set(37, 95, -1.8);
        resolve(gltf.scene);
      });
  });
  // 发动机
  let p11 = new Promise((resolve) => {
    loader
      .setPath("./model/")
      .setDRACOLoader(new DRACOLoader().setDecoderPath("./js/draco/gltf/"))
      .load("GLTF/fadongji.glb", function (gltf) {
        gltf.scene.position.set(26.4, 102, -1.8);
        resolve(gltf.scene);
      });
  });

  // 舱体底部元件1
  let p12 = new Promise((resolve) => {
    loader
      .setPath("./model/")
      .setDRACOLoader(new DRACOLoader().setDecoderPath("./js/draco/gltf/"))
      .load("GLTF/dizuo_1.glb", (gltf) => {
        gltf.scene.position.set(20, 95, -1.8);
        resolve(gltf.scene);
      });
  });

  // 舱体后主机1
  let p13 = new Promise((resolve) => {
    loader
      .setPath("./model/")
      .setDRACOLoader(new DRACOLoader().setDecoderPath("./js/draco/gltf/"))
      .load("GLTF/houzhuji1.glb", (gltf) => {
        gltf.scene.position.set(40, 102, -1.8);
        resolve(gltf.scene);
      });
  });

  // 舱体电子元件3
  let p14 = new Promise((resolve) => {
    loader
      .setPath("./model/")
      .setDRACOLoader(new DRACOLoader().setDecoderPath("./js/draco/gltf/"))
      .load("GLTF/dianzi_3.glb", (gltf) => {
        gltf.scene.position.set(33.1, 99.5, -1.8);
        resolve(gltf.scene);
      });
  });

  // 舱体底座元件3
  let p15 = new Promise((resolve) => {
    loader
      .setPath("./model/")
      .setDRACOLoader(new DRACOLoader().setDecoderPath("./js/draco/gltf/"))
      .load("GLTF/dizuo_3.glb", (gltf) => {
        gltf.scene.position.set(18, 92, -1.8);
        resolve(gltf.scene);
      });
  });

  // 舱体底座元件4
  let p16 = new Promise((resolve) => {
    loader
      .setPath("./model/")
      .setDRACOLoader(new DRACOLoader().setDecoderPath("./js/draco/gltf/"))
      .load("GLTF/dizuo_4.glb", (gltf) => {
        gltf.scene.position.set(19.5, 95.8, -1.8);
        resolve(gltf.scene);
      });
  });

  // 舱体电子元件1
  let p17 = new Promise((resolve) => {
    loader
      .setPath("./model/")
      .setDRACOLoader(new DRACOLoader().setDecoderPath("./js/draco/gltf/"))
      .load("GLTF/dianzi_1.glb", (gltf) => {
        gltf.scene.position.set(31, 98, -4.8);
        resolve(gltf.scene);
      });
  });

  // 舱体电子元件2
  let p18 = new Promise((resolve) => {
    loader
      .setPath("./model/")
      .setDRACOLoader(new DRACOLoader().setDecoderPath("./js/draco/gltf/"))
      .load("GLTF/dianzi_2.glb", (gltf) => {
        gltf.scene.position.set(39.5, 98.5, -4.8);
        resolve(gltf.scene);
      });
  });

  Promise.all([
    p1,
    p2,
    p3,
    p4,
    p5,
    p6,
    p7,
    p8,
    p9,
    p10,
    p11,
    p12,
    p13,
    p14,
    p15,
    p16,
    p17,
    p18,
  ]).then((result) => {
    fan.add(result[0], result[1]);
    fan.scale.set(0.005, 0.005, 0.005);
    fan.position.set(5.11, 3.19598, 12.016);
    fan.rotation.set(0, 89.5, 0);
    sceneOne.add(fan);
    sceneOne.add(land1); // 把初始的地形123加入场景1
    sceneOne.add(land2);
    sceneOne.add(land3);

    // 修改模型名称
    resetName(result[8], "zhuzhou");
    resetName(result[9], "dizuo_2");
    resetName(result[10], "fadongji");
    resetName(result[11], "dizuo_1");
    resetName(result[12], "houzhuji1");
    resetName(result[13], "dianzi_3");
    resetName(result[14], "dizuo_3");
    resetName(result[15], "dizuo_4");
    resetName(result[16], "dianzi_1");
    resetName(result[17], "dianzi_2");
    // 场景2
    flowTrunk.add(result[5], result[7]);
    flowTrunk.position.set(-2.2235, 99.65812, -1.5);
    flowFan.add(
      result[1].clone(),
      flowTrunk,
      result[8],
      result[9],
      result[10],
      result[11],
      result[12],
      result[13],
      result[14],
      result[15],
      result[16],
      result[17]
    );
    flowFan.scale.set(0.04, 0.04, 0.04);
    flowFan.rotation.set(0, 89.5, 0);
    flowFan.position.set(0, -1, 0);
    sceneTwo.add(flowFan);
    sceneTwo.add(result[6]);
    sceneTwo.visible = false;

    // 克隆风机
    cloneFan(0.74852, 2.56534, 7.44558);
    cloneFan(8.29076, 3.19363, 13.22294);
    cloneFan(5.24642, 3.49002, 10.11123);
    cloneFan(7.75871, 3.77725, 10.01902);
    cloneFan(-0.04669, 2.94419, 3.38853);
    cloneFan(6.58057, 3.38455, 0.83203);
    cloneFan(10.53541, 3.58002, -2.88202);
    cloneFan(-3.43544, 2.85505, 5.41173);
    cloneFan(-2.74084, 2.92708, 11.78238);
    cloneFan(-10.1402, 3.29675, 2.79538);
    cloneFan(-2.99843, 3.09172, -6.22022);
    cloneFan(15.37429, 3.50198, -0.2355);
    cloneFan(9.12926, 3.19598, -14.55305);
    cloneFan(3.46791, 2.56157, 0.53953);

    // 克隆地形1
    cloneLand1(8.70549, 2.86865, -1.35901);
    cloneLand1(18.70101, 2.64197, 10.13471);
    cloneLand1(-5.72901, 2.8955, 15.39313);
    cloneLand1(21.694, 2.56808, 18.92285);
    cloneLand1(7.39948, 2.72258, -13.35363);
    cloneLand1(14.23939, 2.75916, -27.02723);
    cloneLand1(17.33572, 2.73076, -1.85427);
    cloneLand1(17.33572, 2.73076, -14.2357);
    cloneLand1(-10.18862, 2.90444, -18.9965);
    cloneLand1(-15.615, 2.73076, -28.86956);
    cloneLand1(-15.615, 2.73076, -9.47457);
    cloneLand1(5.3932, 2.84328, -33.30284);
    cloneLand1(9.75895, 2.73076, -46.883);
    cloneLand1(17.33572, 2.73076, -37.66992);
    cloneLand1(28.728, 2.89401, -26.29959);
    cloneLand1(28.7608, 2.781, -1.85427);
    cloneLand1(28.702, 2.73076, 18.6807);
    cloneLand1(28.702, 2.73076, 33.284);
    cloneLand1(11.26473, 2.8502, 33.284);
    cloneLand1(-5.63942, 2.73076, 23.43441);
    cloneLand1(-15.615, 2.94279, 33.284);
    cloneLand1(-15.615, 2.73076, 13.04465);
    cloneLand1(-15.615, 2.94475, 0.39202);
    cloneLand1(-15.615, 2.73076, -19.4563);
    cloneLand1(28.702, 2.73076, -46.883);
    cloneLand1(-15.615, 2.73076, -46.883);

    // 克隆地形2
    cloneLand2(-4.39732, 2.31106, -7.5534);
    cloneLand2(-11.34043, 2.31106, 3.62787);
    cloneLand2(7.58834, 2.31106, 24.15814);
    cloneLand2(-0.14082, 2.31106, -18.75615);
    cloneLand2(-5.01234, 2.31106, -32.15209);
    cloneLand2(-4.23729, 2.0619, -46.883);
    cloneLand2(21.94042, 2.31106, -46.88289);
    cloneLand2(28.702, 2.31106, -36.6848);
    cloneLand2(28.68036, 2.28686, -13.23737);
    cloneLand2(28.716, 2.26106, 8.15521);
    cloneLand2(18.61536, 2.31106, 33.284);
    cloneLand2(-0.92105, 2.31106, 33.284);
    cloneLand2(-15.615, 2.31106, 22.367);
    cloneLand2(-15.615, 2.14719, -38.39289);
    land1.position.set(17.33572, 2.7305, -1.85427);
    land2.position.set(20.758, 1.73076, -3.13564);
    land3.position.set(6.37, 2.578, 11.246);
    land3.rotation.set(0, 29.87, 0);
    scene.add(sceneOne);
    scene.add(sceneTwo);
  });
}

// 三个按钮：有先后顺序：风机视角 -》舱体视角 -》 爆炸分裂； 关闭就要逆着来
function showFan() {
  if (isShowDetail.value || isPlit.value || loading.value) return; // 走到了后面步骤的，不允许直接操作前面
  isShowFan.value = !isShowFan.value;
  loading.value = true;
  if (isShowFan.value) {
    gsap.to(camera.position, {
      duration: 5,
      x: 9.536,
      y: 3.862,
      z: 15.19,
      repeat: 0,
      ease: "expo",
      onComplete: () => {
        sceneOne.visible = false;
        sceneTwo.visible = true;
        loading.value = false;
      },
    });
  } else {
    sceneOne.visible = true;
    sceneTwo.visible = false;
    gsap.to(camera.position, {
      duration: 5,
      x: 10,
      y: 7,
      z: 20,
      repeat: 0,
      ease: "expo",
      onComplete: () => {
        loading.value = false;
      },
    });
  }
}

function showDetail() {
  if (!isShowFan.value || isPlit.value) return; // 风机视角下才可以操作
  isShowDetail.value = !isShowDetail.value;
}

function splitFan() {
  if (!isShowFan.value || !isShowDetail.value) return; // 风机视角下才可以操作
  isPlit.value = !isPlit.value;
}

function start() {
  const { value } = isStart;
  isStart.value = !value;
  console.log(value, isStart.value);
  arrTweenAn.forEach((an) => {
    isStart.value ? an.play() : an.pause();
  });
}

onMounted(() => {
  init();
  addModel();
});
</script>

<style scoped lang="scss">
.main,
#container {
  width: 100%;
  height: 100%;
}
.btn-group {
  position: fixed;
  bottom: 0;
  left: 50%;
  transform: translate(0, -50%);
  display: flex;
  .btn-item {
    width: 78px;
    height: 73px;
    text-align: center;
    line-height: 73px;
    margin-right: 30px;
    color: #f8f8f8;
    cursor: pointer;
  }
  .start-stop {
    background: url("@/assets/btn_nor_fjkq.png") center center no-repeat;
  }
  .start-stop.active {
    background-image: url("@/assets/btn_hot_fjkq.png");
  }
  .fan-view {
    background: url("@/assets/btn_nor_szfj.png") center center no-repeat;
  }
  .fan-view.active {
    background: url("@/assets/btn_hot_szfj.png") center center no-repeat;
  }
  .detail-view {
    background: url("@/assets/btn_nor_ctsj.png") center center no-repeat;
  }
  .detail-view.active {
    background: url("@/assets/btn_hot_ctsj.png") center center no-repeat;
  }
  .split-fan {
    background: url("@/assets/btn_nor_bz.png") center center no-repeat;
  }
  .split-fan.active {
    background: url("@/assets/btn_hot_bz.png") center center no-repeat;
  }
}
</style>
