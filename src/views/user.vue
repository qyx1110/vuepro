<template>
<el-card class="box-card">
  <!-- 面包屑 -->
  <el-breadcrumb separator="/">
    <el-breadcrumb-item>首页</el-breadcrumb-item>
    <el-breadcrumb-item>用户管理</el-breadcrumb-item>
    <el-breadcrumb-item>用户列表</el-breadcrumb-item>
  </el-breadcrumb>

  <!-- 搜索框 -->
  <el-row class="searchArea">
    <el-col :span="24">
      <el-input placeholder="请输入内容" class="searchInput" v-model="searchVal">
        <el-button slot="append" icon="el-icon-search" @click="checkUser()"></el-button>
      </el-input>
      <el-button type="primary" @click="showAddUserDia()">添加用户</el-button>
    </el-col>
  </el-row>

  <!-- 添加用户的对话框 -->
  <el-dialog title="添加用户" :visible.sync="dialogFormVisibleAdduser" @close="close()">
    <el-form :model="formDate">

      <el-form-item label="用户名" :label-width="formLabelWidth">
        <el-input autocomplete="off" v-model="formDate.username"></el-input>
      </el-form-item>

      <el-form-item label="密码" :label-width="formLabelWidth">
        <el-input autocomplete="off" v-model="formDate.password"></el-input>
      </el-form-item>

      <el-form-item label="邮箱" :label-width="formLabelWidth">
        <el-input autocomplete="off" v-model="formDate.email"></el-input>
      </el-form-item>

      <el-form-item label="电话" :label-width="formLabelWidth">
        <el-input autocomplete="off" v-model="formDate.mobile"></el-input>
      </el-form-item>

    </el-form>
    <div slot="footer" class="dialog-footer">
      <el-button @click="dialogFormVisibleAdduser = false">取 消</el-button>
      <el-button type="primary" @click="addUser()">确 定</el-button>
    </div>
  </el-dialog>

  <!-- 编辑用户的对话框 -->
  <el-dialog title="编辑用户" :visible.sync="dialogFormVisibleEdituser" @close="close()">
    <el-form :model="formDate">

      <el-form-item label="用户名" :label-width="formLabelWidth">
        <el-input disabled autocomplete="off" v-model="formDate.username"></el-input>
      </el-form-item>

      <el-form-item label="邮箱" :label-width="formLabelWidth">
        <el-input autocomplete="off" v-model="formDate.email"></el-input>
      </el-form-item>

      <el-form-item label="电话" :label-width="formLabelWidth">
        <el-input autocomplete="off" v-model="formDate.mobile"></el-input>
      </el-form-item>

    </el-form>
    <div slot="footer" class="dialog-footer">
      <el-button @click="dialogFormVisibleEdituser = false">取 消</el-button>
      <el-button type="primary" @click="editUser()">确 定</el-button>
    </div>
  </el-dialog>

  <!-- 分配权限对话框 -->
  <el-dialog title="分配角色" :visible.sync="dialogFormVisibleSetrole" @close="close()">
    <el-form :model="formDate">

      <el-form-item label="用户名" :label-width="formLabelWidth">
        {{currUserName}}
      </el-form-item>

      <el-form-item label="角色" :label-width="formLabelWidth">
        <el-select v-model="currRoleId">
          <el-option disabled label="请选择" :value = "-1"></el-option>
          <el-option v-for="(item,index) in roles" :key="index" :label="item.roleName" :value="item.id">
          </el-option>
        </el-select>
      </el-form-item>

    </el-form>
    <div slot="footer" class="dialog-footer">
      <el-button @click="dialogFormVisibleSetrole = false">取 消</el-button>
      <el-button type="primary" @click="setRole()">确 定</el-button>
    </div>
  </el-dialog>

  <!-- 表格 -->
  <el-table v-loading="loading" :data="list" style="width: 100%">
    <el-table-column type="index" label="#" width="80">
    </el-table-column>

    <el-table-column prop="username" label="姓名" width="120">
    </el-table-column>

    <el-table-column prop="email" label="邮箱" width="200">
    </el-table-column>

    <el-table-column prop="mobile" label="电话" width="100">
    </el-table-column>

    <el-table-column prop="create_time" label="创建日期" width="180">
      <template slot-scope="scope">
        {{scope.row.create_time | fmtDate}}
      </template>

    </el-table-column>

    <el-table-column prop="date" label="用户状态" width="120">
      <template slot-scope="scope">
        <el-switch @change="changeSwitchMgstate(scope.row)" v-model="scope.row.mg_state" active-color="#13ce66" inactive-color="#ff4949">
        </el-switch>
      </template>

    </el-table-column>

    <el-table-column prop="date" label="操作" width="200">
      <template slot-scope="scope">
        <el-row>
          <el-button type="primary" icon="el-icon-edit" circle size="mini" plain @click="showEditBox(scope.row.id)"></el-button>
          <el-button type="success" icon="el-icon-check" circle size="mini" plain @click="showRoleBox(scope.row)"></el-button>
          <el-button type="danger" icon="el-icon-delete" circle size="mini" plain @click="showDeleteBox(scope.row.id)"></el-button>
        </el-row>
      </template>
    </el-table-column>

  </el-table>

  <!-- 分页 -->
  <el-pagination @size-change="handleSizeChange" @current-change="handleCurrentChange" :current-page="currentPage" :page-sizes="[2, 4, 6, 8]" :page-size="pageSize" layout="total, sizes, prev, pager, next, jumper" :total="total">
  </el-pagination>
