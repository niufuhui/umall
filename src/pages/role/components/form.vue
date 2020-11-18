<template>
  <div>
    <!-- 5.绑定info.isshow到模板 -->
    <el-dialog :title="info.title" :visible.sync="info.isshow" @closed="closed">
      <el-form :model="user" :rules="rules">
        <el-form-item label="角色名称" label-width="120px" prop="rolename">
          <!-- 通过v-model将user绑定到表单上 -->
          <el-input v-model="user.rolename" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="角色权限" label-width="120px">
          <el-tree
            :data="menuList"
            :props="{ children: 'children', label: 'title' }"
            show-checkbox
            ref="tree"
            node-key="id"
          >
          </el-tree>
        </el-form-item>

        <el-form-item label="状态" label-width="120px">
          <el-switch
            v-model="user.status"
            :active-value="1"
            :inactive-value="2"
          ></el-switch>
        </el-form-item>
      </el-form>

      <div slot="footer" class="dialog-footer">
        <el-button @click="cancel">取 消</el-button>
        <el-button type="primary" @click="add" v-if="info.title === '添加角色'"
          >添 加</el-button
        >
        <el-button type="primary" @click="update" v-else>修 改</el-button>
      </div>
    </el-dialog>
  </div>
</template>

<script>
import { mapGetters, mapActions } from "vuex";
import {
  reqMenuList,
  reqRoleAdd,
  reqRoleDetail,
  reqRoleUpdate,
} from "../../../utils/http";
import { successAlert,errorAlert } from "../../../utils/alert";
export default {
  // 4.接受info
  //   
  props: ["info"],
  data() {
    return {
      rules: {
        rolename: [{ required: true, message: "请输入角色名称", trigger: "blur" }],
      },
      // 初始化user
      user: {
        rolename: "",
        menus: "",
        status: 1,
      },
      
      // 初始化树形控件
      menuList: [],
     
    };
  },
  computed: {
    ...mapGetters({}),
  },
  mounted() {
    // 一进来先获取菜单列表数据
    reqMenuList().then((res) => {
      if (res.data.code == 200) {
        this.menuList = res.data.list;
      }
    });
  },
  methods: {
    ...mapActions({}),
    // 点击了取消
    cancel() {
      this.info.isshow = false;
    },
    // user置空
    empty() {
      this.user = {
        rolename: "",
        menus: "",
        status: 1,
      };
      // 把树清空
      this.$refs.tree.setCheckedKeys([]);
    },
    // 验证
    check() {
      return new Promise((resolve, reject) => {
        //验证
        if (this.user.rolename === "") {
          errorAlert("角色名称为空");
          return;
        }
        resolve();
      });
    },
    // 点击添加按钮
    add() {
      this.check().then(() => {
      // 将树形控件的数据取出，变成字符串数组，赋值给user.menus
      this.user.menus = JSON.stringify(this.$refs.tree.getCheckedKeys());
      // 发起添加的请求
      reqRoleAdd(this.user).then((res) => {
        if (res.data.code === 200) {
          // 弹个成功
          successAlert("添加成功");
          // 弹框消失
          this.cancel();
          // form置空
          this.empty();
          // 列表数据刷新list
          this.$emit("init");
        }
      });
      });
    },
    // 获取一条数据
    getOne(id) {
      reqRoleDetail(id).then((res) => {
        // 此时user上是没有id的
        this.user = res.data.list;
        // 处理树形控件的数据
        this.$refs.tree.setCheckedKeys(JSON.parse(this.user.menus));
        // 补id
        this.user.id = id;
      });
    },
    // 点击修改
    update() {
      this.check().then(() => {
      this.user.menus = JSON.stringify(this.$refs.tree.getCheckedKeys());
      reqRoleUpdate(this.user).then((res) => {
        if (res.data.code === 200) {
          // 成功的弹框
          successAlert("修改成功");
          // 弹框消失
          this.cancel();
          // form重置
          this.empty();
          // 列表刷新
          this.$emit("init");
        }
      });
      });
    },
    // 弹框消失
    closed() {
      // 如果是添加出现，点击了取消，此时，什么都不做
      // 如果是编辑出现，点击了取消，此时，form置空
      if (this.info.title === "编辑角色") {
        this.empty();
      }
    },
  },
};
</script>

<style>
</style>