<template>
  <div class="about">
    <!-- <h1>This is an about page</h1> -->
    <!-- {{ $route.params.id }} -->
    <div>{{ title }}</div>

    <div
      v-loading.fullscreen.lock="loading"
      element-loading-text="拼命加载中"
      element-loading-spinner="el-icon-loading"
      element-loading-background="rgba(0, 0, 0, 0.8)"
    >
      {{ $route.query.id }}
    </div>
    <img :src="img" />
  </div>
</template>

<script>
// @ is an alias to /src
// import HelloWorld from '@/components/HelloWorld.vue'
// import About from "../views/About";
// import { Loading } from "element-ui";

export default {
  data() {
    return {
      id: this.$route.query.id,
      img: "",
      title: "",
      loading: true,
    };
  },
  created() {
    var that = this;
    that.onload();
    //当前页面监视键盘输入
    document.onkeydown = function(e) {
      //事件对象兼容
      let e1 =
        e || event || window.event || arguments.callee.caller.arguments[0];

      if (e1 && e1.keyCode == 32) {
        that.$router.go(-1);
        // console.log("回车");
      }
    };
  },
  // 过滤器
  filters: {
    // 解决403图片缓存问题
    getImages(_url) {
      if (_url !== undefined) {
        let _u = _url.substring(7);
        return "https://images.weserv.nl/?url=" + _u;
      }
    },
  },
  methods: {
    onload() {
      var that = this;
      this.$axios
        .get("https://douban.uieee.com/v2/movie/subject/" + that.id, {
          params: {
            apikey: "0df993c66c0c636e29ecbb5344252a4a",
          },
        })
        .then(function(res) {
          console.log(res);
          that.title = res.data.title;
          that.img = res.data.images.small;

          // console.log(that.content);
        })
        .catch(function(error) {
          console.log(error);
        });
    },
  },
  mounted() {},
  updated() {
    this.$nextTick(() => {
      this.loading = false;
    });
  },
};
</script>
<style></style>
