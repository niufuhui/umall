<template>
  <div>
    <!-- 5.绑定info.isshow到模板 -->
    <el-dialog :title="info.title" :visible.sync="info.isshow">
      <el-form :model="user">
        <el-form-item label="手机号" label-width="120px">
           <el-input v-model="user.phone" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="昵称" label-width="120px">
          <!-- 通过v-model将user绑定到表单上 -->
          <el-input v-model="user.nickname" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="密码" label-width="120px">
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
        <el-button type="primary" @click="update">修 改</el-button>
      </div>
    </el-dialog>
  </div>
</template>

<script>
import { mapGetters, mapActions } from "vuex";
import {
  reqVipList,
  reqVipDetail,
  reqVipUpdate
} from "../../../utils/http";
import { successAlert } from "../../../utils/alert";
export default {
  // 4.接受info
  //   
  props: ["info"],
  data() {
    return {
      // 初始化user
      user: {
        uid:"",
        nickname: "",
        phone:"",
        password: "",
        status: 1,
      },
      list: [],
     
    };
  },
  computed: {
    ...mapGetters({}),
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
        uid:"",
        nickname: "",
        phone:"",
        password: "",
        status: 1,
      };
    },
    // 获取一条数据
    getOne(uid) {
      console.log(this.uid);
      reqVipDetail(uid).then((res) => {
        // 此时user上是没有id的,有uid修改uid
        this.user = res.data.list;
        // 处理密码
        this.user.password = "";
      });
    },
    // 点击修改
    update() {
      reqVipUpdate(this.user).then((res) => {
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
    },
    
  },
  mounted() {
    // 一进来先获取菜单列表数据
    reqVipList().then((res) => {
      this.list = res.data.list;
    });
  },
};
</script>

<style>
</style>