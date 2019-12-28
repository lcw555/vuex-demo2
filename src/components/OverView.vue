<template>
  <div class="OverView">
    <ul class="navigation">
      <!-- 全部和收藏 -->
      <li
        v-for="(item, key) in company"
        :key="key + 'A'"
        :class="defaultNavigation == item.LABEL ? 'selectedLi' : ''"
        @click="chnageNavigation(item.LABEL)"
      >{{ item.LABEL }}</li>
    </ul>
    <!-- 显示多选框 压力，流量，水质 -->
    <div class="radio">
      <el-radio-group v-model="radio" @change="changeShow">
        <el-radio :label="val" v-for="(val, key) in checkedFun" :key="key + 'E'">{{ val }}</el-radio>
      </el-radio-group>
    </div>

    <!-- 内容展示区 -->
    <div class="contain">
      <ul class="inlineBox">
        <!-- 内容块 showArr-->
        <li v-for="(val, key) in showArr" :key="key + 'B'">
          <div class="title" @click.stop>
            <!-- 更换定位图标 -->
            <i
              class="icon-more icon-msnui-foresight iconfont"
              title="定位"
              @click="position(val)"
              style="float:left;"
            >&#xe601;</i>
            {{  dataArr[0].ZBMARK }}
            <i
              title="收藏"
              @click="collect(val, key)"
              :class="
                val.ISCOLLECT == '1'
                  ? 'icon-more icon-wujiaoxing1 collected iconfont'
                  : 'icon-more icon-kongxinwujiaoxing uncollected iconfont'
              "
            >&#xe607;</i>
            <!-- 收藏后没有变为实体星星 -->
          </div>
          <div class="measureBox">
            <!-- 展示数据 -->
            <div
              class="containBox"
              v-for="(item, k, index) in val"
              :key="k + 'C'"
              v-show="item && checkedFun[index] == radio"
            >
              <!-- v-show="item && checkedFun[index] == radio" -->
              <div class="smallTitle">
                <span></span>
                {{ checkedFun[index] }}
              </div>

              <div class="measure" v-for="(list, num) in item" :key="num + 'D'">
                <!-- 压力（Mpa） -->
                <div>{{ list.LABEL + '(' + list.UNIT + ') : ' }}</div>
                <!-- 内容 -->
                <div v-if="list.ALERTTYPE == null" style="color:white">{{ list.VALUE }}</div>

                <div v-if="list.ALERTTYPE == '压力下限'" class="unusualA">
                  {{ list.VALUE }}
                  <i class="icon-more icon-jiantouxiajiang iconfont">&#xe605;</i>
                </div>
                <div v-if="list.ALERTTYPE == '压力上限'" class="unusualA">
                  {{ list.VALUE }}
                  <i class="icon-more icon-jiantoushangsheng iconfont">&#xe606;</i>
                </div>
                <div
                  v-if="list.ALERTTYPE == '零值'"
                  class="unusualC"
                  title="零值"
                >{{ list.VALUE + ' (零值)' }}</div>
                <div v-if="list.ALERTTYPE == '平值'" class="unusualA" title="平值">
                  {{ list.VALUE }}
                  <i
                    class="icon-more icon-hengxian1"
                    style="width: 15px margin-left: 18px"
                  >-</i>
                </div>
                <div v-if="list.ALERTTYPE == '超时'" class="unusualB" title="超时">
                  (通讯中断)
                  <i class="icon-more icon-zhongduan iconfont">&#xe7d0;</i>
                </div>
              </div>
            </div>
          </div>
        </li>
      </ul>
    </div>
  </div>
</template>

