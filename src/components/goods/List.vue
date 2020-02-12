<template>
  <div>
    <!-- 面包导航区域 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>商品列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片视图区域 -->
    <el-card>
      <!-- 搜索框与添加按钮区域 -->
      <el-row :gutter="20">
        <el-col :span="8">
          <el-input
            placeholder="请输入内容"
            v-model="queryInfo.query"
            clearable
            @clear="getGoodsList"
          >
            <el-button
              slot="append"
              icon="el-icon-search"
              @click="getGoodsList"
            ></el-button>
          </el-input>
        </el-col>
        <el-col :span="4">
          <el-button type="primary" @click="getAddPage">添加商品</el-button>
        </el-col>
      </el-row>
      <!-- 表格区域 -->
      <el-table :data="goodsList" border stripe>
        <el-table-column type="index" label="#"> </el-table-column>
        <el-table-column prop="goods_name" label="商品名称"> </el-table-column>
        <el-table-column prop="goods_price" label="商品价格(元)" width="105px">
        </el-table-column>
        <el-table-column prop="goods_weight" label="商品重量" width="80px">
        </el-table-column>
        <el-table-column prop="add_time" label="创建时间" width="160px">
          <template slot-scope="scope">
            {{ scope.row.add_time | dateFormat }}
          </template>
        </el-table-column>
        <el-table-column label="操作" width="120px">
          <template slot-scope="scope">
            <!-- 编辑按钮 -->
            <el-button
              type="primary"
              size="mini"
              icon="el-icon-edit"
              @click="getEditPage(scope.row.goods_id)"
            ></el-button>
            <!-- 删除按钮 -->
            <el-button
              type="danger"
              size="mini"
              icon="el-icon-delete"
              @click="removeById(scope.row.goods_id)"
            ></el-button>
          </template>
        </el-table-column>
      </el-table>

      <!-- 分页区域 -->
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="queryInfo.pagenum"
        :page-sizes="[5, 10, 15, 20]"
        :page-size="queryInfo.pagesize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total"
        background
      >
      </el-pagination>
    </el-card>
  </div>
</template>
<script>
export default {
  data() {
    const checkNum = (rule, value, callback) => {
      if (value < 0) {
        return callback(new Error('不能小于0'))
      }
      return callback()
    }
    return {
      // 查询参数对象
      queryInfo: {
        query: '',
        pagenum: 1,
        pagesize: 5
      },
      //  商品列表
      goodsList: [],
      // 总条数
      total: 0,
      // 控制编辑商品对话框的显示与隐藏
      editGoodDialogVisible: false,
      // 编辑商品表单数据对象
      editGoodForm: {
        goods_name: '',
        goods_price: '',
        goods_weight: '',
        goods_number: '',
        goods_introduce: '',
        pics: [],
        attrs: []
      },
      // 编辑商品表单的验证规则
      editGoodRules: {
        goods_name: [
          {
            required: true,
            message: '请输入商品的名称',
            trigger: 'blur'
          }
        ],
        goods_price: [
          {
            required: true,
            message: '请输入商品的价格',
            trigger: 'blur'
          },
          { validator: checkNum, trigger: ['blur', 'change'] }
        ],
        goods_weight: [
          {
            required: true,
            message: '请输入商品的重量',
            trigger: 'blur'
          },
          { validator: checkNum, trigger: ['blur', 'change'] }
        ],
        goods_number: [
          {
            required: true,
            message: '请输入商品的数量',
            trigger: 'blur'
          },
          { validator: checkNum, trigger: ['blur', 'change'] }
        ]
      }
    }
  },
  created() {
    this.getGoodsList()
  },
  methods: {
    //   根据分页获取对应的商品列表
    async getGoodsList() {
      if (this.queryInfo.pagenum >= 1) {
        const { data: res } = await this.$http.get('goods', {
          params: this.queryInfo
        })
        if (res.meta.status !== 200) {
          return this.$message.error('获取商品列表数据失败')
        }
        this.goodsList = res.data.goods
        this.total = res.data.total
        // 判断获取的商品数据是否为空，若为空则获取上一页数据
        if (this.goodsList.length <= 0) {
          // 说明商品数据为空,则请求上一页数据
          this.queryInfo.pagenum -= 1
          this.getGoodsList()
        }
      }
    },
    // 当前的pagesize,每一页显示的条数发生改变时触发该函数
    handleSizeChange(newSize) {
      this.queryInfo.pagesize = newSize
      this.getGoodsList()
    },
    // 当前的页码pagenum发生改变时触发
    handleCurrentChange(newNum) {
      this.queryInfo.pagenum = newNum
      this.getGoodsList()
    },
    // 根据商品id删除该商品
    async removeById(goodsId) {
      const confirmResult = await this.$confirm(
        '此操作将永久删除该商品, 是否继续?',
        '提示',
        {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }
      ).catch(err => err)
      if (confirmResult !== 'confirm') return
      //   发起请求删除对应id的商品
      const { data: res } = await this.$http.delete(`goods/${goodsId}`)
      if (res.meta.status !== 200) {
        return this.$message.error('删除商品失败')
      }
      this.$message.success('删除商品成功')
      this.getGoodsList()
    },
    // 跳转到添加商品页面
    getAddPage() {
      this.$router.push('/goods/add')
    },
    // 跳转到编辑商品页面
    getEditPage(goodsId) {
      this.$router.push(`/goods/edit/${goodsId}`)
    },
    // 展示编辑商品对话框
    async showEditGoodDialog(goodsId) {
      const { data: res } = await this.$http.get('goods/' + goodsId)
      if (res.meta.status !== 200) {
        this.$message.error('获取商品数据失败')
      }
      console.log(res)
      this.editGoodForm = res.data
      this.editGoodDialogVisible = true
    }
  }
}
</script>
<style lang="less" scoped>
.el-table {
  margin-top: 10px;
}
</style>
