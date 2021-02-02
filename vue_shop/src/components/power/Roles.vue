<template>
  <div>
    <!-- 面包屑导航区域 -->
    <el-breadcrumb separator-class="el-icon-arrow-right">
      <el-breadcrumb-item :to="{ path: '/home' }">首页</el-breadcrumb-item>
      <el-breadcrumb-item>权限管理</el-breadcrumb-item>
      <el-breadcrumb-item>权限列表</el-breadcrumb-item>
    </el-breadcrumb>
    <!-- 卡片视图区域 -->
    <el-card>
      <!-- 添加角色按钮区域 -->
      <el-row>
        <el-col>
          <el-button type="primary" @click="addDialogVisible = true"
            >添加角色</el-button
          >
        </el-col>
      </el-row>

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
                <el-tag closable @close="removeRightById(scope.row, item1.id)">
                  {{ item1.authName }}
                  <i class="el-icon-caret-right"></i>
                </el-tag>
              </el-col>
              <!-- 二级和三级权限 -->
              <el-col :span="19">
                <el-row
                  :class="[i2 == 0 ? '' : 'bdtop', 'vcenter']"
                  v-for="(item2, i2) in item1.children"
                  :key="item2.id"
                >
                  <!-- 二级权限 -->
                  <el-col :span="6">
                    <el-tag
                      type="success"
                      closable
                      @close="removeRightById(scope.row, item2.id)"
                    >
                      {{ item2.authName }}
                      <i class="el-icon-caret-right"></i>
                    </el-tag>
                  </el-col>
                  <!-- 三级权限 -->
                  <el-col :span="18">
                    <el-tag
                      type="warning"
                      v-for="(item3, i3) in item2.children"
                      :key="item3.id"
                      closable
                      @close="removeRightById(scope.row, item3.id)"
                    >
                      {{ item3.authName }}
                      <i class="el-icon-caret-right"></i>
                    </el-tag>
                  </el-col>
                </el-row>
              </el-col>
            </el-row>
            <pre>
             <!-- {{ scope.row }} -->
           </pre>
          </template>
        </el-table-column>
        <!-- 索引列 -->
        <el-table-column type="index"> </el-table-column>
        <el-table-column label="角色名称" prop="roleName"></el-table-column>
        <el-table-column label="角色描述" prop="roleDesc"></el-table-column>
        <el-table-column label="操作" width="300px">
          <template slot-scope="scope">
            <el-button
              type="primary"
              size="mini"
              icon="el-icon-edit"
              @click="showEditDialog(scope.row.id)"
              >编辑</el-button
            >
            <el-button
              type="danger"
              size="mini"
              icon="el-icon-delete"
              @click="removeRolesById(scope.row.id)"
              >删除</el-button
            >
            <el-button
              type="warning"
              size="mini"
              icon="el-icon-setting"
              @click="showSetRightDialog(scope.row)"
              >分配权限</el-button
            >
          </template>
        </el-table-column>
      </el-table>
    </el-card>

    <!-- 添加用户对话框 -->
    <el-dialog
      title="添加角色"
      :visible.sync="addDialogVisible"
      width="50%"
      @close="addDialogClosed"
    >
      <!-- 内容主体部分 -->
      <el-form
        :model="addForm"
        :rules="addFormRules"
        ref="addFormRef"
        label-width="100px"
      >
        <el-form-item label="角色名称" prop="roleName">
          <el-input v-model="addForm.roleName"></el-input>
        </el-form-item>
        <el-form-item label="角色描述" prop="roleDesc">
          <el-input v-model="addForm.roleDesc"></el-input>
        </el-form-item>
      </el-form>
      <!-- 底部区域 -->
      <span slot="footer" class="dialog-footer">
        <el-button @click="addDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="addRoles">确 定</el-button>
      </span>
    </el-dialog>

    <!-- 修改用户的对话框 -->
    <el-dialog
      title="修改用户"
      :visible.sync="editDialogVisible"
      width="50%"
      @close="editDialogClosed"
    >
      <el-form
        :model="editForm"
        :rules="editFormRules"
        ref="editFormRef"
        label-width="100px"
      >
        <el-form-item label="角色名称" prop="roleName">
          <el-input v-model="editForm.roleName"></el-input>
        </el-form-item>
        <el-form-item label="角色描述" prop="roleDesc">
          <el-input v-model="editForm.roleDesc"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="editDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="editRolesInfo">确 定</el-button>
      </span>
    </el-dialog>
    <!-- 分配权限对话框 -->
    <el-dialog
      title="分配权限"
      :visible.sync="setRightDialogVisible"
      width="50%"
      @close="setRightDialogClosed"
    >
      <el-tree
        :data="rightslist"
        :props="treeProps"
        show-checkbox
        node-key="id"
        default-expand-all
        :default-checked-keys="defkeys"
        ref="treeRef"
      ></el-tree>
      <span slot="footer" class="dialog-footer">
        <el-button @click="setRightDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="allotRight">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>