<script>
export default {
  name: "OverView",
  components: {},
  data() {
    return {
      defaultNavigation: "全部", //默认打开的导航栏
      radio: "", // 单选框  压力
      checkedFun: [], //显示多选框 "压力", "流量", "水质"
      dataArr: [], //数据数组
      alarmArr: [], //报警数据
      showArr: [], //最终展示数组
      tagName: {}, //详细站点名
      company: [
        {
          LABEL: "全部"
        },
        {
          LABEL: "收藏"
        }
      ]
    };
  },
  created() {
    // this.getDataTest();
    this.getZhanDianData();
  },
  methods: {
    close() {
      this.$emit("close");
    },
    // 切换导航栏
    chnageNavigation(label) {
      this.defaultNavigation = label;
      this.changeShow();
    },
    //首次获取数据
    getDataTest() {
      this.$axios
        // 具体数值
        .get("http://192.168.7.53:25091/api/scada/real/ids")
        .then(res => {
          console.log(res);
          this.dataArr = res.data; //与后端数据相对应存储-->数组
          console.log(this.dataArr);
        })
        .catch(function(error) {
          console.log(error);
        });
    },
    getZhanDianData() {
      this.$axios
        //站点数据
        .get("http://192.168.7.53:25091/api/scada/tree")
        .then(res => {
          this.dataArr = res.data; 
          this.dataArr.forEach(val => {
            this.checkedFun.push(val.ZBNAME);
             console.log(val.ZBNAME);
          });
          this.radio = this.checkedFun[0];
          this.changeShow();
        })
        .catch(function(error) {
          console.log(error);
        });
    },
        // 切换数据
    changeShow() {
      this.showArr = []; //最终展示数组
      // 获取所有站点(分类后)
      var data = [];
      this.dataArr.forEach(val => {
        console.log(val.children);
        val.children.forEach(item => {
          item.NAME = val.ZBNAME;
          console.log( item.NAME);
        });
        this.showArr = data.concat(val.children);
      });

            // radio显示
      data = data.filter(val => {
        let flag = false;
        if (this.radio == '配电站点') {
          if (val.TYPE.PRESURE.length) {
            flag = true;
          }
        } else if (this.radio == '占位站点（勿删）') {
          if (val.TYPE.YIELD.length) {
            flag = true;
          }
        } else if (this.radio == '智慧水厂占位点') {
          if (val.TYPE.FLOW.length) {
            flag = true;
          }
        } else {
          flag = false;
        }
        return flag;
      });

       // 过滤收藏和区域   问题：showArr 没有被赋值
      // .showArr = data.filter(val => {
      //   let flag = false;
      //   if (this.defaultNavigation == '全部') {
      //     flag = true;
      //   } else if (this.defaultNavigation == '收藏') {
      //     if (val.ISCOLLECT == '1') {
      //       flag = true;
      //     }
      //   }
      //   return flag;
      // });
    console.log(this.showArr);
            // 报警赋值
      if (this.alarmArr != []) {
        this.showArr.forEach(val => {
          for (var i in val.TYPE) {
            if(val.TYPE[i]){
              val.TYPE[i].forEach(it => {
                this.alarmArr.forEach(item => {
                  if (item.SID == it.ID) {
                    it.ALERTTYPE = item.DES;
                  }
                });
              });
            }
          }
        });
      }
    }
  },

watch: {
    // 切换radio关闭弹框
    radio(oldData, newData) {
      if (oldData != newData) {
        this.infoWindowShow = false;
      }
    }
},

};
</script>

<style scoped>
.OverView {
  /* background: var(--menu-bg); */
  /* background: rgb(64, 64, 64); */
  background-color: rgb(64, 64, 64);
  height: 100%;
  width: 100%;
  font-family: "Microsoft YaHei", serif;
}

@font-face {
  font-family: "iconfont";
  src: url("../font/iconfont.eot");
  src: url("../font/iconfont.eot?#iefix") format("embedded-opentype"),
    url("../font/iconfont.woff2") format("woff2"),
    url("../font/iconfont.woff") format("woff"),
    url("../font/iconfont.ttf") format("truetype"),
    url("../font/iconfont.svg#iconfont") format("svg");
}

.iconfont {
  font-family: "iconfont" !important;
  font-size: 16px;
  font-style: normal;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
}

ul,
li {
  list-style: none;
  margin: 0;
  padding: 0;
}

.navigation {
  /* color: var(--main-color); */
  /* background: var(--main-one); */
  color: rgb(40, 40, 40);
  background-color: rgb(40, 40, 40);
  display: flex;
  justify-content: center;
  align-items: center;
  text-align: center;
}

.navigation li {
  width: 100px;
  margin: 6px 4px;
  padding: 2px 0;
  cursor: pointer;
  /* color: var(--main-theme); */
  /* background: var(--main-one); */
  /* border: 1px solid var(--main-theme); */
  color: rgb(33, 163, 241);
  border: 1px solid rgb(33, 163, 241);
  border-radius: 3px;
  transition: all 0.5s;
}

.navigation .selectedLi {
  /* background: var(--main-theme); */
  background: rgb(0, 181, 247);
  transition: all 0.2s;
  color: #fff;
  transition: all 0.5s;
}
.el-radio {
  color: white !important;
  padding: 10px;
}
.radio .el-radio__label {
  /* color: var(--main-color) !important; */
  color: red !important;
}

.radio .el-radio-group {
  text-align: center;
  position: relative;
  left: 50%;
  transform: translateX(-50%);
}

.contain {
  height: calc(100% - 61px);
  overflow: auto;
}

