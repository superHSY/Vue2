<template>
  <div>
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>商品管理</el-breadcrumb-item>
      <el-breadcrumb-item>商品分类</el-breadcrumb-item>
    </el-breadcrumb>

    <!-- 卡片视图 -->
    <el-card>
      <el-row>
        <el-col>
          <el-button type="primary" @click="showAdd">添加商品 </el-button>
        </el-col>
      </el-row>

      <!-- 表格占位 -->
      <tree-table
        :data="cateList"
        :columns="columns"
        :selection-type="false"
        :expand-type="false"
        show-index
        index-text="#"
        border
      >
        <!-- 是否有效 -->
        <template slot="isok" slot-scope="scope">
          <i
            class="el-icon-success"
            v-if="scope.row.cat_deleted === false"
            style="color: lightgreen"
          ></i>
          <i class="el-icon-error" v-else style="color: red"></i>
        </template>
        <!-- 排序 -->
        <template slot="order" slot-scope="scope">
          <el-tag size="mini" v-if="scope.row.cat_level === 0">一级</el-tag>
          <el-tag
            type="success"
            size="mini"
            v-else-if="scope.row.cat_level === 1"
            >二级</el-tag
          >
          <el-tag type="warning" size="mini" v-else>三级</el-tag>
        </template>
        <!-- 操作 -->
        <template slot="opt">
          <el-button type="primary" icon="el-icon-edit" size="mini"
            >编辑</el-button
          >
          <el-button type="warning" icon="el-icon-delete" size="mini"
            >删除</el-button
          >
        </template>
      </tree-table>
      <!-- 分页区域 -->
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

    <!-- 添加分类对话框 -->
    <el-dialog title="添加分类" :visible.sync="addCate" width="50%" @close="addclose">
      <span>
        <el-form
          :model="addCateForm"
          :rules="addCaterules"
          ref="addCateFormRef"
          label-width="100px"
        >
          <el-form-item label="分类名称:" prop="cat_name">
            <el-input v-model="addCateForm.cat_name"></el-input>
          </el-form-item>
          <el-form-item label="父级分类:">
            <el-cascader
			  expand-trigger="hover"
              v-model="select"
              :options="parentCateList"
              :props= "cascaderProps"
              @change="handleChange"
			  clearable change-on-select>
			  </el-cascader>
          </el-form-item>
        </el-form>
      </span>
      <span slot="footer" class="dialog-footer">
        <el-button @click="addCate = false">取 消</el-button>
        <el-button type="primary" @click="addCateNO">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data() {
    return {
      //查询数据
      querInfo: {
        type: 3,
        pagenum: 1,
        pagesize: 5
      },
      //商品数据默认为空
      cateList: [],
      //总数据条数
      total: 0,
      //控制对话框的显示
      addCate: false,
      //添加分类表单数据对象
      addCateForm: {
        cat_name: "",
        //父类ID
        cat_pid: 0,
        //一级分类（默认）
        cat_level: 0
      },
      //规则对象
      addCaterules: {
        cat_name: [
          { required: true, message: "请输入活动名称", trigger: "blur" }
        ]
      },
      //父级分类列表
      parentCateList: [],
	  //配置对象
	  cascaderProps: {
		  value:'cat_id',
		  label:'cat_name',
		  chiildren:'children'
	  },
	  //选中的父级分类id数组
	  select:[],
      columns: [
        {
          label: "商品名称",
          prop: "cat_name"
        },
        {
          label: "是否有效",
          //插槽模板
          type: "template",
          template: "isok"
        },
        {
          label: "排序",
          //插槽模板
          type: "template",
          template: "order"
        },
        {
          label: "操作",
          //插槽模板
          type: "template",
          template: "opt"
        }
      ]
    };
  },
  created() {
    this.getcateList();
  },
  methods: {
    //获取商品分类数据
    async getcateList() {
      const { data: res } = await this.$http.get("categories", {
        params: this.querInfo
      });
      if (res.meta.status !== 200)
        return this.$message.error("获取商品列表失败");
      console.log(res.data);
      this.cateList = res.data.result;
      this.total = res.data.total;
    },
    //监听handleSizeChange事件
    handleSizeChange(newSize) {
      this.querInfo.pagesize = newSize;
      this.getcateList();
    },
    //监听pagenum改变
    handleCurrentChange(newPage) {
      this.querInfo.pagenum = newPage;
      this.getcateList();
    },
    showAdd() {
      //获取父级列表
      this.getParentCateList();
      this.addCate = true;
    },
    //获取父级列表
    async getParentCateList() {
      const { data: res } = await this.$http.get("categories", {
        params: { type: 2 }
      });
      if (res.meta.status !== 200) return this.$message.error("获取数据失败");
      console.log(res.data)
      this.parentCateList = res.data;
    },
	//选中项发生变法触发函数
	handleChange() {
		console.log(this.select)
		//如果select数组的length>0 选中了父级分类
		if(this.select.length>0 ){
			//父级分类的ID
			this.addCateForm.cat_pid = this.select[this.select.length -1]
			this.addCateForm.cat_level = this.select.length
			return
		}
		else{
			this.addCateForm.cat_pid = 0
			this.addCateForm.cat_level = 0
		}
	},
	//点击按钮添加新的分类
	addCateNO() {
		console.log(this.addCateForm)
		this.$refs.addCateFormRef.validate(async valid => {
			//console.log(valid)
			if(!valid) return
			const {data: res} = await this.$http.post('categories', this.addCateForm)
			console.log(res)
			if(res.meta.status !== 201) return this.message.error('添加失败')
			this.$message.success('添加成功')
			this.getcateList()
			this.addCate = false
		})
	},
	//监听对话框的监听事件
	addclose() {
		this.$refs.addCateFormRef.resetFields()
		this.select = []
		this.addCateForm.cat_level = 0
		this.addCateForm.cat_pid = 0
	}
  }
};
</script>

<style lang="less" scoped>
.el-button {
  margin-bottom: 15px;
}

.el-cascader{
	width: 100%;
}
</style>
