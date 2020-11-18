<template>
  <div>
    <el-dialog :title="info.title" :visible.sync="info.isshow" @opened="opened" @closed="closed">
      <el-form :model="user" :rules="rules">
        <el-form-item label="活动名称" label-width="120px" prop="title">
          <el-input  v-model="user.title" placeholder="请输入活动名称"></el-input>
        </el-form-item>
        <!-- 2.绑定数据 -->
        <el-form-item label="一级分类" label-width="120px" prop="first_cateid">
          <el-select placeholder="请选择分类" v-model="user.first_cateid" @change="changeFirst">
            <!-- 6.遍历一级分类 -->
            <el-option v-for="item in cateList" :key="item.id" :label="item.catename" :value="item.id"></el-option>
          </el-select>
        </el-form-item>
        <el-form-item label="二级分类" label-width="120px" prop="second_cateid">
          <el-select placeholder="请选择分类" v-model="user.second_cateid" @change="changeSecond">
            <el-option v-for="item in specsList" :key="item.id" :label="item.specsname" :value="item.id"></el-option>
          </el-select>
        </el-form-item>
        
        <el-form-item label="商品" label-width="120px" prop="goodsid">
          <!-- 11.遍历 -->
          <el-select placeholder="请选择商品"  v-model="user.goodsid" >
            <el-option v-for="item in goodsList" :key="item.id" :label="item.goodsname" :value="item.id"></el-option>
          </el-select>
        </el-form-item>
        
        <el-form-item label="状态" label-width="120px">
          <el-switch  v-model="user.status" :active-value="1" :inactive-value="2"></el-switch>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="cancel">取 消</el-button>
        <el-button type="primary" v-if="info.title == '添加活动'" @click="add">添 加</el-button>
        <!-- <el-button type="primary" v-else @click="update">修 改</el-button> -->
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
  reqSpecsList,
} from "../../../utils/http";
import { successAlert, errorAlert } from "../../../utils/alert";
export default {
  props: ["info"],
  data() {
    return {
      rules:{
        title: [
          { required: true, message: "请输入商品名称", trigger: "blur" }
        ],
        first_cateid: [
          { required: true, message: "请输入一级分类", trigger: "change" }
        ],
        second_cateid: [
          { required: true, message: "请输入二级分类", trigger: "change" }
        ],
        goodsid: [
          { required: true, message: "请输入商品规格", trigger: "change" }
        ]
      },
      // 初始化数据
      user: {
        title:"",
        first_cateid: "",
        second_cateid: "",
        goodsid: "",
        status: 1,
      },
      // 属性值
      secondCateList:[],
      thirdSpecsList:[]
    };
  },
  computed: {
    ...mapGetters({
      // 一级列表分类
      cateList: "cate/list",
      // 二级列表
      specsList:"specs/list",
      // 三级列表
      goodsList:"goods/list"
      
    }),
  },
  methods: {
    ...mapActions({
      // 请求一级分类list
      reqcateList: "cate/reqList",
      //请求二级分类list
      reqSpecsList: "specs/reqList",
      //商品list和总数
      reqGoodsList: "goods/reqList",
      reqGoodsCount: "goods/reqCount"
    }),
    changeFirst() {
      this.user.second_cateid = "";
      this.getSecondList();
    },
    getSecondList() {
      // 获取二级分类list
      reqcateList({ pid: this.user.first_cateid }).then((res) => {
        this.secondCateList = res.data.list;
      });
    },
    changeSecond(){
        this.user.goodsid = "";
        this.getThirdList();
    },
    getThirdList(){
        reqSpecsList({id:this.user.first_cateid}).then((res)=>{
            this.thirdSpecsList = res.data.list
        })
    },
    // changeSpecsId() {
    //   // 先将specsattr置空
    //   this.user.specsattr = [];
    //   this.getAttrs();
    // },
    // getAttrs() {
    //   // 取出 specsList ,哪条数据的id和this.user.specsid是一样的
    //   let obj = this.thirdSpecsList.find(item => item.id === this.user.specsid);
    //   //就将这条数据的attrs取出来，赋值给attrsList
    //   this.attrsList = obj.attrs;
    // },
    // 点击了取消
    cancel() {
      this.info.isshow = false;
    },
    //新增商品属性
    addAttr() {
      this.attrArr.push({
        value: "",
      });
    },
    //删除商品属性
    delAttr(index) {
      this.attrArr.splice(index, 1);
    },
    // user置空
    empty() {
      this.user = {
        title:"",
        first_cateid: "",
        second_cateid: "",
        goodsid: "",
        status: 1,
      };
      //二级分类的list
      this.secondCateList = [];
    },
    // 验证
    check() {
      return new Promise((resolve, reject) => {
        //验证
        if (this.user.first_cateid === "") {
          errorAlert("一级分类不能为空");
          return;
        }
        if (this.user.second_cateid === "") {
          errorAlert("二级分类不能为空");
          return;
        }
        if (this.user.title === "") {
          errorAlert("活动名称为空");
          return;
        }
        if (this.user.goodsid === "") {
          errorAlert("请选择商品");
          return;
        }
        resolve();
      });
    },
    add() {
      console.log(this.user);
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
            this.reqSecksList();
          }
        });
      });
    },
    // 获取详情
    getOne(id) {
      reqSeckDetail(id).then((res) => {
        // 此刻user没有id
        this.user = res.data.list;
        // 补id
        this.user.id = id;
        //请二级list
        this.getSecondList();
        //  属性
        this.user.specsattr = JSON.parse(this.user.specsattr);
        //计算规格属性的list
        this.getAttrs();
      });
    },
    // 修改
    update() {
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
            this.reqSeckList();
          }
        });
      });
    },
    // 处理消失
    closed() {
      if (this.info.title === "编辑商品") {
        this.empty();
      }
    },
    opened(){
      // this.create()
    }
  },
  mounted() {
    // 一进来请求一级分类list
    this.reqcateList();
    // 请求二级分类
    this.reqSpecsList();
    // 请求三级分类
    this.reqGoodsList();

    // 一进来就请求规格list,参数true，是为了取到所有的规格
    // this.reqSeckList(true);
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