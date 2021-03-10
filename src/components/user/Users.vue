<template>
  <div>
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>用户管理</el-breadcrumb-item>
      <el-breadcrumb-item>用户列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!--卡片视图-->
    <el-card>
      <el-row :gutter="20">
        <el-col :span="8">
          <el-input
            placeholder="请输入内容"
            v-model="queryInfo.query"
            clearable
            @clear="getUser()"
          >
            <el-button
              slot="append"
              icon="el-icon-search"
              @click="getUser"
            ></el-button>
          </el-input>
        </el-col>
        <el-col :span="4">
          <el-button type="primary" @click="addBoolen = true"
            >添加用户</el-button
          >
        </el-col>
      </el-row>
      <!--用户列表-->
      <el-table :data="userlist" style="width: 100%" border stripe>
        <el-table-column type="index"></el-table-column>
        <el-table-column prop="username" label="姓名"> </el-table-column>
        <el-table-column prop="email" label="邮箱"> </el-table-column>
        <el-table-column prop="mobile" label="电话"> </el-table-column>
        <el-table-column prop="role_name" label="角色"> </el-table-column>
        <el-table-column label="状态">
          <template slot-scope="scope">
            <el-switch
              v-model="scope.row.mg_state"
              active-color="#13ce66"
              inactive-color="#ff4949"
              @change="userChange(scope.row)"
            >
            </el-switch>
          </template>
        </el-table-column>
        <el-table-column label="操作" width="180px">
          <template slot-scope="scope">
            <el-button
              type="primary"
              icon="el-icon-edit"
              size="mini"
              @click="edit(scope.row.id)"
            ></el-button>
            <el-button
              type="danger"
              icon="el-icon-delete"
              size="mini"
              @click="remove(scope.row.id)"
            ></el-button>
            <el-tooltip
              effect="dark"
              content="分配角色"
              placement="top"
              :enterable="false"
            >
              <el-button
                type="warning"
                icon="el-icon-setting"
                size="mini"
                @click="setRole(scope.row)"
              ></el-button>
            </el-tooltip>
          </template>
        </el-table-column>
      </el-table>
      <!-- 分页区域 -->
      <el-pagination
        @size-change="handleSizeChange"
        @current-change="handleCurrentChange"
        :current-page="queryInfo.pagenum"
        :page-sizes="[1, 2, 5, 10]"
        :page-size="queryInfo.pagesize"
        layout="total, sizes, prev, pager, next, jumper"
        :total="total"
      >
      </el-pagination>
    </el-card>
    <!-- 添加用户对话框 -->
    <el-dialog
      title="添加用户"
      :visible.sync="addBoolen"
      width="50%"
      @close="addclose"
    >
      <span>
        <el-form
          :model="addFrom"
          :rules="addrules"
          ref="ruleFormRef"
          label-width="100px"
        >
          <el-form-item label="用户名" prop="username">
            <el-input v-model="addFrom.username"></el-input>
          </el-form-item>
          <el-form-item label="密码" prop="password">
            <el-input v-model="addFrom.password"></el-input>
          </el-form-item>
          <el-form-item label="邮箱" prop="email">
            <el-input v-model="addFrom.email"></el-input>
          </el-form-item>
          <el-form-item label="手机" prop="mobile">
            <el-input v-model="addFrom.mobile"></el-input>
          </el-form-item>
        </el-form>
      </span>
      <span slot="footer" class="dialog-footer">
        <el-button @click="addBoolen = false">取 消</el-button>
        <el-button type="primary" @click="adduser">确 定</el-button>
      </span>
    </el-dialog>
    <!-- 修改用户的对话框-->
    <el-dialog
      title="修改用户"
      :visible.sync="editchange"
      width="50%"
      @close="editclose"
    >
      <span>
        <el-form
          :model="editFrom"
          :rules="editrules"
          ref="editFormRef"
          label-width="70px"
        >
          <el-form-item label="用户名">
            <el-input v-model="editFrom.username" disabled></el-input>
          </el-form-item>
          <el-form-item label="邮箱" prop="email">
            <el-input v-model="editFrom.email"></el-input>
          </el-form-item>
          <el-form-item label="手机号" prop="mobile">
            <el-input v-model="editFrom.mobile"></el-input>
          </el-form-item>
        </el-form>
      </span>
      <span slot="footer" class="dialog-footer">
        <el-button @click="editchange = false">取 消</el-button>
        <el-button type="primary" @click="edituserInfo">确 定</el-button>
      </span>
    </el-dialog>
    <!-- 分配角色对话框 -->
    <el-dialog title="提示" :visible.sync="setUserdialogVisible" width="50%" @close="changeset">
      <div>
        <p>当前用户：{{ userInfo.username }}</p>
        <p>当前角色：{{ userInfo.role_name }}</p>
        <p>
          分配角色：
          <el-select v-model="seletRoles" placeholder="请选择">
            <el-option
              v-for="item in setlist"
              :key="item.id"
              :label="item.roleName"
              :value="item.id"
            >
            </el-option>
          </el-select>
        </p>
      </div>
      <span slot="footer" class="dialog-footer">
        <el-button @click="setUserdialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="save"
          >确 定</el-button
        >
      </span>
    </el-dialog>
  </div>
