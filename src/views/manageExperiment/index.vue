<template>
  <div class="manageExperiment" v-if="Role>=1">
    <el-button type="primary" @click="openaddDialog()" style="margin-top: 20px;margin-left: 30px">新建</el-button>
    <div class="experiment">
      <template>
        <div class="experimentTree">
          <el-input
            placeholder="输入关键字进行过滤"
            v-model="filterText"
          >
          </el-input>

          <el-tree
            class="filter-tree"
            :data="treeData"
            :props="defaultProps"
            default-expand-all
            accordion
            @node-click="handleNodeClick"
            :filter-node-method="filterNode"
            ref="tree"
          >
          </el-tree>
        </div>
        <div class="experimenttable"></div>
        <el-table
          :data="experimentTable.filter(data => !search || data.expName.toLowerCase().includes(search.toLowerCase()))"
          border style="width: 100%;margin-top:30px;"
        >
          <el-table-column align="center" label="名称" prop="expName"></el-table-column>
          <el-table-column align="center" label="所属课程" prop="courseId" :formatter="formatRole2"></el-table-column>
          <el-table-column align="center" label="实验室" prop="classRoom" :formatter="formatRole"></el-table-column>
          <el-table-column align="center" label="课程类型" prop="expProp" :formatter="formatBelong"></el-table-column>
          <el-table-column align="center" width="300">
            <template slot="header" slot-scope="scope">
              <el-input v-model="search" size="mini" placeholder="输入关键字搜索"/>
            </template>
            <template slot-scope="scope">
              <el-button size="mini" @click="openEditDialog(scope.$index, scope.row)">编辑</el-button>
              <el-button size="mini" type="danger" @click="openleDelete(scope.$index, scope.row)">删除</el-button>
              <el-button size="mini" type="primary" @click="opentimneDialog(scope.$index, scope.row)">排课</el-button>
            </template>
          </el-table-column>
        </el-table>
      </template>
    </div>
    <div class="dialog">
      <el-dialog title="编辑实验信息" :visible.sync="editDialogVisible">
        <el-form :model="editExperimentForm" label-width="100px" label-position="left">
          <el-form-item label="实验名称" required>
            <el-input v-model="editExperimentForm.expName"/>
          </el-form-item>
          <el-form-item label="归属学科" required>
            <el-select v-model="selectedCourse" filterable placeholder="请选择">
              <el-option
                v-for="course in courseTable"
                :key="course.id"
                :label="course.courseName"
                :value="course.id"
              >
              </el-option>
            </el-select>
          </el-form-item>
          <el-form-item label="组人数" required>
            <el-input v-model="editExperimentForm.groupNum"/>
          </el-form-item>
          <el-form-item label="辅导老师">
            <el-select v-model="selectedAssistTeacher" filterable placeholder="请选择">
              <el-option
                v-for="user in userTable"
                :key="user.id"
                :label="user.userName"
                :value="user.id"
              >
              </el-option>
            </el-select>
          </el-form-item>
          <el-form-item label="实验项目性质" required>
            <el-radio-group v-model="editExperimentForm.expProp">
              <el-radio :label="1">设计</el-radio>
              <el-radio :label="2">综合</el-radio>
              <el-radio :label="3">基础</el-radio>
            </el-radio-group>
          </el-form-item>
          <el-form-item label="实验要求" required>
            <el-radio-group v-model="editExperimentForm.expRequirement">
              <el-radio :label="1">必修</el-radio>
              <el-radio :label="2">选修</el-radio>
            </el-radio-group>
          </el-form-item>
          <el-form-item>
            <el-button type="primary" @click="saveEdit">保存</el-button>
            <el-button @click="cancelEdit">取消</el-button>
          </el-form-item>
        </el-form>
      </el-dialog>

      <el-dialog title="删除实验信息" :visible.sync="deleteDialogVisible">
        <el-form :model="editExperimentForm" label-width="100px" label-position="left">
          <el-form-item>
            <el-button type="primary" @click="confirm">确定</el-button>
            <el-button @click="cancelDelete">取消</el-button>
          </el-form-item>
        </el-form>
      </el-dialog>

      <el-dialog title="添加实验信息" :visible.sync="addDialogVisible">
        <el-form :model="addExperimentForm" label-width="100px" label-position="left">
          <el-form-item label="实验名称" required>
            <el-input v-model="addExperimentForm.expName"/>
          </el-form-item>
          <el-form-item label="归属学科" required>
            <el-select v-model="selectedCourse" filterable placeholder="请选择">
              <el-option
                v-for="course in courseTable"
                :key="course.id"
                :label="course.courseName"
                :value="course.id"
              >
              </el-option>
            </el-select>
          </el-form-item>
          <el-form-item label="组人数" required>
            <el-input v-model="addExperimentForm.groupNum"/>
          </el-form-item>
          <el-form-item label="辅导老师">
            <el-select v-model="selectedAssistTeacher" filterable placeholder="请选择">
              <el-option
                v-for="user in userTable"
                :key="user.id"
                :label="user.userName"
                :value="user.id"
              >
              </el-option>
            </el-select>
          </el-form-item>
          <el-form-item label="实验项目性质" required>
            <el-radio-group v-model="addExperimentForm.expProp">
              <el-radio :label="1">设计</el-radio>
              <el-radio :label="2">综合</el-radio>
              <el-radio :label="3">基础</el-radio>
            </el-radio-group>
          </el-form-item>
          <el-form-item label="实验要求" required>
            <el-radio-group v-model="addExperimentForm.expRequirement">
              <el-radio :label="1">必修</el-radio>
              <el-radio :label="2">选修</el-radio>
            </el-radio-group>
          </el-form-item>
          <el-form-item>
            <el-button type="primary" @click="saveadd">保存</el-button>
            <el-button @click="canceladd">取消</el-button>
          </el-form-item>
        </el-form>
      </el-dialog>

      <el-dialog title="排课" :visible.sync="timeDialogVisible" width="1050px">
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
        <el-select v-model="selectedLabs" filterable style="margin-left: 20px;" placeholder="请选择实验室">
          <el-option
            v-for="labs in labsTable"
            :key="labs.id"
            :label="labs.roomName"
            :value="labs.id"
          ></el-option>
        </el-select>
        <el-table
          ref="singleTable"
          :data="tableData"
          highlight-current-row
          style="width: 100%"
          border
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
            width="120"
            align="center"
          >
            <template v-slot="{ row, column }">
              <el-button
                v-if="timeExperimentForm.classNumber === tableData.indexOf(row) && timeExperimentForm.classDay === dayIndex"
                type="danger"
                @click="timeDelete(row,column)"
              >取消
              </el-button>
              <el-button
                v-else
                type="primary" @click="addTime(row,column,tableData.indexOf(row),dayIndex)"
              >确定
              </el-button>
            </template>
          </el-table-column>
        </el-table>
        <el-form-item>
          <el-button type="primary" @click="saveadd">保存</el-button>
          <el-button @click="canceladd">取消</el-button>
        </el-form-item>
      </el-dialog>
    </div>
  </div>
