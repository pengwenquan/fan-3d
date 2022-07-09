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
    <div class="cabin-popup" ref="Alarm_pop" :style="{ display: bPopus1 }">
      <div class="cabin-popup-dian"></div>
      <div class="cabin-popup-line"></div>
      <div class="cabin-popup-image"></div>
      <div class="cabin-popup-logo"></div>
      <div class="air-cooling">
        <span class="cabin-popup-name1">{{ PopupArrValue.title }}</span>
        <span class="cabin-popup-name2">{{ PopupArrValue.name1 }}</span>
        <span class="cabin-popup-name3">{{ PopupArrValue.name2 }}</span>
        <span class="cabin-popup-name4">{{ PopupArrValue.name3 }}</span>
        <span class="cabin-popup-name5">{{ PopupArrValue.name4 }}</span>
        <span class="cabin-popup-name6">{{ PopupArrValue.number1 }}</span>
        <span class="cabin-popup-name7">{{ PopupArrValue.number2 }}</span>
        <span class="cabin-popup-name8">{{ PopupArrValue.number3 }}</span>
        <span class="cabin-popup-name9">{{ PopupArrValue.number4 }}</span>
        <span class="cabin-popup-name10">{{ PopupArrValue.Symbol1 }}</span>
        <span class="cabin-popup-name11">{{ PopupArrValue.Symbol2 }}</span>
        <span class="cabin-popup-name12">{{ PopupArrValue.Symbol3 }}</span>
        <span class="cabin-popup-name13">{{ PopupArrValue.Symbol4 }}</span>
      </div>
    </div>
  </div>
</template>

<script setup>
import { onMounted, ref, reactive } from "vue";
import * as THREE from "three";
import { OrbitControls } from "three/examples/jsm/controls/OrbitControls";
import { GLTFLoader } from "three/examples/jsm/loaders/GLTFLoader";
import { DRACOLoader } from "three/examples/jsm/loaders/DRACOLoader";
import gsap, { Linear } from "gsap";

let renderer = new THREE.WebGLRenderer({
  alpha: true,
  antialias: true,
  logarithmicDepthBuffer: true,
});
let scene = new THREE.Scene();
let camera = new THREE.PerspectiveCamera(
  45,
  window.innerWidth / window.innerHeight,
  0.1,
  10000
);
let light = new THREE.DirectionalLight(0xffffff);
let loader = new GLTFLoader();
let arrTweenAn = [];
let fan = new THREE.Object3D();
let land1, land2, land3;
let sceneOne = new THREE.Object3D();
let sceneTwo = new THREE.Object3D();
// 流光风机
let flowTrunk = new THREE.Object3D();
let flowFan = new THREE.Object3D();
let textureloader = new THREE.TextureLoader();
let textureFlow = textureloader.load("./texture/light-animation.png");
let textureMaterial;
let beLoaded = false;

// 描边相关参数
let arrNeedOutline = [];
let nowSelected;

// 按钮点中状态
let isStart = ref(true);
let isShowFan = ref(false);
let isShowDetail = ref(false);
let isPlit = ref(false);
let loading = ref(false);
let PopupArrValue = reactive({
  title: "风冷装置",
  name1: "风冷功率",
  name2: "风冷温度",
  name3: "功率",
  name4: "温度",
  number1: "8",
  number2: "300",
  number3: "200",
  number4: "24",
  Symbol1: "°C",
  Symbol2: "KWh",
  Symbol3: "KW",
  Symbol4: "°C",
});
let bPopus1 = ref("none");
let Alarm_pop = ref(null);

