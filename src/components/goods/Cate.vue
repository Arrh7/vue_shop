<template>
  <div>
    <!-- 面包屑导航区域 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>商品分类</el-breadcrumb-item>
    </el-breadcrumb>

    <!-- 卡片视图区域 -->
    <el-card>
      <el-row>
        <el-col>
          <el-button type="primary" @click="showAddCateDialog"
            >添加分类</el-button
          >
        </el-col>
      </el-row>

      <!-- 表格占位 -->
      <tree-table
        class="treeTable"
        :data="catelist"
        :columns="columns"
        :selection-type="false"
        :expand-type="false"
        show-index
        index-text="#"
        border
        :show-row-hover="false"
      >
        <!-- 是否有效 -->
        <!--   eslint-disable  -->
        <template slot="isok" slot-scope="scope">
          <i
            class="el-icon-success"
            v-if="scope.row.cat_deleted === false"
            style="color:lightgreen"
          ></i>
          <i class="el-icon-error" v-else style="color:lightgreen"></i>
        </template>
        <!--  排序 -->
        <template slot="order" slot-scope="scope">
          <el-tag size="mini" v-if="scope.row.cat_level === 0">一级</el-tag>
          <el-tag
            size="mini"
            type="success"
            v-else-if="scope.row.cat_level === 1"
            >二级</el-tag
          >
          <el-tag size="mini" type="warning" v-else>三级</el-tag>
        </template>
        <!-- 操作 -->
        <template slot="opt" slot-scope="scope">
          <el-button
          type="primary"
           icon="el-icon-edit"
            size="mini"
            @click="CateEditDialog(scope.row.cat_id)"
            >编辑</el-button
          >
          <el-button
          type="danger"
          icon="el-icon-delete"
          size="mini"
          @click="removeCate(scope.row.cat_id)"
            >删除</el-button
          >
        </template>
      </tree-table>

      <!-- 分页占位 -->
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="querInfo.pagenum"
        :page-sizes="[3, 5, 10, 15]"
        :page-size="querInfo.pagesize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total"
      >
      </el-pagination>
    </el-card>

    <!-- 添加用户对话框 -->
    <el-dialog
      title="添加分类"
      :visible.sync="addCateDialogVisible"
      width="50%"
      @close="addCateDialogClosed"
       >
      <el-form
        :model="addCateForm"
        :rules="addCateFormRules"
        ref="addCateFormRef"
        label-width="100px"
      >
        <el-form-item label="分类名称:" prop="cat_name">
          <el-input v-model="addCateForm.cat_name"></el-input>
        </el-form-item>
        <el-form-item label="父级分类:">
          <!-- options用来指定数据源 -->
          <!-- props用来指定配置对象 -->
          <el-cascader
            clearable
             expandTrigger="hover"
            v-model="selectedKeys"
            :options="parentCateList"
            :props="cascaderProps"
            @change="parentCateChanged"
            class="el-cascader-panel"
          ></el-cascader>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="addCateDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addCate"
          >确 定</el-button
        >
      </span>
    </el-dialog>

      <!-- 修改用户对话框 -->
    <el-dialog
      title="修改分类"
      :visible.sync="CatedialogVisible"
      width="50%"
       @close="CateEditDialogClosed">
      <el-form
      ref="CateformRef"
      :model="CateEditform"
       :rules="CeteEditFormRules"
       label-width="80px">
      <el-form-item label="分类名称" prop="cat_name">
        <el-input v-model="CateEditform.cat_name"></el-input>
      </el-form-item>
     </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="CatedialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="CateUserInfo">确 定</el-button>
      </span>

    </el-dialog>
  </div>
</template>

