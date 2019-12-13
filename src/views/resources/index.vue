<template>
  <div class="resources">
    <div class="title">资源管理</div>
    <div class="content">
      <div class="header">
        <template>
          <el-dialog class="modal res-modal" :visible.sync="addModalVisible" title="新增资源" footer-hide>
            <el-form ref="searchForm" :model="self" label-width="100px">
              <el-row :gutter="24">
                <el-col :xs="24" :sm="5">
                  <el-form-item label="资源名称" prop="permissionName">
                    <el-input v-model="self.permissionName" class="ipt" autocomplete="off" :maxlength="maxlength"/>
                  </el-form-item>
                </el-col>
                <el-col :xs="24" :sm="5" :offset="7">
                  <el-form-item label="父级名称" prop="nickName">
                   <el-input type="text" :value="self.parentName" disabled class="ipt-disable" />
                  </el-form-item>
                </el-col>
              </el-row>
              <el-row :gutter="24">
                <el-col :xs="24" :sm="5">
                  <el-form-item label="地址" prop="url">
                     <el-input v-model="self.url" class="ipt" autocomplete="off" :maxlength="maxlength"/>
                  </el-form-item>
                </el-col>
                <el-col :xs="24" :sm="5" :offset="7">
                  <el-form-item label="状态启用" prop="permissionIsEnable">
                    <template>
                      <el-radio-group type="button" v-model="self.permissionIsEnable">
                        <el-radio :label="1" value="是">是</el-radio>
                        <el-radio :label="0" value="否">否</el-radio>
                      </el-radio-group>
                    </template>
                  </el-form-item>
                </el-col>
              </el-row>
              <div class="btns">
                <el-form-item style="margin-left:0px">
                  <el-button class="btn" type="primary" @click="create">确认</el-button>
                </el-form-item>
              </div>
            </el-form>
          </el-dialog>
          <el-dialog class="modal res-modal" :visible.sync="editModalVisible" title="修改资源" footer-hide>
            <el-form ref="searchForm" :model="self" label-width="100px">
              <el-row :gutter="24">
                <el-col :xs="24" :sm="5">
                  <el-form-item label="资源名称" prop="permissionName">
                    <el-input v-model="self.permissionName" class="ipt" autocomplete="off" :maxlength="maxlength"/>
                  </el-form-item>
                </el-col>
                <el-col :xs="24" :sm="5" :offset="7">
                  <el-form-item label="父级名称" prop="nickName">
                   <el-cascader :show-all-levels="false" class="ipt-cascader" :options="parentdata" v-model="self.parentId" :props="{ checkStrictly: true }" clearable></el-cascader>
                  </el-form-item>
                </el-col>
              </el-row>
              <el-row :gutter="24">
                <el-col :xs="24" :sm="5">
                  <el-form-item label="地址" prop="url">
                     <el-input v-model="self.url" class="ipt" autocomplete="off" :maxlength="maxlength"/>
                  </el-form-item>
                </el-col>
                <el-col :xs="24" :sm="5" :offset="7">
                  <el-form-item label="状态启用" prop="permissionIsEnable">
                    <template>
                      <el-radio-group type="button" v-model="self.permissionIsEnable">
                        <el-radio :label="1" value="是">是</el-radio>
                        <el-radio :label="0" value="否">否</el-radio>
                      </el-radio-group>
                    </template>
                  </el-form-item>
                </el-col>
              </el-row>
              <div class="btns">
                <el-form-item style="margin-left:0px">
                  <el-button class="btn" type="primary" @click="edit">确认</el-button>
                </el-form-item>
              </div>
            </el-form>
          </el-dialog>
        </template>
      </div>
      <div class="list" style="padding-top: 50px; ">
        <el-table :data="gridData" row-key="id" max-height="500" :tree-props="{children: 'children', hasChildren: 'hasChildren'}" lazy :load="load">
          <el-table-column prop="nickName" label="资源"></el-table-column>
          <el-table-column prop="url" label="地址"></el-table-column>
          <el-table-column label="状态">
            <template slot-scope="scope">
              <span v-if="scope.row.isEnable == 1" style="color: #69C638">是</span>
              <span v-else style="color: #FF5622">否</span>
            </template>
          </el-table-column>
          <el-table-column label="是否为菜单项">
            <template slot-scope="scope">
              <span v-if="scope.row.permissionIsMenu == 1" style="color: #69C638">是</span>
              <span v-else style="color: #FF5622">否</span>
            </template>
          </el-table-column>
          <el-table-column label="操作">
            <template slot-scope="scope">
              <svg-icon v-if="scope.row.children != null" icon-class="add" class-name="add" @on-click="add(scope.row)"></svg-icon>
              <svg-icon v-if="scope.row.url != undefined" icon-class="alter_green" class-name="alter_green" @on-click="update(scope.row)"></svg-icon>
              <svg-icon v-if="scope.row.url != undefined" icon-class="delete_red" class-name="delete_red" @on-click="remove(scope.row)"></svg-icon>
            </template>
          </el-table-column>
        </el-table>
      </div>
    </div>
  </div>
