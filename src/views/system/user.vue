/**
 * 系统管理 用户管理
 */
<template>
  <div>
    <!-- 面包屑导航 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item>首页</el-breadcrumb-item>
      <el-breadcrumb-item>用户管理</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 搜索筛选 -->
    <el-form :inline="true" :model="formInline" class="user-search">
      <el-form-item label="搜索：">
        <el-select size="small" v-model="formInline.userType" placeholder="请选择">
          <el-option label="全部" value=""></el-option>
          <el-option label="教师" value="教师"></el-option>
          <el-option label="学生" value="学生"></el-option>
        </el-select>
      </el-form-item>
      <el-form-item label="">
        <el-input size="small" v-model="formInline.userId" placeholder="输入用户Id"></el-input>
      </el-form-item>
      <el-form-item label="">
        <el-input size="small" v-model="formInline.name" placeholder="输入姓名"></el-input>
      </el-form-item>
      <el-form-item label="">
        <el-input size="small" v-model="formInline.studentId" placeholder="输入学号/工号"></el-input>
      </el-form-item>
      <el-form-item>
        <el-button size="small" type="primary" icon="el-icon-search" @click="search">搜索</el-button>
        <el-button size="small" type="primary" icon="el-icon-plus" @click="handleEdit()">添加</el-button>
        <el-button size="small" type="primary" icon="el-icon-plus" @click="handleEdit()">批量导入数据</el-button>
      </el-form-item>
    </el-form>
    <!--列表-->
    <el-table size="small" @selection-change="selectChange" :data="userData" highlight-current-row v-loading="loading" border element-loading-text="拼命加载中" style="width: 100%;">
      <el-table-column align="center" type="selection" width="50">
      </el-table-column>
      <el-table-column align="center" sortable prop="userId" label="用户Id" width="80">
      </el-table-column>
      <el-table-column align="center" sortable label="头像" width="80">
        <template slot-scope="scope">
            <el-popover placement="top-start" title="" trigger="hover">
              <img :src="scope.row.avatar" alt="" style="width: 150px;height: 150px">
              <img slot="reference" :src="scope.row.avatar" style="width:auto;height: 40px">
            </el-popover>
        </template>
      </el-table-column>
      <el-table-column align="center" sortable prop="nick" label="昵称" width="80">
      </el-table-column>
      <el-table-column align="center" sortable prop="signature" label="个性签名" width="120">
      </el-table-column>
      <el-table-column align="center" sortable prop="name" label="真实姓名" width="80">
      </el-table-column>
      <el-table-column align="center" sortable prop="studentId" label="学号/工号" width="120">
      </el-table-column>
      <el-table-column align="center" sortable prop="passWord" label="密码" width="120">
      </el-table-column>
      <!-- <el-table-column align="center" sortable prop="telephone" label="手机号码" min-width="50">
      </el-table-column> -->
      <el-table-column align="center" sortable prop="openId" label="绑定的OpenId" width="120">
      </el-table-column>
      <el-table-column align="center" sortable prop="userType" label="用户类型" min-width="50">
      </el-table-column>
      <el-table-column align="center" sortable prop="isActive" label="是否激活" min-width="50">
        <template slot-scope="scope">
          <el-switch v-model="scope.row.isActive=='1'?nshow:fshow" active-color="#13ce66" inactive-color="#ff4949" @change="editActiveType(scope.$index, scope.row)">
          </el-switch>
        </template>
      </el-table-column>
      <el-table-column align="center" sortable prop="isPublish" label="是否可发布" min-width="50">
        <template slot-scope="scope">
          <el-switch v-model="scope.row.isPublish=='1'?nshow:fshow" active-color="#13ce66" inactive-color="#ff4949" @change="editType(scope.$index, scope.row)">
          </el-switch>
        </template>
        <!-- <template slot-scope="scope">
          <div>{{scope.row.editTime|timestampToTime}}</div>
        </template> -->
      </el-table-column>
      
      <el-table-column label="操作" min-width="300">
        <template slot-scope="scope">
          <el-button size="mini" @click="handleEdit(scope.$index, scope.row)">编辑</el-button>
          <el-button size="mini" type="danger" @click="deleteUser(scope.$index, scope.row)">删除</el-button>
          <el-button size="mini" type="success" @click="refreshCache(scope.$index, scope.row)">刷新数据</el-button>
        </template>
      </el-table-column>
    </el-table>
    <!-- 分页组件 -->
    <Pagination v-bind:child-msg="pageparm" @callFather="callFather"></Pagination>

    <!-- 编辑界面 -->
    <el-dialog :title="title" :visible.sync="editFormVisible" width="30%" @click='closeDialog("edit")'>
      <el-form label-width="80px" ref="editForm" :model="editForm" :rules="rules">
        <el-form-item label="用户Id" prop="userId">
          <el-input size="small" v-model="editForm.userId" auto-complete="off" placeholder="请输入用户名"></el-input>
        </el-form-item>
        <el-form-item label="昵称" prop="nick">
          <el-input size="small" v-model="editForm.nick" auto-complete="off" placeholder="请输入昵称"></el-input>
        </el-form-item>
        <el-form-item label="个性签名" prop="signature">
          <el-input size="small" v-model="editForm.signature" auto-complete="off" placeholder="请输入个性签名"></el-input>
        </el-form-item>
        <el-form-item label="姓名" prop="name">
          <el-input size="small" v-model="editForm.name" auto-complete="off" placeholder="请输入姓名"></el-input>
        </el-form-item>
        <!-- <el-form-item label="性别" prop="userSex">
          <el-radio v-model="editForm.userSex" label="男">男</el-radio>
          <el-radio v-model="editForm.userSex" label="女">女</el-radio>
        </el-form-item> -->
        <el-form-item label="学号/工号" prop="studentId">
          <el-input size="small" v-model="editForm.studentId" auto-complete="off" placeholder="请输入学号/工号"></el-input>
        </el-form-item>
        <el-form-item label="密码" prop="passWord">
          <el-input size="small" v-model="editForm.passWord" auto-complete="off" placeholder="请输入密码"></el-input>
        </el-form-item>
        <el-form-item label="OpenId" prop="openId">
          <el-input size="small" v-model="editForm.openId" auto-complete="off" placeholder="请输入OpenId"></el-input>
        </el-form-item>
        <el-form-item label="用户类型" prop="usrType">
          <el-select size="small" v-model="editForm.usrType" placeholder="请选择" class="userRole">
            <el-option label="学生" value="学生"></el-option>
            <el-option label="教师" value="教师"></el-option>
          </el-select>
        </el-form-item>
        <!-- <el-form-item label="手机号" prop="userMobile">
          <el-input size="small" v-model="editForm.userMobile" placeholder="请输入手机号"></el-input>
        </el-form-item> -->
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button size="small" @click='closeDialog("edit")'>取消</el-button>
        <el-button size="small" type="primary" :loading="loading" class="title" @click="submitForm('editForm')">保存</el-button>
      </div>
    </el-dialog>
  </div>
