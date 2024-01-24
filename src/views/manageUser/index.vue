<template>
  <div class="manageUser" v-if="Role===2">
    <el-button type="primary" @click="openaddDialog()" style="margin-top: 20px;margin-left: 30px">新建</el-button>

    <div class="usertable">
      <el-table :data="userTable.filter(data => !search || data.userName.toLowerCase().includes(search.toLowerCase()))"
                border style="width: 100%;margin-top:30px;"
      >
        <el-table-column align="center" label="员工号" prop="employeeId"></el-table-column>
        <el-table-column align="center" label="姓名" prop="userName"></el-table-column>
        <el-table-column align="center" label="身份" prop="isAdmin" :formatter="formatRole"></el-table-column>
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
      <el-dialog title="编辑用户信息" :visible.sync="editDialogVisible">
        <el-form :model="editUserForm" label-width="100px" label-position="left" ref="editUserForm">
          <el-form-item label="姓名" required>
            <el-input v-model="editUserForm.userName"/>
          </el-form-item>
          <el-form-item label="员工号" required>
            <el-input v-model="editUserForm.employeeId"/>
          </el-form-item>
          <el-form-item label="密码">
            <el-input v-model="editUserForm.password" placeholder="不修改密码则留空"></el-input>
          </el-form-item>
          <el-form-item label="身份" required>
            <el-radio-group v-model="editUserForm.isAdmin">
              <el-radio :label="0">参观</el-radio>
              <el-radio :label="1">管理</el-radio>
              <el-radio :label="2">超管</el-radio>
            </el-radio-group>
          </el-form-item>
          <el-form-item>
            <el-button type="primary" @click="saveEdit">保存</el-button>
            <el-button @click="cancelEdit">取消</el-button>
          </el-form-item>
        </el-form>
      </el-dialog>

      <el-dialog title="删除用户信息" :visible.sync="deleteDialogVisible">
        <el-form :model="editUserForm" label-width="100px" label-position="left">
          <el-form-item>
            <el-button type="primary" @click="confirm">确定</el-button>
            <el-button @click="cancelDelete">取消</el-button>
          </el-form-item>
        </el-form>
      </el-dialog>

      <el-dialog title="添加用户信息" :visible.sync="addDialogVisible">
        <el-form :model="addUserForm" label-width="100px" label-position="left">
          <el-form-item label="姓名" required>
            <el-input v-model="addUserForm.userName"/>
          </el-form-item>
          <el-form-item label="员工号" required>
            <el-input v-model="addUserForm.employeeId"/>
          </el-form-item>
          <el-form-item label="密码">
            <el-input v-model="addUserForm.password"></el-input>
          </el-form-item>
          <el-form-item label="身份" required>
            <el-radio-group v-model="addUserForm.isAdmin">
              <el-radio :label="0">参观</el-radio>
              <el-radio :label="1">管理</el-radio>
              <el-radio :label="2">超管</el-radio>
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

var token = getToken()
export default {
  data() {
    return {
      userTable: [],
      search: '',
      editDialogVisible: false,
      editUserForm: {
        id: '',
        userName: '',
        employeeId: '',
        isAdmin: '',
        password: ''
      },
      selectedIndex: -1, // 记录编辑的行索引
      id: '',
      deleteDialogVisible: false,
      addUserForm: {
        userName: '',
        employeeId: '',
        isAdmin: '',
        password: ''
      },
      addDialogVisible: false,
      Role: ''
    }
  },
  methods: {
    openEditDialog(index, row) {
      this.selectedIndex = index
      this.editUserForm = { ...row } // 将当前行的数据加载到编辑表单中
      this.editUserForm.password = null
      this.editDialogVisible = true
    },
    saveEdit() {
      // 发送编辑后的数据到后端保存
      const editedUser = this.editUserForm
      if (!editedUser.password || (editedUser.password && editedUser.password.length >= 6)) {
        axios.put(`http://localhost:8081/user/updateuser`, editedUser, {
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
                this.userTable[this.selectedIndex] = { ...editedUser }
                this.editDialogVisible = false
                this.$message.success('编辑成功')
              }
            }
          )
          .catch(error => {
            console.error('发生了一些错误', error.message)
          })
      } else {
        // 密码长度不足6位
        console.error('密码长度需大于等于6个字符！')
        this.$message.error('密码长度需大于等于6个字符！')
      }
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
      axios.put(`http://localhost:8081/user/deleteuser`, null, {
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
      this.addDialogVisible = true
    },
    saveadd() {
      // 发送编辑后的数据到后端保存
      if (!this.addUserForm.password || (this.addUserForm.password && this.addUserForm.password.length >= 6)) {
        axios.post(`http://localhost:8081/user/adduser`, {
          userName: this.addUserForm.userName,
          employeeId: this.addUserForm.employeeId,
          password: this.addUserForm.password,
          isAdmin: this.addUserForm.isAdmin
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
                this.getUserData()
              }
            }
          )
          .catch(error => {
            console.error('发生了一些错误', error.message)
          })
      } else {
        // 密码长度不足6位
        console.error('密码长度需大于等于6个字符！')
        this.$message.error('密码长度需大于等于6个字符！')
      }
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
          console.log(this.userTable)
        })
        .catch(function(error) {
          console.error(error)
        })
    },
    formatRole(row) {
      switch (row.isAdmin) {
        case 0:
          return '参观'
        case 1:
          return '管理'
        case 2:
          return '超管'
        default:
          return '未知'
      }
    }
  }
  ,
  created: function() {
    // 使用 Axios 的配置项，在请求中添加包含 token 的 header
    axios.get('http://localhost:8081/user/getuser', {
      headers: {
        'X-token': token // 添加 Authorization 头部，格式为 Bearer Token
      }
    })
      .then((response) => {
        this.userTable = response.data
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
      })
      .catch(function(error) {
        console.error(error)
      })
  }
}
</script>

<style>
.manageUser {
  .usertable {
    margin: 30px;
  }
}
</style>
