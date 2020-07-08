<template>
  <div>
   <!-- 面包屑导航区域 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>分类参数</el-breadcrumb-item>
    </el-breadcrumb>

    <!-- 卡片视图区 -->
    <el-card>
      <!-- 注意警告 -->
      <el-alert
        title="注意:只允许为第三级分类设置相关参数！"
        type="warning"
        show-icon
        :closable="false"
        >
      </el-alert>
      <el-row class="cat_opt">
        <el-col>
          <span>选择商品分类：</span>
          <!-- 联级选择框 -->
          <el-cascader
            v-model="selectedCateKeys"
            :options="catelist"
            :props="cateProps"
            expandTrigger="hover"
            @change="handleChange"></el-cascader>
        </el-col>
      </el-row>

      <!-- tab页签区 -->
      <el-tabs v-model="activeName" @tab-click="handleTabClick">
            <!-- t添加动态参数的面板 -->
            <!-- name="many"动态参数 -->
          <el-tab-pane label="动态参数" name="many">
          <el-button type="primary" size="mini"
          :disabled="isBtnDisabled" @click="addDialogVisible=true">添加参数</el-button>
          <!-- 动态参数表格 -->
          <el-table :data="manyTableData" border stripe>
            <!-- 索引列 -->
            <!-- 展开行 -->
            <!--   eslint-disable  -->
              <el-table-column type="expand">
                <template slot-scope="scope">
                   <el-tag v-for="(item,i) in scope.row.attr_vals" :key="i" closable>{{item}}</el-tag>
                </template>
              </el-table-column>
            <el-table-column type="index" label="#"></el-table-column>
            <el-table-column label="参数名称" prop="attr_name"></el-table-column>
              <el-table-column label="操作" >
                <!--   eslint-disable  -->
                <template slot-scope="scope">
                  <el-button
                    type="primary"
                    icon="el-icon-edit"
                      size="mini"
                    @click="showEditDialog(scope.row.attr_id)"
                    >编辑</el-button>
                    <el-button
                    type="danger"
                    icon="el-icon-delete"
                    size="mini"
                    @click="removeDialog(scope.row.attr_id)"
                    >删除</el-button
                    >
                </template>
              </el-table-column>
        </el-table>
        </el-tab-pane>
         <!-- t添加静态参数的面板 -->
         <!-- name="only"静态参数 -->
          <el-tab-pane label="静态属性" name="only">
            <el-button type="primary" size="mini"
            :disabled="isBtnDisabled" @click="addDialogVisible=true">添加属性</el-button>
            <!-- 添加静态参数的面板 -->
              <el-table :data="onlyTableData" border stripe>

              <!-- 展开行 -->
                <el-table-column type="expand"></el-table-column>
                  <!-- 索引列 -->
              <el-table-column type="index" label="#"></el-table-column>
              <el-table-column label="属性名称" prop="attr_name"></el-table-column>
                <el-table-column label="操作" >
                  <!--   eslint-disable  -->
                  <template slot-scope="scope">
                    <el-button
                      type="primary"
                      icon="el-icon-edit"
                        size="mini"
                        @click="showEditDialog(scope.row.attr_id)"
                      >编辑</el-button>
                      <el-button
                      type="danger"
                      icon="el-icon-delete"
                      size="mini"
                       @click="removeDialog(scope.row.attr_id)"
                      >删除</el-button
                      >
                  </template>
                </el-table-column>
            </el-table>
          </el-tab-pane>
    </el-tabs>
    </el-card>

    <!-- 添加参数的对话框 -->
    <el-dialog
      :title="'添加'+titleText"
      :visible.sync="addDialogVisible"
      width="30%" @close="addDialogClosed">
      <!-- 添加参数的对话框 -->
      <el-form :model="addForm" :rules="addFormRules" ref="addFormRef" label-width="100px" >
      <el-form-item :label="titleText" prop="attr_name">
        <el-input v-model="addForm.attr_name"></el-input>
      </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="addDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addParams">确 定</el-button>
      </span>
    </el-dialog>
    <!-- 修改对话框 -->
        <!-- 修改参数的对话框 -->
    <el-dialog
      :title="'修改'+titleText"
      :visible.sync="editDialogVisible"
      width="30%"
       @close="editDialogClosed">
      <!-- 添加参数的对话框 -->
      <el-form :model="editForm" :rules="editFormRules" ref="editFormRef" label-width="100px" >
      <el-form-item :label="titleText" prop="attr_name">
        <el-input v-model="editForm.attr_name"></el-input>
      </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="editDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="editParams">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data() {
    return {
      // 商品列表
      catelist: [],
      // 级联选择框配置对象
      cateProps: {
        value: 'cat_id',
        label: 'cat_name',
        children: 'children'
      },
      // 级联选择框双向绑定到的数组
      selectedCateKeys: [],
      // 被激活的页签名称
      activeName: 'many',
      // 动态参数数据
      manyTableData: [],
      // 静态属性数据
      onlyTableData: [],
      // 控制添加对话框的显示与隐藏
      addDialogVisible: false,
      // 添加参数的表单数据对象
      addForm: {
        attr_name: ''
      },
      // 添加表单的验证规则对象
      addFormRules: {
        attr_name: [{
          required: true,
          message: '请输入参数名称',
          trigger: 'blur'
        }]
      },
      // 控制修改对话框的显示与fou
      editDialogVisible: false,
      // 修改的表单数据对象
      editForm: {},
      //  修改表单的验证规则对象
      editFormRules: {
        attr_name: [{
          required: true,
          message: '请输入参数名称',
          trigger: 'blur'
        }]
      }
    }
  },
  created() {
    this.getCateList()
  },
  methods: {
    // 获取所有商品的分类
    async getCateList() {
      const { data: res } = await this.$http.get('categories')
      if (res.meta.status !== 200) {
        return this.$message.error('获取商品分类失败！')
      }
      this.catelist = res.data
      console.log(this.catelist)
    },
    // 2.级联选择框选中变化，会触发这个函数
    handleChange() {
      this.getParamsData()
    },
    // 3.tab页签点击事件处理函数
    handleTabClick() {
      console.log(this.activeName)
      this.getParamsData()
    },
    // 1.获取参数列表数据
    async getParamsData() {
      // console.log(this.selectedCateKeys)
      // 证明选中不是三级
      if (this.selectedCateKeys.length !== 3) {
        // 如果不是三级选中就清空输入框
        this.selectedCateKeys = []
        return false
      }
      // 否则就是选中的三级
      console.log(this.selectedCateKeys)
      // 根据所选分类id，和当前所处的面板，获取对应的参数
      const { data: res } = await this.$http.get(`categories/${this.cateId}/attributes`, { params: { sel: this.activeName } })
      if (res.meta.status !== 200) {
        return this.$message.error('获取参数失败')
      }

      // 获取到的数据先做判断，是many就存如manyTableData中
      if (this.activeName === 'many') {
        // 在赋值之前，每一项都做循环，每拿到一个item项就用split做字符串分隔
        res.data.forEach(item => {
          item.attr_vals = item.attr_vals.split(' ')
        })
        console.log(res.data)
        this.manyTableData = res.data
      } else {
        // 是only就存入onlyTableData
        this.onlyTableData = res.data
      }
    },
    // 监听添加对话框的关闭事件
    addDialogClosed() {
      // 对话框重置
      this.$refs.addFormRef.resetFields()
    },
    // 点击按钮添加
    addParams() {
      this.$refs.addFormRef.validate(async valid => {
        if (!valid) return
        const { data: res } = await this.$http.post(`categories/${this.cateId}/attributes`, {
          attr_name: this.addForm.attr_name,
          attr_sel: this.activeName

        })
        if (res.meta.status !== 201) {
          return this.$message.error('添加参数失败')
        }
        this.$message.success('添加数据成功')
        this.addDialogVisible = false
        this.getParamsData()
      })
    },
    // 点击按钮修改
    /* eslint-disable*/
    async showEditDialog(attr_id) {
      // 查询当前参数的信息
      const { data: res } = await this.$http.get(`categories/${this.cateId}/attributes/${attr_id}`, { params: { attr_sel: this.activeName } })
      if (res.meta.status !== 200) {
        return this.$message.error('获取参数失败')
      }
      this.editForm=res.data
      this.editDialogVisible = true
    },
    //
    editDialogClosed() {
      this.$refs.editFormRef.resetFields()
    },
    // 点击按钮修改参数信息
    editParams() {
       this.$refs.editFormRef.validate(async valid => {
        if (!valid) return
        const { data: res } = await this.$http.put(`categories/${this.cateId}/attributes/${this.editForm.attr_id}`, {
          attr_name: this.editForm.attr_name,
          attr_sel: this.activeName

        })
        if (res.meta.status !== 200) {
          return this.$message.error('修改参数失败')
        }
        this.$message.success('修改数据成功')
        this.editDialogVisible = false
        this.getParamsData()
      })
    },
    // 点击删除删除信息
    async removeDialog(attr_id){
         // console.log(id)
      // 弹框询问用户是否删除数据
      // confirmResult返回的是文本
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
      const { data: res } = await this.$http.delete(`categories/${this.cateId}/attributes/${attr_id}`)
      if (res.meta.status !== 200) {
        return this.$message.error('删除用户失败')
      }
      this.$message.success('删除用户成功！')
      this.getParamsData()
    }



  },
  computed: {
    // 如果按钮需要被禁用，则返回true,否则返回false
    isBtnDisabled() {
      if (this.selectedCateKeys.length !== 3) {
        return true
      }
      return false
    },
    // 当前选中的三级分类的ID
    cateId() {
      if (this.selectedCateKeys.length === 3) {
        // 如果长度等于3 就选中了三级分类就有ID值
        return this.selectedCateKeys[2]
      }
      // 如果长度小于3 就选中了三级分类就没有ID值
      return null
    },
    // 动态计算标题文本
    titleText() {
      if (this.activeName === 'many') {
        return '动态参数'
      }
      return '静态参数'
    }
  }
}
</script>

<style lang="less" scoped>
.cat_opt{
  padding: 15px 0;
}
.el-tag{
  margin: 10px;
}
</style>
