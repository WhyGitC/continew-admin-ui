<template>
  <a-spin :loading="loading" style="width: 100%">
    <a-card class="general-card" title="访问趋势">
      <template #extra>
        <a-radio-group v-model:model-value="dateRange" type="button" size="small" @change="onChange as any">
          <a-radio :value="7">近7天</a-radio>
          <a-radio :value="30">近30天</a-radio>
        </a-radio-group>
      </template>
      <Chart :option="chartOption" style="height: 460px" />
    </a-card>
  </a-spin>
</template>

<script lang="ts" setup>
import { type EChartsOption, graphic } from 'echarts'
import { type DashboardAccessTrendResp, getDashboardAccessTrend as getData } from '@/apis'
import { useChart } from '@/hooks'

// 提示框
const tooltipItemsHtmlString = (items) => {
  return items
    .map(
      (el) => `<div class="content-panel">
        <p>
          <span style="background-color: ${el.color}" class="tooltip-item-icon"></span>
          <span>${el.seriesName}</span>
        </p>
        <span class="tooltip-value">
        ${el.value}
        </span>
      </div>`
    )
    .join('')
}

const xAxis = ref<string[]>([])
const pvChartData = ref<number[]>([])
const ipChartData = ref<number[]>([])
const { chartOption } = useChart((isDark: EChartsOption) => {
  return {
    grid: {
      left: '38',
      right: '5',
      top: '10',
      bottom: '50'
    },
    legend: {
      bottom: -3,
      icon: 'circle',
      textStyle: {
        color: '#4E5969'
      }
    },
    xAxis: {
      type: 'category',
      offset: 2,
      data: xAxis.value,
      boundaryGap: false,
      axisLabel: {
        color: '#4E5969',
        formatter(value: number, idx: number) {
          if (idx === 0) return ''
          if (idx === xAxis.value.length - 1) return ''
          return `${value}`
        }
      },
      axisLine: {
        show: false
      },
      axisTick: {
        show: false
      },
      splitLine: {
        show: true,
        interval: (idx: number) => {
          if (idx === 0) return false
          return idx !== xAxis.value.length - 1
        },
        lineStyle: {
          color: isDark ? '#3F3F3F' : '#E5E8EF'
        }
      },
      axisPointer: {
        show: true,
        lineStyle: {
          color: '#23ADFF',
          width: 2
        }
      }
    },
    yAxis: {
      type: 'value',
      axisLabel: {
        formatter(value: any, idx: number) {
          if (idx === 0) return value
          if (value >= 1000) {
            return `${value / 1000}k`
          }
          return `${value}`
        }
      },
      axisLine: {
        show: false
      },
      splitLine: {
        lineStyle: {
          type: 'dashed',
          color: isDark ? '#3F3F3F' : '#E5E8EF'
        }
      }
    },
    tooltip: {
      show: true,
      trigger: 'axis',
      formatter(params) {
        const [firstElement] = params
        return `<div>
            <p class="tooltip-title">${firstElement.axisValueLabel}</p>
            ${tooltipItemsHtmlString(params)}
          </div>`
      },
      className: 'echarts-tooltip-diy'
    },
    series: [
      {
        name: '访问次数',
        data: pvChartData.value,
        type: 'line',
        smooth: true,
        showSymbol: false,
        color: '#246EFF',
        symbol: 'circle',
        symbolSize: 10,
        emphasis: {
          focus: 'series',
          itemStyle: {
            borderWidth: 2,
            borderColor: '#E0E3FF'
          }
        },
        areaStyle: {
          opacity: 0.8,
          color: new graphic.LinearGradient(0, 0, 0, 1, [
            {
              offset: 0,
              color: 'rgba(17, 126, 255, 0.16)'
            },
            {
              offset: 1,
              color: 'rgba(17, 128, 255, 0)'
            }
          ])
        }
      },
      {
        name: '独立IP',
        data: ipChartData.value,
        type: 'line',
        smooth: true,
        showSymbol: false,
        color: '#00B2FF',
        symbol: 'circle',
        symbolSize: 10,
        emphasis: {
          focus: 'series',
          itemStyle: {
            borderWidth: 2,
            borderColor: '#E2F2FF'
          }
        },
        areaStyle: {
          opacity: 0.8,
          color: new graphic.LinearGradient(0, 0, 0, 1, [
            {
              offset: 0,
              color: 'rgba(17, 126, 255, 0.16)'
            },
            {
              offset: 1,
              color: 'rgba(17, 128, 255, 0)'
            }
          ])
        }
      }
    ]
  }
})

const loading = ref(false)
const dateRange = ref(30)
// 查询图表数据
const getChartData = async (days: number) => {
  try {
    loading.value = true
    xAxis.value = []
    pvChartData.value = []
    ipChartData.value = []
    const { data: chartData } = await getData(days)
    chartData.forEach((item: DashboardAccessTrendResp) => {
      xAxis.value.push(item.date)
      pvChartData.value.push(item.pvCount)
      ipChartData.value.push(item.ipCount)
    })
  } finally {
    loading.value = false
  }
}

// 切换
const onChange = (days: number) => {
  getChartData(days)
}

onMounted(() => {
  getChartData(30)
})
</script>
