<template>
  <div ref="chart" style="width: 100%; height: 400px"></div>
</template>
  
  <script>
import * as echarts from 'echarts'
export default {
  props: {
    data: {
      type: Array,
      required: true,
    },
    legendData: {
      type: Array,
      default: () => [],
    },
  },
  mounted() {
    this.chart = echarts.init(this.$refs.chart)
    this.drawChart()
  },
  methods: {
    drawChart() {
      this.chart.setOption({
        series: [
          {
            type: 'pie',
            radius: '65%',
            center: ['50%', '50%'],
            data: this.data,
            label: {
              show: true,
              position: 'outside',
              color: '#7F8FA4',
              fontSize: 12,
            },

            legend: {
              top: '5%',
              left: 'center',
            //   data: this.legendData,
            },

            labelLine: {
              show: true,
              length: 50,
              lineStyle: {
                width: 2,
                type: 'solid',
              },
            },
            
          },
          // 画内部百分比的饼图
            {
              type: 'pie',
              data: this.data,

              label: {
                show: true,
                position: 'inside',
                formatter: `{d}%`,
              },
            },

          //     {
          //       type: 'pie',
          //       data: this.data,
          //       radius: "65%",
          //   center: ["50%", "50%"],
          //   label: {
          //     show: true,
          //     position: "inside",
          //     formatter: `{d}%`,
          //   },

          //   labelLine: {
          //     show: true,
          //   },

          //     },
        ],
      })
    },
  },
  watch: {
    data() {
      this.drawChart()
    },
  },
  beforeDestroy() {
    if (this.chart) {
      this.chart.dispose()
      this.chart = null
    }
  },
}
</script>