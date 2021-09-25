<template>
  <div id="app">
    <div id="sceneCanvas" style="height: 100vh; width: 100%"></div>
    <el-row style="position: fixed; width: 100%; padding: 0; top: 0">
      <el-col :span="22">
        <el-header>
          <el-menu @select="handleSelect" class="el-menu-demo" mode="horizontal">
            <el-submenu index="1">
              <template slot="title">文件</template>
              <el-menu-item index="1-1">新场景</el-menu-item>
              <el-menu-item index="1-2">导入</el-menu-item>
              <el-menu-item index="1-3">导入fbx</el-menu-item>
            </el-submenu>
            <el-submenu index="2">
              <template slot="title">添加</template>
              <el-menu-item index="2-1">正方体</el-menu-item>
              <el-menu-item index="2-2">圆</el-menu-item>
              <el-menu-item index="2-3">圆柱体</el-menu-item>
            </el-submenu>
          </el-menu>
        </el-header>
      </el-col>
      <el-col :span="2" style="height: 60px;line-height:60px;">
        <el-checkbox v-model="isState" @change="setIsState">帧数/内存</el-checkbox>
      </el-col>
    </el-row>
    <el-aside
      style="width: 15%;position:absolute;height:calc(100% - 60px);top: 60px;right: 0px;background-color: #eee;">
      <el-row>
        <el-col :span="24">
          <div class="grid-content bg-purple">
            场景
          </div>
        </el-col>
        <el-col :span="24">
          <div class="box-card" style=" height: 300px;background-color: #ffffff;width: 95%;margin-left: 2.5%;">
            <el-tree :data="data" :props="defaultProps" @node-click="handleNodeClick">
            </el-tree>
          </div>
        </el-col>
        <el-col :span="24">
          <div class="box-card"
            style="text-align: left;min-height: 300px;background-color: #eee;width: 95%;margin-left: 2.5%;overflow:hidden;white-space: nowrap;">
            <el-tabs>
              <el-tab-pane label="属性" name="first">
                <el-row style="margin-top: 10px;">
                  <el-col :span="6">
                    type
                  </el-col>
                  <el-col :span="18">
                    {{selectData.type}}
                  </el-col>
                </el-row>
                <el-row style="margin-top: 10px;">
                  <el-col :span="6">
                    uuid
                  </el-col>
                  <el-col :span="18">
                    {{selectData.uuid}}
                  </el-col>
                </el-row>
                <el-row style="margin-top: 10px;">
                  <el-col :span="6">
                    name
                  </el-col>
                  <el-col :span="18">
                    <el-input height=20 v-model="selectData.name" placeholder="请输入内容"></el-input>
                  </el-col>
                </el-row>
                <el-row style="margin-top: 10px;">
                  <el-col :span="6">
                    type
                  </el-col>
                  <el-col :span="18">
                    typexxxxxx
                  </el-col>
                </el-row>
                <el-row style="margin-top: 10px;">
                  <el-col :span="6">
                    type
                  </el-col>
                  <el-col :span="18">
                    typexxxxxx
                  </el-col>
                </el-row>
              </el-tab-pane>
              <el-tab-pane label="几何" name="second">
                几何
              </el-tab-pane>
              <el-tab-pane label="材质" name="third">
                材质
              </el-tab-pane>
            </el-tabs>
          </div>
        </el-col>


      </el-row>
    </el-aside>
  </div>
</template>