</template>

<script>
// 导入请求方法
import {
  userList,
  userSave,
  userDelete,
  userPwd,
  userExpireToken,
  userFlashCache,
  userLock,
  UserDeptTree,
  UserDeptSave,
  UserDeptdeptTree,
  UserChangeDept,
  userActive
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
      // 编辑与添加
      editForm: {
        userId: '',
        avatar:'',
        nick: '',
        signature: '',
        name:'',
        studentId: '',
        passWord: '',
        openId: '',
        usrType:'',
        isActive:'',
        isPublish:'',
        telephone:''
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
        limit: 10,
        name:"",
        studentId:"",
        userId:"",
        userType:""
      },
      //用户数据
      userData: [],
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
  watch: {},

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
    // 获取数据方法
    getdata(parameter) {
      this.loading = true

      /***
       * 调用接口，注释上面模拟数据 取消下面注释
       */
      // 获取用户列表
      userList(parameter).then(res => {
        this.loading = false
        console.log(res);
        if (res.ok == false) {
          this.$message({
            type: 'info',
            message: res.message
          })
        } else {
          this.userData = res.data.records 
          console.log(this.userData);
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
    // 修改type
    editType: function(index, row) {
      console.log(row);
      this.loading = true
      let parm = {
        isPublish: '',
        userId: '',
        token: localStorage.getItem('logintoken')
      }
      parm.userId = row.userId
      let isPublish = row.isPublish
      console.log(isPublish);
      if (isPublish == '0') {
        parm.isPublish = '1'
      } else {
        parm.isPublish = '0'
      }
      console.log(parm.isPublish);
      // 修改状态
      userLock(parm).then(res => {
        this.loading = false
        if (res.ok == false) {
          this.$message({
            type: 'info',
            message: res.message
          })
        } else {
          this.$message({
            type: 'success',
            message: '状态修改成功'
          })
          this.getdata(this.formInline)
        }
      })
    },
    //修改激活状态
    editActiveType: function(index, row) {
      console.log(row);
      this.loading = true
      let parm = {
        isActive: '',
        userId: '',
        token: localStorage.getItem('logintoken')
      }
      parm.userId = row.userId
      let isActive = row.isActive
      console.log(isActive);
      if (isActive == '0') {
        parm.isActive = '1'
      } else {
        parm.isActive = '0'
      }
      console.log(parm.isActive);
      // 修改状态
      console.log(parm);
      userActive(parm).then(res => {
        this.loading = false
        if (res.ok == false) {
          this.$message({
            type: 'info',
            message: res.message
          })
        } else {
          this.$message({
            type: 'success',
            message: '状态修改成功'
          })
          this.getdata(this.formInline)
        }
      })
    },
    //显示编辑界面
    handleEdit: function(index, row) {
      this.editFormVisible = true
      if (row != undefined && row != 'undefined') {
        this.title = '修改用户'
        this.editForm.userId = row.userId
        this.editForm.avatar = row.avatar
        this.editForm.nick = row.nick
        this.editForm.signature = row.signature
        this.editForm.name = row.name
        this.editForm.studentId = row.studentId
        this.editForm.passWord = row.passWord
        this.editForm.usrType = row.usrType
        this.editForm.openId = row.openId
        this.editForm.telephone = row.telephone
      } else {
        this.title = '添加用户'
        this.editForm.userId = ''
        this.editForm.avatar = 'http://localhost:8080/taian/upload/heard/avatar.jpg'
        this.editForm.nick = ''
        this.editForm.signature = ''
        this.editForm.name = ''
        this.editForm.studentId = ''
        this.editForm.passWord = ''
        this.editForm.usrType = ''
        this.editForm.openId = ''
        this.editForm.telephone = ''
        this.editForm.openId = ''
        this.editForm.telephone = ''
        this.editForm.isPublish = '0'
        this.editForm.isActive = '0'
        
      }
    },
    // 编辑、添加提交方法
    submitForm(editData) {
      this.$refs[editData].validate(valid => {
        if (valid) {
          // 请求方法
          userSave(this.editForm)
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
    // 显示部门设置
    handleunit: function(index, row) {
      this.unitAccessshow = true
      let parms = 0
      UserDeptdeptTree(parms)
        .then(res => {
          if (res.data.success) {
            this.UserDept = this.changeArr(res.data.data)
          } else {
            this.$message({
              type: 'info',
              message: res.msg
            })
          }
        })
        .catch(err => {
          this.loading = false
          this.$message.error('配置权限失败,请稍后再试！')
        })
    },
    handleClick(data, checked, node) {
      if (checked) {
        this.$refs.tree.setCheckedNodes([])
        this.$refs.tree.setCheckedNodes([data])
        this.unitparm.deptId = data.id
        this.unitparm.deptName = data.name
        //交叉点击节点
      } else {
      }
    },
    // 保存部门
    unitPermSave() {
      let len = this.selectdata
      let ids = []
      if (len != 0) {
        for (let i = 0; i < len.length; i++) {
          ids.push(len[i].userId)
        }
      }
      this.unitparm.userIds = ids.join(',')
      UserChangeDept(this.unitparm)
        .then(res => {
          this.unitAccessshow = false
          if (res.success) {
            this.$message({
              type: 'success',
              message: '部门设置成功！'
            })
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
          this.$message.error('部门设置失败,请稍后再试！')
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
          userDelete(row.userId)
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
    //数据格式化
    changeArr(data) {
      var pos = {}
      var tree = []
      var i = 0
      while (data.length != 0) {
        if (data[i].pId == 0) {
          tree.push({
            id: data[i].id,
            name: data[i].name,
            pId: data[i].pId,
            open: data[i].open,
            checked: data[i].checked,
            children: []
          })
          pos[data[i].id] = [tree.length - 1]
          data.splice(i, 1)
          i--
        } else {
          var posArr = pos[data[i].pId]
          if (posArr != undefined) {
            var obj = tree[posArr[0]]
            for (var j = 1; j < posArr.length; j++) {
              obj = obj.children[posArr[j]]
            }

            obj.children.push({
              id: data[i].id,
              name: data[i].name,
              pId: data[i].pId,
              open: data[i].open,
              checked: data[i].checked,
              children: []
            })
            pos[data[i].id] = posArr.concat([obj.children.length - 1])
            data.splice(i, 1)
            i--
          }
        }
        i++
        if (i > data.length - 1) {
          i = 0
        }
      }
      return tree
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

 