</template>

<script>
const reg = /^$|(\/\w){1,3}/
export default {
  name: 'container',
  data() {
    return {
      maxlength: 30,                //输入框的最大长度
      addModalVisible: false,       //添加模态框的状态
      editModalVisible: false,      //修改模态框的状态                          
      self: {
        appId: sessionStorage.getItem('appId'),
        permissionId: '',
        permissionName: '',
        url: '',
        permissionIsEnable: '',
        permissionIsMenu: '',
        parentId: [],
        parentName:''
      },                            //添加和修改传入的数据
      gridData: [
        {
          nickName: '',
          children: ''
        }
      ],                            //嵌套表格的数据
      parentdata: []                //级联的数据
    }
  },
  mounted() {
    this.selectresources()
  },
  methods: {
    selectresources() {
      this.parentdata = [{ label: '', value: '', children: [] }]
      let appId = sessionStorage.getItem('appId')
      this.$http.post('/permission/list', { appId: appId }).then(response => {
        if (response.data && response.data.data) {
          this.gridData = [{ nickName: '根节点', id: 1, permissionIsMenu: 1,url:undefined, hasChildren: true, isEnable: 1, value: 1, children: [] }]
          this.gridData[0].children = response.data.data.data.children
          this.gridData[0].children.forEach(fun1 => {
            fun1.parentName = response.data.data.data.nickName
            fun1.permissionIsMenu = 1
            this.$set(fun1, 'hasChildren', true)
            fun1.children.forEach(fun2 => {
              fun2.parentName = fun1.nickName
              fun2.permissionIsMenu = 0
            })
          })
          this.parentdata[0].label = '根节点'
          this.parentdata[0].value = '1'
          let resourceslist = Object.values(response.data.data.data.children)
          resourceslist.forEach(cascadervalue => {
            this.parentdata[0].children.push({ label: cascadervalue.nickName, value: cascadervalue.id })
          })
        }
      })
    },
    add(father) {
      this.self = {
        appId: sessionStorage.getItem('appId'),
        permissionId: '',
        permissionName: '',
        url: '',
        permissionIsEnable: '',
        permissionIsMenu: '',
        parentId: [],
        parentName: ''
      }
      this.addModalVisible = true
      this.self.permissionIsMenu = father.permissionIsMenu
      this.self.parentId = father.id
      this.self.parentName = father.nickName
      this.self.permissionIsEnable = 0
    },
    create() {
      var result = reg.test(this.self.url)
      if (this.self.permissionIsEnable === undefined || !this.self.permissionName) {
        this.$message({
          message: '资源名称，父级名称，状态启用均不能为空',
          type: 'warning'
        })
      } else if (!result) {
        this.$message({
          message: '你输入的地址不正确,地址中必须含有/',
          type: 'warning'
        })
      } else {
        this.$http.post('/permission/save', this.self).then(response => {
          this.selectresources()
          this.addModalVisible = false
          this.$message({
            message: '添加成功',
            type: 'success'
          })
        })
      }
    },
    update(self) {
      this.editModalVisible = true
      this.self.permissionName = self.nickName
      this.self.url = self.url
      this.self.permissionId = self.id
      this.self.permissionIsEnable = self.isEnable
      this.self.parentName = self.parentName
      this.self.parentId = self.parentId
    },
    edit() {
      var result = reg.test(this.self.url)
      if (!this.self.permissionName || this.self.permissionIsEnable === undefined || !this.self.parentId ) {
        this.$message({
          message: '资源名称，父级名称，状态启用均不能为空',
          type: 'warning'
        })
      } else if (!result) {
        this.$message({
          message: '你输入的地址不正确,地址中必须含有/',
          type: 'warning'
        })
      } else {
        this.$http.post('/permission/edit', this.self).then(response => {
          this.selectresources()
          this.editModalVisible = false
          this.$message({
            message: '修改成功',
            type: 'success'
          })
        })
      }
    },
    remove(self) {
      let appId = self.appId
      let permissionId = self.id
      this.$confirm('此操作将永久删除该资源, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
        }).then(() => {
          this.$http.post('/permission/delete', { appId: appId, permissionId: permissionId }).then(response => {
            this.selectresources()
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
.resources {
  width: 100%;
  height: 100%;
  overflow: hidden;
  position: relative;
}
.content {
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
.res-modal {
  .ipt {
    width: 253px;
    height: 40px;
    margin-right: 160px;
  }
  .ipt-disable {
    width: 253px;
    height: 40px;
  }
  .ipt-cascader {
    width: 253px;
    height: 40px;
    display: inline-block;
  }
}
</style>
