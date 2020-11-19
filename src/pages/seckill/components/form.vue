<template>
  <div>
    <el-dialog
      :title="info.title"
      :visible.sync="info.isshow"
      @opened="opened"
      @closed="closed"
    >
      <el-form :model="user" :rules="rules">
        <el-form-item label="活动名称" label-width="120px" prop="title">
          <el-input
            v-model="user.title"
            placeholder="请输入活动名称"
          ></el-input>
        </el-form-item>

        <div class="box">
          <el-form-item label="活动期限" label-width=" 120px ">
            <!-- <el-card class="box-card">
              <el-date-picker
                v-model="value"
                type="datetimerange"
                range-separator="至"
                start-placeholder="开始日期"
                end-placeholder="结束日期"
                size="medium"
                value-format="timestamp"
              ></el-date-picker>
            </el-card> -->


             <el-date-picker
      v-model="value"
      type="datetimerange"
      range-separator="至"
      start-placeholder="开始日期"
      end-placeholder="结束日期"
       value-format="timestamp"
      >
    </el-date-picker>
          </el-form-item>
        </div>

        <!-- 2.绑定数据 -->
        <el-form-item label="一级分类" label-width="120px" prop="first_cateid">
          <el-select
            placeholder="请选择分类"
            v-model="user.first_cateid"
            @change="changeFirst"
          >
            <!-- 6.遍历一级分类 -->
            <el-option
              v-for="item in cateList"
              :key="item.id"
              :label="item.catename"
              :value="item.id"
            ></el-option>
          </el-select>
        </el-form-item>
        <el-form-item label="二级分类" label-width="120px" prop="second_cateid">
          <el-select
            placeholder="请选择分类"
            v-model="user.second_cateid"
            @change="changeSecond"
          >
            <el-option
              v-for="item in secondCateList"
              :key="item.id"
              :label="item.catename"
              :value="item.id"
            ></el-option>
          </el-select>
        </el-form-item>

        <el-form-item label="商品" label-width="120px" prop="goodsid">
          <!-- 11.遍历 -->
          <el-select placeholder="请选择商品" v-model="user.goodsid">
            <el-option
              v-for="item in thirdSpecsList"
              :key="item.id"
              :label="item.goodsname"
              :value="item.id"
            ></el-option>
          </el-select>
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
        <el-button type="primary" v-if="info.title == '添加活动'" @click="add"
          >添 加</el-button
        >
        <el-button type="primary" v-else @click="update">修 改</el-button>
      </div>
    </el-dialog>
  </div>
</template>

<script>
import { mapGetters, mapActions } from "vuex";
import {
  reqcateList,
  reqSeckAdd,
  reqSeckDetail,
  reqSeckUpdate,
  reqspecsList,
  reqgoodsList,
} from "../../../utils/http";
import { successAlert, errorAlert } from "../../../utils/alert";

