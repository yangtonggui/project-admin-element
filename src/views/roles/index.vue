<template>
  <!--  角色管理 -->
  <div class="role">
    <div class="title">角色管理</div>
    <div class="content">
      <div class="search-form">
        <el-form ref="searchForm" :model="formItem" label-width="100px">
          <el-row :gutter="24">
            <el-col :xs="24" :sm="5" :offset="1">
              <el-form-item label="名称" prop="nameZh">
                <el-input class="ipt" v-model="formItem.nameZh"></el-input>
              </el-form-item>
            </el-col>
            <el-col :xs="24" :sm="5" :offset="4">
              <el-form-item label="状态" prop="isEnable">
                <el-select
                  class="ipt"
                  placeholder="全部状态"
                  v-model="formItem.isEnable"
                  @keyup.enter.native="search"
                  clearable
                >
                  <el-option
                    v-for="(item,index) in stateList"
                    :value="item.value"
                    :key="index"
                    :label="item.label"
                  ></el-option>
                </el-select>
              </el-form-item>
            </el-col>
          </el-row>
          <div class="btns">
            <el-form-item>
              <el-button class="btn" type="primary" @click="search()">确认</el-button>
              <el-button class="btn btn_add" @click="showDialog(null)">新增角色</el-button>
            </el-form-item>
          </div>
        </el-form>
      </div>
      <!--  表格部分 -->
      <div class="list">
        <el-table :data="datalist" style="width: 100%;text-align:center">
          <el-table-column label="名称" width="280" prop="nameZh">
            <!-- <template slot-scope="scope">
              <span style="margin-left: 10px">{{ scope.row.nameZh }}</span>
            </template> -->
          </el-table-column>
          <el-table-column label="描述" width="280" prop="description">
            <!-- <template slot-scope="scope">
              <span style="margin-left: 10px">{{ scope.row.description }}</span>
            </template> -->
          </el-table-column>
          <el-table-column label="分配资源" width="350" prop="permissionStr">
            <template slot-scope="scope">
              <el-popover
                word-wrap
                width="80"
                trigger="hover"
                placement="right-start"
                :content="scope.row.permissionStr"
              >
                <span style="margin-left: 10px" slot="reference">{{ scope.row.permissionStr }}</span>
              </el-popover>
            </template>
          </el-table-column>
          <el-table-column label="状态" width="280" prop="isEnable">
            <template slot-scope="scope">
              <span v-if="scope.row.isEnable == 1" style="color: #69C638">是</span>
              <span v-else style="color: #FF5622">否</span>
            </template>
          </el-table-column>
          <el-table-column label="操作" width="280">
            <template slot-scope="scope">
              <svg-icon icon-class="alter" class-name="alter" @on-click="showDialog(scope.row)"></svg-icon>
              <svg-icon icon-class="delete" class-name="delete" @on-click="remove(scope.row)"></svg-icon>
            </template>
          </el-table-column>
        </el-table>
        <!-- 分页 -->
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
    <el-dialog :visible.sync="modalVisible" :title="title" class="dialog role-dialog">
      <el-form ref="roleform" :rules="rules" :model="editItem" label-width="100px">
        <el-row :gutter="24">
          <el-col :xs="24" :sm="5" :offset="1">
            <el-form-item label="角色名称" prop="nameZh">
              <el-input class="ipt" v-model="editItem.nameZh"></el-input>
            </el-form-item>
          </el-col>
        </el-row>
        <el-row>
          <el-col :xs="24" :sm="5" :offset="1">
            <el-form-item label="角色描述" prop="description">
              <el-input class="ipt ipt-desc" v-model="editItem.description"></el-input>
            </el-form-item>
          </el-col>
        </el-row>
        <el-row>
          <el-col :offset="1" :sm="5">
            <el-form-item label="分配资源" prop="appUrl">
              <el-select v-model="editItem.permissions" class="ipt ipt-sel" multiple collapse-tags>
                <el-option-group
                  :label="item.nickName"
                  v-for="(item,index) in permissionList"
                  :key="index"
                >
                  <el-option
                    v-for="(child,index) in item.children "
                    :value="child.id"
                    :key="index"
                    :label="child.nickName"
                  ></el-option>
                </el-option-group>
              </el-select>
            </el-form-item>
          </el-col>
          <el-col :xs="24" :sm="5" :offset="5">
            <el-form-item label="状态启用" prop="description">
              <el-radio-group v-model="editItem.isEnable" type="button">
                <el-radio :label="1">是</el-radio>
                <el-radio :label="0">否</el-radio>
              </el-radio-group>
            </el-form-item>
          </el-col>
        </el-row>
        <div class="btns">
          <el-form-item style="margin-left:0px">
            <el-button class="btn" type="primary" @click="dialogForm('roleform')">确认</el-button>
          </el-form-item>
        </div>
      </el-form>
    </el-dialog>
  </div>