<script>
export default {
  data() {
    return {
      // 商品分类数据，默认为空
      catelist: [],

      // 总数据条数
      total: 0,
      // 为table指定列的定义
      columns: [
        {
          label: '分类名称',
          prop: 'cat_name' // 传的数据赋值
        },
        {
          label: '是否有效',
          type: 'template', // 表示当前列定义为模板列
          template: 'isok' // 表示当前列使用模板名称
        },
        {
          label: '排序',
          type: 'template', // 表示当前列定义为模板列
          template: 'order' // 表示当前列使用模板名称
        },
        {
          label: '操作',
          type: 'template', // 表示当前列定义为模板列
          template: 'opt' // 表示当前列使用模板名称
        }
      ],
      addCateDialogVisible: false,
      // 添加分类的表单分类数据对象
      addCateForm: {
        // 将要添加的分类名称
        cat_name: '',
        // 父级分类的id
        cat_pid: 0,
        // 分类的等级，默认添加一级分类
        cat_level: 0
      },
      addCateFormRules: {
        cat_name: {
          required: true,
          message: '请输入分类名称',
          trigger: 'blur'
        }
      },
      // 查询条件
      querInfo: {
        type: 3,
        pagenum: 1,
        pagesize: 5
      },
      // 获取父级分类的列表
      parentCateList: [],
      // 指定级联选择器的配置对象
      cascaderProps: {
        value: 'cat_id',
        // 指定选中的值
        label: 'cat_name',
        // 指定看到的值
        children: 'children'
        // 父子嵌套的属性
      },
      // 选中的父级分类的id数组
      selectedKeys: [],
      // 控制修改用户对话框
      CatedialogVisible: false,
      // 查询到的用户信息对象保存
      CateEditform: {},
      // 修改验证规则
      CeteEditFormRules: {
        cat_name: {
          required: true,
          message: '请输入分类名称',
          trigger: 'blur'
        }
      }
    }
  },

  methods: {
    async getCateList() {
      const { data: res } = await this.$http.get('categories', {
        params: this.querInfo
      })
      if (res.meta.status !== 200) {
        return this.$message.error('获取商品分类失败！')
      }
      // console.log(res.data)
      this.catelist = res.data.result
      this.total = res.data.total
    },
    // 监听pagesize改变
    handleSizeChange(newSize) {
      this.querInfo.pagesize = newSize
      this.getCateList()
    },
    // 监听pageName改变
    handleCurrentChange(newPage) {
      this.querInfo.pagenum = newPage
      this.getCateList()
    },
    // 点击按钮展示添加分类对话框
    showAddCateDialog() {
      // 先获取父级分类对话框
      this.getParentCateList()
      this.addCateDialogVisible = true
    },
    // 获取父级分类数据列表
    async getParentCateList() {
      const { data: res } = await this.$http.get('categories', {
        params: { type: 2 }
      })
      if (res.meta.status !== 200) {
        return this.$message.error('获取父级分类数据失败！')
      }
      console.log(res.data)
      this.parentCateList = res.data
    },
    // 选择项父级分类触发这个函数
    parentCateChanged() {
      console.log(this.selectedKeys)
      // 如果selectedKeys数组中的length大于0，证明选中的父级分类
      // 反之就说明没有选中任何父级分类
      if (this.selectedKeys.length > 0) {
        // 父级分类的id
        this.addCateForm.cat_pid = this.selectedKeys[
          this.selectedKeys.length - 1
        ]
        // 当前分类的等级赋值
        this.addCateForm.cat_level = this.selectedKeys.length
        return false
      } else {
        // 父级分类的id
        this.addCateForm.cat_pid = 0
        this.addCateForm.cat_level = 0
      }
    },
    // 点击按钮确定添加
    addCate() {
      this.$refs.addCateFormRef.validate(async valid => {
        if (!valid) return
        const { data: res } = await this.$http.post('categories', this.addCateForm)
        console.log(this.addCateForm)
        if (res.meta.status !== 201) {
          return this.$message.error('添加分类失败')
        }
        this.$message.success('添加分类成功！')
        this.getCateList()
        this.addCateDialogVisible = false
      })
    },
    // 监听对话框的关闭事件，重置表单数据
    addCateDialogClosed() {
      this.$refs.addCateFormRef.resetFields()
      this.selectedKeys = []
      this.addCateForm.cat_level = 0
      this.addCateForm.cat_pid = 0
    },
    // 展示编辑分类对话框
    async CateEditDialog(id) {
      console.log(id)
      // 多台数据要用字符串拼接
      // 先获取数据 再修改
      const { data: res } = await this.$http.get('categories/' + id)
      if (res.meta.status !== 200) {
        return this.$message.error('修改用户信息失败')
      }
      // this.getParentCateList()
      this.CateEditform = res.data
      this.CatedialogVisible = true
    },
    // 监听修改对话框关闭事件
    CateEditDialogClosed() {
      this.$refs.CateformRef.resetFields()
    },
    // 修改分类并提交
    CateUserInfo() {
      this.$refs.CateformRef.validate(async valid => {
        // console.log(valid)
        if (!valid) return
        // 可以发起添加用户的网络请求
        const { data: res } = await this.$http.put(
          'categories/' + this.CateEditform.cat_id,
          {
            cat_name: this.CateEditform.cat_name
          }
        )
        if (res.meta.status !== 200) {
          return this.$message.error('修改用户失败！')
        }
        this.$message.success('修改用户成功！')

        // 隐藏添加用户对话框
        this.CatedialogVisible = false
        // 刷新用户列表
        this.getCateList()
      })
    },

    // 删除分类
    async removeCate(id) {
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
      const { data: res } = await this.$http.delete('categories/' + id)
      if (res.meta.status !== 200) {
        return this.$message.error('删除用户失败')
      }
      this.$message.success('删除用户成功！')
      this.getCateList()
    }

  },
  created() {
    this.getCateList()
  }
}
</script>

<style lang="less" scoped>
.treeTable {
  margin-top: 15px;
}
.el-cascader {
  width: 100%;
  // height: 200px;
}

</style>
