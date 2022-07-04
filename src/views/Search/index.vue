<template>
  <div>
    <van-search
      shape="round"
      v-model="value"
      placeholder="请输入搜索关键词"
      @input="inputFn"
    />
    <div class="search_wrap" v-if="resultList.length === 0">
      <!-- 每个热搜的关键词 -->
      <span
        class="hot_item"
        v-for="(obj, index) in hotArr"
        :key="index"
        @click="fn(obj.first)"
        >{{ obj.first }}</span
      >
    </div>
    <!-- 搜索结果 -->
    <div class="search_wrap">
      <!-- 标题 -->
      <p class="hot_title">最佳匹配</p>
      <van-list
        v-model="loading"
        :finished="finished"
        finished-text="没有更多了"
        @load="onLoad"
      >
        <SongItem
          v-for="obj in resultList"
          :key="obj.id"
          :name="obj.name"
          :author="obj.ar[0].name"
          :id="obj.id"
        ></SongItem>
      </van-list>
    </div>
  </div>
</template>

<script>
import { hotSearchAPI, searchResultListAPI } from "@/api";
import SongItem from "@/components/SongItem";
export default {
  data() {
    return {
      value: "",
      hotArr: [],
      resultList: [],
      loading: false,
      finished: false,
      page: 1,
      timer: null,
    };
  },
  async created() {
    const res = await hotSearchAPI();
    console.log(res);
    this.hotArr = res.data.result.hots;
  },
  //难点：async修饰的函数=》默认返回一个全新的promise对象
  // 这个promise对象的结果就是async函数内return的值
  //把搜索结果return出去
  //拿到getListFn的返回值用await提取结果
  methods: {
    async getListFn() {
      return await searchResultListAPI({
        keywords: this.value,
        limit: 20,
        offset: (this.page - 1) * 20,
      });
    },
    async fn(val) {
      // 点击热搜关键词
      this.page = 1;
      this.finished = false;
      this.value = val; // 选中的关键词显示到搜索框
      const res = await this.getListFn();
      console.log(res);
      this.resultList = res.data.result.songs;
      this.loading = false;
    },
    async fn(val) {
      // 点击热搜关键词
      this.page = 1; // 点击重新获取第一页数据
      this.finished = false; // 点击新关键词-可能有新的数据
      this.value = val; // 选中的关键词显示到搜索框
      const res = await this.getListFn();
      console.log(res);
      this.resultList = res.data.result.songs;
      this.loading = false; // 本次数据加载完毕-才能让list加载更多
    },
    async inputFn() {
      // 目标: 输入框改变-逻辑代码-慢点执行
      // 解决: 防抖
      // 概念: 计时n秒, 最后执行一次, 如果再次触发, 重新计时
      // 效果: 用户在n秒内不触发这个事件了, 才会开始执行逻辑代码
      if (this.timer) clearTimeout(this.timer);
      this.timer = setTimeout(async () => {
        this.page = 1; // 点击重新获取第一页数据
        this.finished = false; // 输入框关键字改变-可能有新数据(不一定加载完成了)
        // 输入框值改变
        if (this.value.length === 0) {
          // 搜索关键词如果没有, 就把搜索结果清空阻止网络请求发送(提前return)
          this.resultList = [];
          return;
        }
        const res = await this.getListFn();
        console.log(res);
        // 如果搜索结果响应数据没有songs字段-无数据
        if (res.data.result.songs === undefined) {
          this.resultList = [];
          return;
        }
        this.resultList = res.data.result.songs;
        this.loading = false;
      }, 900);
    },
  },
  components: {
      SongItem,
    },
};
</script>

<style scoped>
/* 搜索容器的样式 */
.search_wrap {
  padding: 0.266667rem;
}
/* 热门搜索文字标题样式 */
.hot_title {
  font-style: 0.32rem;
  color: #666;
}
/* 搜索容器 */
.hot_name_wrap {
  margin: 0.266667rem 0;
}
/* 热搜词——容器 */
.hot_item {
  display: inline-block;
  height: 0.853333rem;
  margin-right: 0.213333rem;
  margin-bottom: 0.213333rem;
  padding: 0 0.373333rem;
  font-size: 0.373333rem;
  line-height: 0.853333rem;
  color: #333;
  border-color: #d3d4da;
  border-radius: 0.853333rem;
  border: 1px solid #d3d4da;
}
.van-cell {
  border-bottom: 1px solid lightgray;
}
</style>