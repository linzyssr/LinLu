<template>
  <div id="root">
    
      <header>
        <!-- <Publicity v-show="!running" /> -->
        <el-select v-model="className" placeholder="请选择班级" class="select" @change="onChange">
          <el-option v-for="item in options" :key="item.value" :label="item.label" :value="item.value">
          </el-option>
        </el-select>
        <!-- <el-button class="res" type="text" @click="showResult = true">
        结果
      </el-button> -->
        <el-button class="con" type="text" @click="showConfig = true">
          配置
        </el-button>
      </header>
      <div id="main" :class="{ mask: showRes }"></div>
      <div id="tags">
        <ul v-for="item in datas" :key="item.key">
          <li>
            <a href="javascript:void(0);" :style="{
              color: '#fff',
            }">
              {{ item.name ? item.name : item.key }}
              <img v-if="item.photo" :src="item.photo" :width="50" :height="50" />
            </a>
          </li>
        </ul>
      </div>
      <transition name="bounce">
        <div id="resbox" v-show="showRes">
          <p @click="showRes = false">{{ categoryName }}结果：</p>
          <div class="container">
            <span v-for="item in resArr" :key="item" class="itemres" :style="resCardStyle" :data-id="item"
              @click="showRes = false" :class="{
                numberOver:
                  !!photos.find((d) => d.id === item) ||
                  !!list.find((d) => d.key === item),
              }">
              <span class="cont" v-if="!photos.find((d) => d.id === item)">
                <span v-if="!!list.find((d) => d.key === item)" :style="{
                  fontSize: '40px',
                }">
                  {{ list.find((d) => d.key === item).name }}
                </span>
                <span v-else>
                  {{ item }}
                </span>
              </span>
              <img v-if="photos.find((d) => d.id === item)" :src="photos.find((d) => d.id === item).value" alt="photo"
                :width="160" :height="160" />
            </span>
          </div>
        </div>
      </transition>

      <el-button class="audio" type="text" @click="() => {
        playAudio(!audioPlaying);
      }
        ">
        <i class="iconfont" :class="[audioPlaying ? 'iconstop' : 'iconplay1']"></i>
      </el-button>

      <LotteryConfig :visible.sync="showConfig" @resetconfig="reloadTagCanvas" />
      <Tool @toggle="toggle" @resetConfig="reloadTagCanvas" @getPhoto="getPhoto" :running="running"
        :closeRes="closeRes" />
      <Result :visible.sync="showResult"></Result>

      <span class="copy-right"> Copyright©linzyssr@gmail.com </span>

      <audio id="audiobg" preload="auto" controls autoplay loop @play="playHandler" @pause="pauseHandler">
        <source :src="audioSrc" />
        你的浏览器不支持audio标签
      </audio>
  </div>