<script>
export default {
  data() {
    return {
      // 所有角色列表数据
      rolelist: [],
      rightslist: [],
      // 控制添加用户对话框的显示与隐藏
      addDialogVisible: false,
      addForm: {
        roleName: "",
        roleDesc: ""
      },
      addFormRules: {
        roleName: [
          { required: true, message: "请输入角色名称", trigger: "blur" }
        ]
      },
      // 控制修改对话框的显示与隐藏
      editDialogVisible: false,

      editForm: {},
      // 修改表单的验证
      editFormRules: {
        roleName: [
          { required: true, message: "请输入角色名称", trigger: "blur" }
        ],
        roleDesc: [
          { required: true, message: "请输入角色描述", trigger: "blur" }
        ]
      },
      // 控制对话框的显示与隐藏
      setRightDialogVisible: false,
      // 树形控件的属性绑定对象
      treeProps: {
        label: "authName",
        children: "children"
      },
      // 默认选中的ID值
      defkeys: [],
      // 当前即将分配权限的角色ID
      roleId: ""
    };
  },
  created() {
    this.getRolesList();
  },
  methods: {
    //   获取所有角色列表
    async getRolesList() {
      const { data: res } = await this.$http.get("roles");
      if (res.meta.status !== 200) {
        return this.$message.error("获取角色列表失败");
      }
      this.rolelist = res.data;
      // console.log(this.rolelist);
    },
    // 监听添加对话框关闭事件
    addDialogClosed() {
      this.$refs.addFormRef.resetFields();
    },
    // 监听修改对话框关闭事件
    editDialogClosed() {
      this.$refs.editFormRef.resetFields();
    },

    // 展示编辑用户的对话框
    async showEditDialog(id) {
      // console.log(id);
      const { data: res } = await this.$http.get("roles/" + id);
      if (res.meta.status !== 200) {
        return this.$message.error("查询用户信息失败!");
      }
      this.editForm = res.data;
      this.editDialogVisible = true;
    },
    editRolesInfo() {
      this.$refs.editFormRef.validate(async valid => {
        // console.log(valid);
        if (!valid) return;
        const { data: res } = await this.$http.put(
          "roles/" + this.editForm.roleId,
          {
            roleName: this.editForm.roleName,
            roleDesc: this.editForm.roleDesc
          }
        );
        if (res.meta.status !== 200) {
          return this.$message.error("更新用户信息失败!");
        }
        // 关闭对话框
        this.editDialogVisible = false;
        // 刷新数据
        this.getRolesList();
        // 提示修改成功
        this.$message.success("更新用户信息成功!");
      });
    },
    addRoles() {
      this.$refs.addFormRef.validate(async valid => {
        // console.log(valid);
        if (!valid) return;
        const { data: res } = await this.$http.post("roles", this.addForm);
        if (res.data.status !== 201) {
          this.$message.error("添加用户失败");
        }
        this.$message.success("添加用户成功");
        this.addDialogVisible = false;
        this.getRolesList();
      });
    },
    // 根据ID删除用户的信息
    async removeRolesById(id) {
      // 询问用户是否删除
      const confirmResult = await this.$confirm(
        "此操作将永久删除该用户, 是否继续?",
        "提示",
        {
          confirmButtonText: "确定",
          cancelButtonText: "取消",
          type: "warning"
        }
      ).catch(err => err);

      if (confirmResult !== "confirm") {
        return this.$message.info("已经取消删除");
      }
      const { data: res } = await this.$http.delete("roles/" + id);
      if (res.meta.status != 200) {
        return this.$message.error("删除用户失败!");
      }
      this.$message.success("删除用户成功!");
      if (document.querySelectorAll(".el-card tbody tr").length === 1) {
        this.queryInfo.pagenum =
          this.queryInfo.pagenum > 1 ? this.queryInfo.pagenum - 1 : 1;
      }
      this.getRolesList();
    },
    // 根据id删除对于的权限
    async removeRightById(role, rightId) {
      // 弹框提示是否删除
      const confirmResult = await this.$confirm(
        "此操作将永久删除该文件, 是否继续?",
        "提示",
        {
          confirmButtonText: "确定",
          cancelButtonText: "取消",
          type: "warning"
        }
      ).catch(err => err);
      if (confirmResult !== "confirm") {
        return this.$message.info("取消了删除!");
      }
      const { data: res } = await this.$http.delete(
        `roles/${role.id}/rights/${rightId}`
      );
      if (res.meta.status !== 200) {
        return this.$message.error("删除权限失败!");
      }
      //  this.getRolesList()
      role.children = res.data;
      this.$message.success("删除权限成功!");
    },
    // 展示分配权限对话框
    async showSetRightDialog(role) {
      this.roleId = role.id;
      // 获取所有权限的数据
      const { data: res } = await this.$http.get("rights/tree");

      if (res.meta.status !== 200) {
        return this.$message.error("获取权限数据失败");
      }
      this.rightslist = res.data;
      this.getLeafKeys(role, this.defkeys);
      this.setRightDialogVisible = true;
    },
    // 通过递归的形式,获取角色下所有的三级权限的id,并保存到defkey数组中
    getLeafKeys(node, arr) {
      // 如果当前node节点不包含children属性,则是三级节点
      if (!node.children) {
        return arr.push(node.id);
      }
      node.children.forEach(item => {
        this.getLeafKeys(item, arr);
      });
    },
    setRightDialogClosed() {
      this.defkeys = [];
    },
    // 点击为角色分配权限
    async allotRight() {
      const keys = [
        ...this.$refs.treeRef.getCheckedKeys(),
        ...this.$refs.treeRef.getHalfCheckedKeys()
      ];
      // console.log(keys);
      const idStr = keys.join(",");
      const { data: res } = await this.$http.post(
        `roles/${this.roleId}/rights`,
        { rids: idStr }
      );
      if (res.meta.status !== 200) {
        return this.$message.error("分配权限失败!");
      }
      this.$message.success("分配权限成功!");
      this.getRolesList();
      this.setRightDialogVisible = false;
    }
  }
};
</script>

<style lang="less" scoped>
.el-tag {
  margin: 7px;
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
