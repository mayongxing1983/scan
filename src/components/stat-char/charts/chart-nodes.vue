// template define
<template>
    <div class='grid-content bg-purple'>
      <div id='statChartsNodes' class="statChartsClass" :style="{ 'min-width': '300px', width: '835px', height: '500px'}"></div>
    </div>
</template>

// script define
<script>
import { mapActions } from 'vuex'
import _ from 'lodash'
import { formatNumber } from '../../../untils/format'
import echarts from 'echarts'
export default {
  mounted () {
    this.getNodesByShardChart()
    this.drawStatChartsNodes()
  },
  computed: {
    statData: {
      get () {
        return this.$store.state.chart.nodesChart
      }
    }
  },
  watch: {
    statData: {
      handler: function (val, oldval) {
        this.drawStatChartsNodes()
      }
    }
  },
  filters: {
    numberFilter (value) {
      return formatNumber(value)
    }
  },
  methods: {
    ...mapActions(['getNodesByShardChart']),
    drawStatChartsNodes () {
      // charts locate
      var statChartsElement = document.getElementById('statChartsNodes')
      // charts instantiatte
      var statCharts = echarts.init(statChartsElement)
      // clean cache
      statCharts.clear()
      if (_.isUndefined(this.statData.code)) {
        statCharts.showLoading({
          text: this.$t('statcharts.common.loading')
        })
      } else {
        statCharts.hideLoading()
      }
      // method define:
      //    getting data
      //    tip display
      // define  charts option
      var option = {
        toolbox: {
          padding: [0, 25, 0, 0],
          show: true,
          feature: {
            mark: {show: true},
            dataView: {show: true, readOnly: false},
            magicType: {
              show: true,
              type: ['pie', 'funnel']
            },
            restore: {show: true},
            saveAsImage: {show: true}
          }
        },
        tooltip: {
          trigger: 'item',
          formatter: '{b} ({d}%) <br/> ' + this.$t('statcharts.node.nodesByShardTipname') + ': {c} '
        },
        legend: {
          type: 'scroll',
          orient: 'vertical',
          x: 'left',
          right: 10,
          top: 20,
          bottom: 20,
          data: this.statData.legendData,
          selected: this.statData.selected
        },
        series: [
          {
            name: this.$t('statcharts.node.nodesByShard'),
            type: 'pie',
            smooth: true,
            symbol: 'circle',
            symbolSize: 5,
            sampling: 'average',
            radius: '40%',
            center: ['50%', '50%'],
            itemStyle: {
              emphasis: {
                shadowBlur: 10,
                shadowOffsetX: 0,
                shadowColor: 'rgba(0, 0, 0, 0.5)'
              }
            },
            data: this.statData.seriesData
          }
        ]
      }
      // clean cache
      statCharts.clear()
      // setting charts option
      statCharts.setOption(option, true)
    }
  }
}
</script>
<style>
  @import "../../../assets/css/chars.less";
</style>
