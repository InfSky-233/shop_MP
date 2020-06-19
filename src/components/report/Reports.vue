<template>
  <div>
    <!-- 面包屑导航区域 -->
    <el-breadcrumb separator="/">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>角色列表</el-breadcrumb-item>
    </el-breadcrumb>

    <!-- 卡片视图区域 -->
    <el-card>
      <!-- 2. 为ECharts准备一个具备大小（宽高）的Dom -->
      <div id="main" style="width: 800px;height:400px;"></div>
    </el-card>
  </div>
</template>

<script>
// 1. 导入 echarts
import echarts from 'echarts'
import _ from 'lodash'
export default {
  data() {
    return {
      // 需要合并的数据
      options: {
        title: {
          text: '用户来源'
        },
        tooltip: {
          trigger: 'axis',
          axisPointer: {
            type: 'cross',
            label: {
              backgroundColor: '#E9EEF3'
            }
          }
        },
        grid: {
          left: '3%',
          right: '4%',
          bottom: '3%',
          containLabel: true
        },
        xAxis: [
          {
            boundaryGap: false
          }
        ],
        yAxis: [
          {
            type: 'value'
          }
        ]
      }
    }
  },
  created() {},
  mounted() {
    // 3. 基于准备好的DOM，初始化echarts实列
    var myChart = echarts.init(document.getElementById('main'))

    this.$http
      .get('reports/type/1')
      .then(res => {
        // 4. 准备图表的配置项和数据
        const result = _.merge(res.data.data, this.options)
        // 5. 使用刚指定的配置项和数据显示图表
        myChart.setOption(result)
      })
      .catch(() => {
        this.$message.error('获取数据失败！')
      })
  },
  methods: {}
}
</script>

<style lang="less" scoped>
</style>
