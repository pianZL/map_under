<template>
<div>
  <div id="echarts"></div>
</div>
</template>
<script>
export default {
  name: "Echarts",
  data() {
    return {
      mapName: '全国',
      mapData: [],
      oldNameList: [],
      level: 0
    };
  },
  methods: {
    // 下一级
    next(param) {
      const self = this
      const index = self.oldNameList.findIndex(e=>{
        return e.name === param.name
      })
      if(index === -1 && self.mapName !== param.name) {
        const oldMapInfo = {name:param.name,parent:self.mapName,level:self.level}
        self.mapName = param.name
        try{
          self.init()
          self.oldNameList.push(oldMapInfo)
          self.oldMapName = param.name
          self.level += 1
          console.log("next",self.level)
        }catch (e) {
          self.mapName = self.oldMapName
          self.$message.error("目前还未有该城市信息，请等候数据上传")
        }
      } else {
        this.$message.error("已经是最底层了")
      }
    },
    // 上一级
    pre() {
      const self = this
      self.level -= 1
      console.log("pre",self.level)
      const index = self.oldNameList.findIndex(e=>{
        return e.level === self.level
      })
      // console.log(index)
      if(index > -1) {
        self.mapName = self.oldNameList[index].parent
        console.log(self.mapName)
        self.init()
        self.oldNameList.splice(index,1)
      } else {
        self.oldNameList = []
        self.level = 0
        self.$message.error("已经是最顶层了")
      }
    },
    init() {
      const self = this;
      let data;
        data = require("../../assets/json/" + self.mapName + ".json");
        self.mapData = data;
        self.$echarts.registerMap(self.mapName, self.mapData);
        let map = self.$echarts.init(document.getElementById("echarts"));
        const option = {
          tooltip: {
            trigger: "item",
            showDelay: 0,
            transitionDuration: 0.2,
            formatter: function(params) {
              return (
                params.seriesName + "<br/>" + params.name + ": " + params.value
              );
            }
          },
          // 视觉映射组件
          visualMap: {
            left: "right",
            min: 100,
            max: 3800,
            inRange: {
              color: [
                "#313695",
                "#4575b4",
                "#74add1",
                "#abd9e9",
                "#e0f3f8",
                "#ffffbf",
                "#fee090",
                "#fdae61",
                "#f46d43",
                "#d73027",
                "#a50026"
              ]
            },
            text: ["High", "Low"], // 文本，默认为数值文本
            calculable: true
          },
          // 工具箱
          toolbox: {
            show: true,
            left: "left",
            top: "top",
            feature: {
              dataView: { readOnly: false }, // readOnly为true的时候 不显示刷新 就是唯一的数据，不能修改 ，false 则带有刷新
              // restore: {}, // 还原数据
              saveAsImage: {} // 将地图下载为图片
            }
          },
          layoutCenter: ['50%','57%'],
          layoutSize: 1000,
          // 数据
          series: [
            {
              name: "1",
              type: "map",
              map: self.mapName,
              zoom: 0.6,
              label: {
                show: true
              },
              mapLocation: {
                y: 30,
                x: 60
              },
              itemSytle: {
                normal: {
                  color: "#fecb2e",
                  shadowBlur: 12,
                  shadowColor: "#0d5ebd"
                }
              },
              coordinateSystem: "geo",
              data: [
                { name: "锦江区", value: "2333" },
                { name: "双流区", value: "2666" },
                { name: "青羊区", value: "1333" }
              ]
            }
          ]
        };
        map.setOption(option);
        map.off("click");
        map.off("contextmenu");
        // 打开地图点击事件
        map.on("click", function(params) {
          self.next(params)
        });
        // 打开鼠标右键事件
        map.on("contextmenu", function(params) {
          self.pre(params)
        });
    }
  },
  mounted() {
    this.init()
  }
};
</script>
<style lang="less" scoped>
  #echarts{
    width: 100%;
    height: 740px;
    margin-left: auto;
    margin-right: auto;
    background-color: aqua;
    float: right;
  }
</style>