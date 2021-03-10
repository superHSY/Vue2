<template>
  <div>
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>角色列表</el-breadcrumb-item>
    </el-breadcrumb>

    <el-card>
      <el-row>
        <el-col>
          <!-- 添加用户 -->
          <el-button type="primary" @click="isadd = true">添加列表</el-button>
          <el-dialog
            title="添加列表"
            :visible.sync="isadd"
            width="50%"
            @close="addUser"
          >
            <span>
              <el-form
                :model="addUserForm"
                :rules="addUserrules"
                ref="addUserFormRef"
                label-width="100px"
              >
                <el-form-item label="角色名称" prop="roleName">
                  <el-input v-model="addUserForm.roleName"></el-input>
                </el-form-item>
                <el-form-item label="角色描述" prop="roleDesc">
                  <el-input v-model="addUserForm.roleDesc"></el-input>
                </el-form-item>
              </el-form>
            </span>
            <span slot="footer" class="dialog-footer">
              <el-button @click="isadd = false">取 消</el-button>
              <el-button type="primary" @click="isUser">确 定</el-button>
            </span>
          </el-dialog>
          <el-dialog
            title="修改角色"
            :visible.sync="editgo"
            width="50%"
            @close="editclosed"
          >
            <span>
              <el-form
                :model="editUserFrom"
                :rules="editrules"
                ref="editFromRef"
                label-width="100px"
              >
                <el-form-item label="角色名称" prop="roleName">
                  <el-input v-model="editUserFrom.roleName"></el-input>
                </el-form-item>
                <el-form-item label="角色描述">
                  <el-input v-model="editUserFrom.roleDesc"></el-input>
                </el-form-item>
              </el-form>
            </span>
            <span slot="footer" class="dialog-footer">
              <el-button @click="editgo = false">取 消</el-button>
              <el-button type="primary" @click="editInfo">确 定</el-button>
            </span>
          </el-dialog>
        </el-col>
      </el-row>

      <!-- 角色列表 -->
      <el-table :data="rolelist" border stripe>
        <!-- 展开列 -->
        <el-table-column type="expand">
          <template slot-scope="scope">
            <el-row
              :class="['bdbottom', i1 === 0 ? 'bdtop' : '', 'vcenter']"
              v-for="(item1, i1) in scope.row.children"
              :key="item1.id"
            >
              <!-- 一级权限 -->
              <el-col :span="5">
                <el-tag closable @close="removeById(scope.row, item1.id)">
                  {{ item1.authName }}
                </el-tag>
                <i class="el-icon-caret-right"></i>
              </el-col>
              <!-- 二级权限 -->
              <el-col :span="19">
                <!-- for循环渲染二级权限 -->
                <el-row
                  :class="[i2 === 0 ? '' : 'bdtop', 'vcenter']"
                  v-for="(item2, i2) in item1.children"
                  :key="item2.id"
                >
                  <el-col :span="6">
                    <el-tag
                      type="success"
                      closable
                      @close="removeById(scope.row, item2.id)"
                    >
                      {{ item2.authName }}
                    </el-tag>
                    <i class="el-icon-caret-right"></i>
                  </el-col>
                  <el-col :span="18">
                    <el-tag
                      v-for="item3 in item2.children"
                      :key="item3.id"
                      type="warning"
                      closable
                      @close="removeById(scope.row, item3.id)"
                    >
                      {{ item3.authName }}
                    </el-tag>
                  </el-col>
                </el-row>
              </el-col>
            </el-row>
            
            <!--<pre>
            {{ scope.row }}
            </pre> -->
          </template>
        </el-table-column>
        <!-- 索引列 -->
        <el-table-column type="index"></el-table-column>
        <el-table-column label="角色名称" prop="roleName"></el-table-column>
        <el-table-column label="角色描述" prop="roleDesc"></el-table-column>
        <el-table-column label="操作" width="300px">
          <template slot-scope="scope">
            <el-button
              type="primary"
              icon="el-icon-edit"
              size="mini"
              @click="change(scope.row.id)"
              >编辑</el-button
            >
            <el-button
              type="danger"
              icon="el-icon-delete"
              size="mini"
              @click="remove(scope.row.id)"
              >删除</el-button
            >
            <el-button
              type="warning"
              icon="el-icon-setting"
              size="mini"
              @click="showSet(scope.row)"
              >分配权限</el-button
            >
          </template>
        </el-table-column>
      </el-table>
    </el-card>

    <!-- 分配权限对话框 -->
    <el-dialog
      title="分配权限"
      :visible.sync="setdialogVisible"
      width="50%"
      @close="setClose"
      >
      <span>
        <!-- 树形控件-->
        <el-tree :data="rightslist" :props="treeProps" 
        show-checkbox node-key='id' default-expand-all
        :default-checked-keys="defKeys" ref="treeRef"></el-tree>
      </span>
      <span slot="footer" class="dialog-footer">
        <el-button @click="setdialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="allotRights">确 定</el-button>
      </span>
    </el-dialog>

   
    
  </div>
</template>

