<template>
  <div>
    <!-- 5.绑定info.isshow到模板 -->
    <el-dialog :title="info.title" :visible.sync="info.isshow" @closed="closed">
      <el-form :model="user" :rules="rules">
        <el-form-item label="所属角色" label-width="120px" prop="roleid">
          <!-- 通过v-model将user绑定到表单上 -->
          <el-select v-model="user.roleid" placeholder="请选择角色">
            <el-option
              v-for="item in roleList"
              :key="item.id"
              :label="item.rolename"
              :value="item.id"
            >
            </el-option>
          </el-select>
        </el-form-item>
        <el-form-item label="用户名" label-width="120px" prop="username">
          <!-- 通过v-model将user绑定到表单上 -->
          <el-input v-model="user.username" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="密码" label-width="120px" prop="password">
          <!-- 通过v-model将user绑定到表单上 -->
          <el-input v-model="user.password" autocomplete="off"></el-input>
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
        <el-button
          type="primary"
          @click="add"
          v-if="info.title === '添加管理员'"
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
  reqRoleList,
  reqUserAdd,
  reqUserDetail,
  reqUserUpdate,
} from "../../../utils/http";
import { successAlert,errorAlert } from "../../../utils/alert";
export default {
  // 4.接受info
  //
  props: ["info"],
  data() {
    return {
      rules: {
        roleid: [
          { required: true, message: "请输入角色名称", trigger: "blur" },
        ],
        username: [
          { required: true, message: "请输入用户名", trigger: "blur" },
        ],
        password: [{ required: true, message: "请输入密码", trigger: "blur" }],
      },
      // 初始化user
      user: {
        roleid: "",
        username: "",
        password: "",
        status: 1,
      },

      // 初始化树形控件
      roleList: [],
    };
  },
  computed: {
    ...mapGetters({}),
  },
  mounted() {
    // 一进来先获取菜单列表数据
    reqRoleList().then((res) => {
      if (res.data.code == 200) {
        this.roleList = res.data.list;
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
        roleid: "",
        username: "",
        password: "",
        status: 1,
      };
    },
    // 验证
    check() {
      return new Promise((resolve, reject) => {
        //验证
        if (this.user.roleid === "") {
          errorAlert("所属角色为空");
          return;
        }
        if (this.user.username === "") {
          errorAlert("用户名为空");
          return;
        }
        if (this.user.password === "") {
          errorAlert("密码为空");
          return;
        }
        resolve();
      });
    },
    // 点击添加按钮
    add() {
      this.check().then(() => {
        // 发起添加的请求
        reqUserAdd(this.user).then((res) => {
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
    getOne(uid) {
      reqUserDetail(uid).then((res) => {
        // 此时user上是没有id的,有uid修改uid
        this.user = res.data.list;
        // 处理密码
        this.user.password = "";
      });
    },
    // 点击修改
    update() {
      this.check().then(() => {
        reqUserUpdate(this.user).then((res) => {
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
      if (this.info.title === "编辑管理员") {
        this.empty();
      }
    },
  },
};
</script>

<style>
</style>