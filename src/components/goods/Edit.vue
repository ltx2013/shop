<template>
  <div>
    <!-- 面包导航区域 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item :to="{ path: '/goods' }">商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>编辑商品</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片视图区域 -->
    <el-card>
      <!-- 提示区域 -->
      <el-alert
        title="编辑商品信息"
        type="info"
        show-icon
        :closable="false"
        center
      ></el-alert>
      <!-- 步骤条区域 -->
      <el-steps
        :space="200"
        :active="activeIndex - 0"
        finish-status="success"
        align-center
      >
        <el-step title="基本信息"></el-step>
        <el-step title="商品参数"></el-step>
        <el-step title="商品属性"></el-step>
        <el-step title="商品图片"></el-step>
        <el-step title="商品内容"></el-step>
        <el-step title="完成"></el-step>
      </el-steps>
      <!-- tab栏区域 -->
      <el-form
        :model="editForm"
        :rules="editFormRules"
        ref="editFormRef"
        label-width="100px"
        label-position="top"
      >
        <el-tabs
          tab-position="left"
          v-model="activeIndex"
          :before-leave="beforeTabLeave"
          @tab-click="tabClicked"
        >
          <el-tab-pane label="基本信息" name="0">
            <el-form-item label="商品名称" prop="goods_name">
              <el-input v-model="editForm.goods_name"></el-input>
            </el-form-item>
            <el-form-item label="商品价格(/元)" prop="goods_price">
              <el-input v-model="editForm.goods_price" type="number"></el-input>
            </el-form-item>
            <el-form-item label="商品重量(/kg)" prop="goods_weight">
              <el-input
                v-model="editForm.goods_weight"
                type="number"
              ></el-input>
            </el-form-item>
            <el-form-item label="商品数量" prop="goods_number">
              <el-input
                v-model="editForm.goods_number"
                type="number"
              ></el-input>
            </el-form-item>
            <el-form-item label="商品分类" prop="goods_cat">
              <!-- 级联选择器 -->
              <el-cascader
                v-model="editForm.goods_cat"
                :options="cateList"
                :props="cateProps"
                @change="handleChange"
              ></el-cascader>
            </el-form-item>
          </el-tab-pane>
          <el-tab-pane label="商品参数" name="1">
            <!-- 渲染表单的item项 -->
            <el-form-item
              :label="item.attr_name"
              v-for="item in manyTableData"
              :key="item.attr_id"
            >
              <el-checkbox-group v-model="item.attr_vals">
                <el-checkbox
                  border
                  :label="it"
                  v-for="(it, index) in item.attr_vals"
                  :key="index"
                ></el-checkbox>
              </el-checkbox-group>
            </el-form-item>
          </el-tab-pane>

          <el-tab-pane label="商品属性" name="2">
            <el-form-item
              :label="item.attr_name"
              v-for="item in onlyTableData"
              :key="item.attr_id"
            >
              <el-input v-model="item.attr_vals"></el-input>
            </el-form-item>
          </el-tab-pane>
          <el-tab-pane label="商品图片" name="3">
            <!-- action 表示图片上传到后台的API地址 -->
            <el-upload
              :action="uploadURL"
              :on-preview="handlePreview"
              :on-remove="handleRemove"
              list-type="picture"
              :headers="headerObj"
              :on-success="handleSuccess"
              :file-list="fileList"
              :limit="limit"
            >
              <el-button size="small" type="primary">点击上传</el-button>
            </el-upload>
          </el-tab-pane>
          <el-tab-pane label="商品内容" name="4">
            <!-- 富文本编辑器组件 -->
            <quill-editor v-model="editForm.goods_introduce"></quill-editor>
            <!-- 编辑商品的按钮 -->
            <el-button type="primary" @click="editGood">编辑商品</el-button>
          </el-tab-pane>
        </el-tabs>
      </el-form>
    </el-card>
    <!-- 图片预览对话框 -->
    <el-dialog title="图片预览" :visible.sync="previewVisible" width="50%">
      <div>
        <el-image style="width: 100%;" :src="previewURL"> </el-image>
      </div>
    </el-dialog>
  </div>
