<template>
  <div class="mapWrapper">
    <div id="map"></div>
  </div>
</template>

<script>
  import axios from "axios";
  import echarts from "echarts";
  import cityMap from "../../public/js/china-main-city-map"

  var chinaId = 100000;
  var chinaName = "china";
  var chinaJson = null;

  var mapStack = [];
  var parentId = null;
  var parentName = null;

  var myChart = null;

  export default {
    name: "myMap",
    mounted: function(){
      this.drawChina("map")
    },
    methods: {
      drawChina(container_id){
        axios.get("./map/" + chinaId + ".json", {}).then(res => {
          const mapJson = res.data;
          chinaJson = mapJson;
          myChart = echarts.init(document.getElementById(container_id));
          registerAndsetOption(myChart, chinaId, chinaName, mapJson, false);
          parentId = chinaId;
          parentName = "china";
          myChart.on("click", function (param) {
            var cityId = cityMap[param.name];
            if(cityId){
              //有下级地图
              axios.get("./map/" + cityId + ".json", {}).then(res => {
                const mapJson = res.data;
                console.log(mapJson)
                registerAndsetOption(
                  myChart,
                  cityId,
                  param.name,
                  mapJson,
                  true
                )
              });
            } else {
              //没有下级地图，回到一级地图，并将mapStack清空
              registerAndsetOption(myChart, chinaId, chinaName, chinaJson, false);
              mapStack = [];
              parentId = chinaId;
              parentName = chinaName;
            }
          })
          // for (let i=0; i<mapJson.features.length; i++) {
          //   console.log
          // }
        })
      },

    }
  }

  function registerAndsetOption(myChart, id, name, mapJson, flag) {
    echarts.registerMap(name, mapJson);
    myChart.setOption({
      series: [
        {
          type: "map",
          map:name,
          itemStyle: {
            areaColor: "rgba(23, 27, 57,0)",
            borderColor: "#1dc199",
            borderWidth: .8
          },
          data: initMapData(mapJson)
        }
      ]
    });

    if(flag){
      mapStack.push({
        mapId: parentId,
        mapName: parentName
      });
      parentId = id;
      parentName = name;
    }
  }

  function initMapData(mapJson) {
    var mapData = [];
    for (var i=0; i<mapJson.features.length;i++){
      mapData.push({
        name:mapJson.features[i].properties.name
      })
    }
    return mapData
  }
</script>

<style scoped>

  #map{
    width: 100%;
    height: 600px;
  }

</style>