</template>
<script>
const name = 'role'
export default {
  name: 'Role',
  data() {
    return {
      modalVisible: false,
      formItem: {
        nameZh: '',
        isEnable: '',
        appId: '',
        startPage: 1
      },
      /* 当前页 */
      currentPage: 1,
      total: 0,
      option: 'save',
      title: '',
      stateList: [
        {
          value: '1',
          label: '启用'
        },
        {
          value: '0',
          label: '关闭'
        }
      ],
      datalist: [],
      editItem: {
        appId: '',
        id: '',
        nameZh: '',
        description: '',
        isEnable: 0,
        permissions: [],
        permissionStr: '',
        startPage: 1
      },
      temp: {},
      permissionList: [],
      rules: {
        nameZh: [
          { required: true, message: '请输入角色名称', trigger: 'blur' },
          { min: 2, max: 10, message: '长度在 2 到 10 个字符', trigger: 'blur' }
        ],
        description: [{ min: 2, max: 100, message: '长度在 2到 100 个字符', trigger: 'blur' }]
      }
    }
  },
  mounted() {
    this.currentPage = 1
    this.search()
    this.selectresources()
  },
  computed: {
    appId() {
      return this.$store.getters.appId
    }
  },
  methods: {
    initEditItem() {
      this.editItem = {
        nameZh: '',
        description: '',
        isEnable: 0,
        permissions: [],
        permissionStr: '',
        startPage: 1
      }
    },
    selectresources() {
      this.$http.post('/permission/list', { appId: this.appId }).then(response => {
        if (response.data && response.data.data) {
          this.permissionList = response.data.data.data.children
          // console.log('资源',response.data.data)
        }
      })
    },
    showDialog(data) {
      this.modalVisible = true
      this.title = data ? '修改角色' : '增加角色'
      this.option = data ? 'edit' : 'save'
      this.editItem.isEnable = 0
      this.editItem = { ...data } || {
        nameZh: '',
        description: '',
        isEnable: 0,
        permissions: [],
        permissionStr: ''
      }
    },
    dialogForm(formName) {
      if (this.option === 'save') {
        this.create(formName)
      } else {
        this.edit(formName)
      }
    },
    //删除
    remove(index) {
      // console.log('index',index)
      this.$confirm('确认删除?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      })
        .then(() => {
          this.$http.post('/role/delete', { appId: this.appId, id: index.id }).then(response => {
            if (response.data.code == '200')
              this.$message({
                type: 'success',
                message: '删除成功!'
              })
            this.search()
          })
        })
        .catch(() => {
          this.$message({
            type: 'info',
            message: '已取消删除'
          })
        })
    },
    create(formName) {
      this.$refs[formName].validate(valid => {
        if (valid) {
          this.$http.post('/role/save', { ...this.editItem, appId: this.appId }).then(response => {
            // console.log(response.data)
            if (response.data.code == '200') {
              this.$message({
                type: 'success',
                message: '创建角色成功',
                showClose: true
              })
              this.search()
              this.initEditItem()
              this.modalVisible = false
            }
          })
        }
      })
    },
    edit(formName) {
      this.$refs[formName].validate(valid => {
        if (valid) {
          Promise.all([
            this.$http.post('/role/edit', { ...this.editItem, appId: this.appId }),
            this.$http.post('/role/allocate', { appId: this.appId, roleId: this.editItem.id, permissions: this.editItem.permissions })
          ]).then(responses => {
            // console.log('edit', responses)
            if (responses[0].data.code == '200' || responses[1].data.code == '200') {
              this.$message({
                type: 'success',
                message: '修改角色成功',
                showClose: true
              })
              this.search()
              this.modalVisible = false
              this.initEditItem()
            }
          })
        }
      })
    },
    //搜索
    search() {
      if (this.formItem.appNameZh || this.formItem.createTime) {
        if (this.currentPage) {
          this.formItem.startPage = this.currentPage
        } else {
          this.formItem.startPage = 1
        }
      } else {
        this.formItem.startPage = this.currentPage
      }
      this.$http
        .post('/role/search', {
          ...this.formItem,
          appId: this.appId
        })
        .then(response => {
          // console.log('search', response.data)
          if (response.data && response.data.data) {
            let respData = response.data.data
            this.total = respData.total
            console.log('角色total',respData)
            this.datalist = respData.list.filter(item => item.roleName.indexOf('_ADMIN') < 0)
            this.datalist.forEach(item => {
              let permissionStr = ''
              item.permissions.forEach((permission, index) => {
                // // console.log('index',index)
                // let icon = index % 2  === 0 ? '\t' : '\n\r'
                // // console.log('1\n\r2')
                // permissionStr += permission.nickName + icon
                 permissionStr += permission.nickName + (index % 2 === 0 ? '\r\n' : '\t')
              })
              item['permissionStr'] = permissionStr
            })
          }
        })
    },
    //分页
    changePage(page) {
      this.currentPage = page

      this.search()
    }
  }
}
</script>

<style lang="scss" scoped>
.role-dialog {
  .ipt {
    width: 252px;
    height: 40px;
  }
  .ipt-desc {
    width: 700px;
  }
  .ipt-sel {
    display: inline-block;
    margin-right: 10%;
  }
}
.role {
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
</style>