</template>
<script>
import { getToken } from '@/utils/auth'
import axios from 'axios'
import { Message } from 'element-ui'
import message from 'element-ui/packages/message'
import courseTable from 'core-js/internals/array-iteration'
import item from '../../layout/components/Sidebar/Item.vue'

var token = getToken()
export default {
  computed: {
    item() {
      return item
    },
    defaultProps() {
      return {
        children: 'children',
        label: 'label'
      }
    }
  },
  data() {
    return {
      experimentTable: [],
      semesters: [],
      selectedSemester: '',
      selectedWeek: '',
      selectedLabs: '',
      search: '',
      editDialogVisible: false,
      editExperimentForm: {
        assistTeacher: '',
        classDay: '',
        classNumber: '',
        classRoom: '',
        classWeek: '',
        courseId: '',
        expName: '',
        expProp: '',
        expRequirement: '',
        groupNum: '',
        id: '',
        itemId: ''
      },
      selectedCourse: '',
      selectedAssistTeacher: '',
      userTable: [],
      labsTable: [],
      selectedIndex: -1, // 记录编辑的行索引
      id: '',
      deleteDialogVisible: false,
      addExperimentForm: {
        assistTeacher: '',
        classDay: '',
        classNumber: '',
        classRoom: '',
        classWeek: '',
        courseId: '',
        expName: '',
        expProp: '',
        expRequirement: '',
        groupNum: '',
        itemId: ''
      },
      addDialogVisible: false,
      treeData: [],
      courseTable: [],
      filterText: '',
      labsName: '',
      clickData: '',
      tableData: [], // 表格数据
      daysOfWeek: [
        { property: 'name', label: '星期一' },
        { property: 'name', label: '星期二' },
        { property: 'name', label: '星期三' },
        { property: 'name', label: '星期四' },
        { property: 'name', label: '星期五' },
        { property: 'name', label: '星期六' },
        { property: 'name', label: '星期日' }
      ],
      weekOptions: [],
      timeDialogVisible: false,
      timeExperimentForm: {
        classDay: '',
        classNumber: '',
        classRoom: '',
        classWeek: '',
        id: '',
        assistTeacher: '',
        courseId: '',
        expName: '',
        expProp: '',
        expRequirement: '',
        groupNum: '',
        itemId: ''
      }, Role: ''
    }
  },
  watch: {
    filterText(val) {
      this.$refs.tree.filter(val)
    },
    selectedSemester(newVal) {
      // 当 value1 变化时触发更新第二个选项的数据
      this.updateWeekOptions()
    }
  },
  methods: {
    addTime(row, column, rowIndex, dayIndex) {
      this.timeExperimentForm = this.editExperimentForm
      this.timeExperimentForm.classWeek = this.selectedWeek
      this.timeExperimentForm.classDay = dayIndex
      this.timeExperimentForm.classNumber = rowIndex
      this.timeExperimentForm.classRoom = this.selectedLabs
      this.timeExperimentForm.itemId = this.selectedSemester
      this.timeExperimentForm.id = this.id
      const editedUser = this.timeExperimentForm
      console.log(editedUser)
      axios.put(`http://localhost:8081/experiment/updateexperiment`, editedUser, {
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
              this.experimentTable[this.selectedIndex] = { ...editedUser }
              this.editDialogVisible = false
              this.$message.success('编辑成功')
            }
          }
        )
        .catch(error => {
          console.error('发生了一些错误', error.message)
        })
    },
    timeDelete(row, column) {
      this.timeExperimentForm = this.editExperimentForm
      this.timeExperimentForm.classWeek = null
      this.timeExperimentForm.classDay = null
      this.timeExperimentForm.classNumber = null
      this.timeExperimentForm.classRoom = null
      this.timeExperimentForm.itemId = null
      this.timeExperimentForm.id = this.id
      const editedUser = this.timeExperimentForm
      axios.put(`http://localhost:8081/experiment/updateexperiment`, editedUser, {
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
              this.experimentTable[this.selectedIndex] = { ...editedUser }
              this.editDialogVisible = false
              this.$message.success('编辑成功')
              this.getExperiment(this.clickData.id)
            }
          }
        )
        .catch(error => {
          console.error('发生了一些错误', error.message)
        })
    },
    opentimneDialog(index, row) {
      this.selectedIndex = index
      this.editExperimentForm = { ...row } // 将当前行的数据加载到编辑表单中
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
      this.timeExperimentForm = this.editExperimentForm
      console.log(this.editExperimentForm)
      this.timeDialogVisible = true
      this.id = row.id
    }
    ,
    updateWeekOptions() {
      // 根据选中的学期动态生成周数选项
      const selectedSemester = this.semesters.find(semester => semester.id === this.selectedSemester)
      if (selectedSemester) {
        const totalWeeks = selectedSemester.totalWeeks
        this.weekOptions = Array.from({ length: totalWeeks }, (_, index) => index + 1)
        this.selectedWeek = 1 // 重置选中的周数
      }
      console.log(selectedSemester)
      // 将后端返回的 classConfig 字符串解析为 JavaScript 对象数组
      this.tableData = JSON.parse(selectedSemester.classConfig)
      console.log(this.tableData)
    }
    ,

    indexMethod(index) {
      return this.tableData[index].start + '-' + this.tableData[index].end
    }
    ,
    handleNodeClick(data) {
      this.clickData = data
      this.getExperiment(data)
    }
    ,
    filterNode(value, data) {
      if (!value) return true
      return data.label.indexOf(value) !== -1
    }
    ,
    getExperiment(data) {
      axios.put(`http://localhost:8081/experiment/getexperiment`, null, {
        headers: {
          'X-token': token // 添加 Authorization 头部，格式为 Bearer Token
        },
        params: {
          id: data.id
        }
      })
        .then(response => {
            // 成功保存后更新前端表格数据
            this.experimentTable = response.data
            this.deleteDialogVisible = false
            this.$message.success('获取成功')
          }
        )
        .catch(error => {
          console.error('发生了一些错误', error.message)
        })
    }
    ,
    openEditDialog(index, row) {

      this.selectedIndex = index
      this.editExperimentForm = { ...row } // 将当前行的数据加载到编辑表单中
      const select1 = this.courseTable.find(course => course.id === this.editExperimentForm.courseId)
      this.selectedCourse = select1.courseName

      const select2 = this.userTable.find(user => user.id === this.editExperimentForm.assistTeacher)
      this.selectedAssistTeacher = select2.userName
      this.editDialogVisible = true
    }
    ,
    saveEdit() {
      // 发送编辑后的数据到后端保存
      if (typeof this.selectedCourse === 'string') {
        const select1 = this.courseTable.find(course => course.courseName === this.selectedCourse)
        this.editExperimentForm.courseId = select1.id
        console.log('myNumber 是字符串类型')
      } else {
        this.editExperimentForm.courseId = this.selectedCourse
        console.log('myNumber 不是字符串类型')
      }
      if (typeof this.selectedAssistTeacher === 'string') {
        const select2 = this.userTable.find(user => user.userName === this.selectedAssistTeacher)
        this.editExperimentForm.assistTeacher = select2.id
        console.log('myNumber 是字符串类型')
      } else {
        this.editExperimentForm.assistTeacher = this.selectedAssistTeacher
        console.log('myNumber 不是字符串类型')
      }
      console.log(this.editExperimentForm)
      const editedUser = this.editExperimentForm

      axios.put(`http://localhost:8081/experiment/updateexperiment`, editedUser, {
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
              this.experimentTable[this.selectedIndex] = { ...editedUser }
              this.editDialogVisible = false
              this.$message.success('编辑成功')
              this.getExperiment(this.clickData.id)
            }
          }
        )
        .catch(error => {
          console.error('发生了一些错误', error.message)
        })
    }
    ,
    cancelEdit() {
      // 取消编辑，关闭对话框
      this.editDialogVisible = false
    }
    ,

    openleDelete(index, row) {
      this.selectedIndex = index
      // this.editUserForm = { ...row } // 将当前行的数据加载到编辑表单中
      this.deleteDialogVisible = true
      this.id = row.id
    }
    ,
    confirm() {
      // 发送编辑后的数据到后端保存
      const id = this.id
      axios.put(`http://localhost:8081/experiment/deleteexperiment`, null, {
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
              this.experimentTable.splice(this.selectedIndex, 1)
              this.deleteDialogVisible = false
              this.$message.success('删除成功')
            }
          }
        )
        .catch(error => {
          console.error('发生了一些错误', error.message)
        })
    }
    ,
    cancelDelete() {
      // 取消编辑，关闭对话框
      this.deleteDialogVisible = false
    }
    ,

    openaddDialog() {
      this.addExperimentForm = {
        assistTeacher: '',
        classDay: '',
        classNumber: '',
        classRoom: '',
        classTime: '',
        classWeek: '',
        courseId: '',
        expName: '',
        expProp: '',
        expRequirement: '',
        groupNum: '',
        itemId: ''
      }
      this.selectedAssistTeacher = ''
      this.selectedCourse = ''
      this.addDialogVisible = true
    }
    ,
    saveadd() {
      // 发送编辑后的数据到后端保存
      axios.post(`http://localhost:8081/experiment/addexperiment`, {
        assistTeacher: this.selectedAssistTeacher,
        courseId: this.selectedCourse,
        expName: this.addExperimentForm.expName,
        expProp: this.addExperimentForm.expProp,
        expRequirement: this.addExperimentForm.expRequirement,
        groupNum: this.addExperimentForm.groupNum
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
              this.getExperiment(this.clickData)
            }
          }
        )
        .catch(error => {
          console.error('发生了一些错误', error.message)
        })
    }
    ,
    canceladd() {
      // 取消编辑，关闭对话框
      this.addDialogVisible = false
    }
    ,
    formatBelong(row) {
      switch (row.expProp) {
        case 1:
          return '设计'
        case 2:
          return '综合'
        case 3:
          return '基础'
        default:
          return '未知'
      }
    }
    ,
    formatRole(row) {
      for (let i = 0; i < this.labsTable.length; i++) {
        if (row.classRoom === this.labsTable[i].id) {
          return this.labsTable[i].roomName
        }
      }
    }
    ,
    formatRole2(row) {
      for (let i = 0; i < this.courseTable.length; i++) {
        if (row.courseId === this.courseTable[i].id) {
          return this.courseTable[i].courseName
        }
      }
    }
    ,
    Tree() {
      // 创建根节点数组
      let treeData = []

      // 遍历用户列表
      this.userTable.forEach(user => {
        // 创建用户节点对象
        let userNode = {
          id: user.id,
          label: user.userName,
          children: []
        }
        // 在课程列表中查找该用户所教的课程
        let userCourses = this.courseTable.filter(course => course.teacher === user.id)
        // 遍历课程列表，为用户节点创建课程子节点
        userCourses.forEach(course => {
          let courseNode = {
            id: course.id,
            label: course.courseName
          }
          userNode.children.push(courseNode) // 将课程节点添加为用户节点的子节点
        })

        // 将用户节点添加到根节点数组
        treeData.push(userNode)
      })

      // 更新 Vue 实例中的树结构数据
      this.treeData = treeData
    }
  }
  ,
  created: function() {
    // 使用 Axios 的配置项，在请求中添加包含 token 的 header
    axios.get('http://localhost:8081/course/getcourse', {
      headers: {
        'X-token': token // 添加 Authorization 头部，格式为 Bearer Token
      }
    })
      .then((response) => {
        this.courseTable = response.data
      })
      .catch(function(error) {
        console.error(error)
      })
    axios.get('http://localhost:8081/item/getitems', {
      headers: {
        'X-token': token // 添加 Authorization 头部，格式为 Bearer Token
      }
    })
      .then((response) => {
        this.semesters = response.data
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
    axios.get('http://localhost:8081/user/getuser', {
      headers: {
        'X-token': token // 添加 Authorization 头部，格式为 Bearer Token
      }
    })
      .then((response) => {
        this.userTable = response.data
        this.Tree()
      })
      .catch(function(error) {
        console.error(error)
      })
    axios.get('http://localhost:8081/user/getrole', {
      headers: {
        'X-token': token // 添加 Authorization 头部，格式为 Bearer Token
      }
    })
      .then((response) => {
        this.Role = response.data.data.role
      })
      .catch(function(error) {
        console.error(error)
      })
  }

}
</script>
<style>
.manageExperiment {
  .experimenttable {
    margin: 30px;
  }
}

.experiment {
  display: flex;
}

.experimentTree {
  flex: 0 0 14%;
  margin: 30px;
}

.experimenttable {
}
</style>
