<template>
  <div class="home">
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
    <div
      class="right_content"
      v-loading.fullscreen.lock="lod"
      element-loading-text="拼命加载中"
      element-loading-spinner="el-icon-loading"
      element-loading-background="rgba(0, 0, 0, 0.8)"
    >
      <div
        class="right_content_default"
        v-for="(item, index) in content"
        :id="item.id"
        :key="item.id"
        ref="content_on"
      >
        <!-- <img :src="item.images.small | getImages(item.images.small)" /> -->
        <img :src="item.images.small" />
        <div class="text">{{ item.title }}{{ index }}</div>
      </div>
    </div>
    <div class="page">{{ page }} / {{ total / count }}页</div>
  </div>
</template>

<script>
// @ is an alias to /src
// import HelloWorld from '@/components/HelloWorld.vue'
import About from "../views/About";

export default {
  name: "Home",
  data() {
    return {
      content: [],
      page: 1,
      start: 0,
      count: "",
      total: "",
      nav_style1: true,
      nav_style2: false,
      right_content1: false,
      right_content2: false,
      right_column: "",
      right_row: "",
      setout: false,
      test_row: "",
      test_content2: true,
      test_content1: false,
      show_page1: false,
      ca_page: 0,
      id: "",
      lod: true,
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

      // 右侧操作
      if (that.right_content1 == true || that.right_content2 == true) {
        console.log("在右侧操作");
        // 右方向建
        if (e1 && e1.keyCode == 39) {
          if (that.right_column < 5) {
            that.right_column++;
            that.right_in();
          }
        }
        // 左方向建
        if (e1 && e1.keyCode == 37) {
          if (that.right_column > 0) {
            // 在最左侧时，x轴为1，如果为零值，退出
            that.right_column--;
            that.right_in();
          }
          // 退出右侧
          if (that.right_column == 0) {
            that.on_style("out");
            that.right_column = "";
            that.right_row = "";
            // 判断属于哪个导航
            if (that.right_content1 == true) {
              that.right_content1 = false;
              that.nav_style1 = true;
            } else if (that.right_content2 == true) {
              that.right_content2 = false;
              that.nav_style2 = true;
            }
          }
        }

        // 上方向
        if (e1 && e1.keyCode == 38) {
          if (that.page == 1) {
            if (that.right_row > 1) {
              that.right_row--;
              that.right_in();
            }
          }
          // 向上翻页
          if (that.page > 1) {
            if (that.right_row > 0) {
              that.right_row--;
              that.right_in();
              if (that.right_row == 0) {
                // 暂存翻页之前的页码
                that.ca_page = that.page;

                that.start -= 10;
                that.page--;
                // 单独处理第一页行位置（2）

                if (that.right_content1 == true) {
                  that.onload(2);
                } else if (that.right_content2 == true) {
                  that.content2_onload(2);
                }
              }
            }
          }
        }
        // 下方向
        if (e1 && e1.keyCode == 40) {
          // 如果y轴小于3，继续自增，请求数据
          if (that.right_row < 3) {
            that.right_row++;
            that.right_in();
          }
          if (that.page < that.total / 10) {
            if (that.right_row == 3) {
              // 暂存翻页之前的页码
              that.ca_page = that.page;
              that.start += 10;
              that.page++;

              if (that.right_content1 == true) {
                that.onload();
              } else if (that.right_content2 == true) {
                that.content2_onload();
              }
            }
          }
        }
      }

      // 导航1
      if (that.nav_style1 == true) {
        // 判断导航
        if (e1 && e1.keyCode == 40) {
          console.log("下");
          that.nav_style1 = false;
          that.nav_style2 = true;
          that.test_content2 = true;
        }

        // 进入右侧
        if (e1 && e1.keyCode == 39) {
          console.log("右");
          that.nav_style1 = false;
          that.right_content1 = true;
          // 右侧初始值
          if (that.right_column == "") {
            that.right_column = 1;
          }
          if (that.right_row == "") {
            that.right_row = 1;
          }
          // 判断右侧内容
          that.right_in();
        }
      }

      // 导航2
      if (that.nav_style2 == true) {
        if (that.start != 0) {
          that.start = 0;
        }
        // 是否请求数据
        if (that.test_content2 == true) {
          that.content2_onload();
          that.test_content2 = false;
        }
        if (e1 && e1.keyCode == 38) {
          console.log("上");
          that.nav_style1 = true;
          that.nav_style2 = false;
          that.test_content1 = true;
          // 加载导航1数据
          that.nav1();
        }
        // 进入右侧
        if (e1 && e1.keyCode == 39) {
          console.log("右");
          that.nav_style2 = false;
          that.right_content2 = true;

          // 右侧初始值
          if (that.right_column == "") {
            that.right_column = 1;
          }
          if (that.right_row == "") {
            that.right_row = 1;
          }
          // 判断右侧内容
          that.right_in();
        }
      }
      if (e1 && e1.keyCode == 13) {
        that.$router.push({
          path: "/about/",
          component: About,
          query: {
            id: that.id,
          },
        });
        // console.log("回车");
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
      // https://www.cnblogs.com/Shyno/p/11561632.html
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
    onload(value) {
      var that = this;
      this.$axios
        .get("https://douban.uieee.com/v2/movie/top250", {
          params: {
            apikey: "0df993c66c0c636e29ecbb5344252a4a",
            start: that.start,
            count: 10,
          },
        })
        .then(function(res) {
          console.log(res);
          that.content = res.data.subjects;
          that.count = res.data.count;
          that.total = res.data.total;
          console.log(that.content);
          that.lod = true;
          // 记录位置
          if (value == 2) {
            that.test_row = 2;
          } else {
            that.test_row = "";
          }
        })
        .catch(function(error) {
          console.log(error);
        });
    },
    content2_onload(value) {
      var that = this;

      this.$axios
        .get("https://douban.uieee.com/v2/movie/coming_soon", {
          params: {
            apikey: "0df993c66c0c636e29ecbb5344252a4a",
            start: that.start,
            count: 10,
          },
        })
        .then(function(res) {
          console.log(res);
          if (res.data.subjects == "") {
            console.log("空");
          }
          that.content = res.data.subjects;
          that.count = res.data.count;
          that.total = res.data.total;
          console.log(that.content);
          that.lod = true;
          // 记录位置
          if (value == 2) {
            that.test_row = 2;
          } else {
            that.test_row = "";
          }
        })
        .catch(function(error) {
          console.log(error);
        });
    },
    right_in() {
      switch (this.right_column) {
        case 1:
          console.log("1");
          if (this.right_row == 1) {
            this.on_style(0);
          } else if (this.right_row == 2) {
            this.on_style(5);
          }
          break;
        case 2:
          console.log("2");
          if (this.right_row == 1) {
            this.on_style(1);
          } else if (this.right_row == 2) {
            this.on_style(6);
          }
          break;
        case 3:
          console.log("3");
          if (this.right_row == 1) {
            this.on_style(2);
          } else if (this.right_row == 2) {
            this.on_style(7);
          }
          break;
        case 4:
          console.log("4");
          if (this.right_row == 1) {
            this.on_style(3);
          } else if (this.right_row == 2) {
            this.on_style(8);
          }
          break;
        case 5:
          console.log("5");
          if (this.right_row == 1) {
            this.on_style(4);
          } else if (this.right_row == 2) {
            this.on_style(9);
          }
          break;

        default:
          break;
      }
    },
    on_style(value) {
      // vue $refs获取DOM
      for (let i = 0; i < this.$refs.content_on.length; i++) {
        this.$refs.content_on[i].style = "";
        if (value != "out") {
          this.$refs.content_on[value].style =
            "box-shadow: 6px 6px 15px #000000;";
          // 获取id
          this.id = this.$refs.content_on[value].id;
          console.log(this.id);
        }
        // console.log(this.$refs.content_on[i]);
      }
    },
    enter_on() {},
    page_row() {
      var that = this;
      // 逻辑乱了— — — —注释掉
      // if (that.test_row == 2) {
      //   that.right_row = 2;
      // }
      // if (that.show_page2 == true) {
      //   that.right_row = 2;
      //   that.show_page2 == false;
      // }
      if (that.page > 1) {
        // 边框
        that.right_row = 1;
        that.right_column = this.right_column;
        that.show_page1 = true;
        // if (that.show_page1 == false) {
        //   that.right_row = 2;
        // }
      }

      // 判断是否向上翻页，true为2
      if (that.page < that.ca_page) {
        that.right_row = 2;
        that.show_page1 = false;
        // if (that.page == 1) {
        //   that.right_row = 1;
        // }
      }

      if (that.page == 1 && that.show_page1 == true) {
        that.right_row = 2;
        that.show_page1 = false;
      }
    },
    nav1() {
      var that = this;
      if (that.test_content1 == true) {
        console.log("true");
        that.onload();
        that.test_content1 = false;
      }
    },
  },
  mounted() {},
  beforeUpdate() {
    // this.lod = false;
  },
  // 由于数据更改导致的虚拟 DOM 重新渲染和打补丁，在这之后会调用该钩子;注意 updated 不会保证所有的子组件也都一起被重绘。如果你希望等到整个视图都重绘完毕，可以在 updated 里使用 vm.$nextTick：
  updated() {
    this.$nextTick(() => {
      this.right_in();
      this.page_row();
      this.lod = false;

      // Code that will run only after the
      // entire view has been re-rendered
    });
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
.home {
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
  /* border: blue solid 1px; */
  width: 80%;
  height: 90%;
  margin-left: 70px;
  display: flex;
  flex-flow: row wrap;
  align-items: center;
  justify-content: space-around;
  padding: 80px 0px;
}
.right_content_default {
  border: solid 2px rgb(170, 170, 170);
  width: 160px;
  height: 220px;
  margin: 20px;
  position: relative;
}
.right_content_default img {
  width: 100%;
  height: 100%;
}
.right_content_default .text {
  position: absolute;
  width: 100%;
  line-height: 1.5em;
  text-align: center;
  background-color: #aaaaaa;
  z-index: 999;
  bottom: 0;
  color: white;
  font-size: 18px;
}
.content_on {
  box-shadow: 6px 6px 15px #000000;
}
.page {
  font-size: 16px;
  position: absolute;
  right: 0;
  bottom: 0;
}
</style>
