<template>
  <div class="demo-tool" id="app">
    <span> 估计通信半径 </span>
    <input value="20000" ref="Radius" @mouseleave="updateConfigs">
    <span> 切换颜色 </span>
    <select ref="RadiusColor" @mouseleave="updateConfigs2">
      <option value="RED">红色</option>
      <option value="GREEN">绿色</option>
      <option value="GOLD">金色</option>
      <option value="HOTPINK">粉色</option>
      <option value="AQUA">蓝色</option>
      <option value="CHARTREUSE">棕色</option>
    </select>
    <select ref="selectedScenario" @mouseleave="ScenarioChanged" @mouseenter="ScenarioMaintained">
      <option value="想定1">想定1</option>
      <option value="想定2">想定2</option>
      <option value="想定3">想定3</option>
      <option value="想定4">想定4</option>
    </select>



  <vc-viewer
    class="viewer"
    :animation="animation"
    :timeline="timeline"
    :base-layer-picker="baseLayerPicker"
    :camera.sync="camera"
    :scene-mode-picker="sceneModePicker"
    :navigation-help-button="homeButton"
    @ready="ready"
  >
    <vc-navigation :options="options" />
    <vc-layer-imagery :imagery-provider="imageryProvider" />
    <vc-provider-terrain-cesium :terrain-provider="terrainProvider" />

  </vc-viewer>

</div>
</template>