.inlineBox {
  height: 100%;
  display: flex;
  justify-content: center;
  flex-wrap: wrap;
  align-content: flex-start;
  overflow: auto;
}

.inlineBox::-webkit-scrollbar,
.contain::-webkit-scrollbar {
  display: none;
}

.inlineBox::after {
  content: "";
  height: 0;
  display: block;
  visibility: hidden;
  clear: both;
}

.contain li {
  width: 200px;
  height: 100px;
  margin: 5px 10px 10px;
  float: left;
  overflow: hidden;
  border-radius: 3px;
  /* box-shadow: 0px 0px 12px var(--popup-shadow);  阴影*/
  box-shadow: 0px 0px 12px #11111111;
  cursor: pointer;
  transition: all 0.2s;
}

.contain li:hover {
  box-shadow: 0px 0px 45px #888888;
}

.measureBox {
  height: calc(100% - 24px);
  overflow: auto;
  font-size: 14px;
}

.measureBox::-webkit-scrollbar {
  display: none;
}

.title {
  font-size: 14px;
  height: 24px;
  line-height: 24px;
  text-align: center;
  background: rgb(40, 40, 40);
  /* color: var(--main-titlecolor); */
  color: white;
  padding: 0 10px;
}

.title span {
  /* color: var(--main-theme); */
  float: left;
  font-size: 12px;
  cursor: pointer;
}

.title i {
  font-size: 12px;
  line-height: 24px;
  cursor: pointer;
}

.collected {
  float: right;
  /* color: var(--start-collect); */
}

.uncollected {
  float: right;
  color: var(--main-titlecolor);
}

/* .containBox {
        border-bottom: 1px solid var(--main-three);
    } */

.containBox:last-child {
  border: 0;
}

.smallTitle {
  /* color: var(--main-theme); */
  color: rgb(33, 163, 241);
  padding: 4px 10px;
  border-bottom: 1px solid rgb(100, 100, 100);
}

.smallTitle > span {
  display: inline-block;
  width: 3px;
  height: 12px;
  margin-left: 4px;
  background: rgb(33, 163, 241);
  margin-right: 6px;
}

.measure {
  padding: 4px;
  width: calc(100% - 8px);
}

.measure > div {
  white-space: nowrap;
  overflow: hidden;
  text-overflow: ellipsis;
}

.measure > div:nth-child(2n + 1) {
  float: left;
  text-indent: 0.5em;
  color: rgb(180, 180, 180);
}

.measure > div:nth-child(2n) {
  float: right;
  text-align: right;
  font-weight: bold;
  /* color: var(--main-titlecolor); */
  position: relative;
}

.measure .unusualA {
  padding-right: 20px;
  color: red !important;
  animation: abnormalA 1s infinite;
}

.measure .unusualC {
  color: red !important;
  animation: abnormalA 1s infinite;
}

.measure .unusualB {
  color: rgb(90, 90, 90) !important;
}

.unusualB i {
  padding-left: 5px;
  font-size: 13px;
}

.unusualA i {
  font-size: 15px;
  position: absolute;
}

.measure::after {
  content: "";
  display: block;
  visibility: hidden;
  height: 0;
  clear: both;
}

.OverView .pagemask {
  width: 100vw;
  height: 100vh;
  position: absolute;
  left: 0;
  top: 0;
  z-index: 2;
  background: rgba(0, 0, 0, 0.5);
}

.OverView .pageBox {
  width: 90%;
  height: calc(100vh - 100px);
  background: #fff;
  position: absolute;
  left: 0;
  right: 0;
  top: -18px;
  bottom: 0;
  margin: 40px auto;
  z-index: 3;
  border-radius: 4px;
  box-shadow: 4px 6px 15px 0px #11111111;
  background-color: rgb(40, 40, 40);
}

.OverView .pageTitle {
  width: 100%;
  height: 30px;
  /* background: var(--popup-titlebg); */
  background: red;
  color: #fff;
  line-height: 30px;
  padding-left: 10px;
  border-radius: 4px 4px 0 0;
}

.OverView .pageTitle i {
  float: right;
  line-height: 30px;
  color: #fff;
  font-size: 16px;
  margin-right: 10px;
  cursor: pointer;
}

.HjInfoWindowBox .title {
  background: var(--popup-titlebg);
  /* background :red; */
}

.HjInfoWindowBox .title span {
  color: #fff;
}
.close-over-view {
  position: absolute;
  top: 10px;
  right: 10px;
  font-size: 16px;
  cursor: pointer;
  color: #fff;
}

.icon-kongxinwujiaoxing {
  color: yellow;
}
</style>