<template>
  <div class="department">
    <div class="title">部门管理</div>
    <div class="content">
      <div class="search-form">
         <el-form ref="searchForm" label-width="100px">
          <el-row :gutter="24">
            <el-col :xs="24" :sm="5" :offset="1">
              <el-form-item label="部门名称" prop="searchName">
                <el-input class="ipt_name form-input" placeholder="全部部门" v-model="searchName" :maxlength="maxlength" @keyup.enter.native="search()"/>
              </el-form-item>
            </el-col>
            <el-col :xs="24" :sm="5" :offset="4">
              <el-form-item label="状态" prop="searchState">
                <el-select class="ipt_state" placeholder="全部状态" v-model="searchState" clearable @keyup.enter.native="search()">
                  <el-option v-for="(item, index) in state" :key="index" :value="item.label">{{ item.label }}</el-option>
                </el-select>
              </el-form-item>
            </el-col>
          </el-row>
          <div class="btns">
            <el-form-item>
              <el-button class="btn" type="primary" @click="search()">确认</el-button>
              <el-button class="btn btn_add" @click="showModal(null)">新增部门</el-button>
            </el-form-item>
          </div>
        </el-form>
      </div>
      <div class="list">
        <el-table ref="table" :data="griddata" row-key="id" max-height="450" :tree-props="{children: 'children', hasChildren: 'hasChildren'}" lazy :load="load">
          <el-table-column prop="name" label="部门"></el-table-column>
          <el-table-column prop="code" label="编码"></el-table-column>
          <el-table-column label="状态">
            <template slot-scope="scope">
              <span v-if="scope.row.isEnable == 1" style="color: #69C638">是</span>
              <span v-else style="color: #FF5622">否</span>
            </template>
          </el-table-column>
          <el-table-column label="操作">
            <template slot-scope="scope">
              <svg-icon icon-class="alter" class-name="alter" @on-click="showModal(scope.row)"></svg-icon>
              <svg-icon icon-class="delete" class-name="delete" @on-click="remove(scope.row)"></svg-icon>
            </template>
          </el-table-column>
        </el-table>
      </div>
      <el-dialog class="dialog app-dialog" :visible.sync="modalVisible" :title="title" footer-hide>
        <el-form ref="searchForm" :model="editdata" label-width="100px">
          <el-row :gutter="24">
            <el-col :xs="24" :sm="5">
              <el-form-item label="部门名称" prop="name">
                <el-input v-model="editdata.name" class="ipt" autocomplete="off" :maxlength="maxlength"/>
              </el-form-item>
            </el-col>
            <el-col :xs="24" :sm="5" :offset="7">
              <el-form-item label="父级部门" prop="parentId">
                <el-cascader :show-all-levels="false" class="ipt-cascader" :options="parentDepartment" v-model="editdata.parentId" :props="{ checkStrictly: true }" clearable></el-cascader>
              </el-form-item>
            </el-col>
          </el-row>
          <el-row :gutter="24">
            <el-col :xs="24" :sm="5">
              <el-form-item label="编码" prop="code">
                <el-input v-model="editdata.code" class="ipt" autocomplete="off" :maxlength="maxlength"/>
              </el-form-item>
            </el-col>
            <el-col :xs="24" :sm="5" :offset="7">
              <el-form-item label="状态启用" prop="isEnable">
                <template>
                  <el-radio-group type="button" v-model="editdata.isEnable">
                    <el-radio :label="1" value="是">是</el-radio>
                    <el-radio :label="0" value="否">否</el-radio>
                  </el-radio-group>
                </template>
              </el-form-item>
            </el-col>
          </el-row>
          <div class="btns">
            <el-form-item style="margin-left:0px">
              <el-button class="btn" type="primary" @click="confirmModal">确认</el-button>
            </el-form-item>
          </div>
        </el-form>
      </el-dialog>
    </div>
  </div>
