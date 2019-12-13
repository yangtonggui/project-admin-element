<template>
  <div class="person">
    <div class="title">用户管理</div>
    <div class="content">
      <div class="search-form">
        <label>用户名</label>
        <!-- <i-input class="ipt_name" v-model="searchData.userName" placeholder="请输入" :maxlength="15" /> -->
        <el-input
          placeholder="请输入内容"
          v-model="searchData.userName"
          clearable
          :maxlength="15"
          class="ipt_name"
        ></el-input>
        <label>角色</label>
        <!-- <Select v-model="searchData.roleId" clearable class="ipt_name">
      <Option v-for="item in rolesList" :value="item.roleId" :key="item.roleId">{{ item.nameZh }}</Option>
        </Select>-->

        <el-select v-model="searchData.roleId" clearable placeholder="请选择">
          <el-option
            v-for="item in rolesList"
            :key="item.roleId"
            :label="item.nameZh"
            :value="item.roleId"
          ></el-option>
        </el-select>

        <!-- <a href="#" @click="getdata">查看数据</a> -->
        <label style="margin-left:15%">部门</label>
        <!-- <Cascader :data="departmentData" change-on-select v-model="searchData.deptIds" class="ipt_name inputstyle"></Cascader> -->
        <el-cascader
          v-model="searchData.deptIds"
          :options="departmentData"
          change-on-select="true"
          class="ipt_name inputstyle"
        ></el-cascader>

        <div class="btns">
          <el-button class="btn" type="primary" @click="search">确认</el-button>
          <el-button class="btn btn_add" @click="showDialog(null)">新增用户</el-button>
        </div>
      </div>

      <!-- 表格列表 -->
      <!-- <div class="list">
        <Table :columns="userstitle" :data="datalist">
          <template slot-scope="{ row }" slot="userName">
            <span class="rowhead">{{ row.userName }}</span>
          </template>
          <template slot-scope="{ row }" slot="action">
            <svg-icon icon-class="alter" class-name="alter listBtn" @on-click="showDialog(row)"></svg-icon>
            <svg-icon icon-class="delete" class-name="delete listBtn" @on-click="remove(row)"></svg-icon>
          </template>
        </Table>
        <div class="page">
          <Page :total="total" show-total :page-size="7" show-elevator @on-change="changePage" />
        </div>
      </div>-->

      <div class="list">
        <el-table :data="datalist" style="width: 100%;text-align:center">
          <el-table-column label="用户名" width="280" prop="userName">
            <template slot-scope="scope">
              <span style="margin-left: 10px">{{ scope.row.userName }}</span>
            </template>
          </el-table-column>
          <el-table-column label="姓名" prop="userRealName">
            <template slot-scope="scope">
              <span style="margin-left: 10px">{{ scope.row.userRealName }}</span>
            </template>
          </el-table-column>
          <el-table-column label="部门" prop="departmentName">
            <template slot-scope="scope">
              <span style="margin-left: 10px">{{ scope.row.departmentName }}</span>
            </template>
          </el-table-column>
          <el-table-column label="角色" prop="roles">
            <template slot-scope="scope">
              <span style="margin-left: 10px">{{ scope.row.roleStr }}</span>
            </template>
          </el-table-column>
          <el-table-column label="最后登录时间" prop="lastLoginTime">
            <template slot-scope="scope">
              <span style="margin-left: 10px">{{ scope.row.lastLoginTime }}</span>
            </template>
          </el-table-column>
          <el-table-column label="状态" prop="isEnable">
            <template slot-scope="scope">
              <span v-if="scope.row.isEnable == 1" style="color: #69C638">是</span>
              <span v-else style="color: #FF5622">否</span>
              <!-- <span style="margin-left: 10px">{{ scope.row.isEnable }}</span> -->
            </template>
          </el-table-column>
          <el-table-column label="操作">
            <template slot-scope="scope">
              <svg-icon icon-class="alter" class-name="alter" @on-click="showDialog(scope.row)"></svg-icon>
              <svg-icon
                icon-class="delete"
                class-name="delete listBtn"
                @on-click="remove(scope.row)"
              ></svg-icon>
            </template>
          </el-table-column>
        </el-table>
        <div class="page">
          <el-pagination
            background
            @current-change="changePage"
            :page-size="7"
            layout="total, prev, pager, next, jumper"
            :total="total"
          ></el-pagination>
        </div>
      </div>
    </div>
    <!-- 模态框部分 -->
    <!-- <Modal v-model="dialogVisible" :title="title" class="modal person-modal" footer-hide>
      <p>
        <label class="form-label">姓 &nbsp;&nbsp;&nbsp;&nbsp;&nbsp;名</label>
        <i-input v-model="editItem.userRealName" placeholder="请输入" class="ipt" :maxlength="5" />
        <label class="form-label">所属部门</label>
        <Cascader :data="departmentData" v-model="editItem.deptIds" change-on-select class="ipt" :render-format="formatDept"></Cascader>
      </p>
      <p>
        <label class="form-label">用 户 名</label>
        <i-input @on-blur="userRepeat" v-model="editItem.userName" placeholder="请输入" class="ipt ipt_name" ref="ipt_userName" :maxlength="15" />
        <label class="form-label">用户密码</label>
        <Input type="password" v-model="editItem.userPassword" placeholder="****" class="ipt ipt_name" />
      </p>
      <p>
        <label class="form-label">角色授权</label>
        <i-select v-model="editItem.roleIds" clearable filterable  multiple class="ipt ipt_name">
          <Option v-for="item in rolesList" :value="item.roleId" :key="item.roleId">{{ item.nameZh }}</Option>
        </i-select>
        <span style="margin:40px" class="form-label">状态启用</span>
        <template>
          <Radio-group class="form-label form-button" type="button" v-model="editItem.isEnable">
            <Radio label="1">是</Radio>
            <Radio class="btn-lf" label="0">否</Radio>
          </Radio-group>
        </template>
      </p>
      <div class="btns">
        <i-button class="btn" type="primary" @click="dialogForm">确认</i-button>
      </div>
    </Modal>-->

    <el-dialog class="dialog user-dialog" :title="title" :visible.sync="dialogVisible" width="50%">
      <el-form ref="searchForm" :model="editItem" label-width="100px">
        <el-row :gutter="24">
          <el-col :xs="24" :sm="5">
            <el-form-item label="姓名" prop="userRealName">
              <el-input
                v-model="editItem.userRealName"
                placeholder="请输入"
                class="ipt"
                :maxlength="5"
              ></el-input>
            </el-form-item>
          </el-col>
          <el-col :xs="24" :sm="5" :offset="7">
            <el-form-item label="所属部门" prop="deptIds">
              <el-cascader
                class="ipt-cascader"
                :options="departmentData"
                v-model="editItem.deptIds"
                change-on-select="true"
                clearable
              ></el-cascader>
            </el-form-item>
          </el-col>
        </el-row>

        <el-row :gutter="24">
          <el-col :xs="12" :sm="5">
            <el-form-item label="用户名" prop="userName">
              <el-input
                @blur="userRepeat"
                v-model="editItem.userName"
                class="ipt"
                autocomplete="off"
                :maxlength="maxlength"
              />
            </el-form-item>
          </el-col>
          <el-col :xs="24" :sm="5" :offset="7">
            <el-form-item label="用户密码" prop="userPassword">
              <el-input
                :placeholder="placeholder"
                type="password"
                v-model="editItem.userPassword"
                class="ipt"
                autocomplete="off"
                :maxlength="maxlength"
              />
            </el-form-item>
          </el-col>
        </el-row>

        <el-row :gutter="24">
          <el-col :xs="12" :sm="5">
            <el-form-item label="角色授权" prop="userName">
              <el-select
                v-model="editItem.roleIds"
                clearable
                placeholder="请选择"
                multiple
                class="ipt"
              >
                <el-option
                  v-for="item in rolesList"
                  :key="item.roleId"
                  :label="item.nameZh"
                  :maxlength="maxlength"
                  :value="item.roleId"
                ></el-option>
              </el-select>
            </el-form-item>
          </el-col>

          <el-col :xs="24" :sm="5" :offset="7">
            <el-form-item label="状态启用" prop="isEnable">
              <template>
                <el-radio-group type="button" v-model="editItem.isEnable">
                  <el-radio :label="1">是</el-radio>
                  <el-radio :label="0">否</el-radio>
                </el-radio-group>
              </template>
            </el-form-item>
          </el-col>
        </el-row>

        <div class="btns">
          <el-form-item style="margin-left:0px">
            <el-button class="btn" type="primary" @click="dialogForm">确认</el-button>
          </el-form-item>
        </div>
      </el-form>
    </el-dialog>
  </div>
