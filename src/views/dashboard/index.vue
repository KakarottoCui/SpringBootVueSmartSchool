<template>
  <div class="app-container">
    <el-row :gutter="20">
      <el-col :span="16">
        <el-card class="box">
          <div slot="header" class="clearfix">
            <span>项目信息</span>
            <el-button style="float: right; padding: 3px 0; font-size: 12px" type="text">系统版本1.0</el-button>
          </div>

          <div style="line-height: 32px;">

            <div class="title" style="padding-top: 0px">系统介绍</div>
            考试系统是一款基于SpringBoot+Vue开发的智慧校园管理系统。包含在线考试、用户体系、错题训练、考试规则、智能算分等功能，流程通畅。

            <div class="title">技术栈</div>
            <div class="tags">
              <el-tag>SpringBoot</el-tag>
              <el-tag>Shiro</el-tag>
              <el-tag>MyBatis Plus</el-tag>
              <el-tag>Redis</el-tag>
              <el-tag>JWT</el-tag>
              <el-tag>Vue</el-tag>
              <el-tag>Vue2</el-tag>
              <el-tag>ElementUI</el-tag>
            </div>

            <div class="title">功能</div>
            <div>
              <div>系统完善：完善的前后端分离架构、规范的接口文档</div>
              <div>权限控制：基于Shiro和JWT开发的权限控制功能。</div>
              <div>基础功能：系统配置、用户管理、部门管理、角色管理等。</div>
              <div>题库管理：支持单选题、多选题、判断题、支持试题批量导入导出</div>
              <div>考试权限：支持指定考试范围：完全公开、指定某些部门人员考试</div>
              <div>在线考试：学员在线考试、查看分数、考试错题训练。</div>
            </div>

          </div>

        </el-card>
      </el-col>

    </el-row>
  </div>
</template>

<script>
import { mapGetters } from 'vuex'
import config from '../../../package.json'

export default {
  name: 'Dashboard',
  data() {
    return {
      currentRole: 'adminDashboard',
      version: config.version
    }
  },
  computed: {
    ...mapGetters([
      'roles',
      'siteData'
    ])
  },
  created() {

  }
}
</script>

<style scoped>
.box{
  height: calc(100vh - 120px);
  font-size: 14px;
  line-height: 28px;
}
.title{
  font-size: 18px;
  font-weight: 700;
  padding-bottom: 10px;
  padding-top: 20px;
  margin-bottom: 10px;
  border-bottom: #eee 1px dotted;
}

.tags span{
  margin-right: 10px;
}

.box a{
  color: #20a0ff;
}

.box a:hover{
  color: #ff0000;
}
</style>
