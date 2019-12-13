<template>
  <div class="application">
    <!-- 应用管理 -->
    <div class="title">应用管理</div>
    <!-- 内容部分 -->
    <div class="content">
      <!-- 搜索表单 -->
      <div class="search-form">
        <el-form ref="searchForm" :model="formItem" label-width="100px">
          <el-row :gutter="24">
            <el-col :xs="24" :sm="5" :offset="1">
              <el-form-item label="名称">
                <el-input class="ipt" v-model="formItem.appNameZh"></el-input>
              </el-form-item>
            </el-col>
            <el-col :xs="24" :sm="5" :offset="4">
              <el-form-item label="创建日期" prop="createTime">
                <el-date-picker
                  class="ipt"
                  type="date"
                  format="yyyy-MM-dd"
                  placeholder="选择日期"
                  v-model="formItem.createTime"
                ></el-date-picker>
              </el-form-item>
            </el-col>
          </el-row>
          <div class="btns">
            <el-form-item>
              <el-button class="btn" type="primary" @click="search()">确认</el-button>
              <el-button class="btn btn_add" @click="showDialog(null)">新增应用</el-button>
            </el-form-item>
          </div>
        </el-form>
      </div>
      <!-- 表格部分 -->
      <div class="list">
        <el-table :data="datalist" style="width: 100%;text-align:center">
          <el-table-column label="名称" width="280" prop="appNameZh">
            <template slot-scope="scope">
              <span style="margin-left: 10px">{{ scope.row.appNameZh }}</span>
            </template>
          </el-table-column>
          <el-table-column label="管理员" width="280" prop="userName">
            <template slot-scope="scope">
              <span style="margin-left: 10px">{{ scope.row.userName }}</span>
            </template>
          </el-table-column>
          <el-table-column label="地址" width="350" prop="appUrl" >
            <template slot-scope="scope">
             <el-popover width="200" trigger='hover' placement="right-start" :content="scope.row.appUrl">
                <span style="margin-left:10px" slot="reference">{{scope.row.appUrl}}</span>
             </el-popover>
              <!-- <span style="margin-left: 10px">{{ scope.row.appUrl }}</span> -->
            </template>
          </el-table-column>
          <el-table-column label="创建日期" width="280" prop="createTime">
            <template slot-scope="scope">
              <span style="margin-left: 10px">{{ scope.row.createTime }}</span>
            </template>
          </el-table-column>
          <el-table-column label="操作" width="280">
            <template slot-scope="scope">
              <svg-icon icon-class="alter" class-name="alter" @on-click="showDialog(scope.row)"></svg-icon>
            </template>
          </el-table-column>
        </el-table>
        <div class="page">
          <el-pagination
            background
            :total="total"
            layout="total, prev, pager, next,jumper"
            :current-page.sync="currentPage"
            @current-change="changePage"
            :page-size="7"
          ></el-pagination>
        </div>
      </div>
    </div>
    <el-dialog :visible.sync="modalVisible" :title="title" class="dialog app-dialog">
      <el-form ref="appform" :model="editItem" :rules="rules" label-width="100px">
        <el-row :gutter="24">
          <el-col :xs="24" :sm="5" :offset="1">
            <el-form-item label="应用名称" prop="appNameZh">
              <el-input class="ipt" v-model="editItem.appNameZh"></el-input>
            </el-form-item>
          </el-col>
          <el-col :xs="24" :sm="5" :offset="5">
            <el-form-item label="管理员" prop="userName">
              <el-input class="ipt" v-model="editItem.userName"></el-input>
            </el-form-item>
          </el-col>
        </el-row>
        <el-row>
          <el-col :offset="1" :sm="8">
            <el-form-item label="应用地址" prop="appUrl">
              <el-input class="ipt ipt-addr" v-model="editItem.appUrl"></el-input>
            </el-form-item>
          </el-col>
        </el-row>
        <div class="btns">
          <el-form-item style="margin-left:0px">
            <!-- <el-button class="btn" type="primary" @click="confirmDialog">确认</el-button> -->
            <el-button class="btn" type="primary" @click="dialogForm('appform')">确认</el-button>
          </el-form-item>
        </div>
      </el-form>
    </el-dialog>
  </div>
