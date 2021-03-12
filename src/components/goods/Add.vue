<template>
  <div>
    <!-- 导航区域 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>添加商品</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片视图 -->
    <el-card>
      <!--提示区域-->
      <el-alert
        title="添加商品信息"
        type="info"
        center
        show-icon
        :closable="false"
      >
      </el-alert>
      <!--步骤条-->
      <el-steps
        :space="200"
        :active="active - 0"
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

      <!-- tab -->
      <el-form
        :model="addForm"
        :rules="addrules"
        ref="addruleFormRef"
        label-width="100px"
        label-position="top"
      >
        <el-tabs v-model="active" :tab-position="'left'" :before-filter="befor">
          <el-tab-pane label="基本信息" name="0">
            <el-form-item label="商品名称" prop="goods_name">
              <el-input v-model="addForm.goods_name"></el-input>
            </el-form-item>
            <el-form-item label="商品价格" prop="goods_price">
              <el-input v-model="addForm.goods_price" type="number"></el-input>
            </el-form-item>
            <el-form-item label="商品重量" prop="goods_weight">
              <el-input v-model="addForm.goods_weight" type="number"></el-input>
            </el-form-item>
            <el-form-item label="商品数量" prop="goods_number">
              <el-input v-model="addForm.goods_number" type="number"></el-input>
            </el-form-item>
            <el-form-item label="商品分类" prop="goods_cat">
              <el-cascader
                v-model="catelist"
                :options="cateProp"
                :props="{ expandTrigger: 'hover' }"
                @change="handleChange"
              ></el-cascader>
            </el-form-item>
          </el-tab-pane>
          <el-tab-pane label="商品参数" name="1">配置管理</el-tab-pane>
          <el-tab-pane label="商品属性" name="2">角色管理</el-tab-pane>
          <el-tab-pane label="商品图片" name="3">角色管理</el-tab-pane>
          <el-tab-pane label="商品内容" name="4">定时任务补偿</el-tab-pane>
        </el-tabs>
      </el-form>
    </el-card>
  </div>
</template>

<script>
export default {
  data() {
    return {
      active: "0",
      //添加商品的表单
      addForm: {
        goods_name: "",
        goods_price: 0,
        goods_weight: 0,
        goods_cat: []
      },
      //列表
      catelist: [],
      cateProp: [{
        label: 'cat_name',
        value: 'cat_id',
        children: [{
		label: 'cat_name',
        value: 'cat_id',
		children: [{
		label: 'cat_name',
        value: 'cat_id',
		}]
		}]
      }],
      addrules: {
        goods_name: [
          { required: true, message: "请输入商品名称", trigger: "blur" }
        ],
        goods_price: [
          { required: true, message: "请输入商品价格", trigger: "blur" }
        ],
        goods_weight: [
          { required: true, message: "请输入商品重量", trigger: "blur" }
        ],
        goods_number: [
          { required: true, message: "请输入商品数量", trigger: "blur" }
        ],
        goods_cat: [
          { required: false, message: "请输入商品分类", trigger: "blur" }
        ]
      }
    };
  },
  created() {
    this.getCatList();
  },
  methods: {
    //获取商品所有数据
    async getCatList() {
      const { data: res } = await this.$http.get("categories");
      if (res.meta.status !== 200) {
        return this.$message.error("获取商品失败");
      }
      //this.$message.success('获取成功')
      this.catelist = res.data;
      console.log(this.catelist);
	  //console.log(res.data.id)
    },
    //选择器变化触发函数
    handleChange() {
      console.log(this.addForm.goods_cat);
    },
    befor(activeName, oldActiveName) {
      console.log(activeName, oldActiveName);
    }
  }
};
</script>

<style lang="less" scope></style>