</el-card>
</template>

<script>
export default {
  data() {
    return {
      list: [],
      loading: false,
      currentPage: 1,
      pageSize: 2,
      pagenum: 1,
      total: 0,
      searchVal: '',
      dialogFormVisibleAdduser: false,
      formDate: {},
      dialogFormVisibleEdituser: false,
      formLabelWidth: '100px',
      dialogFormVisibleSetrole: false,
      
      //当前用户名
      currUserName: '',

      //当前的角色id
      currRoleId: -1,

      //所有角色(主管, 测试 等 )
      roles: [],

      //当前用户id
      currUserId: -1


    }
  },

  created() {
    this.loadTableDate()
  },

  methods: {
    //分配权限 发送请求
    async setRole(){
      const res = await this.$http.put(`users/${this.currUserId}/role`,{
        rid: this.currRoleId
      })
      const {
        meta: {
          status,
          msg
        }
      } = res.data

      if(status === 200){
        this.$message.success(msg)
        this.dialogFormVisibleSetrole = false
        this.currRoleId = -1
      }

    },

    //分配权限  显示对话框
    async showRoleBox(user){
      this.dialogFormVisibleSetrole = true
      console.log(user)
      this.currUserId = user.id
      this.currUserName = user.username
      const res = await this.$http.get(`roles`)
      console.log(res)
      this.roles = res.data.data
      const res2 = await this.$http.get(`users/${user.id}`)
      this.currRoleId = res2.data.data.rid
      // console.log(res2)

    },

    

    close() {
      this.formDate = {}
    },
    //编辑用户
    //显示编辑用户的提示框
    async showEditBox(userId) {
      this.dialogFormVisibleEdituser = true
      const res = await this.$http.get(`users/${userId}`)

      // console.log(res)

      this.formDate = res.data.data

    },

    async editUser() {
      //关闭提示框
      this.dialogFormVisibleEdituser = false

      const res = await this.$http.put(`users/${this.formDate.id}`, this.formDate)
      const {
        meta: {
          status,
          msg
        }
      } = res.data

      if (status === 200) {
        this.loadTableDate()
        this.$message.success(msg)
      }
    },

    //添加用户
    //显示添加用户的提示框
    showAddUserDia() {
      this.dialogFormVisibleAdduser = true
    },

    async addUser() {
      //关闭对话框
      this.dialogFormVisibleAdduser = false

      //发送请求
      const res = await this.$http.post(`users`, this.formDate)
      //  console.log(res)
      const {
        meta: {
          msg,
          status
        }
      } = res.data

      if (status === 201) {
        this.loadTableDate()
        this.$message.success(msg)
        //  this. formDate = {}
        //for(const key in this.formDate){
        //  if(this.formDate.hasOwnProperty(key)){
        //    this.formDate[key]=''
        //  }
        //}
      }
    },

    //删除用户

    //显示删除的提示框
    showDeleteBox(userId) {
      this.$confirm('Sure?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(async () => {
        const res = await this.$http.delete(`users/${userId}`)

        const {
          meta: {
            status,
            msg
          }
        } = res.data

        if (status === 200) {
          //刷新数据
          this.loadTableDate()

          //提示信息
          this.$message({
            type: 'success',
            message: msg
          })
        }
      }).catch(() => {
        this.$message({
          type: 'info',
          message: '已取消删除'
        })
      })
    },

    //改变用户状态
    async changeSwitchMgstate(user) {
      //请求路径：users/:uId/state/:type
      const res = await this.$http.put(`users/${user.id}/state/${user.mg_state}`)
      // console.log(res)
      const {
        meta: {
          status,
          msg
        }
      } = res.data

      if (status === 200) {
        this.$message.success(msg)
      } else {
        this.$message.error(msg)
      }
    },

    // 查询用户
    checkUser() {
      this.loadTableDate()
    },

    handleSizeChange(val) {
      this.pageSize = val
      this.loadTableDate()
    },
    handleCurrentChange(val) {
      this.pagenum = val
      this.loadTableDate()
    },

    async loadTableDate() {
      this.loading = true
      //除了登录功能 其他功能的接口都需要加入token才能请求
      const AUTH_TOKEN = sessionStorage.getItem('token')
      this.$http.defaults.headers.common['Authorization'] = AUTH_TOKEN
      const res = await this.$http.get(`users?pagenum=${this.pagenum}&pagesize=${this.pageSize}&query=${this.searchVal}`)
      // console.log(res)

      this.total = res.data.data.total
      // console.log(this.total)
      const {
        meta: {
          msg,
          status
        },
        data: {
          users
        }
      } = res.data

      if (status === 200) {
        this.loading = false
        this.list = users
        // console.log(this.list)
        this.pagenum = 1
      }
    }
  }
}
</script>

<style>
.box-card {
  height: 100%;
}

.searchArea {
  margin-top: 10px;
  margin-bottom: 10px;
}

.searchArea .searchInput {
  width: 350px;
}
</style>
