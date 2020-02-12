<template>
  <div>
    <!-- 面包导航区域 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>订单管理</el-breadcrumb-item>
      <el-breadcrumb-item>订单列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片视图区域 -->
    <el-card class="box-card">
      <!-- 搜索框区域 -->
      <el-row :gutter="20">
        <el-col :span="8">
          <el-input
            placeholder="请输入内容"
            v-model="queryInfo.query"
            clearable
            @clear="getOrderList"
          >
            <el-button
              slot="append"
              icon="el-icon-search"
              @click="getOrderList"
            ></el-button>
          </el-input>
        </el-col>
      </el-row>
      <!-- 订单表格区域 -->
      <el-table :data="orderList" border stripe>
        <el-table-column type="index" label="#"> </el-table-column>
        <el-table-column label="订单编号" prop="order_number">
        </el-table-column>
        <el-table-column label="订单价格" prop="order_price" width="120px">
        </el-table-column>
        <el-table-column label="是否付款" prop="pay_status" width="100px">
          <template slot-scope="scope">
            <el-tag type="danger" v-if="scope.row.pay_status === '0'"
              >没付款</el-tag
            >
            <el-tag type="success" v-else>已付款</el-tag>
          </template>
        </el-table-column>
        <el-table-column label="是否发货" prop="is_send" width="90px">
        </el-table-column>
        <el-table-column label="下单时间" prop="create_time" width="160px">
          <template slot-scope="scope">
            {{ scope.row.create_time | dateFormat }}
          </template>
        </el-table-column>
        <el-table-column label="操作" width="180px">
          <template>
            <!--修改地址  -->
            <el-tooltip
              class="item"
              effect="dark"
              content="修改地址"
              placement="top"
              :enterable="true"
            >
              <el-button
                size="mini"
                type="primary"
                icon="el-icon-location-information"
                @click="showEditAddressDialog"
              ></el-button>
            </el-tooltip>
            <!-- 查看物流 -->
            <el-tooltip
              class="item"
              effect="dark"
              content="查看物流"
              placement="top"
              :enterable="true"
            >
              <el-button
                size="mini"
                type="success"
                icon="el-icon-truck"
                @click="showLogisticsDialog"
              ></el-button>
            </el-tooltip>
            <!-- 修改订单状态 -->
            <el-tooltip
              class="item"
              effect="dark"
              content="修改订单状态"
              placement="top"
              :enterable="true"
            >
              <el-button
                size="mini"
                type="warning"
                icon="el-icon-s-order"
              ></el-button>
            </el-tooltip>
          </template>
        </el-table-column>
      </el-table>

      <!-- 分页区域 -->
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="queryInfo.num"
        :page-sizes="[5, 10, 15]"
        :page-size="queryInfo.pagesize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total"
        background
      >
      </el-pagination>
    </el-card>
    <!-- 修改地址对话框 -->
    <el-dialog
      title="修改地址"
      :visible.sync="editAddressDialogVisible"
      width="50%"
      @close="editAddressDialogClosed"
    >
      <el-form
        :model="editAddressForm"
        :rules="editAddressFormRules"
        ref="editAddressFormRef"
        label-width="100px"
      >
        <el-form-item label="省市区/县" prop="address1">
          <el-cascader
            v-model="editAddressForm.address1"
            :options="cityData"
            :props="{ expandTrigger: 'hover' }"
          ></el-cascader>
        </el-form-item>
        <el-form-item label="详细地址" prop="address2">
          <el-input v-model="editAddressForm.address2"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="editAddressDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="editAddressDialogVisible = false"
          >确 定</el-button
        >
      </span>
    </el-dialog>

    <!-- 查看物流对话框 -->
    <el-dialog title="物流" :visible.sync="logisticsDialogVisible" width="50%">
      <el-timeline-item>
        <el-timeline-item
          v-for="(activity, index) in logisticsInfo"
          :key="index"
          :timestamp="activity.time"
        >
          {{ activity.context }}
        </el-timeline-item>
      </el-timeline-item>
    </el-dialog>
  </div>
</template>
<script>
import cityData from './city_data2017_element'
export default {
  data() {
    return {
      queryInfo: {
        query: '',
        pagenum: 1,
        pagesize: 10
      },
      //   订单列表
      orderList: [],
      // 订单总条数
      total: 0,
      //   控制修改地址对话框的显示与隐藏
      editAddressDialogVisible: false,
      //   修改地址的表单数据对象
      editAddressForm: {
        address1: [],
        address2: ''
      },
      //   修改地址表单的验证规则
      editAddressFormRules: {
        address1: [
          {
            required: true,
            message: '请选择省市区/县',
            trigger: 'blur'
          }
        ],
        address2: [
          {
            required: true,
            message: '请输入详细的地址',
            trigger: 'blur'
          }
        ]
      },
      //   省市区
      cityData,
      //   控制物流对话框的显示与隐藏
      logisticsDialogVisible: false,
      //   物流信息
      logisticsInfo: []
    }
  },
  created() {
    this.getOrderList()
  },
  methods: {
    //   获取订单列表数据
    async getOrderList() {
      const { data: res } = await this.$http.get('orders', {
        params: this.queryInfo
      })
      if (res.meta.status !== 200) {
        return this.$message.error('获取订单列表数据失败')
      }
      this.orderList = res.data.goods
      this.total = res.data.total
    },
    // 每一页显示条数pagesize发生改变时触发
    handleSizeChange(newSize) {
      this.queryInfo.pagesize = newSize
      this.getOrderList()
    },
    // 当前页pagenum发生改变时触发
    handleCurrentChange(newPage) {
      this.queryInfo.pagenum = newPage
      this.getOrderList()
    },
    // 展示修改地址对话框
    showEditAddressDialog() {
      this.editAddressDialogVisible = true
    },
    // 监听修改地址对话框关闭事件
    editAddressDialogClosed() {
      this.$refs.editAddressFormRef.resetFields()
    },
    // 展示物流对话框
    async showLogisticsDialog() {
      const { data: res } = await this.$http.get('/kuaidi/' + 1106975712662)
      if (res.meta.status !== 200) {
        return this.$message.error('获取物流数据失败')
      }
      this.logisticsInfo = res.data
      this.logisticsDialogVisible = true
    }
  }
}
</script>
<style lang="less" scoped>
.el-table {
  margin-top: 15px;
}
.el-cascader {
  width: 100%;
}
</style>
