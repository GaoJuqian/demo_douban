<template>
  <div id="app">
    <div id="nav" class="left_nav">
      <div :class="{ nav_style1: nav_style1 }" class="nav_router">
        <div>豆瓣电影TOP250</div>
        <div style="color:white" v-if="right_content1">
          -----
        </div>
      </div>
      <div class="nav_router" :class="{ nav_style2: nav_style2 }">
        <div to="/about">即将上映的电影</div>
        <div style="color:white" v-if="right_content2">
          -----
        </div>
      </div>
    </div>
    <div class="right_content"></div>
    <!-- <router-view /> -->
  </div>
</template>

<script>
export default {
  data() {
    return {
      content: [],
      nav_style1: true,
      nav_style2: false,
      right_content1: false,
      right_content2: false,
    };
  },
  created() {
    var that = this;
    //当前页面监视键盘输入
    document.onkeydown = function(e) {
      //事件对象兼容
      let e1 =
        e || event || window.event || arguments.callee.caller.arguments[0];
      if (that.nav_style1 == true) {
        // 判断导航
        if (e1 && e1.keyCode == 40) {
          console.log("下");
          that.nav_style1 = false;
          that.nav_style2 = true;
        }
        // 判断内容
        if (e1 && e1.keyCode == 39) {
          console.log("右");
          that.nav_style1 = false;
          that.right_content1 = true;
        }
      }
      if (that.nav_style2 == true) {
        if (e1 && e1.keyCode == 38) {
          console.log("上");
          that.nav_style1 = true;
          that.nav_style2 = false;
        }
      }

      // if (e1 && e1.keyCode == 37) {
      //   console.log("左");
      // } else if (e1 && e1.keyCode == 38) {
      //   console.log("上");
      // } else if (e1 && e1.keyCode == 39) {
      //   console.log("右");
      // } else if (e1 && e1.keyCode == 40) {
      //   console.log("下");
      // }
    };
  },
};
</script>
<style>
* {
  margin: 0;
  padding: 0;
  list-style: none;
  box-sizing: border-box;
  font-size: 20px;
}
#app {
  position: absolute;
  width: 1280px;
  height: 720px;
  display: flex;
  flex-flow: row nowrap;
}
.left_nav {
  width: 200px;
  background: rgba(0, 0, 0, 0.6);
  height: 100%;
  display: flex;
  flex-flow: column wrap;
  justify-content: flex-start;
}
.nav_router {
  line-height: 1.5em;
  height: 1.5em;
  margin: 50px 0px;
  margin-left: 25px;
  padding-left: 4px;
  color: white;
}
.nav_style1 {
  background-color: white;
  color: black;
}
.nav_style2 {
  background-color: white;
  color: black;
}
.right_content {
  border: blue solid 1px;
  width: 80%;
  height: 100%;
  margin-left: 70px;
}
</style>
