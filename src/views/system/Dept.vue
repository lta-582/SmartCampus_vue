/**
 * 系统管理
 */
<template>
  <div>
    <!-- 面包屑导航 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item>首页</el-breadcrumb-item>
      <el-breadcrumb-item>系统轮播图管理</el-breadcrumb-item>
    </el-breadcrumb>
    <el-form>
    <el-form-item>
        <el-button size="small" type="primary" icon="el-icon-plus" @click="handleEdit()">添加轮播图</el-button>
      </el-form-item>
    </el-form>
    <!--列表-->
    <el-table size="small" @selection-change="selectChange" :data="sActiveData" highlight-current-row v-loading="loading" border element-loading-text="拼命加载中" style="width: 100%; margin-top: 20px;">
      <el-table-column align="center" sortable prop="imageId" label="图片Id" width="80">
      </el-table-column>
      <el-table-column align="center" sortable prop="imagePath" label="图片" width="300">
        <template slot-scope="scope">
            <el-popover placement="top-start" title="" trigger="hover">
              <img :src="scope.row.imagePath" alt="" style="width: 300px;height: 300px">
              <img slot="reference" :src="scope.row.imagePath" style="width: auto;height: 120px">
            </el-popover>
        </template>
      </el-table-column>
      <el-table-column align="center" sortable prop="uploadTime" label="创建时间" width="120">
      </el-table-column>
      <el-table-column label="操作" min-width="300">
        <template slot-scope="scope">
          <el-button size="mini" @click="handleEdit(scope.$index, scope.row)">修改</el-button>
          <el-button size="mini" type="danger" @click="deleteUser(scope.$index, scope.row)">删除</el-button>
        </template>
      </el-table-column>
    </el-table>
    <!-- 分页组件 -->
    <Pagination v-bind:child-msg="pageparm" @callFather="callFather"></Pagination>

    <!-- 编辑界面 -->
    <el-dialog :title="title" :visible.sync="editFormVisible" width="30%" @click='closeDialog("edit")'>
      <el-form label-width="80px" ref="editForm" :model="editForm" :rules="rules">
        <el-form-item label="文章Id" prop="articleId">
          <el-input :disabled="true" size="small" v-model="editForm.imageId" auto-complete="off" placeholder=""></el-input>
        </el-form-item>
        <el-form-item label="上传时间" prop="uploadTime">
          <el-input :disabled="true" size="small" v-model="editForm.uploadTime" auto-complete="off" placeholder=""></el-input>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button size="small" @click='closeDialog("edit")'>取消</el-button>
        <el-button size="small" type="primary" :loading="loading" class="title" @click="submitForm('editForm')">保存</el-button>
      </div>
      <el-upload
  action="/api/upload/image/"
  list-type="picture-card"
  :on-preview="handlePictureCardPreview"
  :on-remove="handleRemove"
  :multiple="false"
  :on-success="handleSuccess">
  <i class="el-icon-plus"></i>
</el-upload>
<el-dialog :visible.sync="dialogVisible">
  <img width="100%" :src="dialogImageUrl" alt="">
  {{ dialogImageUrl }}
</el-dialog>
    </el-dialog>
  </div>
</template>

<script>
// 导入请求方法
import {
  imageList,
  imageSave,
  imageDelete
} from '../../api/userMG'
import Pagination from '../../components/Pagination'
export default {
  data() {
    return {
      dialogImageUrl: '123',
      dialogVisible: false,
      loading: false, //是显示加载
      title: '添加轮播图',
      editFormVisible: false, //控制编辑页面显示与隐藏
      dataAccessshow: false, //控制数据权限显示与隐藏
      unitAccessshow: false, //控制所属单位隐藏与显示
      // 编辑与添加
      editForm: {
        imageId:'',
        imagePath:'',
        uploadTime:''
      },
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
        limit: 10
      },
      //用户数据
      sActiveData: [],
      // 选中
      checkmenu: [],
      //参数role
      saveroleId: '',
      // 分页参数
      pageparm: {
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
  watch: {
    onChangeImgUrl: {
    handler(newName) {
      var aa = document.querySelector('.el-upload--picture-card')
      if (newName) {
        aa.style.display = 'none' 
      } else {
        setTimeout(() => {
          aa.style.display = 'inline-block'
        }, 1100)
      }
    }
  }
  },

  /**
   * 创建完毕
   */
  created() {
    this.getdata(this.formInline)
  },

  /**
   * 里面的方法只有被调用才会执行
   */
  methods: {
    handleSuccess(res){
      this.editForm.imagePath = res
      console.log(res);
    },
    handleRemove(file, fileList) {
        console.log(file.url);
      },
      handlePictureCardPreview(file) {
        this.dialogImageUrl = file.url;
        this.dialogVisible = true;
      },
    // 获取数据方法
    getdata(parameter) {
      this.loading = true

      /***
       * 调用接口，注释上面模拟数据 取消下面注释
       */
      // 获取文章列表
      imageList(parameter).then(res => {
        this.loading = false
        console.log(res);
        if (res.ok == false) {
          this.$message({
            type: 'info',
            message: res.message
          })
        } else {
          this.sActiveData = res.data.records 
          console.log(this.sActiveData);
          // 分页赋值
          this.pageparm.currentPage = this.formInline.page
          this.pageparm.pageSize = this.formInline.limit
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
    //搜索事件
    search() {
      this.getdata(this.formInline)
    },
     //显示编辑界面
    handleEdit: function(index, row) {
      this.editFormVisible = true
      if (row != undefined && row != 'undefined') {
        this.title = '修改轮播图'
        this.editForm.articleId = row.articleId
        this.editForm.title = row.title
        this.editForm.contentText = row.contentText
        this.editForm.imagePath = row.imagePath
      } else {
        this.title = '添加轮播图'
        this.editForm.articleId = ''
        this.editForm.title = ''
        this.editForm.contentText = ''
        this.editForm.imagePath = ''
      }
    },
    // 编辑、添加提交方法
    submitForm(editData) {
      this.$refs[editData].validate(valid => {
        if (valid) {
          // 请求方法
          imageSave(this.editForm)
            .then(res => {
              this.editFormVisible = false
              this.loading = false
              if (res.ok) {
                this.getdata(this.formInline)
                this.$message({
                  type: 'success',
                  message: '数据保存成功！'
                })
              } else {
                this.$message({
                  type: 'info',
                  message: res.message
                })
              }
            })
            .catch(err => {
              this.editFormVisible = false
              this.loading = false
              this.$message.error('保存失败，请稍后再试！')
            })
        } else {
          return false
        }
      })
    },
    
    // 选择复选框事件
    selectChange(val) {
      this.selectdata = val
    },
    // 关闭编辑、增加弹出框
    closeDialog(dialog) {
      if (dialog == 'edit') {
        this.editFormVisible = false
      } else if (dialog == 'perm') {
        this.dataAccessshow = false
      } else if (dialog == 'unit') {
        this.unitAccessshow = false
      }
    },
    // 删除用户
    deleteUser(index, row) {
      this.$confirm('确定要删除吗?', '信息', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      })
        .then(() => {
          // 删除
          console.log(row.imageId);
          imageDelete(row.imageId)
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

 