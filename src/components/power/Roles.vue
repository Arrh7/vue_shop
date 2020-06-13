<template>
  <div>
    <!-- 面包屑导航区域 -->
    <el-breadcrumb separator="/">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>角色列表</el-breadcrumb-item>
    </el-breadcrumb>

    <!-- 卡片视图 -->
    <el-card>
      <!-- 添加角色 -->
      <el-row>
        <el-col>
          <el-button type="primary" @click="addDialogVisible = true"
            >添加角色</el-button
          >
        </el-col>
      </el-row>
      <!-- 角色列表区域 -->
      <el-table :data="roleslist" border stripe>
        <!-- 展开列 -->
        <el-table-column type="expand">
          <template slot-scope="scope">
            <!--   eslint-disable  -->
            <el-row
              :class="['bdbottom', i1 === 0 ? 'bdtop' : '', 'vcenter']"
              v-for="(item1, i1) in scope.row.children"
              :key="item1.id"
            >
              <!-- 渲染一级权限 -->
              <el-col :span="5">
                <el-tag
                  closable
                  @close="removeRightById(scope.row, item1.id)"
                  >{{ item1.authName }}</el-tag
                >
                <i class="el-icon-caret-right"></i>
              </el-col>
              <!-- 渲染二级和三级 -->
              <el-col :span="19">
                <!-- 通过for循环嵌套渲染二级权限 -->
                <el-row
                  :class="[i2 === 0 ? '' : 'bdtop', 'vcenter']"
                  v-for="(item2, i2) in item1.children"
                  :key="item2.id"
                >
                  <el-col :span="6">
                    <el-tag
                      type="success"
                      closable
                      @close="removeRightById(scope.row, item2.id)"
                      >{{ item2.authName }}</el-tag
                    >
                    <i class="el-icon-caret-right"></i>
                  </el-col>
                  <el-col :span="18">
                    <el-tag
                      type="warning"
                      v-for="(item3, i3) in item2.children"
                      :key="item3.id"
                      closable
                      @close="removeRightById(scope.row, item3.id)"
                    >
                      {{ item3.authName }}
                    </el-tag>
                  </el-col>
                </el-row>
              </el-col>
            </el-row>
          </template>
        </el-table-column>
        <!-- 索引列 -->
        <el-table-column type="index"></el-table-column>
        <el-table-column label="角色名称" prop="roleName"></el-table-column>
        <el-table-column label="角色描述" prop="roleDesc"></el-table-column>
        <el-table-column label="操作" width="300px">
          <!--   eslint-disable  -->
          <template slot-scope="scope">
            <el-button
              size="mini"
              type="primary"
              icon="el-icon-edit"
              @click="showEditDialog(scope.row.id)"
              >编辑</el-button
            >
            <el-button
              size="mini"
              type="danger"
              icon="el-icon-delete"
              @click="removeUserById(scope.row.id)"
              >删除</el-button
            >
            <el-button
              size="mini"
              type="warning"
              icon="el-icon-setting"
              @click="showSetRightDialog(scope.row)"
              >分配权限</el-button
            >
          </template>
        </el-table-column>
      </el-table>
    </el-card>

    <!-- 分配权限的对话框 -->
    <el-dialog
      title="分配权限"
      :visible.sync="setRightDialogVisible"
      width="50%"
      @close="setRightDialogClosed"
    >
      <el-tree
      :data="rightlist"
      :props="treeProps"
      show-checkbox node-key="id" default-expand-all
      :default-checked-keys="defKeys"
      ref="treeRef"></el-tree>
      <span slot="footer" class="dialog-footer">
        <el-button @click="setRightDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="allotRights"
          >确 定</el-button
        >
      </span>
    </el-dialog>

    <!-- 添加用户对话框 -->
    <el-dialog
      title="添加用户"
      :visible.sync="addDialogVisible"
      width="50%"
      @close="addDialogClosed"
    >
      <!-- 内容主体区域 -->
      <el-form
        :model="addForm"
        :rules="addFormRules"
        ref="addFormRef"
        label-width="80px"
      >
        <el-form-item label="角色名称" prop="roleName">
          <el-input v-model="addForm.roleName"></el-input>
        </el-form-item>
        <el-form-item label="角色描述" prop="roleDesc">
          <el-input v-model="addForm.roleDesc"></el-input>
        </el-form-item>
      </el-form>
      <!-- 底部按钮 -->
      <span slot="footer" class="dialog-footer">
        <el-button @click="addDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addUser">确 定</el-button>
      </span>
    </el-dialog>

    <!-- 修改用户对话框 -->
    <el-dialog
      title="修改用户"
      :visible.sync="editdialogVisible"
      width="50%"
      @close="editdialogClosed"
    >
      <el-form
        ref="editFormRef"
        :model="editForm"
        :rules="editFormRules"
        label-width="80px"
      >
        <el-form-item label="用户Id">
          <el-input v-model="editForm.roleId" disabled></el-input>
        </el-form-item>
        <el-form-item label="角色名称" prop="roleName">
          <el-input v-model="editForm.roleName"></el-input>
        </el-form-item>
        <el-form-item label="角色描述" prop="roleDesc">
          <el-input v-model="editForm.roleDesc"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="editdialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="editUserInfo">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data() {
    return {
      //  权限列表
      roleslist: [],
      // 控制添加用户的对话框
      addDialogVisible: false,
      // 控制分配权限对话框的显示与隐藏
      setRightDialogVisible: false,
      // 所有权限的数据
      rightlist: [],

      // 树形控件绑定对象
      treeProps: {
        label: 'authName',
        children: 'children'
      },
      // 默认选中的节点id值
      defKeys: [],
      // 当前即将分配权限的角色id
      roleId: '',
      // 添加用户数据绑定
      addForm: {
        roleId: '',
        roleName: '',
        roleDesc: ''
      },
      // 添加表单验证规则对象
      addFormRules: {
        roleName: [
          { required: true, message: '角色名称', trigger: 'blur' },
          {
            min: 3,
            max: 6,
            message: '不可以为空',
            trigger: 'blur'
          }
        ],
        roleDesc: [
          { required: true, message: '角色描述', trigger: 'blur' },
          { min: 0, max: 9, message: '可以为空', trigger: 'blur' }
        ]
      },
      // 控制修改用户对话框
      editdialogVisible: false,
      // 查询到的用户信息对象保存
      editForm: {},
      // 修改表单的验证规则对象
      editFormRules: {
        roleName: [
          { required: true, message: '角色名称', trigger: 'blur' },
          {
            min: 3,
            max: 6,
            message: '不可以为空',
            trigger: 'blur'
          }
        ],
        roleDesc: [
          { required: true, message: '角色描述', trigger: 'blur' },
          { min: 0, max: 9, message: '可以为空', trigger: 'blur' }
        ]
      }
    }
  },
  created() {
    //  获取所有权限
    this.getRolesList()
  },
  methods: {
    //  获取权限数据
    async getRolesList() {
      const { data: res } = await this.$http.get('roles')
      if (res.meta.status !== 200) {
        return this.$message.error('获取权限列表失败！')
      }
      this.roleslist = res.data
      // console.log(this.roleslist)
    },

    // 监听添加角色对话框的关闭清空事件
    addDialogClosed() {
      this.$refs.addFormRef.resetFields()
    },
    // 点击按钮添加新角色
    addUser() {
      this.$refs.addFormRef.validate(async valid => {
        // console.log(valid)
        if (!valid) return
        // 可以发起添加角色的网络请求
        const { data: res } = await this.$http.post('roles', this.addForm)
        if (res.meta.status !== 200) {
          this.$message.error('添加角色失败！')
        }
        this.$message.success('添加角色成功！')

        // 隐藏添加角色对话框
        this.addDialogVisible = false
        // 刷新角色列表
        this.getRolesList()
      })
    },
    // 展示编辑用户对话框
    async showEditDialog(id) {
      // console.log(id)
      // 多台数据要用字符串拼接
      const { data: res } = await this.$http.get('roles/' + id)
      if (res.meta.status !== 200) {
        return this.$message.error('查询用户信息失败')
      }
      this.editForm = res.data
      this.editdialogVisible = true
      // // 刷新用户列表
      // this.getUserList()
    },
    // 监听修改对话框关闭事件
    editdialogClosed() {
      this.$refs.editFormRef.resetFields()
    },
    // 修改角色信息并提交
    editUserInfo() {
      this.$refs.editFormRef.validate(async valid => {
        // console.log(valid)
        if (!valid) return
        // 可以发起添加用户的网络请求
        const { data: res } = await this.$http.put(
          'roles/' + this.editForm.roleId,
          {
            roleName: this.editForm.roleName,
            roleDesc: this.editForm.roleDesc
          }
        )
        if (res.meta.status !== 200) {
          return this.$message.error('修改用户失败！')
        }
        this.$message.success('修改用户成功！')

        // 隐藏添加用户对话框
        this.editdialogVisible = false
        // 刷新用户列表
        this.getRolesList()
      })
    },
    // 删除角色信息
    async removeUserById(id) {
      // console.log(id)
      // 弹框询问用户是否删除数据
      const confirmResult = await this.$confirm(
        '永久删除该用户, 是否继续?',
        '提示',
        {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }
      ).catch(err => err)
      //  如果用户确认删除为confirm
      // 如果取消返回cancel
      // console.log(confirmResult)
      if (confirmResult !== 'confirm') {
        return this.$message.info('已取消删除')
      }
      const { data: res } = await this.$http.delete('roles/' + id)
      if (res.meta.status !== 200) {
        return this.$message.error('删除用户失败')
      }
      this.$message.success('删除用户成功！')
      this.getRolesList()
    },
    // 根据id删除对应的权限
    async removeRightById(role, rightId) {
      // 弹出框提示用户是否删除
      const confirmResult = await this.$confirm('此操作将永久删除该文件, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).catch(err => err)
      if (confirmResult !== 'confirm') {
        return this.$message.info('已取消删除')
      }
      const { data: res } = await this.$http.delete(`roles/${role.id}/rights/${rightId}`)
      if (res.meta.status !== 200) {
        return this.$message.error('删除用户失败')
      }
      this.$message.success('删除用户成功！')
      // this.getRolesList() 重新渲染会这个表格刷新
      // 只刷新数组
      role.children = res.data
    },
    // 展示分配权限的对话框
    async showSetRightDialog(role) {
      this.roleId = role.id
      // 获取所有权限的数据
      const { data: res } = await this.$http.get('rights/tree')
      if (res.meta.status !== 200) {
        return this.$message.error('获取权限数据失败')
      }
      // 把获取到的数据保存到data中
      this.rightlist = res.data
      console.log(this.rightlist)
      // 递归获取三级节点id
      this.getLeafKeys(role, this.defKeys)
      this.setRightDialogVisible = true
    },
    // 通过递归的形式，获取角色下所有三级权限的id，并保存到数组defKeys
    getLeafKeys(node, arr) {
      // 如果当前node节点不包含children属性。则是三级节点
      if (!node.children) {
        return arr.push(node.id)
      }
      // 递归
      node.children.forEach(item =>
        this.getLeafKeys(item, arr)
      )
    },
    // 监听分配权限对话框的关闭事件
    setRightDialogClosed() {
      this.defKeys = []
    },
    // 点击为角色分配权限
    async allotRights() {
      const keys = [
        ...this.$refs.treeRef.getCheckedKeys(),
        ...this.$refs.treeRef.getHalfCheckedKeys()
      ]
      // console.log(keys)
      // 字符串拼接
      const idStr = keys.join(',')
      const { data: res } = await this.$http.post(`roles/${this.roleId}/rights`, { rids: idStr })
      if (res.meta.status !== 200) {
        return this.$message.error('分配权限失败！')
      }
      this.$message.success('分配权限成功')
      this.getRolesList()
      this.setRightDialogVisible = false
    }
  }
}
</script>

<style lang="less" scoped>
.el-tag {
  margin: 7px;
}
.bdtop {
  border-top: 1px solid #eee;
}
.bdbottom {
  border-bottom: 1px solid #eee;
}
.vcenter {
  display: flex;
  align-items: center;
}
</style>