</template>
<script>
const reg = /^[0-9]*$/
export default {
  name: 'Department',
  data() {
    return {
      maxlength: 30,          //输入框的最大长度
      searchName: '',         //搜索时输入的名字
      searchState: '',        //搜索时输入的状态
      state: [
        {
          value: '1',
          label: '启用'
        },
        {
          value: '0',
          label: '禁用'
        }
      ],                      //搜索状态的选项
      total: 0,               //总页数
      currentPage: 1,         //当前页
      modalVisible: false,    //模态框是否开启状态
      options: true,          //判断是添加还是修改操作
      title: '',              //判断是添加还是修改模态框
      editdata: {},           //修改或添加传入的数据
      parentDepartment: [],   //级联内的数据
      griddata: [{}]          //嵌套表格的数据
    }
  },
  mounted: function() {
    this.listTree()
  },
  methods: {
    listTree() {
      this.parentDepartment = [{ label: '', value: '', children: [] }]
      this.$http.post('/department/listTree', { appId: sessionStorage.getItem('appId') }).then(response => {
        if (response.data && response.data.data) {
          this.griddata = response.data.data.children
          this.griddata.forEach(fun1 => {
            fun1.parentName = response.data.data.name
            this.$set(fun1, 'id', fun1.pid)
            this.$set(fun1, 'hasChildren', true)
            fun1.children.forEach(fun2 => {
              fun2.parentName = fun1.name
              this.$set(fun2, 'id', fun2.pid)
            })
          })
          let listTreeData = response.data.data
          this.parentDepartment[0].label = listTreeData.name
          this.parentDepartment[0].value = listTreeData.pid
          let resourceslist = Object.values(response.data.data.children)
          resourceslist.forEach(cascadervalue => {
            this.parentDepartment[0].children.push({ label: cascadervalue.name, value: cascadervalue.pid })
          })
        }
      })
    },
    showModal(data) {
      this.modalVisible = true
      this.title = data ? '修改部门' : '增加部门'
      this.options = data ? false : true
      if (data !== null) {
        data.appId = sessionStorage.getItem('appId')
        this.editdata = { ...data }
      } else {
        this.editdata = {
          name: '',
          parentName: '',
          code: '',
          isEnable: 0,
          parentId: ''
        }
      }
    },
    confirmModal() {
      if (this.options === true) {
        this.create()
      } else {
        this.edit()
      }
    },
    create() {
      var result = reg.test(this.editdata.code)
      if (!this.editdata.name || !this.editdata.code || this.editdata.parentId === undefined || this.editdata.isEnable === undefined) {
        this.$message({
          message: '部门名称，父级部门，编码，启用状态均不能为空',
          type: 'warning'
        })
      } else if (!result) {
        this.$message({
          message: '你输入的编码不正确,编码必须由数字组成',
          type: 'warning'
        })
      } else {
        if (this.editdata.parentId.length == 0) {
          this.$message({
            message: '部门名称，父级部门，编码，启用状态均不能为空',
            type: 'warning'
          })
        } else {
          this.editdata.parentId = this.editdata.parentId[this.editdata.parentId.length - 1]
          this.$http
            .post('/department/save', {
              appId: sessionStorage.getItem('appId'),
              name: this.editdata.name,
              parentId: this.editdata.parentId,
              isEnable: this.editdata.isEnable,
              code: this.editdata.code
            })
            .then(response => {
              this.listTree()
              this.editdata = { name: '', parentName: '', code: '', isEnable: '', parentId: '' }
              this.modalVisible = false
              this.$message({
                message: '添加成功',
                type: 'success'
              })
            })
        }
      }
    },
    edit() {
      var result = reg.test(this.editdata.code)
      if (!this.editdata.name || !this.editdata.parentId || !this.editdata.code || this.editdata.isEnable === undefined) {
        this.$message({
          message: '部门名称，父级部门，编码，启用状态均不能为空',
          type: 'warning'
        })
      } else {
        if (this.editdata.parentId.length <= 2) {
          this.editdata.parentId = this.editdata.parentId[this.editdata.parentId.length - 1]
        } else if (!result) {
          this.$message({
            message: '你输入的编码不正确,编码必须由数字组成',
            type: 'warning'
          })
        } else {
          this.editdata.parentId = this.editdata.parentId
          this.editdata.appId = sessionStorage.getItem('appId')
          this.$http.post('/department/edit', this.editdata).then(response => {
            this.listTree()
            this.editdata = { name: '', parentName: '', code: '', isEnable: '', parentId: '' }
            this.modalVisible = false
            this.$message({
              message: '修改成功',
              type: 'success'
            })
          })
        }
      }
    },
    remove(index) {
       this.$confirm('此操作将永久删除该资源, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
        }).then(() => {
          this.$http.post('/department/delete', { pid: index.pid, appId: sessionStorage.getItem('appId') }).then(response => {
            this.listTree()
            this.$message({
              type: 'success',
              message: '删除成功!'
            });
          })
        }).catch(() => {
          this.$message({
            type: 'info',
            message: '已取消删除'
          });          
        })
    },
    load(tree, treeNode, resolve) {
      resolve(tree.children)
    }
  }
}
</script>
<style lang="scss" scoped>
.department {
  width: 100%;
  height: 100%;
  overflow: hidden;
  position: relative;
  .search-form {
    padding-top: 30px;
    .ipt_name {
      width: 252px;
      height: 40px;
      margin-right: 20%;
    }
    .ipt_state {
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
  .list {
    ::-webkit-scrollbar-track {
      background-color: #f1f1f1;
    }
    ::-webkit-scrollbar-thumb {
      background: linear-gradient(#b6e6ff, #409eff);
      border-radius: 5px;
    }
    ::-webkit-scrollbar {
      width: 5px;
    }
  }
}
.app-dialog {
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
</style>