function init() {
  camera.position.set(10, 7, 20);
  scene.add(camera);

  light.position.set(20, 20, 20);
  scene.add(light);

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
    if (beLoaded) {
      textureMaterial.map.offset.x += 0.01;
    }
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

function addOutLine(obj, strName, bErr) {
  let cloneObj = obj.clone();
  cloneObj.position.x = obj.position.x + 0.1;
  cloneObj.scale.set(1.02, 1.02, 1.02);
  cloneObj.visible = false;
  stroke(cloneObj, bErr);
  arrNeedOutline.push({
    name: strName,
    obj: cloneObj,
    err: bErr,
  });
  flowFan.add(cloneObj);
}

function stroke(obj, bErr) {
  let meshMaterial = new THREE.MeshBasicMaterial();
  obj.traverse((child) => {
    if (child.isMesh) {
      child.material = meshMaterial;
      child.material.side = 1;
      child.material.color = bErr
        ? new THREE.Color(0xff0000)
        : new THREE.Color(0xffffff);
    }
  });
}

function findModelSelected(strName) {
  let msize = arrNeedOutline.length;
  for (let i = 0; i < msize; i++) {
    if (strName.indexOf(arrNeedOutline[i].name) !== -1) {
      nowSelected = arrNeedOutline[i].obj;
      changeStatus(arrNeedOutline[i].obj, true, arrNeedOutline[i].err);
      return arrNeedOutline[i].name;
    }
  }
}

function setErrStatus(strName, bErr) {
  let nSize = arrNeedOutline.length;
  for (var i = 0; i < nSize; ++i) {
    if (strName == arrNeedOutline[i].name) {
      arrNeedOutline[i].err = bErr;
      break;
    }
  }
}

// 监听点击事件
function mouseClick(eve) {
  if (splitFan.value) return;
  eve.preventDefault();
  eve.stopPropagation();
  const mouse = new THREE.Vector2();
  mouse.x = (eve.offsetX / renderer.domElement.offsetWidth) * 2 - 1;
  mouse.y = -(eve.offsetY / renderer.domElement.offsetHeight) * 2 + 1;
  const raycaster = new THREE.Raycaster();
  raycaster.setFromCamera(mouse, camera);
  let intersects = raycaster.intersectObject(flowFan.children, true);

  if (nowSelected !== undefined) {
    changeStatus(nowSelected, false);
  }

  if (intersects[0] !== undefined) return;

  let strSelectName = findModelSelected(intersects[0].object.name);

  switch (strSelectName) {
    case "houzhuji1":
      getMousePos(Alarm_pop.value, this.dom, 0, -24, -215, -113, eve);
      PopupArrValue.title = "风冷装置";
      PopupArrValue.name1 = "风冷功率";
      PopupArrValue.name2 = "风冷温度";
      PopupArrValue.name3 = "功率";
      PopupArrValue.name4 = "温度";
      PopupArrValue.number1 = "8";
      PopupArrValue.number2 = "300";
      PopupArrValue.number3 = "200";
      PopupArrValue.number4 = "24";
      PopupArrValue.Symbol1 = "°C";
      PopupArrValue.Symbol2 = "KWh";
      PopupArrValue.Symbol3 = "KW";
      PopupArrValue.Symbol4 = "°C";
      bPopus1 = "block";
      break;
    case "zhuzhou":
      getMousePos(Alarm_pop.value, this.dom, 0, -124, -215, -113, eve);
      PopupArrValue.title = "主轴";
      PopupArrValue.name1 = "电压";
      PopupArrValue.name2 = "电流";
      PopupArrValue.name3 = "功率";
      PopupArrValue.name4 = "频率";
      PopupArrValue.number1 = "220";
      PopupArrValue.number2 = "30";
      PopupArrValue.number3 = "10";
      PopupArrValue.number4 = "200";
      PopupArrValue.Symbol1 = "伏特";
      PopupArrValue.Symbol2 = "安培";
      PopupArrValue.Symbol3 = "千瓦";
      PopupArrValue.Symbol4 = "赫兹";
      bPopus1 = "block";
      break;
    case "fadongji":
      getMousePos(Alarm_pop.value, this.dom, 0, -124, -215, -113, eve);
      PopupArrValue.title = "油冷装置";
      PopupArrValue.name1 = "功率";
      PopupArrValue.name2 = "容量";
      PopupArrValue.name3 = "油耗";
      PopupArrValue.name4 = "工作时间";
      PopupArrValue.number1 = "15";
      PopupArrValue.number2 = "500";
      PopupArrValue.number3 = "100";
      PopupArrValue.number4 = "72";
      PopupArrValue.Symbol1 = "KW";
      PopupArrValue.Symbol2 = "L";
      PopupArrValue.Symbol3 = "G/KW.H";
      PopupArrValue.Symbol4 = "H";
      bPopus1 = "block";
      break;
    case "dizuo_1":
      getMousePos(Alarm_pop.value, this.dom, 0, -124, -215, -113, eve);
      PopupArrValue.title = "偏航装置";
      PopupArrValue.name1 = "速度";
      PopupArrValue.name2 = "制动力矩";
      PopupArrValue.name3 = "功率";
      PopupArrValue.name4 = "频率";
      PopupArrValue.number1 = "20";
      PopupArrValue.number2 = "25";
      PopupArrValue.number3 = "6";
      PopupArrValue.number4 = "150";
      PopupArrValue.Symbol1 = "R";
      PopupArrValue.Symbol2 = "Nm";
      PopupArrValue.Symbol3 = "Kw";
      PopupArrValue.Symbol4 = "Hrz";
      bPopus1 = "block";
      break;
    case "dizuo_2":
      getMousePos(Alarm_pop.value, this.dom, 0, -24, -215, -113, eve);
      PopupArrValue.title = "底座";
      PopupArrValue.name1 = "耐热温度";
      PopupArrValue.name2 = "损耗程度";
      PopupArrValue.name3 = "湿度";
      PopupArrValue.number1 = "160";
      PopupArrValue.number2 = "10";
      PopupArrValue.number3 = "40";
      PopupArrValue.Symbol1 = "°C";
      PopupArrValue.Symbol2 = "%";
      PopupArrValue.Symbol3 = "%";
      bPopus1 = "block";
      break;
    case "dizuo_3":
      getMousePos(Alarm_pop.value, this.dom, 0, -124, -215, -113, eve);
      PopupArrValue.title = "齿轮系统";
      PopupArrValue.name1 = "油槽温度";
      PopupArrValue.name2 = "入口轴温度";
      PopupArrValue.name3 = "输入轴温度";
      PopupArrValue.name4 = "输出轴温度";
      PopupArrValue.number1 = "60";
      PopupArrValue.number2 = "50";
      PopupArrValue.number3 = "70";
      PopupArrValue.number4 = "65";
      PopupArrValue.Symbol1 = "°C";
      PopupArrValue.Symbol2 = "°C";
      PopupArrValue.Symbol3 = "°C";
      PopupArrValue.Symbol4 = "°C";
      bPopus1 = "block";
      break;
    default:
      bPopus1 = "none";
  }
}

function getMousePos(o, canvas, x, x1, y, y1, event) {
  let rect = canvas.getBoundingClientRect();

  let positionX = event.clientX - rect.left;
  let positionY = event.clientY - rect.top;
  o.style.position = "absolute";
  if (positionY > 600) {
    o.style.top = y1 + positionY + "px"; //用鼠标指针的y轴坐标和传入偏移值设置对象y轴坐标
  } else {
    o.style.top = y + positionY + "px";
  }

  if (positionX > 1200) {
    o.style.left = x1 + positionX + "px"; //用鼠标指针的x轴坐标和传入偏移值设置对象x轴坐标
  } else {
    o.style.left = x + positionX + "px"; //用鼠标指针的x轴坐标和传入偏移值设置对象x轴坐标
  }
}

function changeStatus(cloneObj, bEnter) {
  cloneObj.visible = bEnter;
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
        textureMaterial = gltf.scene.getObjectByName("模型").material;
        textureMaterial.map = textureFlow;
        textureMaterial.transparent = true;
        textureMaterial.map.wrapS = textureMaterial.map.wrapT =
          THREE.RepeatWrapping;

        gltf.scene.getObjectByName("模型").material = textureMaterial;
        gltf.scene.getObjectByName("模型_1").material = textureMaterial;
        gltf.scene.getObjectByName("模型_2").material = textureMaterial;
        gltf.scene.traverse((child) => {
          if (child.isMesh) {
            child.material.side = 2;
          }
        });
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

    // 添加描边
    addOutLine(result[8], "zhuzhou", false);
    addOutLine(result[9], "dizuo_2", false);
    addOutLine(result[10], "fadongji", false);
    addOutLine(result[11], "dizuo_1", false);
    addOutLine(result[12], "houzhuji1", false);
    addOutLine(result[13], "dianzi_3", false);
    addOutLine(result[14], "dizuo_3", false);
    addOutLine(result[15], "dizuo_4", false);
    addOutLine(result[16], "dianzi_1", false);
    addOutLine(result[17], "dianzi_2", false);
    setErrStatus("fadongji", true);
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
    beLoaded = true;
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
  if (!isShowFan.value || isPlit.value || loading.value) return; // 风机视角下才可以操作
  isShowDetail.value = !isShowDetail.value;
  if (isShowDetail.value) {
    gsap.to(camera.position, {
      duration: 5,
      x: 2.29,
      y: 0.66,
      z: 1.187,
      repeat: 0,
      ease: Linear.easeOut,
      onComplete: () => {
        light.position.set(10, 2, 0);
      },
    });

    gsap.to(flowFan.position, {
      duration: 5,
      x: -1,
      y: -4,
      z: 0,
      repeat: 0,
      ease: Linear.easeOut,
    });
  } else {
    gsap.to(camera.position, {
      duration: 5,
      x: 9.536,
      y: 3.862,
      z: 15.19,
      repeat: 0,
      ease: Linear.easeOut,
    });

    gsap.to(flowFan.position, {
      duration: 5,
      x: 0,
      y: -1,
      z: 0,
      repeat: 0,
      ease: Linear.easeOut,
      onComplete: () => {
        light.position.set(10, 10, -10);
      },
    });
  }
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
  window.addEventListener("click", mouseClick.bind(this));
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
.cabin-popup {
  width: 520px;
  height: 231px;
  position: absolute;
  top: 0px;
  pointer-events: none;
}

.cabin-popup-dian {
  width: 23px;
  height: 23px;
  background-image: url("~@/assets/yuandian.png");
  background-repeat: no-repeat;
  position: absolute;
  left: 3px;
  top: 207px;
}

.cabin-popup-line {
  width: 112px;
  height: 90px;
  background-image: url("~@/assets/tankuang_lianxian.png");
  background-repeat: no-repeat;
  position: absolute;
  left: 17px;
  top: 124px;
}

.cabin-popup-image {
  width: 383px;
  height: 191px;
  background-image: url("~@/assets/bg_tankuang.png");
  background-repeat: no-repeat;
  position: absolute;
  left: 122px;
  top: 32px;
  z-index: 10;
}

.cabin-popup-logo {
  width: 110px;
  height: 110px;
  background-image: url("~@/assets/fengji_sbxx.png");
  background-repeat: no-repeat;
  background-size: 106px 97px;
  position: absolute;
  left: 157px;
  top: 74px;
  z-index: 11;
}

.cabin-popup-name1 {
  color: #a3a2d9;
  font-size: 16px;
  position: absolute;
  top: 50px;
  left: 280px;
  z-index: 11;
}

.cabin-popup-name2 {
  color: #fff;
  font-size: 14px;
  position: absolute;
  top: 90px;
  left: 280px;
  z-index: 11;
}

.cabin-popup-name3 {
  color: #fff;
  font-size: 14px;
  position: absolute;
  top: 119px;
  left: 280px;
  z-index: 11;
}

.cabin-popup-name4 {
  color: #fff;
  font-size: 14px;
  position: absolute;
  top: 149px;
  left: 280px;
  z-index: 11;
}

.cabin-popup-name5 {
  color: #fff;
  font-size: 14px;
  position: absolute;
  top: 179px;
  left: 280px;
  z-index: 11;
}

.cabin-popup-name6 {
  color: #f4d310;
  font-size: 16px;
  position: absolute;
  top: 90px;
  left: 376px;
  z-index: 11;
}

.cabin-popup-name7 {
  color: #f4d310;
  font-size: 14px;
  position: absolute;
  top: 119px;
  left: 376px;
  z-index: 11;
}

.cabin-popup-name8 {
  color: #f4d310;
  font-size: 14px;
  position: absolute;
  top: 149px;
  left: 376px;
  z-index: 11;
}

.cabin-popup-name9 {
  color: #f4d310;
  font-size: 14px;
  position: absolute;
  top: 179px;
  left: 376px;
  z-index: 11;
}

.cabin-popup-name10 {
  color: #6f71a2;
  font-size: 14px;
  position: absolute;
  top: 90px;
  left: 442px;
  z-index: 11;
}

.cabin-popup-name11 {
  color: #6f71a2;
  font-size: 14px;
  position: absolute;
  top: 119px;
  left: 442px;
  z-index: 11;
}

.cabin-popup-name12 {
  color: #6f71a2;
  font-size: 14px;
  position: absolute;
  top: 149px;
  left: 442px;
  z-index: 11;
}

.cabin-popup-name13 {
  color: #6f71a2;
  font-size: 14px;
  position: absolute;
  top: 179px;
  left: 442px;
  z-index: 11;
}
</style>