</template>
<script>
import _ from 'lodash'
export default {
  data() {
    let checkNumber = (rule, value, cb) => {
      if (value <= 0) {
        return cb(new Error('不能小于或等于0'))
      }
      return cb()
    }
    return {
      activeIndex: '0',
      // 编辑商品的表单数据对象
      editForm: {
        goods_name: '',
        goods_price: '',
        goods_weight: '',
        goods_number: '',
        // 商品所属的分类
        goods_cat: [],
        // 上传图片的数组
        pics: [],
        // 商品的详情描述
        goods_introduce: '',
        attrs: []
      },
      // 编辑表单的验证规则
      editFormRules: {
        goods_name: [
          {
            required: true,
            message: '请输入商品名称',
            trigger: 'blur'
          }
        ],
        goods_price: [
          {
            required: true,
            message: '请输入商品的价格',
            trigger: 'blur'
          },
          { validator: checkNumber, trigger: ['change', 'blur'] }
        ],
        goods_weight: [
          {
            required: true,
            message: '请输入商品的重量',
            trigger: 'blur'
          },
          { validator: checkNumber, trigger: ['change', 'blur'] }
        ],
        goods_number: [
          {
            required: true,
            message: '请输入商品的数量',
            trigger: 'blur'
          },
          { validator: checkNumber, trigger: ['change', 'blur'] }
        ],
        goods_cat: [
          {
            required: true,
            message: '请选择商品的分类',
            trigger: 'blur'
          }
        ]
      },
      // 所有商品分类列表数据
      cateList: [],
      // 级联选择器的配置对象
      cateProps: {
        expandTrigger: 'hover',
        value: 'cat_id',
        label: 'cat_name',
        children: 'children'
      },
      // 动态参数数据列表
      manyTableData: [],
      // 静态属性
      onlyTableData: [],
      // 上传图片的的URL地址
      uploadURL: 'http://127.0.0.1:8888/api/private/v1/upload',
      // 图片上传的请求头对象
      headerObj: {
        Authorization: window.localStorage.getItem('token')
      },
      // 控制预览图片的显示与隐藏
      previewVisible: false,
      // 预览图片的url地址
      previewURL: '',
      // 图片列表
      fileList: [],
      // 限制上传图片的数量
      limit: 0
    }
  },
  props: ['goodsId'],
  created() {
    this.getGoodInfo()
    this.getCateList()
  },
  methods: {
    // 获取当前编辑商品的信息
    async getGoodInfo() {
      const { data: res } = await this.$http.get(
        'goods/' + this.$route.params.goodsId
      )
      if (res.meta.status !== 200) {
        this.$message.error('获取编辑商品数据失败')
      }
      res.data.goods_cat = res.data.goods_cat.split(',')
      this.editForm = res.data
      this.editForm.goods_cat.forEach((item, i) => {
        this.editForm.goods_cat[i] = parseInt(item)
      })
      this.editForm.pics.forEach(item => {
        this.fileList.push({ url: item.pics_sma_url })
      })
      console.log(this.editForm)
    },
    // 获取所有商品分类
    async getCateList() {
      const { data: res } = await this.$http.get('categories')
      if (res.meta.status !== 200) {
        return this.$message.error('获取所有商品分类失败')
      }
      this.cateList = res.data
    },
    // 级联选择器的选择项发生变化会触发该函数
    handleChange() {
      if (this.editForm.goods_cat.length !== 3) {
        this.editForm.goods_cat = []
      }
    },
    beforeTabLeave(activeName, oldActiveName) {
      if (oldActiveName === '0' && this.editForm.goods_cat.length !== 3) {
        this.$message.error('请选择商品分类!')
        return false
      }
    },
    async tabClicked() {
      // 说明访问动态参数面板
      if (this.activeIndex === '1') {
        const { data: res } = await this.$http.get(
          `categories/${this.cateId}/attributes`,
          {
            params: {
              sel: 'many'
            }
          }
        )
        if (res.meta.status !== 200) {
          return this.$message.error('获取商品动态参数失败')
        }
        res.data.forEach(item => {
          item.attr_vals =
            item.attr_vals.length === 0 ? [] : item.attr_vals.split(' ')
        })
        this.manyTableData = res.data
      } else if (this.activeIndex === '2') {
        const { data: res } = await this.$http.get(
          `categories/${this.cateId}/attributes`,
          {
            params: {
              sel: 'only'
            }
          }
        )
        if (res.meta.status !== 200) {
          return this.$message.error('获取商品分类静态属性失败')
        }
        this.onlyTableData = res.data
      }
    },
    // 处理图片的预览
    handlePreview(file) {
      this.previewURL = file.response ? file.response.data.url : file.url
      this.previewVisible = true
    },
    // 处理图片移除的操作
    handleRemove(file) {
      console.log(file)
      let filePath = ''
      if (file.response) {
        // 1.获取将要删除的图片的临时路径
        filePath = file.response.data.tmp_path
      } else {
        filePath = file.url
      }
      // 2.从pics数组中,找到这个图片对应的索引值
      const i = this.editForm.pics.findIndex(item => {
        return file.response
          ? item.pic === filePath
          : item.pics_sma_url === filePath
      })
      // 3.调用数组的splice方法,把图片信息对象,从pics数组中移除
      this.editForm.pics.splice(i, 1)
      console.log(this.editForm)
    },
    // 图片上传成功后会回调该函数
    handleSuccess(response) {
      // 拼接得到一个图片信息对象
      const picInfo = {
        pic: response.data.tmp_path
      }
      // 将图片信息对象push到pics数组中
      this.editForm.pics.push(picInfo)
    },
    // 编辑商品
    editGood() {
      this.$refs.editFormRef.validate(async valid => {
        if (!valid) {
          return this.$message.error('请填写必要的表单项')
        }
        const form = _.cloneDeep(this.editForm)
        form.goods_cat = form.goods_cat.join(',')
        // 处理动态参数
        this.manyTableData.forEach(item => {
          const newInfo = {
            attr_id: item.attr_id,
            attr_value: item.attr_vals.join(' ')
          }
          this.editForm.attrs.push(newInfo)
        })
        // 处理静态属性
        this.onlyTableData.forEach(item => {
          const newInfo = { attr_id: item.attr_id, attr_value: item.attr_vals }
          this.editForm.attrs.push(newInfo)
        })
        form.attrs = this.editForm.attrs
        console.log(form)

        const { data: res } = await this.$http.put(
          'goods/' + this.$route.params.goodsId,
          form
        )
        if (res.meta.status !== 200) {
          return this.$message.error('编辑商品失败')
        }
        this.$message.success('编辑商品成功')
        this.$router.push('/goods')
      })
    }
  },
  computed: {
    cateId() {
      if (this.editForm.goods_cat.length === 3) {
        return this.editForm.goods_cat[2]
      }
      return null
    }
  }
}
</script>
<style lang="less" scoped>
.el-checkbox {
  margin: 0 10px 0 0 !important;
}
.el-button {
  margin-top: 15px;
}
</style>