</template>
<script>
const name = 'applications'
export default {
  name: 'Applications',
  data() {
    //自定义验证表单元素的方法
    // var validateUserName = (rule, value, callback) => {
    //   if (value == this.editItem.userName) {
    //     callback(new Error('管理员名称已存在'))
    //   } else {
    //     callback()
    //   }
    // }
    return {
      option: 'save',
      modalVisible: false,
      /* 当前页 */
      currentPage: 1,
      total: 0,
      /* 查询app数据列表 */
      datalist: [],
      formItem: {
        appNameZh: '',
        createTime: '',
        startPage: 1
      },
      editItem: {
        appNameZh: '',
        appUrl: '',
        userName: ''
      },
      title: '',
      rules: {
        appNameZh: [
          { required: true, message: '请输入app名称', trigger: 'blur' },
          { min: 2, max: 10, message: '长度在 2 到 10 个字符', trigger: 'blur' }
        ],
        userName: [
          { required: true, message: '请输入管理员名称', trigger: 'blur' },
          { min: 2, max: 6, message: '长度在 2到 6 个字符', trigger: 'blur' }
          // { validator: validateUserName, trigger: 'blur' }
        ],
        appUrl: [
          { required: true, message: '请输入你的url地址', trigger: 'blur' },
          { type: 'url', message: '请输入正确的url地址', trigger: 'blur' },
          { min: 10, max: 100, message: '长度在 10 到 100 个字符', trigger: 'blur' }
        ]
      }
    }
  },
  mounted() {
    this.currentPage = 1
    this.search()
  },
  methods: {
    initEditItem() {
      this.editItem = {
        appNameZh: '',
        appUrl: '',
        userName: '',
        createTime: '',
        startPage: 1
      }
    },
    showDialog(data) {
      this.modalVisible = true
      this.title = data ? '修改应用' : '增加应用'
      this.option = data ? 'edit' : 'save'
      this.editItem = { ...data } || {
        appNameZh: '',
        appUrl: '',
        userName: ''
      }
    },
    dialogForm(formName) {
      if (this.option === 'save') {
        this.create(formName)
      } else {
        this.edit(formName)
      }
    },

    create(formName) {
      this.$refs[formName].validate(valid => {
        if (valid) {
          this.$http.post('/user/searchUserName', { appId: sessionStorage.getItem('appId'), userName: this.editItem.userName }).then(response => {
            // console.log('userName', response.data)
            if (response.data.data) {
              this.$message({
                type: 'warning',
                showClose: true,
                message: '管理员名称已存在!'
              })
              return false
            } else {
              this.$http.post('/app/save', this.editItem).then(response => {
                if (response.data.code == '200') {
                  this.$message({
                    type: 'success',
                    message: '添加应用成功!',
                    showClose: true
                  })
                  this.search()
                  this.initEditItem()
                  this.modalVisible = false
                }
              })
            }
          })
        }
      })
    },
    edit(formName) {
      this.$refs[formName].validate(valid => {
        if (valid) {
          this.$http.post('/app/edit', this.editItem).then(response => {
            // console.log('edit',response)
            if (response.data.code == '200') {
              this.$message({
                showClose: true,
                message: '修改应用成功',
                type: 'success'
              })
              this.search()
              this.modalVisible = false
              this.initEditItem()
            }
          })
        }
      })
    },
    //分页
    changePage(page) {
      this.currentPage = page
      this.search()
    },
    //搜索
    search() {
      if (this.formItem.createTime !== '') {
        this.formItem.createTime = new Date(this.formItem.createTime).toLocaleDateString()
      }
      if (this.formItem.appNameZh || this.formItem.createTime) {
        if (this.currentPage) {
          this.formItem.startPage = this.currentPage
        } else {
          this.formItem.startPage = 1
        }
      } else {
        this.formItem.startPage = this.currentPage
      }
      this.$http.post('/app/search', { ...this.formItem }).then(response => {
        if (response.data && response.data.data) {
          console.log('total',response.data)
          let respData = response.data.data
          this.total = respData.total
          this.datalist = respData.list
          this.initEditItem()
        }
      })
    }
  }
}
</script>
<style lang="scss" scoped>
.application {
  width: 100%;
  height: 100%;
  overflow: hidden;
  position: relative;

  .search-form {
    padding-top: 30px;
    .ipt {
      width: 252px;
      height: 40px;
      margin-right: 20%;
    }
    .btns {
      text-align: center;
      margin: 50px 0 40px 0;
      .btn {
        margin-left: 80px;
      }
    }
  }
}
.app-dialog {
  .ipt {
    width: 252px;
    height: 40px;
  }
  .ipt-addr {
    width: 645px;
  }
}
</style>
