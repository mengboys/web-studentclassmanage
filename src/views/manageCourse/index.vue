<template>
  <div class="manageCourse" v-if="Role>=1">
    <el-button type="primary" @click="openaddDialog()" style="margin-top: 20px;margin-left: 30px">新建</el-button>
    <div class="coursetable">
      <el-table
        :data="courseTable.filter(data => !search || data.courseName.toLowerCase().includes(search.toLowerCase()))"
        border style="width: 100%;margin-top:30px;"
      >
        <el-table-column align="center" label="课程名" prop="courseName"></el-table-column>
        <el-table-column align="center" label="课程编号" prop="courseCode"></el-table-column>
        <el-table-column align="center" label="任课老师" prop="teacher"></el-table-column>
        <el-table-column align="center" label="课程编号" prop="courseCode"></el-table-column>
        <el-table-column align="center" label="课程类型" prop="courseProp" :formatter="formatBelong"></el-table-column>
        <el-table-column align="center" label="上课班级" prop="classTarget"></el-table-column>
        <el-table-column align="center" label="上课人数" prop="studentNumber"></el-table-column>
        <el-table-column align="center">
          <template slot="header" slot-scope="scope">
            <el-input v-model="search" size="mini" placeholder="输入关键字搜索"/>
          </template>
          <template slot-scope="scope">
            <el-button size="mini" @click="openEditDialog(scope.$index, scope.row)">编辑</el-button>
            <el-button size="mini" type="danger" @click="openleDelete(scope.$index, scope.row)">删除</el-button>
          </template>
        </el-table-column>
      </el-table>
    </div>

    <div class="dialog">
      <el-dialog title="编辑课程信息" :visible.sync="editDialogVisible">
        <el-form :model="editCourseForm" label-width="100px" label-position="left">
          <el-form-item label="课程名" required>
            <el-input v-model="editCourseForm.courseName"/>
          </el-form-item>
          <el-form-item label="课程编号" required>
            <el-input v-model="editCourseForm.courseCode"/>
          </el-form-item>
          <el-form-item label="归属学院" required>
            <el-select v-model="selectedDP" filterable placeholder="请选择">
              <el-option
                v-for="department in departmentInfo"
                :key="department.value"
                :label="department.label"
                :value="department.value"
              >
              </el-option>
            </el-select>
          </el-form-item>
          <el-form-item label="上课班级" required>
            <el-input v-model="editCourseForm.classTarget"></el-input>
          </el-form-item>
          <el-form-item label="上课人数" required>
            <el-input v-model.number="editCourseForm.studentNumber"></el-input>
          </el-form-item>
          <el-form-item label="上课学期" required>
            <el-select v-model="editCourseForm.itemId" filterable placeholder="请选择">
              <el-option
                v-for="item in semesters"
                :key="item.id"
                :label="item.itemName"
                :value="item.id"
              >
              </el-option>
            </el-select>
          </el-form-item>
          <el-form-item label="开课专业学期" required>
            <el-input v-model.number="addCourseForm.classItems"></el-input>
          </el-form-item>
          <el-form-item label="归属学科" required>
            <el-select v-model="selectedBP" filterable placeholder="请选择">
              <el-option
                v-for="bT in belongType"
                :key="bT.value"
                :label="bT.label"
                :value="bT.value"
              >
              </el-option>
            </el-select>
          </el-form-item>
          <el-form-item label="任课教师" required>
            <el-select v-model="editCourseForm.teacher" filterable placeholder="请选择">
              <el-option
                v-for="userInfo in userTable"
                :key="userInfo.id"
                :label="userInfo.userName"
                :value="userInfo.id"
              >
              </el-option>
            </el-select>
          </el-form-item>
          <el-form-item label="软件环境">
            <el-input v-model="editCourseForm.softEnv"></el-input>
          </el-form-item>
          <el-form-item label="课程类型" required>
            <el-radio-group v-model="editCourseForm.courseProp">
              <el-radio :label="0">必修</el-radio>
              <el-radio :label="1">限选</el-radio>
              <el-radio :label="2">任选</el-radio>
            </el-radio-group>
          </el-form-item>
          <el-form-item>
            <el-button type="primary" @click="saveEdit">保存</el-button>
            <el-button @click="cancelEdit">取消</el-button>
          </el-form-item>
        </el-form>
      </el-dialog>

      <el-dialog title="删除课程信息" :visible.sync="deleteDialogVisible">
        <el-form :model="editCourseForm" label-width="100px" label-position="left">
          <el-form-item>
            <el-button type="primary" @click="confirm">确定</el-button>
            <el-button @click="cancelDelete">取消</el-button>
          </el-form-item>
        </el-form>
      </el-dialog>

      <el-dialog title="添加课程信息" :visible.sync="addDialogVisible">
        <el-form :model="addCourseForm" label-width="100px" label-position="left">
          <el-form-item label="课程名" required>
            <el-input v-model="addCourseForm.courseName"/>
          </el-form-item>
          <el-form-item label="课程编号" required>
            <el-input v-model="addCourseForm.courseCode"/>
          </el-form-item>
          <el-form-item label="归属学院" required>
            <el-select v-model="selectedDP" filterable placeholder="请选择">
              <el-option
                v-for="department in departmentInfo"
                :key="department.value"
                :label="department.label"
                :value="department.value"
              >
              </el-option>
            </el-select>
          </el-form-item>
          <el-form-item label="上课班级" required>
            <el-input v-model="addCourseForm.classTarget"></el-input>
          </el-form-item>
          <el-form-item label="上课人数" required>
            <el-input v-model.number="addCourseForm.studentNumber"></el-input>
          </el-form-item>
          <el-form-item label="上课学期" required>
            <el-select v-model="addCourseForm.itemId" filterable placeholder="请选择">
              <el-option
                v-for="item in semesters"
                :key="item.id"
                :label="item.itemName"
                :value="item.id"
              >
              </el-option>
            </el-select>
          </el-form-item>
          <el-form-item label="开课专业学期" required>
            <el-input v-model.number="addCourseForm.classItems"></el-input>
          </el-form-item>
          <el-form-item label="归属学科" required>
            <el-select v-model="selectedBP" filterable placeholder="请选择">
              <el-option
                v-for="bT in belongType"
                :key="bT.value"
                :label="bT.label"
                :value="bT.value"
              >
              </el-option>
            </el-select>
          </el-form-item>
          <el-form-item label="任课教师" required>
            <el-select v-model="addCourseForm.teacher" filterable placeholder="请选择">
              <el-option
                v-for="userInfo in userTable"
                :key="userInfo.id"
                :label="userInfo.userName"
                :value="userInfo.id"
              >
              </el-option>
            </el-select>
          </el-form-item>
          <el-form-item label="软件环境">
            <el-input v-model="addCourseForm.softEnv"></el-input>
          </el-form-item>
          <el-form-item label="课程类型" required>
            <el-radio-group v-model="addCourseForm.courseProp">
              <el-radio :label="0">必修</el-radio>
              <el-radio :label="1">限选</el-radio>
              <el-radio :label="2">任选</el-radio>
            </el-radio-group>
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
import message from 'element-ui/packages/message'
import courseTable from 'core-js/internals/array-iteration'
import item from '../../layout/components/Sidebar/Item.vue'

