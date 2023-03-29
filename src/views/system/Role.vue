/**
 * 系统管理 文章管理
 */
<template>
  <div>
    <!-- 面包屑导航 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item>首页</el-breadcrumb-item>
      <el-breadcrumb-item>文章管理</el-breadcrumb-item>
    </el-breadcrumb>
    <!--列表-->
    <el-table size="small" @selection-change="selectChange" :data="articleData" highlight-current-row v-loading="loading" border element-loading-text="拼命加载中" style="width: 100%;">
      <el-table-column align="center" type="selection" width="50">
      </el-table-column>
      <el-table-column align="center" sortable prop="articleId" label="文章Id" width="80">
      </el-table-column>
      <el-table-column align="center" sortable label="头像" width="80">
        <template slot-scope="scope">
            <el-popover placement="top-start" title="" trigger="hover">
              <img :src="scope.row.avatar" alt="" style="width: 150px;height: 150px">
              <img slot="reference" :src="scope.row.avatar" style="width: 40px;height: 40px">
            </el-popover>
        </template>
      </el-table-column>
      <el-table-column align="center" sortable prop="nick" label="昵称" width="80">
      </el-table-column>
      <el-table-column align="center" sortable prop="content" label="发布内容" width="240">
      </el-table-column>
      <el-table-column align="center" sortable label="发布的图片" width="180">
        <template slot-scope="scope">
            <el-popover placement="top-start" title="" trigger="hover">
              <img :src="scope.row.imagePath" alt="" style="width: 200px;height: 200px">
              <img slot="reference" :src="scope.row.imagePath" style="width: auto;height: 150px">
            </el-popover>
        </template>
      </el-table-column>
      <el-table-column align="center" sortable prop="voteCount" label="点赞数量" width="80">
      </el-table-column>
      <el-table-column align="center" sortable prop="commentCount" label="评论数量" width="120">
      </el-table-column>
      <el-table-column align="center" sortable prop="createTime" label="创建时间" width="120">
      </el-table-column>
      <el-table-column align="center" sortable prop="locationName" label="发布定位" min-width="50">
      </el-table-column>
      
      <el-table-column label="操作" min-width="300">
        <template slot-scope="scope">
          <el-button size="mini" type="success" @click="showComment(scope.$index, scope.row)">展开评论</el-button>
          <el-button size="mini" type="danger" @click="deleteArticle(scope.$index, scope.row)">删除</el-button>
        </template>
      </el-table-column>
    </el-table>
    <!-- 分页组件 -->
    <Pagination v-bind:child-msg="pageparm" @callFather="callFather"></Pagination>
    <!-- 编辑界面 -->
    <el-dialog title="文章评论" :visible.sync="editFormVisible" width="60%" @click='closeDialog("edit")'>
      <el-table size="small" :data="commentList" highlight-current-row v-loading="loading" border element-loading-text="拼命加载中" style="width: 100%;">
        <el-table-column align="center" sortable prop="commentId" label="评论Id" width="100">
      </el-table-column>
      <el-table-column align="center" sortable prop="userId" label="评论者ID" width="100">
      </el-table-column>
      <el-table-column align="center" sortable label="评论者头像" width="100">
        <template slot-scope="scope">
            <el-popover placement="top-start" title="" trigger="hover">
              <img :src="scope.row.avatar" alt="" style="width: 150px;height: 150px">
              <img slot="reference" :src="scope.row.avatar" style="width:auto;height: 40px">
            </el-popover>
        </template>
      </el-table-column>
      <el-table-column align="center" sortable prop="nick" label="评论者昵称" width="100">
      </el-table-column>
      <el-table-column align="center" sortable prop="content" label="评论内容" width="300">
      </el-table-column>
      <el-table-column align="center" sortable prop="createTime" label="评论时间" width="130">
      </el-table-column>
      <el-table-column label="操作" min-width="100">
        <template slot-scope="scope">
          <el-button size="mini" type="danger" @click="deletecomment(scope.$index, scope.row)">删除</el-button>
        </template>
      </el-table-column>
        </el-table>
        <Pagination v-bind:child-msg="commentpageparm" @callFather="callcommentFather"></Pagination>
    </el-dialog>
  </div>
