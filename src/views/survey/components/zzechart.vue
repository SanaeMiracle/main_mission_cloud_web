<template>
	<div class="bar" id="bar" />
</template>

<script>
// 自行引入echarts
import * as echarts from 'echarts'

export default {
  name: 'Bar',
  // 接收从父组件传回的值
  props: ['getData'],
  data () {
    return {}
  },
  // 实时监听父组件传过来的值，进而执行drawBar方法，重绘柱状图
  watch: {
    getData: {
      handler (value) {
        this.drawBar(value)
      },
      deep: true
    }
  },
  mounted () {
    this.drawBar()
  },
  methods: {
    drawBar ({
      textTitle = '',
      nameTitle = '',
      xName = '',
      yName = '',
      nameArray = [],
      dataArray = [],
      colorArray = [],
      barWidth = '30%',
      showTopVal = false,
      xAxis = [
        {
          name: yName,
          type: 'category',
          data: nameArray,
          nameLocation: 'end',
          axisTick: {
            show: false
          },
          splitLine: {
            show: false // 去除网格线
          },
          axisLabel: {
            interval: 1,
            formatter: function (value) {
              let str = ''
              let num = 7 // 每行显示字数
              let valLength = value.length // 该项x轴字数
              let rowNum = Math.ceil(valLength / num) // 行数

              if (rowNum > 1) {
                for (let i = 0; i < rowNum; i++) {
                  let temp = ''
                  let start = i * num
                  let end = start + num

                  temp = value.substring(start, end) + '\n'
                  str += temp
                }
                return str
              } else {
                return value
              }
            }
          },
          nameTextStyle: {
            padding: [30, 0, 0, -30]
          }
        }
      ],
      dataZoom = [],
      grid = {
        left: 20,
        right: 20,
        bottom: 20,
        top: 80,
        containLabel: true
      }
    } = {}) {
      let barBox = echarts.init(document.getElementById('bar'))
      let option = {
        title: {
          text: textTitle,
          left: 'center',
          top: 10,
          textStyle: {
            color: '#333',
            fontSize: '18',
            fontWeight: 'bolder'
          }
        },
        grid: grid,
        tooltip: {
          trigger: 'axis',
          axisPointer: {
            type: 'shadow'
          }
        },
        dataZoom: dataZoom,
        xAxis: xAxis,
        yAxis: [
          {
            axisLine: { show: false },
            name: xName,
            type: 'value'
            // axisLabel: {
            //   formatter: value => (value + '元')
            // }
          }
        ],
        series: [
          {
            name: nameTitle,
            type: 'bar',
            barWidth: barWidth,
            data: dataArray,
            label: {
              show: showTopVal, // 开启显示
              position: 'top', // 在上方显示
              textStyle: { // 数值样式
                color: '#333333',
                fontSize: 12
              }
            },
            itemStyle: {
              normal: {
                color: function (params) {
                  const colorList = colorArray
                  return colorArray.length > 0 ? colorList[params.dataIndex] : '#EA8187'
                }
              }
            }
          }
        ]
      }
      barBox.setOption(option, true)
    }
  }
}
</script>

