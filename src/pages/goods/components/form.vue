<template>
  <div>
    <el-dialog :title="info.title"
      :visible.sync="info.isshow"
      @opened="opened"
      @closed="closed"
    >
      <el-form :model="user" :rules="rules">
        <!-- 2.绑定数据 -->
        <el-form-item label="一级分类" label-width="120px" prop="first_cateid">
          <el-select
            placeholder="请选择一级分类"
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
          <el-select placeholder="请选择二级分类" v-model="user.second_cateid">
            <el-option
              v-for="item in secondCateList"
              :key="item.id"
              :label="item.catename"
              :value="item.id"
            ></el-option>
          </el-select>
        </el-form-item>
        <el-form-item label="商品名称" label-width="120px" prop="goodsname">
          <el-input
            v-model="user.goodsname"
            placeholder="请输入商品名称"
          ></el-input>
        </el-form-item>
        <el-form-item label="价格" label-width="120px" prop="price">
          <el-input v-model="user.price" placeholder="请输入价格"></el-input>
        </el-form-item>
        <el-form-item label="市场价格" label-width="120px" prop="market_price">
          <el-input v-model="user.market_price" placeholder="请输入市场价格" ></el-input>
        </el-form-item>
        <el-form-item label="图片" label-width="120px">
          <!-- 上传文件 -->
          <div class="myupload">
            <h3>+</h3>
            <img class="img" v-if="imgUrl" :src="imgUrl" alt />

            <input
              v-if="info.isshow"
              type="file"
              class="ipt"
              @change="changeFile"
            />
          </div>
        </el-form-item>
        <el-form-item label="商品规格" label-width="120px" prop="specsid">
          <!-- 11.遍历 -->
          <el-select
            placeholder="请选择商品规格"
            v-model="user.specsid"
            @change="changeSpecsId"
          >
            <el-option
              v-for="item in specsList"
              :key="item.id"
              :label="item.specsname"
              :value="item.id"
            ></el-option>
          </el-select>
        </el-form-item>
        <el-form-item label="规格属性" label-width="120px" prop="specsattr">
          <el-select
            placeholder="请选择二级分类"
            v-model="user.specsattr"
            multiple
          >
            <el-option
              v-for="item in attrsList"
              :key="item"
              :label="item"
              :value="item"
            ></el-option>
          </el-select>
        </el-form-item>
        <el-form-item label="是否新品" label-width="120px">
          <el-radio v-model="user.isnew" :label="1">是</el-radio>
          <el-radio v-model="user.isnew" :label="2">否</el-radio>
        </el-form-item>
        <el-form-item label="是否热卖" label-width="120px">
          <el-radio v-model="user.ishot" :label="1">是</el-radio>
          <el-radio v-model="user.ishot" :label="2">否</el-radio>
        </el-form-item>
        <el-form-item label="状态" label-width="120px">
          <el-switch
            v-model="user.status"
            :active-value="1"
            :inactive-value="2"
          ></el-switch>
        </el-form-item>
        <el-form-item label="描述" label-width="120px">
          <!-- 富文本编辑器 -->

          <div v-if="info.isshow" id="edit"></div>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="cancel">取 消</el-button>
        <el-button type="primary" v-if="info.title == '添加商品'" @click="add"
          >添 加</el-button
        >
        <el-button type="primary" v-else @click="update">修 改</el-button>
      </div>
    </el-dialog>
  </div>
</template>

