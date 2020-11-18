<template>
  <div>
    <el-button type="primary" @click="willAdd">添加</el-button>

   
    <v-list  @edit="edit"></v-list>

    
    <v-form :info="info" ref="form"></v-form>
  </div>
</template>
<script>
import { mapGetters, mapActions } from "vuex";
import vList from "./components/list.vue";
import vForm from "./components/form.vue";
import { reqbannerList} from "../../utils/http";
export default {
  components:{
    vList,
    vForm
  },
  computed: {
    ...mapGetters({})
  },
  data() {
    return {
      info:{
          isshow:false,
          title:"轮播图添加"
      }
      

    }
  },
  methods: {
    ...mapActions({}),
    // 点击添加按钮
    willAdd(){
      this.info = {
        isshow : true,
        title :"轮播图添加"
      }
    },
    // 编辑
    edit(id){
      this.info={
        isshow:true,
        title:"编辑轮播图"
      };
      this.$refs.form.getOne(id);
    }
  },
  mounted() {}
};
</script>
<style scoped>
</style>