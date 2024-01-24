<template>
  <div class="manageItem" v-if="Role===2">
    <el-button type="primary" @click="openaddDialog()" style="margin-top: 20px;margin-left: 30px">新建</el-button>
    <div class="itemtable">
      <el-table :data="itemTable.filter(data => !search || data.itemName.toLowerCase().includes(search.toLowerCase()))"
                border
                style="width: 100%;margin-top:30px;"
      >
        <el-table-column align="center" label="学期名称" prop="itemName"></el-table-column>
        <el-table-column align="center" label="学期开始日期" prop="startDate"></el-table-column>
        <el-table-column align="center" label="总周数" prop="totalWeeks"></el-table-column>
        <el-table-column align="center" label="一日课时" prop="classPerDay"></el-table-column>
        <el-table-column align="center" label="当前学期" prop="isSelected" :formatter="formatRole2"></el-table-column>
        <el-table-column align="center" width="300px">
          <template slot="header" slot-scope="scope">
            <el-input v-model="search" size="mini" placeholder="输入关键字搜索"/>
          </template>
          <template slot-scope="scope" >
            <el-button size="mini" @click="openEditDialog(scope.$index, scope.row)">编辑</el-button>
            <el-button size="mini" @click="openSelected(scope.$index, scope.row)">设为当前学期</el-button>
            <el-button size="mini" type="danger" @click="openleDelete(scope.$index, scope.row)">删除</el-button>
          </template>
        </el-table-column>
      </el-table>
    </div>

    <div class="dialog">
      <el-dialog title="编辑学期信息" :visible.sync="editDialogVisible">
        <el-form :model="editItemForm" label-width="100px">
          <el-form-item label="学期名称" required>
            <el-input v-model="editItemForm.itemName"/>
          </el-form-item>
          <el-form-item label="学校开始日期" prop="startDate" required>
            <el-date-picker v-model="editItemForm.startDate" type="date" placeholder="选择日期" format="yyyy-MM-dd"/>
          </el-form-item>
          <el-form-item label="总周数" required>
            <el-input v-model.number="editItemForm.totalWeeks"/>
          </el-form-item>
          <el-form-item label="一日课时" required>
            <el-input v-model.number="editItemForm.classPerDay"/>
          </el-form-item>
          <el-form-item v-for="index in editItemForm.classPerDay" :key="index" label="课程时间" required>
            <!-- 时间选择器 -->
            <el-time-select
              v-model="editItemForm.classConfig[index-1].start"
              :picker-options="{  start: '08:00',step: '00:10',end: '21:00' }"
              placeholder="开始时间"
            ></el-time-select>
            至
            <el-time-select
              v-model="editItemForm.classConfig[index-1].end"
              :picker-options="{ start: '08:00',step: '00:10',end: '21:00' }"
              placeholder="结束时间"
            ></el-time-select>
          </el-form-item>
          <el-form-item>
            <el-button type="primary" @click="saveEdit">保存</el-button>
            <el-button @click="cancelEdit">取消</el-button>
          </el-form-item>
        </el-form>
      </el-dialog>

      <el-dialog title="删除用户信息" :visible.sync="deleteDialogVisible">
        <el-form :model="editItemForm" label-width="100px">
          <el-form-item>
            <el-button type="primary" @click="confirm">确定</el-button>
            <el-button @click="cancelDelete">取消</el-button>
          </el-form-item>
        </el-form>
      </el-dialog>

      <el-dialog title="设置当前学期" :visible.sync="selectDialogVisible">
        <el-form :model="editItemForm" label-width="100px">
          <el-form-item>
            <el-button type="primary" @click="Selected">确定</el-button>
            <el-button @click="cancelSelected">取消</el-button>
          </el-form-item>
        </el-form>
      </el-dialog>

      <el-dialog title="添加学期信息" :visible.sync="addDialogVisible">
        <el-form :model="addItemForm" label-width="100px">
          <el-form-item label="学期名称" required>
            <el-input v-model="addItemForm.itemName"/>
          </el-form-item>
          <el-form-item label="学校开始日期" prop="startDate" required>
            <el-date-picker v-model="addItemForm.startDate" type="date" placeholder="选择日期" format="yyyy-MM-dd"/>
          </el-form-item>
          <el-form-item label="总周数" required>
            <el-input v-model.number="addItemForm.totalWeeks"/>
          </el-form-item>
          <el-form-item label="一日课时" required>
            <el-input v-model.number="addItemForm.classPerDay"/>
          </el-form-item>
          <el-form-item v-for="index in addItemForm.classPerDay" :key="index" label="课程时间" required>
            <!-- 时间选择器 -->
            <el-time-select
              v-model="addItemForm.classConfig[index-1].start"
              :picker-options="{  start: '08:00',step: '00:10',end: '21:00' }"
              placeholder="开始时间"
            ></el-time-select>
            至
            <el-time-select
              v-model="addItemForm.classConfig[index-1].end"
              :picker-options="{ start: '08:00',step: '00:10',end: '21:00' }"
              placeholder="结束时间"
            ></el-time-select>
          </el-form-item>
          <el-form-item>
            <el-button type="primary" @click="saveadd">保存</el-button>
            <el-button @click="canceladd">取消</el-button>
          </el-form-item>
        </el-form>
      </el-dialog>
    </div>
  </div>