</template>
<script>
export default {
  name: 'User',
  data() {
    return {
      placeholder: ' ',
      maxlength: 30,
      departmentData: [],
      title: '',
      dialogVisible: false,
      // userstitle: [
      //   {
      //     title: '用户名',
      //     slot: 'userName',
      //     width: 200,
      //     height: '200px'
      //   },
      //   {
      //     title: '姓名',
      //     key: 'userRealName'
      //   },
      //   {
      //     title: '部门',
      //     key: 'departmentName'
      //   },
      //   {
      //     title: '角色',
      //     key: 'roleStr'
      //   },
      //   {
      //     title: '最后登录时间',
      //     key: 'lastLoginTime'
      //   },
      //   {
      //     title: '状态',
      //     key: 'isEnable',
      //     render: (h, params) => {
      //       const row = params.row
      //       const text = row.isEnable == 1 ? '是' : '否'
      //       if (row.isEnable) {
      //         return h('span', { style: { color: '#69C638' } }, text)
      //       } else {
      //         return h('span', { style: { color: '#FF5622' } }, text)
      //       }
      //     }
      //   },
      //   {
      //     title: '操作',
      //     slot: 'action',
      //     width: 150,
      //     align: 'center'
      //   }
      // ],
      total: 0,
      rolesList: [],
      departmentList: [],
      datalist: [], //获取数据列表
      editItem: {
        id: '',
        userName: '', //姓名
        userRealName: '', //用户名
        userPassword: '', //用户密码
        isEnable: '', //用户是否可以启用
        deptIds: [],
        roleIds: []
      },
      option: 'save', //是新增或修改
      searchData: {
        currentPage: 1, //当前页
        deptIds: [],
        userName: '',
        roleId: ''
      },

      isUserRepeat: false
    }
  },
  computed: {
    appId() {
      return this.$store.getters.appId
    }
  },
  mounted() {
    this.searchRoles()
    this.searcDepartment()
    this.search()
  },
  methods: {
    formatDept(lable) {
      // console.log(9999)
      return lable[lable.length - 1] || this.editItem.departmentName
    },
    validateForm() {
      if (!this.editItem.userRealName) {
        this.$message.error('用户名称不可为空')
        return false
      }
      if (!this.editItem.userName) {
        this.$message.error('姓名不可为空')

        return false
      }
      if (this.option === 'save' && !this.editItem.userPassword) {
        this.$message.error('用户密码不可为空')
        return false
      }
      if (!this.editItem.deptIds || !this.editItem.deptIds.length) {
        this.$message.error('部门不可为空')
        return false
      }
      if (!this.editItem.roleIds) {
        this.$message.error('角色授权不可为空')
        return false
      }
      if (!this.editItem.isEnable) {
        this.$message.error('状态启用未选择')
        return false
      }
      return true
    },
    userRepeat() {
      console.log(6666)
      this.$http.post('/user/searchUserName ', { userName: this.editItem.userName, appId: this.appId }).then(response => {
        this.isUserRepeat = response.data.data
        if (this.isUserRepeat) {
          this.$message.error('该用户已存在')
          this.$refs['ipt_userName'].focus()
        }
      })
    },
    //迭代 (函数自调用)
    getChildren(dept) {
      dept['value'] = dept.pid
      dept['label'] = dept.name
      if (dept.children && dept.children.length) {
        dept.children.forEach(child => {
          child['value'] = child.pid
          child['label'] = child.name
          if (child.children && child.children.length) {
            this.getChildren(child)
          }
        })
      }
    },
    searcDepartment() {
      this.departmentData = []
      this.$http.post('/department/listTree', { appId: this.appId }).then(response => {
        if (response.data.code == 200) {
          let dept = response.data.data
          this.getChildren(dept)
          dept['value'] = dept.pid
          dept['label'] = dept.name
          this.departmentData.push(dept)
        }
      })
    },
    searchRoles() {
      this.$http
        .post('/role/list', {
          startPage: '1',
          // size: '1',
          appId: this.appId
        })
        .then(response => {
          if (response.data.code == 200) {
            let list = response.data.data.list
            let rolesList = []
            if (list && list.length) {
              list.forEach(roleItem => {
                rolesList.push({ roleId: roleItem.id, nameZh: roleItem.nameZh })
              })
            }
            this.rolesList = rolesList
          }
        })
    },

    showDialog(data, index) {
      this.placeholder = data ? '****' : ' '
      this.dialogVisible = true
      this.title = data ? '修改用户' : '新增用户'
      this.option = data ? 'edit' : 'save'
      this.editItem = data ? { ...data } : {}

      if (this.editItem.departmentId) {
        this.editItem.deptIds = []
        this.editItem.deptIds.push(this.editItem.departmentId)
      }
      if (this.editItem.roles && this.editItem.roles.length) {
        this.editItem.roleIds = this.editItem.roles.map(item => item.id)
      }

      console.log(this.editItem.departmentId)
      console.log(this.editItem.departmentName)
    },
    //新增或修改
    dialogForm() {
      if (this.isUserRepeat) {
        return
      } else if (this.validateForm()) {
        this.option === 'save' ? this.add() : this.update()
      }
    },
    //分页
    changePage(page) {
      this.searchData.currentPage = page
      this.search()
    },
    //增
    add() {
      this.$http
        .post('/user/save', {
          ...this.editItem,
          appId: this.appId,
          departmentId: this.editItem.deptIds[this.editItem.deptIds.length - 1]
        })
        .then(response => {
          // this.$message.success('新增成功')
          this.dialogVisible = false
          this.search()
        })
    },
    //删
    remove(data, index) {
      this.$confirm('确定删除吗?', {
        type: 'warning',
        // center:true,
        showCancelButton: false
      }).then(() => {
        this.$http.post('/user/delete', { id: data.id, appId: this.appId }).then(response => {
          this.$message.success('删除成功')
          this.search()
        })
      })
    },
    //改
    update(index) {
      this.$http
        .post('/user/edit', {
          ...this.editItem,
          appId: this.appId,
          departmentId: this.editItem.deptIds[this.editItem.deptIds.length - 1]
        })
        .then(response => {
          this.$message.success('修改成功')
          this.dialogVisible = false
          this.search()
        })
    },
    // 查
    search() {
      this.$http
        .post('/user/search', {
          ...this.searchData,
          appId: this.appId,
          startPage: this.searchData.currentPage,
          departmentId: this.searchData.deptIds[this.searchData.deptIds.length - 1] || ''
        })
        .then(response => {
          // console.log(response)
          let respdata = response.data.data
          this.datalist = respdata.list
          this.total = respdata.total
          this.datalist.forEach(item => {
            // item.isEnable=item.isEnable? '是':item.isEnable=item.isEnable==0?'否':''
            let roleStr = ''
            if (item.roles) {
              item.roles.forEach(item => {
                roleStr += item.nameZh + ','
              })
              roleStr = roleStr.substr(0, roleStr.length - 1)
            }
            item['roleStr'] = roleStr
          })
        })
    }
  }
}
</script>
<style lang="scss">
.user-dialog {
  .ipt {
    width: 253px;
    height: 40px;
    margin-right: 160px;
  }
  .ipt-cascader {
    width: 253px;
    height: 40px;
    display: inline-block;
  }
}

.el-table th {
  background-color: #f7f8fa !important;
  //  background-color:red !important
}
.person {
  width: 100%;
  height: 100%;
  overflow: hidden;
  position: relative;
  .search-form {
    padding-top: 30px;
    label {
      font-size: 18px;
      height: 40px;
      margin-right: 10px;
    }

    .ipt_name {
      width: 252px;
      height: 40px;
      margin-right: 13%;
      display: inline-block;
    }
    .inputstyle {
      display: inline-block;
      width: 252px;
    }
    .btns {
      text-align: center;
      margin: 50px 0 40px 0;
      .btn {
        margin-left: 80px;
        height: 40px;
        width: 144px;
      }
    }
  }
  .list {
    .page {
      margin-top: 20px;
      text-align: center;
    }
    .ivu-table th,
    .ivu-table-cell {
      text-align: center;
    }
    .listBtn {
      transform: translate(3px, 0);
    }
  }
}
</style>
<style lang="scss" scoped>
.person-modal {
  .ipt {
    display: inline-block;
    width: 252px;
  }
  label {
    margin: 40px;
  }
  .form-button {
    height: 0;
    transform: translate(-51px, -60px);
  }
  .btn-lf {
    transform: translate(-49px, 0);
  }
}
</style>
