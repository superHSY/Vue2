<template>
  <div>
    <!-- 导航区域 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>商品列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片视图 -->
    <el-card>
      <el-row :gutter="20">
        <el-col :span="8">
          <el-input placeholder="请输入内容" v-model="queryInfo.query" clearable @clear="getGoodList">
            <el-button slot="append" icon="el-icon-search" @click="getGoodList"></el-button>
          </el-input>
        </el-col>
        <el-col :span="4">
          <el-button type="primary" @click="goAdd">添加商品</el-button>
        </el-col>
      </el-row>
      <!-- table -->
      <el-table :data="goodlist" border stripe>
        <el-table-column type="index"></el-table-column>
        <el-table-column label="商品名称" prop="goods_name"></el-table-column>
        <el-table-column
          label="商品价格(元)"
          prop="goods_price"
          width="95px"
        ></el-table-column>
        <el-table-column
          label="商品重量"
          prop="goods_weight"
          width="70px"
        ></el-table-column>
        <el-table-column label="创建时间" prop="add_time" width="180px">
          <template slot-scope="scope">
            {{ scope.row.add_time | dataFormat }}
          </template>
        </el-table-column>
        <el-table-column label="操作" width="130px">
          <template slot-scope="scope">
            <el-button
              type="primary"
              icon="el-icon-edit"
              size="mini"
            ></el-button>
            <el-button
              type="danger"
              icon="el-icon-delete"
              size="mini"
			  @click="remove(scope.row.goods_id)"
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
    return {
      queryInfo: {
        query: "",
        pagenum: 1,
        pagesize: 10
      },
      //商品列表
      goodlist: [],
      //商品的总条数
      total: 0
    };
  },
  created() {
    this.getGoodList();
  },
  methods: {
    //根据分类获得列表
    async getGoodList() {
      const { data: res } = await this.$http.get("goods", {
        params: this.queryInfo
      });
      if (res.meta.status !== 200) return this.$message.error("获取失败");
      this.$message.success("获取用户成功");
      console.log(res.data);
      this.goodlist = res.data.goods;
      this.total = res.data.total;
    },
	handleSizeChange(newsize) {
		this.queryInfo.pagesize = newsize
		this.getGoodList()
	},
	handleCurrentChange(newpage) {
		this.queryInfo.pagenum = newpage
		this.getGoodList()
	},
	async remove(id) {
		const confirmove = await this.$confirm('此操作将永久删除该商品, 是否继续?', '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).catch(err =>err)
		if(confirmove!== 'confirm') return this.$message.error('已经取消删除')
		const {data: res} = await this.$http.delete(`goods/${id}`)
		if(res.meta.status !== 200) return this.$$message.error('删除失败')
		this.$message.success('删除商品成功')
		this.getGoodList()
	},
	goAdd() {
		this.$router.push('/goods/add')
	}
  }
};
</script>

<style lang="less" scope></style>