<script>
export default {
  data() {
    return {
      //所有角色列表数据
      rolelist: [],
      //按钮关闭
      isadd: false,
      editgo: false,
       //控制分配权限对话框
      setdialogVisible: false,
      //所有权限的数据
      rightslist:[],
      //树形控件的绑定对象
      treeProps:{
        label: 'authName',
        children: 'children'
      },
      defKeys:[],
      //查询到角色信息对象
      editUserFrom: {},
      //绑定添加数据
      addUserForm: {
        roleName: "",
        roleDesc: ""
      },
      //即将分配权限的角色
      roleId:'',
      //添加角色规则
      addUserrules: {
        roleName: [
          { required: true, message: "请输入正确角色名", trigger: "blur" }
        ],
        roleDesc: [
          { required: true, message: "请输入角色描述", trigger: "blur" }
        ]
      },
      //编辑角色规则
      editrules: {
        roleName: [
          { required: true, message: "请输入正确角色名", trigger: "blur" }
        ]
      }
    };
  },
  created() {
    this.getRoles();
  },
  methods: {
    async getRoles() {
      const { data: res } = await this.$http.get("roles");
      if (res.meta.status !== 200) return this.$message.error("获取列表失败");
      this.rolelist = res.data;
      console.log(this.rolelist);
    },
    //表单重新刷新
    addUser() {
      this.$refs.addUserFormRef.resetFields();
    },
    //添加角色
    isUser() {
      this.$refs.addUserFormRef.validate(async vaild => {
        if (!vaild) return;
        const { data: res } = await this.$http.post("roles", this.addUserForm);
        console.log(res.data);
        if (res.meta.status !== 201)
          return this.$message.error("获取角色列表失败");
        this.$message.success("添加角色成功");
        //隐藏对话框
        this.isadd = false;
        //重新获得列表数据
        this.getRoles();
      });
    },
    //根据id查询客户
    async change(id) {
      console.log(id);
      const { data: res } = await this.$http.get("roles/" + id);
      if (res.meta.status !== 200) return this.$message.error("测试角色失败！");
      console.log(res.data);
      this.editUserFrom = res.data;
      this.editgo = true;
    },
    //表单刷新
    editclosed() {
      this.$refs.editFromRef.resetFields();
    },
    //编辑修改
    editInfo() {
      this.$refs.editFromRef.validate(async vaild => {
        //console.log(vaild)
        if (!vaild) return;
        const { data: res } = await this.$http.put(
          "roles/" + this.editUserFrom.roleId,
          {
            roleName: this.editUserFrom.roleName,
            roleDesc: this.editUserFrom.roleDesc
          }
        );
        //console.log( this.editUserFrom.id)
        if (res.meta.status !== 200) return this.$message.error("编辑失败！");
        this.$message.success("获取成功");
        this.editgo = false;
        this.getRoles();
      });
    },
    //删除列表表单
    async remove(id) {
      const deleteres = await this.$confirm(
        "此操作将永久删除该用户, 是否继续?",
        "提示",
        {
          confirmButtonText: "确定",
          cancelButtonText: "取消",
          type: "warning"
        }
      ).catch(error => {
        return error;
      });
      if (deleteres !== "confirm") {
        return this.$message.info("取消删除");
      }
      const { data: res } = await this.$http.delete("roles/" + id);
      if (res.meta.status !== 200) return this.$message.error("删除用户失败");
      this.$message.success("删除用户成功！");
      this.getRoles();
    },
    //删除tag标签
    async removeById(role, rightId) {
      const isdelete = await this.$confirm(
        "此操作将永久删除该文件, 是否继续?",
        "提示",
        {
          confirmButtonText: "确定",
          cancelButtonText: "取消",
          type: "warning"
        }
      ).catch(err => err);
      if (isdelete !== "confirm") {
        return this.$message.info("取消删除");
      }
      //console.log('确认删除')
      const { data: res } = await this.$http.delete(
        `roles/${role.id}/rights/${rightId}`
      );
      if (res.meta.status !== 200) return this.$message.error("删除权限失败！");
      this.$message.success("删除权限成功！");
      role.children = res.data;
      //this.getRoles()
    },
    //分配权限对话框
    async showSet(role) {
      //获取所有权限的数据
      this.roleId = role.id
      const {data: res} = await this.$http.get('rights/tree')
      if(res.meta.status !== 200) return this.$message.error('获取数据失败！')
      //保存数据
      this.rightslist = res.data
      console.log(this.rightslist)

      //递归获取节点id
      this.getKefs(role, this.defKeys)
      this.setdialogVisible = true
    },
    //默认选定的值
    //defKeys : []
    //通过递归的形式取的id并且保存到数组中
    getKefs(node, arr)  {
      //不包含children属性
      if( !node.children) {
        return arr.push(node.id)
      }
      node.children.forEach( item =>{
        this.getKefs(item, arr)
      })
    },
    //监听关闭对话框事件
    setClose() {
      this.defKeys = []
    },
    //点击分配权限
    async allotRights() {
      const keys = [
        //展开运算符
        ...this.$refs.treeRef.getCheckedKeys(),
        ...this.$refs.treeRef.getHalfCheckedNodes()
      ]

      const idStr = keys.join(',')
      const {data: res} = await this.$http.post(`roles/${this.roleId}/rights`, { rids: idStr})
      
      if(res.meta.status !== 200) return this.$message.error('更新失败！')
      this.$message.success('分配权限成功')
      this.getRoles()
      this.setdialogVisible = false
      console.log(keys)
    }
  }
};
</script>

<style lang="less" scope>
.el-tag {
  margin: 10px;
}

.bdtop {
  border-top: 1px solid #eee;
}

.bdbottom {
  border-bottom: 1px solid #eee;
}

.vcenter {
  display: flex;
  align-items: center;
}
</style>