<script>
import axios from 'axios'
import 'vue-cesium/lib/vc-navigation.css'
let TargetModels =[];
let lastScenario;
export default {
  data() {
    return {
      animation: false,
      timeline: false,
      baseLayerPicker: true,
      sceneModePicker: true,
      homeButton: true,
      camera: {
        position: {
          lng: 120.65,
          lat: 25.64,
          height: 70000
        },
        heading: 360,
        pitch: -90,
        roll: 0
      },
      show: true,
      MapStyle: 'cva_c',
      imageryProvider: {},
      terrainProvider: {},
      options: {
        enableCompass: true,
        enableZoomControl: {
          // 缩放比例
          zoomAmount: 2,
          // 用于在使用重置导航重置地图视图时设置默认视图控制。接受的值是经纬度{lng: number, lat: number, height: number}或者 rectangle{west: number,south: number,east: number,north: number}
          defaultResetView: {
            lng: 120.65, lat: 25.64, height: 70000, heading: 360, pitch: -90, roll: 0
          },
          overrideCamera: false
        },
        enableDistanceLegend: true,
        enableLocationBar: true,
        enableCompassOuterRing: true,
        enablePrintView: false,
        enableMyLocation: false,
      }
    }
  },
  methods: {
    ready(cesiumInstance) {
      const { Cesium, viewer } = cesiumInstance;
      this.cesiumInstance = cesiumInstance;

      /*
      const url_targets = 'http://127.0.0.1:4001/targetinfo/all/';
      const url_sources = 'http://127.0.0.1:4003';
      const url_commlinks = 'http://127.0.0.1:4001/commreqrespevents';
      const url_updatecomm = 'http://127.0.0.1:8889/JXCommunicationRequest'
      const url_updatetarget = 'http://127.0.0.1:8889//JXBattleSituation'
      */
      const url_targets = 'http://manet.synology.me:4001/targetinfo/all/';
      const url_sources = 'http://manet.synology.me:4003';
      const url_commlinks = 'http://manet.synology.me:4001/commreqrespevents';
      const url_updatecomm = 'http://60.205.185.202:8889/JXCommunicationRequest';
      const url_updatetarget = 'http://60.205.185.202:8889//JXBattleSituation';

      this.imageryProvider = new Cesium.UrlTemplateImageryProvider({
        url: url_sources + '/maptile/{z}/{x}/{y}.jpg',
        fileExtension: 'jpg'
      });
      this.terrainProvider = new Cesium.CesiumTerrainProvider(
        {
          url: url_sources + '/terrain/'
        }
      );
      // 个人化控件定制
      var terrainViewModels = [];
      terrainViewModels.push(new Cesium.ProviderViewModel({
        name: 'WGS84 Ellipsoid',
        iconUrl: Cesium.buildModuleUrl('Widgets/Images/TerrainProviders/Ellipsoid.png'),
        tooltip: 'WGS84 standard ellipsoid, also known as EPSG:4326',
        category: '卫星测量',
        creationFunction: function() {
          return new Cesium.EllipsoidTerrainProvider()
        }
      }));
      terrainViewModels.push(new Cesium.ProviderViewModel({
        name: '测试用离线地形',
        iconUrl: Cesium.buildModuleUrl('Widgets/Images/TerrainProviders/CesiumWorldTerrain.png'),
        tooltip: '水经注下载离线高程',
        category: '卫星测量',
        creationFunction: function() {
          return new Cesium.CesiumTerrainProvider({
            url: url_sources + '/terrain/'
          })
        }
      }));
      var imageryViewModels = [];
      imageryViewModels.push(new Cesium.ProviderViewModel({
        name: '离线地图',
        iconUrl: Cesium.buildModuleUrl('Widgets/Images/ImageryProviders/openStreetMap.png'),
        tooltip: '水经注离线下载',
        category: '国内常用',
        creationFunction: function() {
          return new Cesium.UrlTemplateImageryProvider({
            url: url_sources + '/maptile/{z}/{x}/{y}.jpg',
            fileExtension: 'jpg'
          })
        }
      }));
      imageryViewModels.push(new Cesium.ProviderViewModel({
        name: '离线海图',
        iconUrl: Cesium.buildModuleUrl('Widgets/Images/ImageryProviders/openStreetMap.png'),
        tooltip: '水经注离线下载',
        category: '国内常用',
        creationFunction: function() {
          return new Cesium.UrlTemplateImageryProvider({
            url: url_sources + '/nauticaltile/{z}/{x}/{y}.jpg',
            fileExtension: 'jpg'
          })
        }
      }));
      viewer.baseLayerPicker.viewModel.imageryProviderViewModels = imageryViewModels;
      viewer.baseLayerPicker.viewModel.terrainProviderViewModels = terrainViewModels;
      /*
      // Create a CesiumWidget without imagery, if you haven't already done so.
      var cesiumWidget = new Cesium.CesiumWidget('cesiumContainer', { imageryProvider: false })

      // Finally, create the baseLayerPicker widget using our view models.
      var layers = cesiumWidget.imageryLayers
      var baseLayerPicker = new Cesium.BaseLayerPicker('baseLayerPickerContainer', {
        globe: viewer.scene.globe,
        imageryProviderViewModels: imageryViewModels,
        terrainProviderViewModels: terrainViewModels
      })
      console.log(layers)
      console.log(baseLayerPicker)
       */

      axios
        .get(url_targets)
        .then(function(response) {
          if (response.data.msg !== 'success') {
            console.log('Receive Error!')
          }
          console.log(response.data);
          // console.log(response.data.data);
          if (response.data.code === 0) {
            for (let i = 0; i < response.data.data.length; i++) {
              const targetdata = response.data.data[i];
              const entityId = targetdata.id;
              const scenarioName = targetdata.scenarioname;
              const targetId = targetdata.targetId;
              const longitude = targetdata.lon;
              const latitude = targetdata.lat;
              const altitude = targetdata.altitude;
              const name = targetdata.targetname;
              const modelType = targetdata.targetclass;
              let modelShow = false;
              const DegreesHeading = targetdata.heading;
              const ellipsoidRadius = 10000;
              const label = name + ' (' + longitude + ',' + latitude + ',' + altitude + ')';
              const targetPosition = Cesium.Cartesian3.fromDegrees(
                longitude,
                latitude,
                altitude
              );
              const Heading = Cesium.Math.toRadians(DegreesHeading);
              const hpr = new Cesium.HeadingPitchRoll(Heading, 0, 135);
              const targetOrientation = Cesium.Transforms.headingPitchRollQuaternion(
                targetPosition,
                hpr
              );
              if (!TargetModels[scenarioName])
                TargetModels[scenarioName]=[];
              TargetModels[scenarioName][targetId] = targetdata;
              viewer.entities.add({
                name: label,
                id: entityId,
                position: targetPosition,
                orientation: targetOrientation,
                label: new Cesium.LabelGraphics({
                  text: label,
                  fillColor: Cesium.Color.GOLD,
                  font: '20px sans-serif',
                  horizontalOrigin: 1,
                  outlineColor: new Cesium.Color(0, 0, 0, 1),
                  outlineWidth: 2,
                  pixelOffset: new Cesium.Cartesian2(17, -5),
                  style: Cesium.LabelStyle.FILL,
                  scaleByDistance: new Cesium.NearFarScalar(1.5e2, 2, 8.0e5, 0.4),
                  show: modelShow
                }),
                model: new Cesium.ModelGraphics({
                  uri: url_sources + '/models/model' + modelType.toString() + '.gltf',
                  maximumScale: 20000,
                  minimumPixelSize: 50,
                  runAnimations: false,
                  show: modelShow
                }),
                ellipsoid: new Cesium.EllipsoidGraphics({
                  radii: new Cesium.Cartesian3(ellipsoidRadius, ellipsoidRadius, ellipsoidRadius),
                  material: Cesium.Color.RED.withAlpha(0.2),
                  outline: true,
                  fill: false,
                  outlineColor: Cesium.Color.CRIMSON,
                  show: modelShow
                })
              })
            }
          }
        })
        .catch(function(error) {
          console.log(error)
        })
        .finally(function() {

          console.log(TargetModels);
          axios
                  .get(url_commlinks)
                  .then(function(response) {
                    if (response.data.msg !== 'success') {
                      console.log('Receive Error!')
                    }
                    console.log(response.data);
                    if (response.data.code === 0) {
                      for (let i = 0; i < response.data.data.list.length; i++) {
                        const linkdata = response.data.data.list[i];
                        console.log(linkdata);
                        let linkColor;
                        const srcTargetId = linkdata.srcTargetId;
                        const dstTargetId = linkdata.dstTargetId;
                        const linkSuccess = linkdata.success;
                        if (linkSuccess) {
                          linkColor = Cesium.Color.GREEN
                        } else {
                          linkColor = Cesium.Color.RED
                        }
                        viewer.entities.add({
                          name: 'link' + linkdata.dataNo,
                          id: srcTargetId + '-' + dstTargetId,
                          polyline: {
                            positions: Cesium.Cartesian3.fromDegreesArrayHeights([
                              TargetModels[srcTargetId].lon,
                              TargetModels[srcTargetId].lat,
                              TargetModels[srcTargetId].altitude,
                              TargetModels[dstTargetId].lon,
                              TargetModels[dstTargetId].lat,
                              TargetModels[dstTargetId].altitude
                            ]),
                            width: 5,
                            material: new Cesium.PolylineDashMaterialProperty({
                              color: linkColor
                            })
                          }
                        })
                      }
                    }
                  })
                  .catch(function(error) {
                    console.log(error)
                  })
                  .finally(function() {
                  });
        });

      setInterval(function () {
        axios
          .get(url_updatecomm)
          .then(function (response) {
            if (response.data.msg !== 'success') {
              console.log('Receive Error!')
            }
            for (let i = 0;i<6;i++)
              for (let j = 0;j<6;j++)
              {
                let entityID = i + '-' + j;
                let tempEntity = viewer.entities.getById(entityID);
                if (tempEntity){
                  viewer.entities.removeById(entityID)
                }
              }
            console.log(response.data);
            if (response.data.code === 0) {
              for (let i = 0; i < response.data.data.length; i++){
                const linkdata = response.data.data[i];
                const sourceList = linkdata.source_list;
                const dstList = linkdata.destination_list;
                // const dataContent = linkdata.data_content
                // alert(dataContent)
                for (let j = 0;j<sourceList.length; j++){
                  const srcTargetId = sourceList[j];
                  for (let k = 0;k<dstList.length; k++){
                    const dstTargetId = dstList[k];
                    const entityID = srcTargetId + '-' + dstTargetId;
                    viewer.entities.add({
                      name: 'link' + entityID,
                      id: entityID,
                      polyline: {
                        positions: Cesium.Cartesian3.fromDegreesArrayHeights([
                          TargetModels[srcTargetId].lon,
                          TargetModels[srcTargetId].lat,
                          TargetModels[srcTargetId].altitude,
                          TargetModels[dstTargetId].lon,
                          TargetModels[dstTargetId].lat,
                          TargetModels[dstTargetId].altitude
                        ]),
                        width: 5,
                        material: new Cesium.PolylineDashMaterialProperty({
                          color: Cesium.Color.RED
                        })
                      }
                    })

                  }
                }
              }
            }
          })
                .catch(function(error) {
                  console.log(error)
                })
                .finally(function() {
                  console.log('link update over')
                })

      },10000);
      setInterval(function () {
        axios
          .get(url_updatetarget)
          .then(function (response) {
            if (response.data.msg !== 'success') {
              console.log('Receive Error!')
            }
            console.log(response.data);
            if (response.data.code === 0){
              for (let i = 0; i < response.data.data.length; i++){
                const targetList = response.data.data[i].target_list;
                // console.log(targetList);
                for (let j=0;j<targetList.length;j++){
                  const targetData = targetList[j];
                  const targetId = targetData.target_id;
                  const targetLongitude = targetData.longitude;
                  const targetLatitude = targetData.latitude;
                  const targetAltitude = targetData.altitude;
                  TargetModels[targetId].lon = targetLongitude;
                  TargetModels[targetId].lat = targetLatitude;
                  TargetModels[targetId].altitude = targetAltitude;
                  let targetEntity = viewer.entities.getById(targetId);
                  const targetPosition = Cesium.Cartesian3.fromDegrees(
                          targetLongitude,
                          targetLatitude,
                          targetAltitude
                  );
                  const name = targetData.name;
                  const updateLabel = name + ' (' + targetLongitude + ',' + targetLatitude + ',' + targetAltitude + ')';
                  targetEntity.position = targetPosition;
                  targetEntity.label.text = updateLabel;
                  targetEntity.name = updateLabel;
              }
            }
          }})
                .catch(function(error) {
                  console.log(error)
                })
                .finally(function() {
                  console.log('target update over')
                })
      },10000)
    },
  updateConfigs(){
    const {  Cesium, viewer } = this.cesiumInstance;
    const radius = this.$refs.Radius.value;
    const changedScenario = this.$refs.selectedScenario.value;
    let temp = TargetModels[changedScenario];
    for (let i=0;i<temp.length;i++)
    {
      const targetData = temp[i];
      const entityId = targetData.id;
      let temp2 = viewer.entities.getById(entityId);
      temp2._ellipsoid.radii = new Cesium.Cartesian3(radius, radius, radius)
    }
    console.log('updateRadius');
    console.log(temp)
  },
    updateConfigs2(){
      const { Cesium, viewer } = this.cesiumInstance;
      const RadiusColor = this.$refs.RadiusColor.value;
      const changedScenario = this.$refs.selectedScenario.value;
      let temp = TargetModels[changedScenario];
      let EntityColor;
      if (RadiusColor === 'RED') EntityColor = Cesium.Color.RED;
      if (RadiusColor === 'GOLD') EntityColor = Cesium.Color.GOLD;
      if (RadiusColor === 'GREEN') EntityColor = Cesium.Color.GREEN;
      if (RadiusColor === 'AQUA') EntityColor = Cesium.Color.AQUA;
      if (RadiusColor === 'CHARTREUSE') EntityColor = Cesium.Color.CHARTREUSE;
      if (RadiusColor === 'HOTPINK') EntityColor = Cesium.Color.HOTPINK;
      for (let i=0;i<temp.length;i++)
      {
        const targetData = temp[i];
        const entityId = targetData.id;
        let temp2 = viewer.entities.getById(entityId);
        temp2._ellipsoid.outlineColor = EntityColor;
      }
      console.log('updateColor');
      console.log(temp)
    }
    ,ScenarioChanged(){
      const {  viewer } = this.cesiumInstance;
      const changedScenario = this.$refs.selectedScenario.value;
      let temp = TargetModels[changedScenario];
      console.log(temp)
      for (let i=0;i<temp.length;i++)
      {
        const targetData = temp[i];
        const entityId = targetData.id;
        let temp2 = viewer.entities.getById(entityId);
        temp2._model.show = true;
        temp2._label.show = true;
        temp2._ellipsoid.show =true;
      }
      lastScenario = changedScenario;
    },
    ScenarioMaintained(){
      const {  viewer } = this.cesiumInstance;
      let temp = TargetModels[lastScenario];
      for (let i=0;i<temp.length;i++)
      {
        const targetData = temp[i];
        const entityId = targetData.id;
        let temp2 = viewer.entities.getById(entityId);
        temp2._model.show = false;
        temp2._label.show = false;
        temp2._ellipsoid.show =false;
      }

    }
  }

}
</script>
<style>
  .viewer {
    width: 100%;
    height: 100%;
  }
  .demo-tool{
    top: 0;
  }
</style>
