<template>
  <div class='com-container'>
    <div class='com-chart' ref='hot_ref'></div>
    <span class="iconfont arr-left" @click="toLeft" :style="comStyle">&#xe6ef;</span>
    <span class="iconfont arr-right" @click="toRight" :style="comStyle">&#xe6ed;</span>
    <span class="cat-name" :style="comStyle">{{ catName }}</span>
  </div>
</template>

<script>
export default {
  name: "Hot",
  data () {
    return {
      chartInstance: null,
      allData: null,
      currentIndex: 0, // 当前所展示出的一级分类数据
      titleFontSize: 0
    }
  },
  computed: {
    catName () {
      if (!this.allData) {
        return ''
      } else {
        return this.allData[this.currentIndex].name
      }
    },
    comStyle () {
      return {
        fontSize: this.titleFontSize + 'px'
      }
    }
  },
  mounted() {
    this.initChart()
    this.getData()
    window.addEventListener('resize', this.screenAdapter)
    this.screenAdapter()
  },
  destroyed() {
    window.removeEventListener('resize', this.screenAdapter)
  },
  methods:{
    initChart(){
      this.chartInstance = this.$echarts.init(this.$refs.hot_ref, 'chalk')
      const initOption={
        title: {
          text: '▎ 热销商品的占比',
          left: 20,
          top: 20
        },
        legend: {
          top: '15%',
          icon: 'circle'
        },
        tooltip:{
          // 使用箭头函数，this指向VM
          formatter: (arg)=>{
            // console.log("this:",this)
            const thirdCategory = arg.data.children
            // 三级中的百分比需要自己手动的计算
            let total=0
            thirdCategory.forEach(item=>{
              total+=item.value
            })
            let str=""
            thirdCategory.forEach(item=>{
              str+=
                  `
                ${item.name}:${parseInt(item.value / total*100)+"%"}
                <br/>
                  `
            })
            return str

          }},
        //箭头函数 为了适配this的问题
        series: [
          {
            type:"pie",
            // 取消原本有的附近展示信息
            label:{
              show:false
            },
            // 高亮显示
            emphasis:{
              label:{
                show:true
              },
              labelLine:{
                show:false
              }
            }
          }
        ]
      }
      this.chartInstance.setOption(initOption)
    },
    async getData(){
      const { data: ret } = await this.$http.get('hotproduct')
      this.allData = ret
      // console.log(this.allData)
      this.updateChart()
    },
    // 更新图表
    updateChart () {
      // 处理图表需要的数据
      const legendData = this.allData[this.currentIndex].children.map(item => {
        return item.name
      })
      // 展示的数据信息
      const seriesData = this.allData[this.currentIndex].children.map(item => {
        return {
          name: item.name,
          value: item.value,
          children: item.children // 新增加children的原因是为了在tooltip中的formatter的回调函数中,来拿到这个二级分类下的三级分类数据
        }
      })
      const dataOption = {
        legend: {
          data: legendData
        },
        series: [
          {
            data: seriesData
          }
        ]
      }
      this.chartInstance.setOption(dataOption)
    },
    // 屏幕适配
    screenAdapter () {
      this.titleFontSize = this.$refs.hot_ref.offsetWidth / 100 * 3.6
      const adapterOption = {
        title: {
          textStyle: {
            fontSize: this.titleFontSize
          }
        },
        legend: {
          itemWidth: this.titleFontSize ,
          itemHeight: this.titleFontSize,
          itemGap: this.titleFontSize / 2,
          textStyle: {
            fontSize: this.titleFontSize / 2
          }
        },
        series: [
          {
            radius: this.titleFontSize * 4.5,
            center: ['50%', '60%']
          }
        ]
      }
      this.chartInstance.setOption(adapterOption)
      this.chartInstance.resize()
    },
    // 移动图标-->向左移动和向右移动
    toLeft () {
      this.currentIndex--
      if (this.currentIndex < 0) {
        this.currentIndex = this.allData.length - 1
      }
      this.updateChart()
    },
    toRight () {
      this.currentIndex++
      if (this.currentIndex > this.allData.length - 1) {
        this.currentIndex = 0
      }
      this.updateChart()
    }
  },
}
</script>



<style lang='less' scoped>
.arr-left {
  position:absolute;
  left: 10%;
  top: 50%;
  transform: translateY(-50%);
  cursor: pointer;
  color: white;
}
.arr-right {
  position:absolute;
  right: 10%;
  top: 50%;
  transform: translateY(-50%);
  cursor: pointer;
  color: white;
}
.cat-name {
  position:absolute;
  left: 80%;
  bottom: 20px;
  color: white;
}
</style>