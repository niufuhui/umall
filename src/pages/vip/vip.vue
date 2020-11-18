<template>
  <div>
    <!-- 传递list数据 给list组件 -->
    <!-- 绑定自定义事件init --> 
    <!-- 绑定edit事件 -->
    <v-list :list="list" @init="init" @edit="edit"></v-list>
    <!-- 传递info给form  -->
    <!-- 绑定自定义事件init -->
    <v-form :info="info" @init="init" ref="form"></v-form>
  </div>
</template>
<script>
import { mapGetters, mapActions } from "vuex";
import vList from "./components/list.vue"
import vForm from "./components/form.vue"
import {reqVipList} from "../../utils/http"
export default {
  components:{
    vList,
    vForm
  },
  data() {
    return {
      info:{
        isshow:false,
        title:"会员编辑"
      },
      // 列表数据初始值
      list:[]
    }
  },
  computed: {
    ...mapGetters({})
  },
  methods: {
    ...mapActions({}),
    // 发送请求
    init(){
      reqVipList().then(res=>{
        this.list = res.data.list
      })
    },
    // 编辑
    edit(uid){
      this.info={
        isshow:true,
        title:"编辑会员"
      }
      this.$refs.form.getOne(uid)
    }
  },
  mounted() {
    // 一进来就发请求
    this.init()
    
  }
};
</script>
<style scoped>
</style>