</template>
<script>
import LotteryConfig from "@/components/LotteryConfig";
import Publicity from "@/components/Publicity";
import Tool from "@/components/Tool";
import bgaudio from "@/assets/bg.mp3";
import beginaudio from "@/assets/begin.mp3";
import {
  getData,
  configField,
  resultField,
  newLotteryField,
  conversionCategoryName,
  listField,
} from "@/helper/index";
import { luckydrawHandler } from "@/helper/algorithm";
import Result from "@/components/Result";
import { database, DB_STORE_NAME } from "@/helper/db";
export default {
  name: "App",

  components: { LotteryConfig, Publicity, Tool, Result },

  computed: {
    resCardStyle() {
      const style = { fontSize: "30px" };
      const { number } = this.config;
      if (number < 100) {
        style.fontSize = "100px";
      } else if (number < 1000) {
        style.fontSize = "80px";
      } else if (number < 10000) {
        style.fontSize = "60px";
      }
      return style;
    },
    config: {
      get() {
        return this.$store.state.config;
      },
    },
    result: {
      get() {
        return this.$store.state.result;
      },
      set(val) {
        this.$store.commit("setResult", val);
      },
    },
    list() {
      return this.$store.state.list;
    },
    allresult() {
      let allresult = [];
      for (const key in this.result) {
        if (this.result.hasOwnProperty(key)) {
          const element = this.result[key];
          allresult = allresult.concat(element);
        }
      }
      return allresult;
    },
    datas() {
      console.log("=============")
      if (this.className === '3') return this.treeList;
      if (this.className === '4') return this.fourList;
      const { number } = this.config;
      const nums = number >= 1500 ? 500 : this.config.number;
      const configNum = number > 1500 ? Math.floor(number / 3) : number;
      const randomShowNums = luckydrawHandler(configNum, [], nums);
      const randomShowDatas = randomShowNums.map((item) => {
        const listData = this.list.find((d) => d.key === item);
        const photo = this.photos.find((d) => d.id === item);
        return {
          key: item * (number > 1500 ? 3 : 1),
          name: listData ? listData.name : "",
          photo: photo ? photo.value : "",
        };
      });
      return randomShowDatas;
    },
    categoryName() {
      return conversionCategoryName(this.category);
    },
    photos() {
      return this.$store.state.photos;
    },
  },
  created() {
    const data = getData(configField);
    if (data) {
      this.$store.commit("setConfig", Object.assign({}, data));
    }
    const result = getData(resultField);
    if (result) {
      this.$store.commit("setResult", result);
    }

    const newLottery = getData(newLotteryField);
    if (newLottery) {
      const config = this.config;
      newLottery.forEach((item) => {
        this.$store.commit("setNewLottery", item);
        if (!config[item.key]) {
          this.$set(config, item.key, 0);
        }
      });
      this.$store.commit("setConfig", config);
    }

    const list = getData(listField);
    if (list) {
      this.$store.commit("setList", list);
    }
  },

  data() {
    return {
      dataList: [],
      running: false,
      showRes: false,
      showConfig: false,
      showResult: false,
      resArr: [],
      category: "",
      audioPlaying: false,
      audioSrc: bgaudio,

      options: [
        {
          value: "3",
          label: "3班",
        },
        {
          value: "4",
          label: "4班",
        },
      ],
      className: "3",

      fourList: Object.freeze([
        {
          "key": 1,
          "name": "陈坤志"
        },
        {
          "key": 2,
          "name": "蒋宇航"
        },
        {
          "key": 3,
          "name": "张宁夏"
        },
        {
          "key": 4,
          "name": "林培鸿"
        },
        {
          "key": 5,
          "name": "林依萱"
        },
        {
          "key": 6,
          "name": "李博文"
        },
        {
          "key": 7,
          "name": "王钰晗"
        },
        {
          "key": 8,
          "name": "伍彦霖"
        },
        {
          "key": 9,
          "name": "苏嘉俐"
        },
        {
          "key": 10,
          "name": "庄梓梵"
        },
        {
          "key": 11,
          "name": "马艺馨"
        },
        {
          "key": 12,
          "name": "郭瑜晨"
        },
        {
          "key": 13,
          "name": "张淑婷"
        },
        {
          "key": 14,
          "name": "黄润民"
        },
        {
          "key": 15,
          "name": "张炜婷"
        },
        {
          "key": 16,
          "name": "陈煜坤"
        },
        {
          "key": 17,
          "name": "许佳倩"
        },
        {
          "key": 18,
          "name": "何庚鸿"
        },
        {
          "key": 19,
          "name": "王耀阳"
        },
        {
          "key": 20,
          "name": "李梓涵"
        },
        {
          "key": 21,
          "name": "王思涵"
        },
        {
          "key": 22,
          "name": "彭梓轩"
        },
        {
          "key": 23,
          "name": "贾思晨"
        },
        {
          "key": 24,
          "name": "陈婉欣"
        },
        {
          "key": 25,
          "name": "林思渟"
        },
        {
          "key": 26,
          "name": "陈志成"
        },
        {
          "key": 27,
          "name": "宁甜妤"
        },
        {
          "key": 28,
          "name": "林海天"
        },
        {
          "key": 29,
          "name": "刘慧珠"
        },
        {
          "key": 30,
          "name": "石夏兰"
        },
        {
          "key": 31,
          "name": "赵庭鋆"
        },
        {
          "key": 32,
          "name": "高楚儿"
        },
        {
          "key": 33,
          "name": "娄忠智"
        },
        {
          "key": 34,
          "name": "张慧灵"
        },
        {
          "key": 35,
          "name": "严厚文"
        },
        {
          "key": 36,
          "name": "赖炜彬"
        },
        {
          "key": 37,
          "name": "罗苑菲"
        },
        {
          "key": 38,
          "name": "郭帅斌"
        },
        {
          "key": 39,
          "name": "郑承皓"
        },
        {
          "key": 40,
          "name": "孙思淇"
        },
        {
          "key": 41,
          "name": "揭佳琪"
        },
        {
          "key": 42,
          "name": "方有韬"
        },
        {
          "key": 43,
          "name": "陈澜"
        },
        {
          "key": 44,
          "name": "张欣如"
        },
        {
          "key": 45,
          "name": "魏登潼"
        },
        {
          "key": 46,
          "name": "刘煜铭"
        },
        {
          "key": 47,
          "name": "佘玮灿"
        },
        {
          "key": 48,
          "name": "杨芳"
        },
        {
          "key": 49,
          "name": "苏锦鸿"
        },
        {
          "key": 50,
          "name": "张宇榕"
        }
      ]),
      treeList: Object.freeze([
        {
          "key": 1,
          "name": "林子涵"
        },
        {
          "key": 2,
          "name": "刘佳钰"
        },
        {
          "key": 3,
          "name": "邱建辉"
        },
        {
          "key": 4,
          "name": "徐浩哲"
        },
        {
          "key": 5,
          "name": "刘其沛"
        },
        {
          "key": 6,
          "name": "陈欣娜"
        },
        {
          "key": 7,
          "name": "杨艳玲"
        },
        {
          "key": 8,
          "name": "华岳"
        },
        {
          "key": 9,
          "name": "廖文博"
        },
        {
          "key": 10,
          "name": "潘学春"
        },
        {
          "key": 11,
          "name": "王泽涵"
        },
        {
          "key": 12,
          "name": "张修涵"
        },
        {
          "key": 13,
          "name": "许辰"
        },
        {
          "key": 14,
          "name": "潘学秋"
        },
        {
          "key": 15,
          "name": "王诗雅"
        },
        {
          "key": 16,
          "name": "温云豪"
        },
        {
          "key": 17,
          "name": "李任阳"
        },
        {
          "key": 18,
          "name": "张涵"
        },
        {
          "key": 19,
          "name": "潘学艳"
        },
        {
          "key": 20,
          "name": "许瀚迪"
        },
        {
          "key": 21,
          "name": "曹义磊"
        },
        {
          "key": 22,
          "name": "邓梅英"
        },
        {
          "key": 23,
          "name": "张景萱"
        },
        {
          "key": 24,
          "name": "蔡子谦"
        },
        {
          "key": 25,
          "name": "曾海婷"
        },
        {
          "key": 26,
          "name": "胡金婷"
        },
        {
          "key": 27,
          "name": "曾凯贤"
        },
        {
          "key": 28,
          "name": "阮先伟"
        },
        {
          "key": 29,
          "name": "黄婧婧"
        },
        {
          "key": 30,
          "name": "巫泉"
        },
        {
          "key": 31,
          "name": "张芸菲"
        },
        {
          "key": 32,
          "name": "徐佳彤"
        },
        {
          "key": 33,
          "name": "上官靖航"
        },
        {
          "key": 34,
          "name": "方浩权"
        },
        {
          "key": 35,
          "name": "叶嘉杭"
        },
        {
          "key": 36,
          "name": "孙诗涵"
        },
        {
          "key": 37,
          "name": "冯烁"
        },
        {
          "key": 38,
          "name": "张欣怡"
        },
        {
          "key": 39,
          "name": "李轩"
        },
        {
          "key": 40,
          "name": "杨典立"
        },
        {
          "key": 41,
          "name": "郭振兴"
        },
        {
          "key": 42,
          "name": "刘玉轩"
        },
        {
          "key": 43,
          "name": "李芯然"
        },
        {
          "key": 44,
          "name": "叶巧伶"
        },
        {
          "key": 45,
          "name": "杨韬"
        },
        {
          "key": 46,
          "name": "苏振升"
        },
        {
          "key": 47,
          "name": "胡意翔"
        },
        {
          "key": 48,
          "name": "谭小飞"
        }
      ])
    };
  },
  watch: {
    photos: {
      deep: true,
      handler() {
        this.$nextTick(() => {
          this.reloadTagCanvas();
        });
      },
    },
  },
  mounted() {
    this.startTagCanvas();
    setTimeout(() => {
      this.getPhoto();
    }, 1000);
    window.addEventListener("resize", this.reportWindowSize);
  },
  beforeDestroy() {
    window.removeEventListener("resize", this.reportWindowSize);
  },
  methods: {
    reportWindowSize() {
      const AppCanvas = this.$el.querySelector("#rootcanvas");
      if (AppCanvas.parentElement) {
        AppCanvas.parentElement.removeChild(AppCanvas);
      }
      this.startTagCanvas();
    },
    playHandler() {
      this.audioPlaying = true;
    },
    pauseHandler() {
      this.audioPlaying = false;
    },
    playAudio(type) {
      if (type) {
        this.$el.querySelector("#audiobg").play();
      } else {
        this.$el.querySelector("#audiobg").pause();
      }
    },
    loadAudio() {
      this.$el.querySelector("#audiobg").load();
      this.$nextTick(() => {
        this.$el.querySelector("#audiobg").play();
      });
    },
    getPhoto() {
      database.getAll(DB_STORE_NAME).then((res) => {
        if (res && res.length > 0) {
          this.$store.commit("setPhotos", res);
        }
      });
    },
    speed() {
      return [0.1 * Math.random() + 0.01, -(0.1 * Math.random() + 0.01)];
    },
    createCanvas() {
      const canvas = document.createElement("canvas");
      canvas.width = document.body.offsetWidth;
      canvas.height = document.body.offsetHeight;
      canvas.id = "rootcanvas";
      this.$el.querySelector("#main").appendChild(canvas);
    },
    startTagCanvas() {
      this.createCanvas();
      const { speed } = this;
      window.TagCanvas.Start("rootcanvas", "tags", {
        textColour: null,
        initial: speed(),
        dragControl: 1,
        textHeight: 20,
        noSelect: true,
        lock: "xy",
      });
    },
    reloadTagCanvas() {
      window.TagCanvas.Reload("rootcanvas");
    },
    closeRes() {
      this.showRes = false;
    },
    toggle(form) {
      const { speed, config } = this;
      if (this.running) {
        this.audioSrc = bgaudio;
        this.loadAudio();

        window.TagCanvas.SetSpeed("rootcanvas", speed());
        this.showRes = true;
        this.running = !this.running;
        this.$nextTick(() => {
          this.reloadTagCanvas();
        });
      } else {
        this.showRes = false;
        if (!form) {
          return;
        }

        this.audioSrc = beginaudio;
        this.loadAudio();

        const { number } = config;
        const { category, mode, qty, remain, allin } = form;
        let num = 1;
        if (mode === 1 || mode === 5) {
          num = mode;
        } else if (mode === 0) {
          num = remain;
        } else if (mode === 99) {
          num = qty;
        }
        const resArr = luckydrawHandler(
          number,
          allin ? [] : this.allresult,
          num,
          this.className
        );
        this.resArr = resArr;

        this.category = category;
        if (!this.result[category]) {
          this.$set(this.result, category, []);
        }
        const oldRes = this.result[category] || [];
        const data = Object.assign({}, this.result, {
          [category]: oldRes.concat(resArr),
        });
        this.result = data;
        window.TagCanvas.SetSpeed("rootcanvas", [5, 1]);
        this.running = !this.running;
      }
    },

    onChange(val) {
      try {
        const data = getData(configField);
      if (data) {
        this.$store.commit("setConfig", Object.assign({}, data));
      }
      const result = getData(resultField);
      if (result) {
        this.$store.commit("setResult", result);
      }

      const newLottery = getData(newLotteryField);
      if (newLottery) {
        const config = this.config;
        newLottery.forEach((item) => {
          this.$store.commit("setNewLottery", item);
          if (!config[item.key]) {
            this.$set(config, item.key, 0);
          }
        });
        this.$store.commit("setConfig", config);
      }

      const list = getData(listField);
      if (list) {
        this.$store.commit("setList", list);
      }
      } catch (error) {

      } finally {
        this.startTagCanvas();
        setTimeout(() => {
          this.getPhoto();
        }, 1000);
        window.addEventListener("resize", this.reportWindowSize);
      }
     
    },
  },
};
</script>
<style lang="scss">
#root {
  height: 100%;
  position: relative;
  background-image: url("./assets/bg1.jpg");
  background-size: 100% 100%;
  background-position: center center;
  background-repeat: no-repeat;
  background-color: #121936;

  .mask {
    -webkit-filter: blur(5px);
    filter: blur(5px);
  }

  header {
    height: 50px;
    line-height: 50px;
    position: relative;

    .el-button {
      position: absolute;
      top: 17px;
      padding: 0;
      z-index: 9999;

      &.con {
        right: 20px;
      }

      &.res {
        right: 100px;
      }

      &.select {
        right: 200px;
      }
    }
  }

  .audio {
    position: absolute;
    top: 100px;
    right: 30px;
    width: 40px;
    height: 40px;
    line-height: 40px;
    border: 1px solid #fff;
    border-radius: 50%;
    padding: 0;
    text-align: center;

    .iconfont {
      position: relative;
      left: 1px;
    }
  }

  .copy-right {
    position: absolute;
    right: 0;
    bottom: 0;
    color: #ccc;
    font-size: 12px;
  }

  .bounce-enter-active {
    animation: bounce-in 1.5s;
  }

  .bounce-leave-active {
    animation: bounce-in 0s reverse;
  }
}

#main {
  height: 100%;
}

#resbox {
  position: absolute;
  top: 50%;
  left: 50%;
  width: 1280px;
  transform: translateX(-50%) translateY(-50%);
  text-align: center;

  p {
    color: red;
    font-size: 50px;
    line-height: 120px;
  }

  .container {
    display: flex;
    justify-content: center;
    flex-wrap: wrap;
  }

  .itemres {
    background: #fff;
    width: 160px;
    height: 160px;
    border-radius: 4px;
    border: 1px solid #ccc;
    line-height: 160px;
    font-weight: bold;
    margin-right: 20px;
    margin-bottom: 20px;
    cursor: pointer;
    display: flex;
    align-items: center;
    justify-content: center;
    position: relative;

    .cont {
      display: flex;
      justify-content: center;
      align-items: center;
    }

    &.numberOver::before {
      content: attr(data-id);
      width: 30px;
      height: 22px;
      line-height: 22px;
      background-color: #fff;
      position: absolute;
      bottom: 0;
      left: 0;
      font-size: 14px;
      // border-radius: 50%;
      z-index: 1;
    }
  }
}
</style>
