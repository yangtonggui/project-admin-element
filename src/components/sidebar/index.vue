<template>
  <div class="sidebar">
    <el-row class="tac">
      <el-menu default-active="2" class="el-menu-vertical-demo" @select="gotoPage">
        <el-menu-item
          :index="item.path"
          :name="item.path"
          v-for="item in menuList"
          :key="item.path"
        >
          <i>
            <svg-icon :icon-class="item.path.slice(1)" class-name="icon"></svg-icon>
          </i>
          <span slot="title">{{ item.name }}</span>
        </el-menu-item>
      </el-menu>
    </el-row>
    <br />
  </div>
</template>
<script>
export default {
  data() {
    return {
      selectedName: '',
      menuList: []
    }
  },
  mounted() {
    this.menuList = this.$store.getters.menuList
  },
  methods: {
    gotoPage(name) {
      this.$router.push({
        path: name || ''
      })
    }
  }
}
</script>

<style lang="scss" >
.sidebar {
  .el-menu {
    margin-bottom: 50px;
    font-size: 20px;
    font-weight: 500;
    line-height: 24px;
    color: rgba(100, 105, 116, 1);
    text-align: center;
  }
  .el-menu-item,
  .el-submenu__title {
    font-size: 20px;
    margin-bottom: 50px;
  }

  .el-menu-item:focus, .el-menu-item:hover {
    outline: 0;
     background-color: #ffffff !important;; 
}
}
</style>
<style lang="scss" scoped>
.sidebar {
  padding-top: 10px;
  margin-top: 10px;
  width: 280px;
  background: rgba(255, 255, 255, 1);
  box-shadow: 4px 0px 4px rgba(122, 122, 122, 0.2);
  .icon {
    width: 24px !important;
    height: 24px !important;
    margin-right: 15px;
  }
}
</style>