</template>

<script>
export default {
  data() {
    //自定义规则
    //邮箱
    var checkemail = (rule, value, callback) => {
      const regemail = /^[a-zA-Z0-9_-]+@[a-zA-Z0-9_-]+(\.[a-zA-Z0-9_-]+)+$/;
      if (regemail.test(value)) {
        return callback();
      }

      callback(new Error("请输入合法邮箱"));
    };
    //电话
    var checkphone = (rule, value, callback) => {
      const regphone = /^(0|86|17951)?(13[0-9]|15[012356789]|17[678]|18[0-9]|14[57])[0-9]{8}$/;
      if (regphone.test(value)) {
        return callback();
      }

      callback(new Error("请输入合法手机号"));
    };
    return {
      //获取用户列表对象
      queryInfo: {
        query: "",
        //当前的页数
        pagenum: 1,
        //每页显示几条数据
        pagesize: 2
      },
      userlist: [],
      total: 0,
      //控制添加用户对话框的显示与隐藏
      addBoolen: false,
      //控制修改用户
      editchange: false,
      //控制分配角色对话框
      setUserdialogVisible: false,
      //需要被分配角色的用户信息
      userInfo: {},
      //所有角色列表
      setlist: [],
      //已选中的ID值
      seletRoles:[],
      //查询到的用户对象
      editFrom: {
        email: "",
        mobile: ""
      },
      //修改表单的验证对象
      editrules: {
        email: [
          { required: true, message: "请输入修改邮箱", trigger: "blur" },
          { validator: checkemail, trigger: "blur" }
        ],
        mobile: [
          { required: true, message: "请输入修改手机号", trigger: "blur" },
          { validator: checkphone, trigger: "blur" }
        ]
      },
      //添加用户表单数据
      addFrom: {
        username: "",
        password: "",
        email: "",
        mobile: ""
      },
      //添加表单验证规则对象
      addrules: {
        username: [
          { required: true, message: "请输入用户名", trigger: "blur" },
          { min: 3, max: 10, message: "长度在 3 到 10 个字符", trigger: "blur" }
        ],
        password: [
          { required: true, message: "请输入密码", trigger: "blur" },
          { min: 5, max: 15, message: "长度在 5 到 15 个字符", trigger: "blur" }
        ],
        email: [
          { required: true, message: "请输入邮箱", trigger: "blur" },
          { validator: checkemail, trigger: "blur" }
        ],
        mobile: [
          { required: true, message: "请输入手机号", trigger: "blur" },
          { validator: checkphone, trigger: "blur" }
        ]
      }
    };
  },
  created() {
    this.getUser();
  },
  methods: {
    async getUser() {
      const { data: res } = await this.$http.get("users", {
        params: this.queryInfo
      });
      console.log(res);
      if (res.meta.status !== 200) return this.$message.error("获取失败");
      this.userlist = res.data.users;
      this.total = res.data.total;
    },
    //监听改变的事件
    handleSizeChange(newSize) {
      console.log(newSize);
      this.queryInfo.pagesize = newSize;
      this.getUser();
    },
    //监听页码值的事件
    handleCurrentChange(newPage) {
      this.queryInfo.pagenum = newPage;
      this.getUser();
    },
    //监听开关的改变
    async userChange(userinfo) {
      const { data: res } = await this.$http.put(
        `users/${userinfo.id}/state/${userinfo.mg_state}`
      );
      if (res.meta.status !== 200) {
        userinfo.mg_state = !userinfo.mg_state;
        return this.$message.error("更新用户失败！");
      }
      this.$message.success("更新用户成功");
    },
    //监听表单关闭
    addclose() {
      this.$refs.ruleFormRef.resetFields();
    },
    //点击按钮 添加用户
    adduser() {
      this.$refs.ruleFormRef.validate(async vaild => {
        if (!vaild) return;
        //发起请求
        const { data: res } = await this.$http.post("users", this.addFrom);
        if (res.meta.status !== 201) return this.$message.error("添加用户失败");
        this.$message.success("添加用户成功！");
        //隐藏对话框
        this.addBoolen = false;
        //重新获得列表数据
        this.getUser();
      });
    },
    //展示编辑用户对话框
    async edit(id) {
      console.log(id);
      const { data: res } = await this.$http.get("users/" + id);
      if (res.meta.status !== 200) return this.$message.error("查询用户失败！");
      console.log(res.data);
      this.editFrom = res.data;
      this.editchange = true;
    },
    //监听修改用户对话框关闭事件
    editclose() {
      this.$refs.editFormRef.resetFields();
    },
    //修改用户信息提交
    edituserInfo() {
      this.$refs.editFormRef.validate(async vaild => {
        if (!vaild) return;
        //发起修改用户信息数据请求
        const { data: res } = await this.$http.put(
          "users/" + this.editFrom.id,
          { email: this.editFrom.email, mobile: this.editFrom.mobile }
        );
        console.log(this.editFrom.id);
        if (res.meta.status !== 200) return this.$message.error("更新用户失败");
        //关闭对话框
        this.editchange = false;
        //刷新列表
        this.getUser();
        //更新用户成功
        this.$message.success("更新用户成功");
      });
    },
    //根据id删除
    async remove(id) {
      //弹框删除
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
      const { data: res } = await this.$http.delete("users/" + id);
      if (res.meta.status !== 200) return this.$message.error("删除用户失败");
      this.$message.success("删除用户成功！");
      this.getUser();
    },
    //展示分配角色对话框
    async setRole(userInfo) {
      //获取所有角色列表
      const { data: res } = await this.$http.get("roles");
      if (res.meta.status !== 200) return this.$message.error("获取失败");
      this.setlist = res.data;
      this.setUserdialogVisible = true;
      this.userInfo = userInfo;
    },
    async save() {
      if(!this.seletRoles) {
        return this.$message.error('请选择要分配的角色')
      }
      const {data: res} = await this.$http.put(`users/${this.userInfo.id}/role`, {rid: this.seletRoles})
      if(res.meta.status!==200) return this.$message.error('更新角色失败！')
      this.$message.success('更新用户成功')
      this.getUser()
      this.setUserdialogVisible = false
    },
    //监听分配角色对话框
    changeset() {
      this.seletRoles = ''
      this.userInfo = {}
    }
  }
};
</script>

<style lang="less" scope></style>