<script>
import E from "wangeditor";
import { mapGetters, mapActions } from "vuex";
import {
  reqcateList,
  reqgoodsAdd,
  reqgoodsDetail,
  reqgoodsUpdate,
  reqspecsDel,
} from "../../../utils/http";
import { successAlert, errorAlert } from "../../../utils/alert";
export default {
  props: ["info"],
  data() {
    return {
      rules:{
          first_cateid: [
          { required: true, message: "请输入一级分类", trigger: "change" }
        ],
        second_cateid: [
          { required: true, message: "请输入二级分类", trigger: "change" }
        ],
        goodsname: [
          { required: true, message: "请输入商品名称", trigger: "blur" }
        ],
        price: [{ required: true, message: "请输入商品价格", trigger: "blur" }],
        market_price: [
          { required: true, message: "请输入商品市场价格", trigger: "blur" }
        ],
        specsid: [
          { required: true, message: "请输入商品规格", trigger: "change" }
        ],
        specsattr: [
          {
            type: "array",
            required: true,
            message: "请至少选择一个规格属性",
            trigger: "change"
          }
        ]
      },
      // 初始化数据
      user: {
        first_cateid: "",
        second_cateid: "",
        goodsname: "",
        price: "",
        market_price: "",
        img: null,
        description: "",
        specsid: "",
        specsattr: [],
        isnew: 1,
        ishot: 1,
        status: 1,
      },
      // 属性值
      secondCateList: [],
      // 图片地址
      imgUrl: "",
      // 规格属性list
      attrsList: [],
    };
  },
  computed: {
    ...mapGetters({
      // 一级列表分类
      cateList: "cate/list",
      // 规格list
      specsList: "specs/list"
    }),
  },
  methods: {
    ...mapActions({
      // 请求一级分类list
      reqcateList: "cate/reqList",
      //请求规格list
      reqspecsList: "specs/reqList",
      //商品list和总数
      reqgoodsList: "goods/reqList",
      reqgoodsCount: "goods/reqCount"
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
    changeFile(e) {
      let file = e.target.files[0];
      this.imgUrl = URL.createObjectURL(file);
      this.user.img = file;
    },
    changeSpecsId() {
      // 先将specsattr置空
      this.user.specsattr = [];
      this.getAttrs();
    },
    getAttrs() {
      // 取出 specsList ,哪条数据的id和this.user.specsid是一样的
      let obj = this.specsList.find(item => item.id === this.user.specsid);
      //就将这条数据的attrs取出来，赋值给attrsList
      this.attrsList = obj.attrs;
    },
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
        first_cateid: "",
        second_cateid: "",
        goodsname: "",
        price: "",
        market_price: "",
        img: null,
        description: "",
        specsid: "",
        specsattr: [],
        isnew: 1,
        ishot: 1,
        status: 1,
      };
      //二级分类的list
      this.secondCateList = [];
      //图片临时地址
      this.imgUrl = "";
      //规格属性list
      this.attrsList = [];
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
        if (this.user.goodsname === "") {
          errorAlert("商品名称为空");
          return;
        }
        if (this.user.price === "") {
          errorAlert("商品价格为空");
          return;
        }
        if (this.user.market_price === "") {
          errorAlert("商品市场价格为空");
          return;
        }
        if (!this.user.img) {
          errorAlert("请选择图片");
          return;
        }
        if (this.user.specsid === "") {
          errorAlert("请选择商品规格");
          return;
        }
        if (this.user.specsattr.length === 0) {
          errorAlert("请选择商品属性");
          return;
        }
        if (this.editor.txt.html() === "") {
          errorAlert("请输入商品描述");
          return;
        }
        resolve();
      });
    },
    add() {
      this.check().then(() => {
        //将编辑器的内容取出来给user.description
        //this.editor.txt.html() 取值
        this.user.description = this.editor.txt.html();
        let d = { ...this.user };
        d.specsattr = JSON.stringify(d.specsattr);
        reqgoodsAdd(d).then((res) => {
          if (res.data.code === 200) {
            // 弹个成功
            successAlert("添加成功");
            // 弹框消失
            this.cancel();
            // form置空
            this.empty();
            // 列表数据刷新list
            this.reqgoodsList();
            this.reqgoodsCount();
          }
        });
      });
    },
    // 获取详情
    getOne(id) {
      reqgoodsDetail(id).then((res) => {
        // 此刻user没有id
        this.user = res.data.list;
        // 补id
        this.user.id = id;
        //请二级list
        this.getSecondList();
        //图片
        this.imgUrl = this.$imgPre + this.user.img;
        //  属性
        this.user.specsattr = JSON.parse(this.user.specsattr);
        //计算规格属性的list
        this.getAttrs();
        //给编辑器赋值
        if (this.editor) {
          this.editor.txt.html(this.user.description);
        }
      });
    },
    // 修改
    update() {
      this.check().then(() => {
        //将编辑器的内容取出来给user.description
        //this.editor.txt.html() 取值
        this.user.description = this.editor.txt.html();
        let d = { ...this.user };
        d.specsattr = JSON.stringify(d.specsattr);
        reqgoodsUpdate(d).then((res) => {
          if (res.data.code === 200) {
            // 弹个成功
            successAlert("添加成功");
            // 弹框消失
            this.cancel();
            // form置空
            this.empty();
            // 列表数据刷新list
            this.reqgoodsList();
            this.reqgoodsCount();
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
      this.editor = new E("#edit");
      this.editor.create()
      this.editor.txt.html(this.user.description)
    }
  },
  mounted() {
    // 一进来请求一级分类list
    this.reqcateList();
    // 一进来就请求规格list,参数true，是为了取到所有的规格
    this.reqspecsList(true);
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