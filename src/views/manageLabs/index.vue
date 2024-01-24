<template>
  <div class="manageLabs" v-if="Role===2">
    <el-button type="primary" @click="openaddDialog()" style="margin-top: 20px;margin-left: 30px">新建</el-button>
    <div class="labstable">
      <el-table :data="labsTable.filter(data => !search || data.roomName.toLowerCase().includes(search.toLowerCase()))"
                border
                style="width: 100%;margin-top:30px;"
      >
        <el-table-column align="center" label="实验室名称" prop="roomName"></el-table-column>
        <el-table-column align="center" label="实验室地址" prop="classroom"></el-table-column>
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
      <el-dialog title="编辑实验室信息" :visible.sync="editDialogVisible">
        <el-form :model="editLabsForm" label-width="100px">
          <el-form-item label="实验室名称" required>
            <el-input v-model="editLabsForm.roomName"/>
          </el-form-item>
          <el-form-item label="实验室地址" required>
            <el-input v-model="editLabsForm.classroom"/>
          </el-form-item>
          <el-form-item label="介绍" required>
            <el-input
              type="textarea"
              placeholder="请输入内容"
              :autosize="{ minRows: 6, maxRows: 10}"
              v-model="editLabsForm.description"
              maxlength="1000"
            ></el-input>
          </el-form-item>
          <el-form-item>
            <el-button type="primary" @click="saveEdit">保存</el-button>
            <el-button @click="cancelEdit">取消</el-button>
          </el-form-item>
        </el-form>
      </el-dialog>

      <el-dialog title="删除实验室信息" :visible.sync="deleteDialogVisible">
        <el-form :model="editLabsForm" label-width="100px">
          <el-form-item>
            <el-button type="primary" @click="confirm">确定</el-button>
            <el-button @click="cancelDelete">取消</el-button>
          </el-form-item>
        </el-form>
      </el-dialog>

      <el-dialog title="添加实验室信息" :visible.sync="addDialogVisible">
        <el-form :model="addLabsForm" label-width="100px">
          <el-form-item label="实验室名称" required>
            <el-input v-model="addLabsForm.roomName"/>
          </el-form-item>
          <el-form-item label="实验室地址" required>
            <el-input v-model="addLabsForm.classroom"/>
          </el-form-item>
          <el-form-item label="简介" required>
            <el-input v-model="addLabsForm.description"></el-input>
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
      labsTable: [],
      search: '',
      editDialogVisible: false,
      editLabsForm: {
        id: '',
        roomName: '',
        classroom: '',
        description: ''
      },
      selectedIndex: -1, // 记录编辑的行索引
      id: '',
      deleteDialogVisible: false,
      addLabsForm: {
        roomName: '',
        classroom: '',
        description: ''
      },
      addDialogVisible: false,
      Role: ''
    }
  },
  methods: {
    //编辑实验室
    openEditDialog(index, row) {
      this.selectedIndex = index
      this.editLabsForm = { ...row } // 将当前行的数据加载到编辑表单中
      this.editDialogVisible = true
    },
    saveEdit() {
      // 发送编辑后的数据到后端保存
      const editedUser = this.editLabsForm
      axios.put(`http://localhost:8081/labs/updatelabs`, editedUser, {
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
              this.labsTable[this.selectedIndex] = { ...editedUser }
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
    // 删除实验室
    openleDelete(index, row) {
      this.selectedIndex = index
      this.deleteDialogVisible = true
      this.id = row.id
    },
    confirm() {
      // 发送编辑后的数据到后端保存
      const id = this.id
      axios.put(`http://localhost:8081/labs/deletelabs`, null, {
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
              this.cancelDelete()
              console.error(res.message)
            } else {
              // 成功保存后更新前端表格数据
              this.getLabsData()
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
    // 添加实验室
    openaddDialog() {
      this.addDialogVisible = true
    },
    saveadd() {
      // 发送编辑后的数据到后端保存
      axios.post(`http://localhost:8081/labs/addlabs`, {
        roomName: this.addLabsForm.roomName,
        classroom: this.addLabsForm.classroom,
        description: this.addLabsForm.description
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
              this.getLabsData()
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
    getLabsData() {
      axios.get('http://localhost:8081/labs/getlabs', {
        headers: {
          'X-token': token // 添加 Authorization 头部，格式为 Bearer Token
        }
      })
        .then((response) => {
          this.labsTable = response.data
          console.log(this.userTable)
        })
        .catch(function(error) {
          console.error(error)
        })
    }
  },
  created: function() {
    // 使用 Axios 的配置项，在请求中添加包含 token 的 header
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
.manageLabs {
  .labstable {
    margin: 30px;
  }
}
</style>
