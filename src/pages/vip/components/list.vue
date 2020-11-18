<template>
  <div>
    <!-- list展示在页面 -->
    <el-table
      :data="list"
      style="width: 100%; margin-bottom: 20px"
      row-key="id"
      border
      :tree-props="{ children: 'children' }">
      <el-table-column prop="uid" label="用户编号" sortable></el-table-column>
      <el-table-column prop="nickname" label="昵称" sortable></el-table-column>
      <el-table-column prop="phone" label="手机号" sortable></el-table-column>
      <el-table-column label="状态">
        <template slot-scope="scope">
          <el-button type="primary" v-if="scope.row.status === 1">启用</el-button>
          <el-button type="info" v-else>禁用</el-button>
        </template>
      </el-table-column>
      <el-table-column label="操作">
        <template slot-scope="scope">
          <!-- scope.row.id直接取到该单元格对象里的id -->
          <el-button type="primary" @click="edit(scope.row.uid)">编辑</el-button>
          <!-- 绑定confirm事件 -->
          <!-- <del-btn @confirm="del(scope.row.uid)"></del-btn> -->
        </template>
      </el-table-column>
    </el-table>
  </div>
</template>

<script>
import { mapGetters, mapActions } from "vuex";


export default {
  // 接收list
  props: ["list"],
  computed: {
    ...mapGetters({}),
  },
  methods: {
    ...mapActions({}),
    
    // 点击了编辑按钮
    edit(uid) {
      // 通知父组件要编辑了
      this.$emit("edit", uid);
    },
  },
  mounted() {},
};
</script>

<style>
</style>