var token = getToken()
export default {
  computed: {
    item() {
      return item
    }
  },
  data() {
    return {
      courseTable: [],
      semesters: [],
      search: '',
      departmentInfo: [{
        value: 1,
        label: '网络'
      }, {
        value: 2,
        label: '计算机'
      }, {
        value: 3,
        label: '软件'
      }, {
        value: 4,
        label: '物联网'
      }, {
        value: 5,
        label: '人工智能'
      }, {
        value: 6,
        label: '大数据'
      }],

      belongType: [{
        value: 1,
        label: '计算机'
      }, {
        value: 2,
        label: '物联网'
      }],
      editDialogVisible: false,
      editCourseForm: {
        id: '',
        itemId: '',
        courseName: '',
        courseCode: '',
        courseProp: '',
        belongType: '',
        department: '',
        classTarget: '',
        studentNumber: '',
        classItems: '',
        softEnv: '',
        teacher: ''
      },
      selectedBelong: {},
      selectedDepartment: {},
      selected: {},
      selectedDP: '',
      selectedBP: '',
      userTable: [],
      selectedIndex: -1, // 记录编辑的行索引
      id: '',
      deleteDialogVisible: false,
      addCourseForm: {
        itemId: '',
        courseName: '',
        courseCode: '',
        courseProp: '',
        belongType: '',
        department: '',
        classTarget: '',
        studentNumber: '',
        classItems: '',
        softEnv: '',
        teacher: ''
      },
      addDialogVisible: false,
      Role: ''
    }
  },
  methods: {
    openEditDialog(index, row) {

      this.selectedIndex = index
      this.editCourseForm = { ...row } // 将当前行的数据加载到编辑表单中

      this.selectedDepartment = this.departmentInfo.find(department => department.value === this.editCourseForm.department)
      this.selectedDP = this.selectedDepartment.value
      this.selectedBelong = this.belongType.find(type => type.value === this.editCourseForm.belongType)
      this.selectedBP = this.selectedBelong.value

      this.selected = this.semesters.find(item => item.id === this.editCourseForm.itemId)
      this.editCourseForm.itemId = this.selected.itemName
      this.selected = this.userTable.find(item => item.id === this.editCourseForm.teacher)
      this.editCourseForm.teacher = this.selected.userName
      this.editDialogVisible = true
    },
    saveEdit() {
      // 发送编辑后的数据到后端保存
      if (typeof this.editCourseForm.itemId === 'string') {
        this.selected = this.semesters.find(item => item.itemName === this.editCourseForm.itemId)
        this.editCourseForm.itemId = this.selected.id
        console.log('myNumber 是字符串类型')
      } else {
        console.log('myNumber 不是字符串类型')
      }
      if (typeof this.editCourseForm.teacher === 'string') {
        this.selected = this.userTable.find(item => item.userName === this.editCourseForm.teacher)
        console.log(this.selected)
        this.editCourseForm.teacher = this.selected.id
        console.log('myNumber 是字符串类型')
      } else {
        console.log('myNumber 不是字符串类型')
      }
      this.editCourseForm.department = this.selectedDP
      this.editCourseForm.belongType = this.selectedBP
      const editedUser = this.editCourseForm

      axios.put(`http://localhost:8081/course/updatecourse`, editedUser, {
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
              this.courseTable[this.selectedIndex] = { ...editedUser }
              this.editDialogVisible = false
              this.$message.success('编辑成功')
              this.getCourseData()
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

    openleDelete(index, row) {
      this.selectedIndex = index
      // this.editUserForm = { ...row } // 将当前行的数据加载到编辑表单中
      this.deleteDialogVisible = true
      this.id = row.id
    },
    confirm() {
      // 发送编辑后的数据到后端保存
      const id = this.id
      axios.put(`http://localhost:8081/course/deletecourse`, null, {
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
              this.userTable.splice(this.selectedIndex, 1)
              this.deleteDialogVisible = false
              this.$message.success('删除成功')
              this.getCourseData()
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

    openaddDialog() {
      this.addCourseForm = {
        itemId: '',
        courseName: '',
        courseCode: '',
        courseProp: '',
        belongType: '',
        department: '',
        classTarget: '',
        studentNumber: '',
        classItems: '',
        softEnv: '',
        teacher: ''
      }
      this.selectedBP = ''
      this.selectedDP = ''
      this.addDialogVisible = true
    },
    saveadd() {
      // 发送编辑后的数据到后端保存
      axios.post(`http://localhost:8081/course/addcourse`, {
        courseCode: this.addCourseForm.courseCode,
        courseName: this.addCourseForm.courseName,
        itemId: this.addCourseForm.itemId,
        courseProp: this.addCourseForm.courseProp,
        belongType: this.selectedBP,
        department: this.selectedDP,
        classTarget: this.addCourseForm.classTarget,
        studentNumber: this.addCourseForm.studentNumber,
        classItems: this.addCourseForm.classItems,
        softEnv: this.addCourseForm.softEnv,
        teacher: this.addCourseForm.teacher
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
              this.getCourseData()
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
    getUserData() {
      axios.get('http://localhost:8081/user/getuser', {
        headers: {
          'X-token': token // 添加 Authorization 头部，格式为 Bearer Token
        }
      })
        .then((response) => {
          this.userTable = response.data
        })
        .catch(function(error) {
          console.error(error)
        })
    },
    getCourseData() {
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
    },
    formatBelong(row) {
      switch (row.belongType) {
        case 1:
          return '必修'
        case 2:
          return '限选'
        case 3:
          return '任选'
        default:
          return '未知'
      }
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
    axios.get('http://localhost:8081/user/getuser', {
      headers: {
        'X-token': token // 添加 Authorization 头部，格式为 Bearer Token
      }
    })
      .then((response) => {
        this.userTable = response.data
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
      })
      .catch(function(error) {
        console.error(error)
      })
  }
}
</script>

<style>
.manageCourse {
  .coursetable {
    margin: 30px;
  }
}
</style>