<script>
  import * as THREE from "three";
  import { OrbitControls } from "@three-ts/orbit-controls";
  import Stats from "three/examples/jsm/libs/stats.module";

  export default {
    name: "App",
    data() {
      return {
        camera: null,
        renderer: null,
        scene: null,
        orbit: null,
        stats: null,
        isState: true,
        drawer: true,
        defaultProps: {
          label: 'type',
          children: 'children'
        },
        data: [],
        selectData: [],
      };
    },
    created() { },
    mounted() {
      var _self = this;
      _self.initThree();
      _self.initScene();
      _self.initCamera();
      _self.initAmLight();
      _self.initMeshFloor();
      _self.initState();
      _self.pushData();
      _self.animate();
    },
    methods: {
      handleNodeClick(data) {
        this.selectData = data;
      },
      pushData() {
        this.data = this.scene.children;
        this.selectData = this.data[0];
      },
      handleSelect(key) {
        if (key === '2-1') {
          //添加正放体
          var cubeGeometry = new THREE.BoxGeometry(4, 4, 4);

          //设置颜色线框显示否
          var cubeMaterial = new THREE.MeshBasicMaterial({ color: 0xff0000});
          var cube = new THREE.Mesh(cubeGeometry, cubeMaterial);

          //设置cube的位置 
          cube.position.x = 0;
          cube.position.y = 0;
          cube.position.z = 0;

          //cube添加到场景中
          this.scene.add(cube);
        }
      },
      setIsState(val) {
        this.stats.domElement.style.display = val == true ? '' : 'none';
      },
      initThree() {
        this.width = document.getElementById("sceneCanvas").clientWidth;
        this.height = document.getElementById("sceneCanvas").clientHeight;
        this.renderer = new THREE.WebGLRenderer({
          antialias: true,
        });
        this.renderer.setSize(this.width, this.height);
        this.renderer.autoClear = false;
        this.renderer.shadowMap.enabled = true;
        this.renderer.shadowMapSoft = true; //关键
        this.renderer.shadowMap.type = THREE.PCFSoftShadowMap; //关键
        document
          .getElementById("sceneCanvas")
          .appendChild(this.renderer.domElement);
      },
      initCamera() {
        this.camera = new THREE.PerspectiveCamera(
          30,
          this.width / this.height,
          0.1,
          10000
        );
        this.camera.position.set(0, 300, 600);
        this.camera.lookAt(0, 0, 0);
        this.orbit = new OrbitControls(this.camera, this.renderer.domElement);
        this.orbit.target = new THREE.Vector3(0, 0, 0); //控制焦点

        this.camera.lookAt(0, 0, 0);
        this.orbit.update();
        this.scene.add(this.camera);
      },
      initScene() {
        this.scene = new THREE.Scene();
        this.scene.background = new THREE.Color("0xa0a0a0");
        // this.scene.fog = new THREE.FogExp2(0xa0a0a0, 0.0001);
      },
      initAmLight() {
        const hemiLight = new THREE.HemisphereLight(0xffffff, 0.5);
        hemiLight.position.set(0, 20, 0);
        hemiLight.name = "环境光";
        this.scene.add(hemiLight);
      },
      initMeshFloor() {
        let groundGeom = new THREE.BoxBufferGeometry(400, 0.2, 400);
        let groundMate = new THREE.MeshPhongMaterial({ color: 0xa0a0a0 });
        let ground = new THREE.Mesh(groundGeom, groundMate);
        ground.position.y = -0.1;
        ground.receiveShadow = true;
        ground.castShadow = true;
        ground.name = "地面";



        this.scene.add(ground);

      },
      initState() {
        this.stats = new Stats();
        this.stats.domElement.style.position = "absolute";
        this.stats.domElement.style.bottom = "0px";
        this.stats.domElement.style.top = "";
        document.getElementById("sceneCanvas").appendChild(this.stats.domElement);
      },
      animate() {
        this.renderer.render(this.scene, this.camera);
        // this.material.uniforms['time'].value = .00025 * (Date.now() - this.start);
        // this.updateOri();
        // this.update();
        this.stats.update();
        requestAnimationFrame(this.animate);
      },
    },
  };
</script>

<style>
  #app {
    font-family: Avenir, Helvetica, Arial, sans-serif;
    -webkit-font-smoothing: antialiased;
    -moz-osx-font-smoothing: grayscale;
    text-align: center;
    color: #2c3e50;
    /* margin-top: 60px; */
  }
</style>