<template>
  <div class="echarts-bar" ref="echartsBar"></div>
</template>
  
  <script>
import * as echarts from 'echarts'
export default {
  name: 'EchartsBar',
  props: {
    legendData: {
      type: Array,
      default: () => [],
    },
    xAxisData: {
      type: Array,
      default: () => [],
    },
    seriesData: {
      type: Array,
      default: () => [],
    },
  },
  mounted() {
    this.initEcharts()
  },
  methods: {
    initEcharts() {
      let that = this
      const chart = echarts.init(this.$refs.echartsBar)

      let option = {
        //     title : {
        //     text: '送检按时完成率',
        //     subtext: '数据来自xxxx有限公司',
        //     x: 'center',
        //     align: 'right'
        //   },//标题

        tooltip: {
          trigger: 'axis',
          axisPointer: {
            type: 'shadow',
          }, //鼠标移上的阴影，默认是线
          formatter: this.legendData,
        },
        legend: {
          data: this.legendData,
        },
        xAxis: {
          type: 'category',
          data: this.xAxisData,
        },
        yAxis: {
          type: 'value',
          axisLabel: {
            interval: 0, // 使x轴文字显示全
            show: true,
            color: '#666666',
            formatter: '{value}%', //y轴数值，带百分号
          },
          min: 0, //最小百分比
          max: 100, //最大百分比
        },

        label: {
          show: true,
          position: 'top', //数据在中间显示
          formatter: '{c}%', //百分比显示
        },
        series: [
          {
            data: this.seriesData,
            type: 'bar',
            barMaxWidth: 100,
            itemStyle: {
              normal: {
                color: function (params) {
                  // 给出颜色组
                  var colorList = ['#cca272', '#74608f', '#d7a02b', '#c8ba23']
                  //循环调用
                  return colorList[params.dataIndex % colorList.length]
                },
              },
            },
          },
        ],

        // series: this.seriesData,
      }
      // chart.setOption()
      option && chart.setOption(option, true)
    },
  },
}
</script>
  
  <style scoped>
.echarts-bar {
  width: 100%;
  height: 400px;
}
</style>