// template define
<template>
    <div class='grid-content bg-purple'>
      <div id='statChartsTxHistory' class="statChartsClass" :style="{ 'min-width': '300px', width: '835px', height: '500px'}"></div>
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
    this.getTxChart(this.shardCharValue)
    this.drawStatChartsTxHistory()
  },
  computed: {
    txChart: {
      get () {
        return this.$store.state.chart.txChart
      }
    },
    shardCharValue: {
      get () {
        return this.$store.state.shardChar.shardCharValue
      }
    }
  },
  watch: {
    txChart: {
      handler: function (val, oldval) {
        this.drawStatChartsTxHistory()
      }
    },
    shardCharValue: {
      handler: function (val, oldval) {
        this.getTxChart(this.shardCharValue)
        this.drawStatChartsTxHistory()
      }
    }
  },
  filters: {
    numberFilter (value) {
      return formatNumber(value)
    }
  },
  methods: {
    ...mapActions(['getTxChart']),
    drawStatChartsTxHistory () {
      var self = this
      // charts locate
      var statChartsElement = document.getElementById('statChartsTxHistory')
      // charts instantiatte
      var statCharts = echarts.init(statChartsElement)
      // clean cache
      statCharts.clear()
      if (_.isUndefined(this.txChart.code)) {
        statCharts.showLoading({
          text: this.$t('statcharts.common.loading')
        })
      } else {
        statCharts.hideLoading()
      }
      // method define:
      //    getting data
      var statChartsDataX = function () {
        return self.txChart.x
      }
      var statChartsDataY = function () {
        return self.txChart.y
      }
      var statChartsTooltipData = function (index) {
        return self.txChart.tooltip[index]
      }
      // define  charts option
      var option = {
        title: {
          text: '(' + (this.shardCharValue === '0' ? this.$t('statcharts.common.shardAll') : (this.$t('statcharts.common.shardTag') + this.shardCharValue)) + ')',
          x: 'left'
        },
        toolbox: {
          padding: [0, 25, 0, 0],
          feature: {
            dataZoom: {
              yAxisIndex: 'none'
            },
            restore: {},
            magicType: {
              type: ['line', 'bar']
            },
            saveAsImage: {}
          }
        },
        tooltip: {
          trigger: 'axis',
          formatter: function (datas) {
            var dataindex = datas[0].dataIndex
            var toopTipData = statChartsTooltipData(dataindex)

            var newDate = new Date()
            newDate.setTime(toopTipData.TimeStamp * 1000)
            var res = newDate.toDateString() + '<br/>'
            res += self.$t('statcharts.transaction.historyTipTotal') + ': ' + toopTipData.TotalTxs + '<br/>'
            res += self.$t('statcharts.transaction.historyTipAvgDifficulty') + ': ' + toopTipData.AvgDifficulty + '<br/>'
            res += self.$t('statcharts.transaction.historyTipHashRate') + ': ' + toopTipData.HashRate + '<br/>'
            res += self.$t('statcharts.transaction.historyTipAvgBlockTime') + ': ' + toopTipData.AvgBlockTime + '<br/>'
            res += self.$t('statcharts.transaction.historyTipTotalBlockCount') + ': ' + toopTipData.TotalBlocks + '<br/>'
            res += self.$t('statcharts.transaction.historyTipNewAddressSeen') + ': ' + toopTipData.NewAddress + '<br/>'
            return res
          },
          axisPointer: {
            type: 'cross'
          }
        },
        xAxis: {
          name: this.$t('statcharts.transaction.historyTxXname'),
          nameLocation: 'end',
          nameTextStyle: {

          },
          type: 'category',
          boundaryGap: false,
          data: statChartsDataX()
        },
        yAxis: {
          name: this.$t('statcharts.transaction.historyTxYname'),
          nameLocation: 'middle',
          nameTextStyle: {

          },
          nameGap: 50,
          type: 'value',
          boundaryGap: [0, '80%']
        },
        dataZoom: [{
          type: 'inside',
          start: 0,
          end: 100
        }, {
          start: 0,
          end: 100,
          handleIcon: 'M10.7,11.9v-1.3H9.3v1.3c-4.9,0.3-8.8,4.4-8.8,9.4c0,5,3.9,9.1,8.8,9.4v1.3h1.3v-1.3c4.9-0.3,8.8-4.4,8.8-9.4C19.5,16.3,15.6,12.2,10.7,11.9z M13.3,24.4H6.7V23h6.6V24.4z M13.3,19.6H6.7v-1.4h6.6V19.6z',
          handleSize: '80%',
          handleStyle: {
            color: '#fff',
            shadowBlur: 3,
            shadowColor: 'rgba(0, 0, 0, 0.6)',
            shadowOffsetX: 2,
            shadowOffsetY: 2
          }
        }],
        series: [
          {
            name: this.$t('statcharts.transaction.historyTxName'),
            type: 'line',
            smooth: true,
            symbol: 'circle',
            symbolSize: 5,
            sampling: 'average',
            itemStyle: {
              normal: {
                color: 'rgb(255, 70, 131)'
              }
            },
            lineStyle: {
              color: 'red'
            },
            data: statChartsDataY()
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
