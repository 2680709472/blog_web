<template>
  <div class="app-container">
    <div>
      <el-form ref="form" :model="form" class="form">
        <el-form-item>
          <el-col :span="5">
            <el-input v-model="form.username" placeholder="用户名"/>
          </el-col>

          <el-col :span="5">
            <el-select v-model="form.type" placeholder="选择类型">
              <el-option label="管理员" value="1"/>
              <el-option label="普通用户" value="2"/>
            </el-select>
          </el-col>
          <el-col :span="2">
            <el-button type="primary" @click="search" style="margin-right: 10px;">查询</el-button>
          </el-col>

        </el-form-item>
        <el-form-item>
          <el-tag type="success" @click="adduser({})" class="tag_hover">新增</el-tag>
          <el-tag type="danger" @click="delete_user({})" class="tag_hover">删除</el-tag>
        </el-form-item>
      </el-form>
    </div>
    <div>
      <el-table
        v-loading="listLoading"
        :data="list"
        element-loading-text="Loading"
        border
        fit
        highlight-current-row
        @selection-change="handleSelectionChange"
      >
        <el-table-column
          type="selection"
          width="55">
        </el-table-column>
        <el-table-column align="center" label="id" width="95" prop="id" sortable>
        </el-table-column>

        <el-table-column label="用户名" align="center" prop="username">
        </el-table-column>
        <el-table-column label="邮箱" align="center" prop="email">
        </el-table-column>
        <el-table-column label="手机" align="center" prop="phone">
        </el-table-column>
        <el-table-column label="个人介绍" align="center" prop="introduce">
        </el-table-column>
        <el-table-column label="类型" align="center" prop="type">
          <template v-slot="scope">
            <el-tag :type="scope.row.type | statusFilter">{{ scope.row.type === 1 ? "管理员" : "普通用户" }}</el-tag>

          </template>
        </el-table-column>
        <el-table-column label="创建时间" align="center" prop="create_time">
          <template v-slot="scope">
            <span>{{ scope.row.create_time.split("T")[0] }}</span>
          </template>
        </el-table-column>
        <el-table-column label="操作" align="center">
          <template v-slot="scope">
            <el-tag type="success" @click="adduser(scope.row)" class="tag_hover">修改</el-tag>
            <el-tag type="danger" @click="delete_user(scope.row)" class="tag_hover">删除</el-tag>
          </template>
        </el-table-column>
      </el-table>
      <div class="block">
        <el-pagination
          @size-change="handleSizeChange"
          @current-change="handleCurrentChange"
          :current-page="1"
          :page-sizes="[10, 20, 30, 40]"
          :page-size="10"
          layout="total, sizes, prev, pager, next, jumper"
          :total="total">
        </el-pagination>
      </div>
    </div>
    <div>
      <el-dialog :title="title" :visible.sync="show">
        <Adduser :adduser_data="adduser_data"></Adduser>
        <div slot="footer" class="dialog-footer">
          <el-button @click="show = false">取 消</el-button>
          <el-button type="primary" @click="submit">确 定</el-button>
        </div>
      </el-dialog>
    </div>
  </div>
</template>
<script>

import Adduser from "@/views/user/user_list/adduser"

export default {
  components: {Adduser},
  filters: {
    statusFilter(status) {
      const statusMap = {
        1: 'success',
        2: 'gray',
        // deleted: 'danger'
      }
      return statusMap[status]
    }
  },
  data() {
    return {
      list: null,
      listLoading: true,
      form: {
        username: "",
        type: ""
      },
      pageSize: 10,
      currentPage: 1,
      total: 0,
      selectId: [],
      adduser_data: {
        username: "",
        type: "",
        phone: "",
        email: "",
        introduce: "",
        password: "",
        avatar:""
      },
      show: false,
      title: "新增用户",
      is_add: true,
      id: ''
    }
  },
  comments: {
    Adduser
  },
  created() {
    this.fetchData()
  },
  methods: {
    fetchData() {
      this.listLoading = true
      this.axios.get("/api/users/list", {
        params: {
          username: this.form.username,
          type: this.form.type,
          pageSize: this.pageSize,
          currentPage: this.currentPage
        }
      }).then(res => {
        console.log(res.data)
        this.list = res.data.data
        this.total = res.data.total
        this.listLoading = false
      }).catch(err => {
        console.log(err)
      })
    },
    search() {
      console.log(this.form)
      this.fetchData()
    },
    handleSizeChange(val) {
      this.pageSize = val
      this.fetchData()
      console.log(`每页 ${val} 条`, val);
    },
    handleCurrentChange(val) {
      this.currentPage = val
      this.fetchData()
      console.log(`当前页: ${val}`);
    },
    adduser(row) {
      this.show = true
      if (row.id) {
        console.log(row.length)

        this.adduser_data.username = row.username
        this.adduser_data.type = row.type
          this.adduser_data.phone = row.phone
        this.adduser_data.email = row.email
        this.adduser_data.introduce = row.introduce
        this.adduser_data.password = row.password
        this.adduser_data.avatar = row.avatar
        this.id = row.id
        this.is_add = false
      }else{
        this.adduser_data={
          username: "",
            type: "",
          phone: "",
          email: "",
          introduce: "",
          password: "",
          avatar:""
        }
      }
    },
    submit() {
      console.log(this.is_add)
      if (this.is_add) {
        this.axios.post("/api/users/register", {
          username: this.adduser_data.username,
          type: this.adduser_data.type,
          phone: this.adduser_data.phone,
          email: this.adduser_data.email,
          introduce: this.adduser_data.introduce,
          password: this.adduser_data.password,
          avatar:this.adduser_data.avatar
        }).then(res => {
          if (res.data.code === 0) {
            this.$message("添加成功")
          } else {
            this.$message("用户名已存在")
          }
        }).catch(err => {
          console.log(err)
        })
      } else {
        console.log(this.adduser_data)
        this.axios.post("/api/users/update", {
          id: this.id,
          username: this.adduser_data.username,
          type: this.adduser_data.type,
          phone: this.adduser_data.phone,
          email: this.adduser_data.email,
          introduce: this.adduser_data.introduce,
          password: this.adduser_data.password,
          avatar:this.adduser_data.avatar
        }).then(res => {
          if (res.data.code === 0) {
            this.$message("修改成功")
          } else {
            this.$message("用户名重复")
          }
        }).catch(err => {
          console.log(err)
        })
      }
      this.fetchData()
      this.show = false
      this.adduser_data = {
        username: "",
        type: "",
        phone: "",
        email: "",
        introduce: "",
        password: "",
      }
    },
    delete_user(row) {
      let ids = []
      this.$confirm('是否删除?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        if (row.id) {
          console.log(row)
          ids.push(row.id)
        } else if (this.selectId.length > 0) {
          ids = this.selectId
        } else {
          this.$message("请选择用户")
        }
        this.axios.post("/api/users/delete", {"ids": ids}).then(res => {
          this.$message("删除成功")
          this.fetchData()

        }).catch(err => {
          console.log(err)
        })
      })

    },
    handleSelectionChange(val) {
      this.selectId = []
      val.forEach(e => {
        this.selectId.push(e.id)
      })
    }
  }
}
</script>
<style scoped>

</style>