</template>
<script>
import { getToken } from '@/utils/auth'
import axios from 'axios'
import { Message } from 'element-ui'

var token = getToken()
export default {
  data() {
    return {
      itemTable: [],
      search: '',
      editDialogVisible: false,
      editItemForm: {
        id: '',
        itemName: '',
        startDate: '',
        totalWeeks: '',
        classPerDay: '',
        classConfig: []
      },
      selectedIndex: -1, // 记录编辑的行索引
      id: '',
      deleteDialogVisible: false,
      addItemForm: {
        itemName: '',
        startDate: '',
        totalWeeks: '',
        classPerDay: '',
        classConfig: []
      },
      addDialogVisible: false,
      originalData: '',
      selectDialogVisible: false,
      Role: ''
    }
  },
  methods: {
    formatRole2(row) {
      switch (row.isSelected) {
        case 0:
          return '否'
        case 1:
          return '是'
        default:
          return '未知'
      }
    },
    //编辑学期
    openEditDialog(index, row) {
      this.selectedIndex = index
      this.editItemForm = { ...row } // 将当前行的数据加载到编辑表单中
      // 将后端返回的 classConfig 字符串解析为 JavaScript 对象数组
      this.editItemForm.classConfig = JSON.parse(row.classConfig)
      this.editDialogVisible = true
      this.originalData = this.editItemForm.classPerDay
    },
    saveEdit() {
      // 发送编辑后的数据到后端保存
      const editedItem = this.editItemForm
      editedItem.classConfig = JSON.stringify(this.editItemForm.classConfig) // 将 classConfig 数组转换为字符串
      if (this.editItemForm.startDate instanceof Date) {
        editedItem.startDate = this.editItemForm.startDate.toISOString().split('T')[0]
      } else if (typeof this.editItemForm.startDate === 'string') {
        // 逻辑来判断日期格式是否需要转换
        const parsedDate = new Date(this.editItemForm.startDate)
        if (!isNaN(parsedDate.getTime())) {
          editedItem.startDate = parsedDate.toISOString().split('T')[0]
        }
      }
      axios.put(`http://localhost:8081/item/updateitem`, editedItem, {
        headers: {
          'X-token': token // 添加 Authorization 头部，格式为 Bearer Token
        }
      })
        .then(response => {
            const res = response.data
            // if the custom code is not 20000, it is judged as an error.
            if (res.code !== 20000) {
              Message({
                message: res.message || 'Error',
                type: 'error',
                duration: 5 * 1000
              })
              console.error(res.message)
            } else {
              // 成功保存后更新前端表格数据
              this.itemTable[this.selectedIndex] = { ...editedItem }
              this.editDialogVisible = false
              this.$message.success('编辑成功')
            }
          }
        )
        .catch(error => {
          console.error('发生了一些错误', error.message)
        })
    },
    cancelEdit() {
      // 取消编辑，关闭对话框
      this.editDialogVisible = false
    },
    // 删除学期
    openleDelete(index, row) {
      this.selectedIndex = index
      this.deleteDialogVisible = true
      this.id = row.id
    },
    confirm() {
      // 发送编辑后的数据到后端保存
      const id = this.id
      axios.put(`http://localhost:8081/item/deleteitem`, null, {
        headers: {
          'X-token': token // 添加 Authorization 头部，格式为 Bearer Token
        },
        params: {
          id: id
        }
      })
        .then(response => {
            const res = response.data
            // if the custom code is not 20000, it is judged as an error.
            if (res.code !== 20000) {
              Message({
                message: res.message || 'Error',
                type: 'error',
                duration: 5 * 1000
              })
              console.error(res.message)
            } else {
              // 成功保存后更新前端表格数据
              this.itemTable.splice(this.selectedIndex, 1)
              this.deleteDialogVisible = false
              this.$message.success('删除成功')
            }
          }
        )
        .catch(error => {
          console.error('发生了一些错误', error.message)
        })
    },
    cancelDelete() {
      // 取消编辑，关闭对话框
      this.deleteDialogVisible = false
    },
    //设置为当前学期
    openSelected(index, row) {
      this.selectedIndex = index
      this.selectDialogVisible = true
      this.id = row.id
    },
    Selected() {
      // 发送编辑后的数据到后端保存
      const id = this.id
      axios.put(`http://localhost:8081/item/setselected`, null, {
        headers: {
          'X-token': token // 添加 Authorization 头部，格式为 Bearer Token
        },
        params: {
          id: id
        }
      })
        .then(response => {
            const res = response.data
            // if the custom code is not 20000, it is judged as an error.
            if (res.code !== 20000) {
              Message({
                message: res.message || 'Error',
                type: 'error',
                duration: 5 * 1000
              })
              this.cancelSelected()
              console.error(res.message)
            } else {
              // 成功保存后更新前端表格数据
              this.getitemData()
              this.selectDialogVisible = false
              this.$message.success('设置成功')
            }
          }
        )
        .catch(error => {
          console.error('发生了一些错误', error.message)
        })
    },
    cancelSelected() {
      // 取消编辑，关闭对话框
      this.selectDialogVisible = false
    },
    // 添加学期
    openaddDialog() {
      this.addDialogVisible = true
    },
    saveadd() {
      // 发送编辑后的数据到后端保存
      this.addItemForm.classConfig = JSON.stringify(this.addItemForm.classConfig) // 将 classConfig 数组转换为字符串
      this.addItemForm.startDate = this.addItemForm.startDate.toISOString().split('T')[0]
      axios.post(`http://localhost:8081/item/additem`, {
        itemName: this.addItemForm.itemName,
        startDate: this.addItemForm.startDate,
        totalWeeks: this.addItemForm.totalWeeks,
        classPerDay: this.addItemForm.classPerDay,
        classConfig: this.addItemForm.classConfig
      }, {
        headers: {
          'X-token': token // 添加 Authorization 头部，格式为 Bearer Token
        }
      })
        .then(response => {
            const res = response.data
            // if the custom code is not 20000, it is judged as an error.
            if (res.code !== 20000) {
              Message({
                message: res.message || 'Error',
                type: 'error',
                duration: 5 * 1000
              })
              console.error(res.message)
            } else {
              // 成功保存后更新前端表格数据
              this.addDialogVisible = false
              this.$message.success('编辑成功')
              this.getitemData()
            }
          }
        )
        .catch(error => {
          console.error('发生了一些错误', error.message)
        })
    },
    canceladd() {
      // 取消编辑，关闭对话框
      this.addDialogVisible = false
    },
    getitemData() {
      axios.get('http://localhost:8081/item/getitems', {
        headers: {
          'X-token': token // 添加 Authorization 头部，格式为 Bearer Token
        }
      })
        .then((response) => {
          this.itemTable = response.data
          console.log(this.userTable)
        })
        .catch(function(error) {
          console.error(error)
        })
    },
    updateClassConfigLength(newLength) {
      // 更新classConfig数组长度
      const currentLength = this.addItemForm.classConfig.length
      if (newLength > currentLength) {
        const diff = newLength - currentLength
        for (let i = 0; i < diff; i++) {
          this.addItemForm.classConfig.push({ start: '', end: '' })
        }
      } else if (newLength < currentLength) {
        this.addItemForm.classConfig.splice(newLength)
      }
    },
    updateEditClassConfigLength(newLength) {
      // 更新classConfig数组长度
      const diff = newLength - this.editItemForm.classConfig.length
      for (let i = 0; i < diff; i++) {
        this.editItemForm.classConfig.push({ start: '', end: '' })
      }
    }
  },
  watch: {
    'addItemForm.classPerDay'(newValue) {
      this.updateClassConfigLength(newValue)
    },
    'editItemForm.classPerDay'(newValue) {
      const originalClassPerDay = this.originalData // 假设 originalData 是你的原始数据

      // 将编辑表单中的每日课时数与原始数据进行比较
      if (newValue > originalClassPerDay) {
        // 如果编辑表单的每日课时数超出了原始数据的每日课时数，更新 classConfig 的长度
        this.updateEditClassConfigLength(newValue)
      }
    }
  },
  created: function() {
    // 使用 Axios 的配置项，在请求中添加包含 token 的 header
    axios.get('http://localhost:8081/item/getitems', {
      headers: {
        'X-token': token // 添加 Authorization 头部，格式为 Bearer Token
      }
    })
      .then((response) => {
        this.itemTable = response.data
        console.log(this.userTable)
      })
      .catch(function(error) {
        console.error(error)
      })
    axios.get('http://localhost:8081/user/getrole',{
      headers: {
        'X-token': token // 添加 Authorization 头部，格式为 Bearer Token
      },
    })
      .then((response) => {
        this.Role = response.data.data.role
        console.log(this.Role)
      })
      .catch(function(error) {
        console.error(error)
      })
  }
}
</script>

<style>
.manageItem {
  .itemtable {
    margin: 30px;
  }
}
</style>
