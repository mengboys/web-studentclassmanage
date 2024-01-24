<template>
  <div class="dashboard-container">
    <div class="select">

      <el-select v-model="selectedSemester" filterable placeholder="请选择">
        <el-option
          v-for="item in semesters"
          :key="item.id"
          :label="item.itemName"
          :value="item.id"
        >
        </el-option>
      </el-select>
      <el-select v-model="selectedWeek" filterable style="margin-left: 20px;" placeholder="请选择周数">
        <el-option
          v-for="week in weekOptions"
          :key="week"
          :label="`第 ${week} 周`"
          :value="week"
        ></el-option>
      </el-select>
      <el-select v-model="selectedLabs" :formatter="formatRole" filterable style="margin-left: 20px;"
                 placeholder="请选择实验室"
      >
        <el-option
          v-for="labs in labsTable"
          :key="labs.id"
          :label="labs.roomName"
          :value="labs.id"
        ></el-option>
      </el-select>
    </div>
    <div class="table">
      <el-table
        ref="singleTable"
        :data="tableData"
        highlight-current-row
        style="width: 100%"
        border
        :cell-style="tableCellStyle"
      >
        <el-table-column
          type="index"
          :index="indexMethod"
          width="150"
          align="center"
        >
        </el-table-column>
        <el-table-column
          v-for="(day, dayIndex) in daysOfWeek"
          :key="dayIndex"
          :property="day.property"
          :label="day.label"
          width="120px"
          align="center"
        >
          <template v-slot="{ row, column }">
            <div v-for="(experiment, index) in experimentTable" :key="index">
              <div v-if="experiment.classNumber === tableData.indexOf(row) && experiment.classDay === dayIndex">
                {{ experiment.expName }}
              </div>
            </div>
          </template>
        </el-table-column>
      </el-table>
    </div>
    <el-select v-model="selectedLabs2" :formatter="formatRole" filterable style="margin-left: 20px;"
               placeholder="请选择实验室"
    >
      <el-option
        v-for="labs in labsTable"
        :key="labs.id"
        :label="labs.roomName"
        :value="labs.id"
      ></el-option>
    </el-select>
    <div style="margin: 30px">{{ description }}</div>
  </div>
</template>

<script>
import { mapGetters } from 'vuex'
import axios from 'axios'
import { getToken } from '@/utils/auth'

export default {
  name: 'Dashboard',
  data() {
    return {
      semesters: [],
      selectedSemester: '',
      selectedWeek: '',
      selectedLabs: '',
      selectedLabs2: '',
      weekOptions: [],
      labsTable: [],
      tableData: [],
      daysOfWeek: [
        { property: 'name', label: '星期一' },
        { property: 'name', label: '星期二' },
        { property: 'name', label: '星期三' },
        { property: 'name', label: '星期四' },
        { property: 'name', label: '星期五' },
        { property: 'name', label: '星期六' },
        { property: 'name', label: '星期日' }
      ],
      experimentTable: [],
      description: ''
    }
  },
  created: function() {
    // 假设你的 token 存储在变量 token 中
    var token = getToken()
    // 使用 Axios 的配置项，在请求中添加包含 token 的 header
    axios.get('http://localhost:8081/item/getitems', {
      headers: {
        'X-token': token // 添加 Authorization 头部，格式为 Bearer Token
      }
    })
      .then((response) => {
        this.semesters = response.data
        console.log(this.semesters)
        // 找到 isSelected 字段为 1 的学期并设置为默认选中
        const defaultSemester = this.semesters.find(semester => semester.isSelected === 1)
        if (defaultSemester) {
          this.selectedSemester = defaultSemester.id

          // 设置默认周数为第一周
          const totalWeeks = defaultSemester.totalWeeks
          this.weekOptions = Array.from({ length: totalWeeks }, (_, index) => index + 1)
          this.selectedWeek = 1

          // 设置默认实验为第一实验
          this.selectedLabs = 21
        }
      })
      .catch(function(error) {
        console.error(error)
      })
    axios.get('http://localhost:8081/labs/getlabs', {
      headers: {
        'X-token': token // 添加 Authorization 头部，格式为 Bearer Token
      }
    })
      .then((response) => {
        this.labsTable = response.data
      })
      .catch(function(error) {
        console.error(error)
      })
    axios.put(`http://localhost:8081/experiment/getallexperiment`, null, {
      headers: {
        'X-token': token // 添加 Authorization 头部，格式为 Bearer Token
      }
    })
      .then(response => {
          // 成功保存后更新前端表格数据
          this.experimentTable = response.data
          console.log(response.data)
          this.deleteDialogVisible = false
          this.$message.success('获取成功')
        }
      )
      .catch(error => {
        console.error('发生了一些错误', error.message)
      })
  },
  methods: {
    handleLabSelect() {
      // 当实验室选择变化时触发
      const selectedLab = this.labsTable.find(lab => lab.id === this.selectedLabs2)
      if (selectedLab) {
        this.description = selectedLab.description // 设置实验室简介
      }
    },
    updateWeekOptions() {
      // 根据选中的学期动态生成周数选项
      const selectedSemester = this.semesters.find(semester => semester.id === this.selectedSemester)
      if (selectedSemester) {
        const totalWeeks = selectedSemester.totalWeeks
        this.weekOptions = Array.from({ length: totalWeeks }, (_, index) => index + 1)
        this.selectedWeek = 1 // 重置选中的周数

        this.tableData = JSON.parse(selectedSemester.classConfig)
      }
    },
    indexMethod(index) {
      return this.tableData[index].start + '-' + this.tableData[index].end
    },
    formatRole(row) {
      for (let i = 0; i < this.labsTable.length; i++) {
        if (row.classRoom === this.labsTable[i].id) {
          return this.labsTable[i].roomName
        }
      }
    }
  },
  watch: {
    selectedSemester(newVal) {
      // 当 value1 变化时触发更新第二个选项的数据
      this.updateWeekOptions()
    },
    selectedLabs2: {
      immediate: true,
      handler() {
        this.handleLabSelect()
      }
    }
  }
}
</script>

<style lang="scss" scoped>
.dashboard {
  &-text {
    font-size: 30px;
    line-height: 46px;
  }
}

.select {
  margin: 30px;
}

.table {
  margin: 30px;
}
</style>