export default {
  props: ["info"],
  data() {
    return {
      rules: {
        title: [{ required: true, message: "请输入活动名称", trigger: "blur" }],
        first_cateid: [
          { required: true, message: "请输入一级分类", trigger: "change" },
        ],
        second_cateid: [
          { required: true, message: "请输入二级分类", trigger: "change" },
        ],
        goodsid: [{ required: true, message: "请输入商品", trigger: "change" }],
      },
      value: [],
      // 初始化数据
      user: {
        title: "",
        begintime: null,
        endtime: null,
        first_cateid: "",
        second_cateid: "",
        goodsid: "",
        status: 1,
      },
      // 属性值
      secondCateList: [],
      thirdSpecsList: [],
      dateTime: [],
    };
  },
  computed: {
    ...mapGetters({
      // 一级列表分类
      cateList: "cate/list",
      // 二级列表
      specsList: "specs/list",
      // 三级列表
      goodsList: "goods/list",
    }),
  },
  methods: {
    ...mapActions({
      // 请求一级分类list
      reqcateList: "cate/reqList",
      //请求二级分类list
      reqspecsList: "specs/reqList",
      //商品list和总数
      reqgoodsList: "goods/reqList",
      reqSeckillList: "seckill/reqList",
    }),
    changeFirst() {
      this.user.second_cateid = "";
      this.user.goodsid = "";
      this.getSecondList();
    },
    getSecondList() {
      // 获取二级分类list
      reqcateList({ pid: this.user.first_cateid }).then((res) => {
        this.secondCateList = res.data.list;
      });
    },
    changeSecond() {
      this.user.goodsid = "";
      this.getThirdList();
    },
    getThirdList() {
      reqgoodsList({
        fid: this.user.first_cateid,
        sid: this.user.second_cateid,
      }).then((res) => {
        this.thirdSpecsList = res.data.list;
      });
    },

    // 点击了取消
    cancel() {
      this.info.isshow = false;
    },
    // user置空
    empty() {
      this.user = {
        title: "",
        begintime: null,
        endtime: null,
        first_cateid: "",
        second_cateid: "",
        goodsid: "",
        status: 1,
      };
      //二级分类的list
      this.secondCateList = [];
      this.thirdSpecsList = [];
      this.dateTime = [];
    },
    // 验证
    check() {
      return new Promise((resolve, reject) => {
        //验证
        if (this.user.title === "") {
          errorAlert("商品名称为空");
          return;
        }
        if (this.user.first_cateid === "") {
          errorAlert("一级分类不能为空");
          return;
        }
        if (this.user.second_cateid === "") {
          errorAlert("二级分类不能为空");
          return;
        }
        if (this.user.goodsid === "") {
          errorAlert("商品为空");
          return;
        }
        if (this.value.length === 0) {
          errorAlert("日期时间为空");
          return;
        }

        resolve();
      });
    },
    add() {
      this.user.begintime = this.value[0];
      this.user.endtime = this.value[1];
      this.check().then(() => {
        reqSeckAdd(this.user).then((res) => {
          if (res.data.code === 200) {
            // 弹个成功
            successAlert("添加成功");
            // 弹框消失
            this.cancel();
            // form置空
            this.empty();
            // 列表数据刷新list
            this.reqSeckillList();
          }
        });
      });
    },
    // 获取详情
    getOne(id) {
      this.value= []
      reqSeckDetail(id).then((res) => {
        // 此刻user没有id
        this.user = res.data.list;
      this.value.push(this.user.begintime,this.user.endtime)

        console.log( this.value);
        // 补id
        this.user.id = id;
        //请二级list
        this.getSecondList();
        // 请求三级
        this.getThirdList();
      });
    },
    // 修改
    update() {
      
      this.user.begintime = this.value[0];
      this.user.endtime = this.value[1];
      console.log(this.user);
      this.check().then(() => {
        reqSeckUpdate(this.user).then((res) => {
          if (res.data.code === 200) {
            // 弹个成功
            successAlert("添加成功");
            // 弹框消失
            this.cancel();
            // form置空
            this.empty();
            // 列表数据刷新list
            this.reqSeckillList();
          }
        });
      });
    },
    // 处理消失
    closed() {
      if (this.info.title === "编辑活动") {
        this.empty();
      }
    },
    opened() {},
  },
  mounted() {
    // 一进来请求一级分类list
    this.reqcateList();
    // 请求二级分类
    this.reqspecsList();
    // 请求三级分类
    this.reqgoodsList();
  },
};
</script>

<style scoped >
.line {
  display: flex;
}
.line .el-input {
  flex: 1;
}
.line .el-button {
  width: auto;
}
.myupload {
  width: 100px;
  height: 100px;
  border-radius: 5px;
  border: 1px dashed #ccc;
  position: relative;
}
.myupload h3 {
  width: 100%;
  height: 100px;
  font-size: 30px;
  text-align: center;
  line-height: 100px;
  color: #666;
  font-weight: 100;
}
.myupload .ipt {
  width: 100px;
  height: 100px;
  position: absolute;
  left: 0;
  top: 0;
  opacity: 0;
}
.myupload .img {
  width: 100px;
  height: 100px;
  position: absolute;
  left: 0;
  top: 0;
}
</style>