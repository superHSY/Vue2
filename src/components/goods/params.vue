<template>
  <div>
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>参数列表</el-breadcrumb-item>
    </el-breadcrumb>

    <!-- 卡片视图 -->
    <el-card>
      <!-- 警告区域 -->
      <el-alert
        title="注意：只允许为第三级分类设置相关参数!"
        type="warning"
        :closable="false"
        show-icon
      >
      </el-alert>

      <!-- 选择商品分类 -->
      <el-row class="cat_opt">
        <el-col>
          <span>选择商品分类：</span>
          <el-cascader expand-trigger="hover" :options="cateList" :props="cateProps" v-model="selectedCateKeys" @change="handleChange">
          </el-cascader>
        </el-col>
      </el-row>

	  <!-- tab页面区域 -->
	 	 <el-tabs v-model="activeName" @tab-click="handleClick">
			<!-- 添加动态参数 -->
    		<el-tab-pane label="动态参数" name="first" >
				<el-button type="primary" size="mini" :disabled="isok">
					添加参数
				</el-button>
        <!-- 动态表格 -->
        <el-table  border stripe>
          <el-table-column type="index"></el-table-column>
          <el-table-column label="参数名称" prop="arr_name"></el-table-column>
          <el-table-column label="操作">
            <template >
              <el-button type="primary" icon="el-icon-edit">编辑</el-button>
              <el-button type="primary" icon="el-icon-delete">删除</el-button>
            </template>
          </el-table-column>

        </el-table>
			</el-tab-pane>
			<!-- 添加静态属性 -->
    		<el-tab-pane label="静态属性" name="second">
				<el-button type="primary" size="mini" :disabled="isok">静态属性</el-button>
				<!-- 静态属性表格 -->
         <el-table border stripe>
          <el-table-column type="index"></el-table-column>
          <el-table-column label="属性名称" prop="arr_name"></el-table-column>
          <el-table-column label="操作">
            <template >
              <el-button type="primary" icon="el-icon-edit">编辑</el-button>
              <el-button type="primary" icon="el-icon-delete">删除</el-button>
            </template>
          </el-table-column>

        </el-table>
        </el-tab-pane>
  		</el-tabs>
    </el-card>
  </div>
</template>

<script>
export default {
  data() {
    return {
      //商品分类
      cateList: [],
	  //级联双向绑定
	  selectedCateKeys:[],
	  //级联选择框配置对象
	  cateProps: {
        value: 'cat_id',
        label: 'cat_name',
        children: 'children'
      },
	  //激活页面名称
	  activeName:'first'
    };
  },
  created() {
    this.getCatalist();
  }, 
  methods: {
    async getCatalist() {
      const { data: res } = await this.$http.get("categories");
      if (res.meta.status !== 200) return this.$message.error("分类失败！");
      this.cateList = res.data;
      console.log(this.cateList);
	    //console.log(this.selectKeys.length);

    },
	//级联触发函数
	handleChange() {
		if(this.selectKeys.length !== 3){
			this.selectKeys = []
			return
		}
		console.log(this.selectKeys)
	},
	//处理函数
	handleClick() {
		console.log(this.activeName)
	}
  },
  computed: {
	  //如果按钮需要被禁用
	  isok() {
		  if(this.selectKeys !== 3){
			  return true
		  }
		  return false
	  }
   }
};

</script>

<style lang="less" scoped>
.cat_opt {
  margin: 15px 0;
}
</style>