</template>

<script>
// 导入请求方法
import {
  articleList,
  articleDelete,
  getCommentList,
  commentDelete
} from '../../api/userMG'
import Pagination from '../../components/Pagination'
export default {
  data() {
    return {
      nshow: true, //switch开启
      fshow: false, //switch关闭
      loading: false, //是显示加载
      title: '添加用户',
      editFormVisible: false, //控制编辑页面显示与隐藏
      dataAccessshow: false, //控制数据权限显示与隐藏
      unitAccessshow: false, //控制所属单位隐藏与显示
      // 选择数据
      selectdata: [],
      // rules表单验证
      rules: {
        userId: [
          { required: true, message: '请输入用户名', trigger: 'blur' }
        ],
        userRealName: [
          { required: true, message: '请输入姓名', trigger: 'blur' }
        ],
        roleId: [{ required: true, message: '请选择角色', trigger: 'blur' }],
        userMobile: [
          { required: true, message: '请输入手机号', trigger: 'blur' },
          {
            pattern: /^1(3\d|47|5((?!4)\d)|7(0|1|[6-8])|8\d)\d{8,8}$/,
            required: true,
            message: '请输入正确的手机号',
            trigger: 'blur'
          }
        ],
        userEmail: [
          { required: true, message: '请输入邮箱', trigger: 'blur' },
          {
            pattern: /^[A-Za-z0-9\u4e00-\u9fa5]+@[a-zA-Z0-9_-]+(\.[a-zA-Z0-9_-]+)+$/,
            required: true,
            message: '请输入正确的邮箱',
            trigger: 'blur'
          }
        ],
        userSex: [{ required: true, message: '请选择性别', trigger: 'blur' }]
      },
      // 删除用户
      seletedata: {
        ids: '',
        token: localStorage.getItem('logintoken')
      },
      // 请求数据参数
      formInline: {
        page: 1,
        limit: 10,
        nick:"",
        userId:"",
        articleId:""
      },
      commentline: {
        page: 1,
        limit: 10,
        articleId:""
      },
      //用户数据
      articleData: [],
      //评论数据
      commentList:[],
      // 选中
      checkmenu: [],
      //参数role
      saveroleId: '',
      // 分页参数
      pageparm: {
        currentPage: 1,
        pageSize: 10,
        total: 10
      },
      commentpageparm: {
        currentPage: 1,
        pageSize: 10,
        total: 10
      }
    }
  },
  // 注册组件
  components: {
    Pagination
  },

  /**
   * 数据发生改变
   */
  watch: {},

  /**
   * 创建完毕
   */
  created() {
    this.getdata(this.formInline);
    this.getCommentData(this.commentline)
  },

  /**
   * 里面的方法只有被调用才会执行
   */
  methods: {
    // 获取数据方法
    getdata(parameter) {
      this.loading = true

      /***
       * 调用接口，注释上面模拟数据 取消下面注释
       */
      // 获取用户列表
      articleList(parameter).then(res => {
        this.loading = false
        console.log(res);
        if (res.ok == false) {
          this.$message({
            type: 'info',
            message: res.message
          })
        } else {
          this.articleData = res.data
          console.log(res);
          // 分页赋值
          this.pageparm.currentPage = this.formInline.page
          this.pageparm.pageSize = this.formInline.limit
          this.pageparm.total = res.count//???
        }
      })
    },
    getCommentData(parameter) {
      this.loading = true

      /***
       * 调用接口，注释上面模拟数据 取消下面注释
       */
      // 获取用户列表
      getCommentList(parameter).then(res => {
        this.loading = false
        console.log("更新数据成功");
        if (res.ok == false) {
          this.$message({
            type: 'info',
            message: res.message
          })
        } else {
          this.commentList = res.data.records
          console.log(res);
          // 分页赋值
          this.pageparm.currentPage = this.commentline.page
          this.pageparm.pageSize = this.commentline.limit
          this.pageparm.total = res.count//???
        }
      })
    },
    // 分页插件事件
    callFather(parm) {
      this.formInline.page = parm.currentPage
      this.formInline.limit = parm.pageSize
      this.getdata(this.formInline)
    },
    // 分页插件事件
    callcommentFather(parm) {
      this.commentline.page = parm.currentPage
      this.commentline.limit = parm.pageSize
      this.getCommentList(this.commentline)
    },
    //搜索事件
    search() {
      this.commentline(this.formInline)
      this.getCommentList(this.commentline)
    },
    // 删除用户
    deleteArticle(index, row) {
      this.$confirm('确定要删除吗?', '信息', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      })
        .then(() => {
          // 删除
          articleDelete(row.articleId)
            .then(res => {
              if (res.ok) {
                this.$message({
                  type: 'success',
                  message: '数据已删除!'
                })
                this.getdata(this.formInline)
              } else {
                this.$message({
                  type: 'info',
                  message: res.message
                })
              }
            })
            .catch(err => {
              this.loading = false
              this.$message.error('数据删除失败，请稍后再试！')
            })
        })
        .catch(() => {
          this.$message({
            type: 'info',
            message: '已取消删除！'
          })
        })
    },
    showComment: function(index, row) {
      this.editFormVisible = true
      this.commentline.articleId = row.articleId
      getCommentList(this.commentline).then(res => {
              if (res.ok) {
                this.commentList = res.data.records;
                console.log("请求成功");
                console.log(res);
                this.commentline(this.commentline)
              } else {
                this.$message({
                  type: 'info',
                  message: res.message
                })
              }
      },fail=>{
        console.log("请求失败");
      })

    },

        // 删除评论
        deletecomment(index, row) {
      this.$confirm('确定要删除吗?', '信息', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      })
        .then(() => {
          // 删除
          commentDelete(row.commentId)
            .then(res => {
              if (res.ok) {
                this.$message({
                  type: 'success',
                  message: '数据已删除!'
                })
                this.getCommentData(this.commentline)
              } else {
                this.$message({
                  type: 'info',
                  message: res.message
                })
              }
            })
            .catch(err => {
              this.loading = false
              this.$message.error('数据删除失败，请稍后再试！')
            })
        })
        .catch(() => {
          this.$message({
            type: 'info',
            message: '已取消删除！'
          })
        })
    },
    // 选中菜单
    changemenu(arr) {
      let change = []
      for (let i = 0; i < arr.length; i++) {
        if (arr[i].checked) {
          change.push(arr[i].id)
        }
      }
      return change
    },
    // 选择复选框事件
    selectChange(val) {
      this.selectdata = val
    },
    // 菜单权限-保存
    menuPermSave() {
      let parm = {
        userId: this.saveroleId,
        deptIds: ''
      }
      let node = this.$refs.tree.getCheckedNodes()
      let moduleIds = []
      if (node.length != 0) {
        for (let i = 0; i < node.length; i++) {
          moduleIds.push(node[i].id)
        }
        parm.deptIds = JSON.stringify(moduleIds)
      }
      UserDeptSave(parm)
        .then(res => {
          if (res.success) {
            this.$message({
              type: 'success',
              message: '权限保存成功'
            })
            this.dataAccessshow = false
            this.getdata(this.formInline)
          } else {
            this.$message({
              type: 'info',
              message: res.msg
            })
          }
        })
        .catch(err => {
          this.loading = false
          this.$message.error('权限保存失败，请稍后再试！')
        })
    },
    // 刷新缓存
    refreshCache(index, row) {
      userFlashCache(row.userId)
        .then(res => {
          if (res.ok) {
            this.$message({
              type: 'success',
              message: '刷新成功！'
            })
            this.getdata(this.formInline)
          } else {
            this.$message({
              type: 'info',
              message: res.message
            })
          }
        })
        .catch(err => {
          this.loading = false
          this.$message.error('刷新失败，请稍后再试！')
        })
    }
  }
}
</script>

<style scoped>
.user-search {
  margin-top: 20px;
}
.userRole {
  width: 100%;
}
</style>

 