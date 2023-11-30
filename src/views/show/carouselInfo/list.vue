<template>
  <div class="app-container">

    <el-card class="operate-container" shadow="never">
      <el-form inline>
        <el-form-item label="关键字">
          <el-input type="text" width="100" placeholder="关键字" v-model="searchObj.keyword" clearable />
        </el-form-item>

        <el-button type="primary" icon="el-icon-search" @click="fetchData()">查询</el-button>
        <el-button type="default" @click="resetSearch">清空</el-button>
      </el-form>
    </el-card>

    <!-- 工具条 -->
    <el-card class="operate-container" shadow="never">
      <i class="el-icon-tickets" style="margin-top: 5px"></i>
      <span style="margin-top: 5px">数据列表</span>
      <el-button class="btn-add" size="mini" @click="add()">添加</el-button>
      <el-button class="btn-add" size="mini" @click="removeRows()" style="margin: 0 10px;">批量删除</el-button>
    </el-card>

    <!-- banner列表 -->
    <el-table v-loading="listLoading" :data="list" stripe border style="width: 100%;margin-top: 10px;"
      @selection-change="handleSelectionChange">

      <el-table-column type="selection" width="55" />

      <el-table-column label="SUK_ID/sku编号" width="120px">
        <template slot-scope="scope">
          <p>ID：{{ scope.row.id }}</p>
          <p>编号：{{ scope.row.skuCode }}</p>
        </template>
      </el-table-column>
      <el-table-column label="商品图片" width="140" align="center">
        <template slot-scope="scope">
          <img style="height: 80px" :src="scope.row.imgUrl">
        </template>
      </el-table-column>
      <el-table-column prop="sort" label="排序" />
      <el-table-column label="标签" width="160px">
        <template slot-scope="scope">
          <p>上架/下架：
            <el-switch @change="handlePublishStatusChange(scope.$index, scope.row)" :inactive-value=0 :active-value=1
              v-model="scope.row.publishStatus">
            </el-switch>
          </p>
        </template>
      </el-table-column>

      <el-table-column label="操作" width="150" align="center">
        <template slot-scope="scope">
          <router-link :to="'/product/skuInfo/edit/' + scope.row.id">
            <el-button type="text" size="mini">修改</el-button>
          </router-link>
          <el-button type="text" size="mini" @click="removeDataById(scope.row.id)">删除</el-button>
        </template>
      </el-table-column>
    </el-table>

    <!-- 分页组件 -->
    <el-pagination :current-page="page" :total="total" :page-size="limit" :page-sizes="[5, 10, 20, 30, 40, 50, 100]"
      style="padding: 30px 0; text-align: center;" layout="sizes, prev, pager, next, jumper, ->, total, slot"
      @current-change="fetchData" @size-change="changeSize" />
  </div>
</template>

<script>
import api from '@/api/product/skuInfo'
import categoryApi from '@/api/product/category'

export default {
  data () {
    return {
      listLoading: true, // 数据是否正在加载
      list: null, // banner列表
      total: 0, // 数据库中的总记录数
      page: 1, // 默认页码
      limit: 10, // 每页记录数
      searchObj: {}, // 查询表单对象
      multipleSelection: [], // 批量选择中选择的记录列表

      categoryList: []
    }
  },

  // 生命周期函数：内存准备完毕，页面尚未渲染
  created () {
    console.log('list created......')
    this.fetchData()
    this.fetchCategoryList()
  },

  // 生命周期函数：内存准备完毕，页面渲染成功
  mounted () {
    console.log('list mounted......')
  },

  methods: {

    // 当页码发生改变的时候
    changeSize (size) {
      console.log(size)
      this.limit = size
      this.fetchData(1)
    },

    add () {
      this.$router.push({ path: '/product/skuInfo/add' })
    },

    // 加载banner列表数据
    fetchData (page = 1) {
      console.log('翻页。。。' + page)
      // 异步获取远程数据（ajax）
      this.page = page

      api.getPageList(this.page, this.limit, this.searchObj).then(
        response => {
          debugger
          this.list = response.data.records
          this.total = response.data.total

          // 数据加载并绑定成功
          this.listLoading = false
        }
      )
    },

    fetchCategoryList () {
      categoryApi.findAllList().then(response => {
        this.categoryList = response.data
      })
    },

    // 重置查询表单
    resetData () {
      console.log('重置查询表单')
      this.searchObj = {}
      this.fetchData()
    },

    // 根据id删除数据
    removeDataById (id) {
      // debugger
      this.$confirm('此操作将永久删除该记录, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => { // promise
        // 点击确定，远程调用ajax
        return api.removeById(id)
      }).then((response) => {
        this.fetchData(this.page)
        if (response.code) {
          this.$message({
            type: 'success',
            message: '删除成功!'
          })
        }
      }).catch(() => {
        this.$message({
          type: 'info',
          message: '已取消删除'
        })
      })
    },

    // 当表格复选框选项发生变化的时候触发
    handleSelectionChange (selection) {
      console.log('handleSelectionChange......')
      console.log(selection)
      this.multipleSelection = selection
    },

    // 批量删除
    removeRows () {
      console.log('removeRows......')

      if (this.multipleSelection.length === 0) {
        this.$message({
          type: 'warning',
          message: '请选择要删除的记录!'
        })
        return
      }

      this.$confirm('此操作将永久删除该记录, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => { // promise
        // 点击确定，远程调用ajax
        // 遍历selection，将id取出放入id列表
        var idList = []
        this.multipleSelection.forEach(item => {
          idList.push(item.id)
          // console.log(idList)
        })
        // 调用api
        return api.removeRows(idList)
      }).then((response) => {
        this.fetchData(this.page)
        if (response.code) {
          this.$message({
            type: 'success',
            message: '删除成功!'
          })
        }
      }).catch(() => {
        this.$message({
          type: 'info',
          message: '已取消删除'
        })
      })
    },

    handlePublishStatusChange (index, row) {
      debugger
      if (row.checkStatus !== 1) {
        this.$message({
          type: 'info',
          message: '审核通过才可以上级或下架'
        })
        row.publishStatus = row.publishStatus === 1 ? 0 : 1
        return
      }
      api.publish(row.id, row.publishStatus).then(response => {
        this.$message({
          type: 'info',
          message: '操作成功'
        })
        this.fetchData()
      })
    },

    handleStatusChange (index, row) {
      debugger
      api.check(row.id, row.checkStatus).then(response => {
        this.$message({
          type: 'info',
          message: '操作成功'
        })
        this.fetchData()
      })
    },

    handleNewPersonChange (index, row) {
      debugger
      api.isNewPerson(row.id, row.isNewPerson).then(response => {
        this.$message({
          type: 'info',
          message: '操作成功'
        })
        this.fetchData()
      })
    }
  }
}
</script>

