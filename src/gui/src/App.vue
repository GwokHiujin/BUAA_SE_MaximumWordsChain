<template>
  <v-app>
    <v-app-bar app color="#626c91" dark>
    <v-toolbar-title class="font-weight-black">
      Maximum Words Chain Gen | GX🌳YY
    </v-toolbar-title>
    </v-app-bar>

    <v-content id="main">
      <v-container fluid>
        <v-row
            class="align-center justify-center"
        >
          <v-col
              cols="6"
              style="height: 100%"
              class="px-15"
          >
            <material-card color="#a0a7e6"
                           class="px-5 py-3 elevation-2">
              <template v-slot:heading>
                <div class="display-2 font-weight-black pb-3">
                  ✨ 设置参数
                </div>

                <v-divider/>

                <v-row class="pt-3">
                  <v-col cols="6" class="subtitle-1 font-weight-light">
                    在下方输入文本和单词链生成参数，获得可视化单词链图！
                  </v-col>
                  <v-col cols="6" class="pl-lg-16">
                    <v-btn
                        depressed
                        color="#626c91"
                        dark
                        class="font-weight-bold"
                        @click="genWordsChain()"
                    >
                      ✅ 求解单词链
                    </v-btn>
                  </v-col>
                </v-row>
              </template>
              <v-alert
                  icon="mdi-shield-lock-outline"
                  text
                  color="teal"
                  border="left"
                  class="mt-5"
                  v-if="emptyWordsAlert"
              >
                <v-row align="center" no-gutters>
                  <v-col class="grow">
                    单词文本不可为空！
                  </v-col>
                  <v-col class="shrink">
                    <v-btn
                        text
                        @click="emptyWordsAlert = false"
                    >
                      <v-icon>
                        mdi-close
                      </v-icon>
                    </v-btn>
                  </v-col>
                </v-row>
              </v-alert>

              <v-alert
                  icon="mdi-shield-lock-outline"
                  text
                  color="teal"
                  border="left"
                  class="mt-5"
                  v-if="conflictParamsAlert"
              >
                <v-row align="center" no-gutters>
                  <v-col class="grow">
                    您输入了不兼容的参数！
                  </v-col>
                  <v-col class="shrink">
                    <v-btn
                        text
                        @click="conflictParamsAlert = false"
                    >
                      <v-icon>
                        mdi-close
                      </v-icon>
                    </v-btn>
                  </v-col>
                </v-row>
              </v-alert>

              <v-alert
                  icon="mdi-shield-lock-outline"
                  text
                  color="teal"
                  border="left"
                  class="mt-5"
                  v-if="exceptionAlert"
              >
                <v-row align="center" no-gutters>
                  <v-col class="grow">
                    {{exceptionMsg}}
                  </v-col>
                  <v-col class="shrink">
                    <v-btn
                        text
                        @click="exceptionAlert = false"
                    >
                      <v-icon>
                        mdi-close
                      </v-icon>
                    </v-btn>
                  </v-col>
                </v-row>
              </v-alert>
              <v-row class="mt-3">
                <v-col cols="6" class="px-5 pt-5">
                  <v-file-input
                      accept=".txt"
                      color="indigo"
                      truncate-length="50"
                      placeholder="上传 .txt 文件"
                      @change="uploadFile()"
                      v-model="fileInfo"
                  ></v-file-input>
                </v-col>

              </v-row>
              <v-row class="px-5">
                <v-textarea
                    color="indigo"
                    placeholder="在此手动输入单词文本或使用文件上传"
                    hint="请输入想生成单词链的目标文本！"
                    auto-grow
                    v-model="rawWords"
                    no-resize
                    rows="3"
                ></v-textarea>
              </v-row>
              <v-form v-model="valid">
                <v-container>
                  <v-radio-group
                      v-model="calType"
                      row
                      class="pb-5"
                      label="请选择单词链计算类型："
                  >
                    <v-radio
                        label="所有单词链"
                        value="1"
                        color="#6be6c1"
                    ></v-radio>
                    <v-radio
                        label="单词数最多"
                        value="2"
                        color="#6be6c1"
                    ></v-radio>
                    <v-radio
                        label="字母数最多"
                        value="3"
                        color="#6be6c1"
                    ></v-radio>
                  </v-radio-group>

                  <v-row class="pb-0 px-5">
                    <v-switch
                        v-model="allowRing"
                        inset
                        label="是否允许成环"
                        color="#6be6c1"
                    ></v-switch>
                  </v-row>
                  <v-row>
                    <v-col cols="4">
                      <v-overflow-btn
                          class="my-2"
                          :items="alphabetOptions"
                          label="必须出现的首字母"
                          hint="必须出现的首字母"
                          item-value="text"
                          color="#6be6c1"
                          v-model="headLetterMust"
                      ></v-overflow-btn>
                    </v-col>
                    <v-col cols="4">
                      <v-overflow-btn
                          class="my-2"
                          :items="alphabetOptions"
                          label="必须出现的尾字母"
                          hint="必须出现的尾字母"
                          item-value="text"
                          color="#6be6c1"
                          v-model="tailLetterMust"
                      ></v-overflow-btn>
                    </v-col>
                    <v-col cols="4">
                      <v-overflow-btn
                          class="my-2"
                          :items="alphabetOptions"
                          label="不能出现的首字母"
                          hint="不能出现的首字母"
                          item-value="text"
                          color="#6be6c1"
                          v-model="headLetterNot"
                      ></v-overflow-btn>
                    </v-col>
                  </v-row>
                </v-container>
              </v-form>
            </material-card>
          </v-col>

          <v-col
              cols="6"
              style="height: 100%"
              class="px-15 py-15"
          >
            <material-card color="#a0a7e6"
                           class="px-5 py-3 elevation-2"
            >
              <template v-slot:heading>
                <div class="display-2 font-weight-black pb-3">
                  😎 单词链求解结果
                </div>

                <v-divider/>

                <v-row class="pt-3">
                  <v-col cols="6" class="subtitle-1 font-weight-light">
                    共求解得到 {{calNum}} 条单词链，用时 {{calTime}} 秒。
                  </v-col>
                  <v-col cols="6" class="pl-lg-16">
                    <v-btn
                        depressed
                        color="#626c91"
                        dark
                        class="font-weight-bold"
                        @click="saveFile()"
                    >
                      📑 导出结果 .txt 文件
                    </v-btn>
                  </v-col>
                </v-row>
              </template>
              <v-container>
                <div id="graph" style="height: 450px; width: 550px; overflow: hidden" class="pb-5"/>
              </v-container>
            </material-card>
          </v-col>
        </v-row>
      </v-container>
    </v-content>
  </v-app>
</template>

<script>
import * as echarts from 'echarts';
import materialCard from "@/MaterialCard.vue";
import saveAs from 'file-saver'

const ffi = require('ffi-napi')
const myDll = ffi.Library('./core.dll', {
  gen_chains_all: ['int',
    ['string']],
  gen_chain_word: ['int',
    ['string', 'char', 'char', 'char', 'bool']],
  gen_chain_char: ['int',
    ['string', 'char', 'char', 'char', 'bool']],
  get_execution_time: ['double',
    []],
  getResult: ['string',
    []]
})

const alphabetOptions = [];
alphabetOptions.push('none')
for (let i = 0; i < 26; i++) {
  const value = String.fromCharCode(97 + i);
  alphabetOptions.push(value)
}

var myChart;

export default {
  name: 'App',
  components: {
    materialCard,
  },
  data: () => ({
    rawWords: "",
    fileInfo: "",
    alphabetOptions: alphabetOptions,
    headLetterMust: 'none',
    headLetterNot: 'none',
    tailLetterMust: 'none',
    emptyWordsAlert: false,
    conflictParamsAlert: false,
    exceptionAlert: false,
    calType: "1",
    allowRing: false,
    results: "",
    calTime: 0,
    calNum: 0,
    exceptionMsg: ""
  }),
  mounted() {
    myChart = echarts.init(document.getElementById('graph'));
    let data = [
      {
        name: '单',
        value: '5',
        x: Math.random() * 30 + Math.random() * 5,
        y: Math.random() * 30 + Math.random() * 5,
        symbolSize: 25,
        id: 0,
        itemStyle: {
          color: '#626c91',
        }
      },
      {
        name: '词',
        value: '8',
        x: Math.random() * 30 + Math.random() * 5,
        y: Math.random() * 30 + Math.random() * 5,
        symbolSize: 40,
        id: 1,
        itemStyle: {
          color: '#6be6c1',
        }
      },
      {
        name: '链',
        value: 6,
        x: Math.random() * 30 + Math.random() * 5,
        y: Math.random() * 30 + Math.random() * 5,
        symbolSize: 30,
        id: 2,
        itemStyle: {
          color: '#6be6c1',
        }
      },
      {
        name: '示例',
        value: '7',
        x: Math.random() * 30 + Math.random() * 5,
        y: Math.random() * 30 + Math.random() * 5,
        symbolSize: 35,
        id: 3,
        itemStyle: {
          color: '#3fb1e3',
        }
      }
    ];
    let edges = [{
      source: 0,
      target: 1,
    }, {
      source: 1,
      target: 2,
    }, {
      source: 2,
      target: 3,
    }];
    var option = {
      tooltip: {},
      animationDurationUpdate: 1500,
      animationEasingUpdate: 'quinticInOut',
      series: [{
        type: 'graph',
        layout: 'none',
        roam: true,
        edgeLabel: {
          fontSize: 20
        },
        force: {
          edgeLength: 5,
          repulsion: 20,
          gravity: 0.2
        },
        draggable: true,
        edgeSymbol: ['circle', 'arrow'],
        emphasis: {
          focus: 'adjacency',
          label: {
            show: false
          }
        },
        data: data,
        edges: edges,
        lineStyle: {
          color: '#626c91',
          width: 0.5,
          opacity: 0.7,
          curveness: 0.3
        }
      }]
    };
    myChart.setOption(option);
  },
  methods: {
    uploadFile() {
      const that = this
      const reader = new FileReader()
      that.rawWords = " ";
      reader.readAsText(that.fileInfo)
      reader.onload = function (e) {
        that.rawWords = e.target.result;
      }
    },
    saveFile() {
      let that = this;
      const data = that.results;
      let str = new Blob([data], {type: 'text/plain; charset=utf8'});
      saveAs(str, 'solution.txt');
    },
    genWordsChain() {
      let that = this;
      console.log({
        rawWords: that.rawWords,
        calType: that.calType,
        hM: that.headLetterMust,
        tM: that.tailLetterMust,
        hN: that.headLetterNot,
        aR: that.allowRing
      })
      let validateFlag = 1;
      if (that.rawWords === '') {
        that.emptyWordsAlert = true;
        validateFlag = 0;
      }
      if (that.calType === "1") {
        if (that.allowRing !== false ||
            that.headLetterMust !== 'none' ||
            that.headLetterNot !== 'none' ||
            that.tailLetterMust !== 'none') {
          that.conflictParamsAlert = true;
          validateFlag = 0;
        }
      }

      if (that.headLetterMust === that.headLetterNot &&
          that.headLetterMust !== 'none') {
        that.conflictParamsAlert = true;
        validateFlag = 0;
      }

      if (validateFlag === 1) {
        that.results = "";
        // gen_chains_all
        if (that.calType === "1") {
          let curNum = myDll.gen_chains_all(that.rawWords + '\x1a');
          if (curNum < 0) {
            that.exceptionMsg = curNum === -12 ?
                "您输入的单词文本中存在环" : curNum === -13 ?
                    "计算结果中存在超过20000条单词链" : "计算过程中发生错误";
            that.exceptionAlert = true;
            that.calNum = 0;
          } else {
            that.calTime = parseFloat(myDll.get_execution_time());
            that.results = myDll.getResult();
            if (that.results === "") {
              that.exceptionAlert = true;
              that.calNum = 0;
              that.exceptionMsg = "不存在满足条件的单词链";
            } else {
              that.calNum = curNum;
              console.log(that.calNum, that.calTime, that.results)
              this.drawGraph(1);
            }
          }
        } else if (that.calType === "2") {
          that.results = "";
          // gen_chain_word
          let curNum = myDll.gen_chain_word(that.rawWords + '\x1a',
              that.headLetterMust === 'none' ? 0 : that.headLetterMust.charCodeAt(0),
              that.tailLetterMust === 'none' ? 0 : that.tailLetterMust.charCodeAt(0),
              that.headLetterNot === 'none' ? 0 : that.headLetterNot.charCodeAt(0),
              that.allowRing);
          if (curNum < 0) {
            that.exceptionMsg = curNum === -12 ?
                "您输入的单词文本中存在环" : curNum === -13 ?
                    "计算结果中存在超过20000条单词链" : "计算过程中发生错误";
            that.exceptionAlert = true;
            that.calNum = 0;
          } else {
            that.calTime = parseFloat(myDll.get_execution_time());
            that.results = myDll.getResult();
            if (that.results === "") {
              that.exceptionAlert = true;
              that.calNum = 0;
              that.exceptionMsg = "不存在满足条件的单词链";
            } else {
              that.calNum = 1;
              console.log(that.calNum, that.calTime, that.results)
              this.drawGraph(0);
            }
          }
        } else {
          that.results = "";
          // gen_chain_char
          let curNum = myDll.gen_chain_char(that.rawWords + '\x1a',
              that.headLetterMust === 'none' ? 0 : that.headLetterMust.charCodeAt(0),
              that.tailLetterMust === 'none' ? 0 : that.tailLetterMust.charCodeAt(0),
              that.headLetterNot === 'none' ? 0 : that.headLetterNot.charCodeAt(0),
              that.allowRing);
          if (curNum < 0) {
            that.exceptionMsg = curNum === -12 ?
                "您输入的单词文本中存在环" : curNum === -13 ?
                    "计算结果中存在超过20000条单词链" : "计算过程中发生错误";
            that.exceptionAlert = true;
            that.calNum = 0;
          } else {
            that.calTime = parseFloat(myDll.get_execution_time());
            that.results = myDll.getResult();
            if (that.results === "") {
              that.exceptionAlert = true;
              that.calNum = 0;
              that.exceptionMsg = "不存在满足条件的单词链";
            } else {
              that.calNum = 1;
              console.log(that.calNum, that.calTime, that.results)
              this.drawGraph(0);
            }
          }
        }
      }
    },
    drawGraph: function (opt) {
      var option;
      let that = this;

      console.log("chart get: " + that.results)
      let rawData = that.results;
      let data = [];
      let edges = [];

      if (that.results === "") {
        data = [
          {
            name: '单',
            value: '5',
            x: Math.random() * 30 + Math.random() * 5,
            y: Math.random() * 30 + Math.random() * 5,
            symbolSize: 25,
            id: 0,
            itemStyle: {
              color: '#626c91',
            }
          },
          {
            name: '词',
            value: '8',
            x: Math.random() * 30 + Math.random() * 5,
            y: Math.random() * 30 + Math.random() * 5,
            symbolSize: 40,
            id: 1,
            itemStyle: {
              color: '#6be6c1',
            }
          },
          {
            name: '链',
            value: 6,
            x: Math.random() * 30 + Math.random() * 5,
            y: Math.random() * 30 + Math.random() * 5,
            symbolSize: 30,
            id: 2,
            itemStyle: {
              color: '#6be6c1',
            }
          },
          {
            name: '示例',
            value: '7',
            x: Math.random() * 30 + Math.random() * 5,
            y: Math.random() * 30 + Math.random() * 5,
            symbolSize: 35,
            id: 3,
            itemStyle: {
              color: '#3fb1e3',
            }
          }
        ];
        edges = [{
          source: 0,
          target: 1,
        }, {
          source: 1,
          target: 2,
        }, {
          source: 2,
          target: 3,
        }];
      } else {
        let subGraph = rawData.split('\n');
        let len = subGraph.length;
        subGraph = subGraph.slice(0, len - 1);
        if (opt === 0) {
          for (let i = 0; i < subGraph.length; i++) {
            let curWord = subGraph[i];
            data.push({
              name: curWord,
              value: curWord.length,
              x: Math.random() * 30 + Math.random() * 5,
              y: Math.random() * 30 + Math.random() * 5,
              symbolSize: curWord.length * 5,
              id: i,
              itemStyle: {
                color: (i === subGraph.length - 1) ?
                    '#3fb1e3' : (i === 0) ?
                    '#626c91' :
                    '#6be6c1',
              }
            });
            if (i !== 0) {
              edges.push({
                source: i - 1,
                target: i,
              })
            }
          }
        } else {
          let idx = 0;
          for (let i = 0; i < subGraph.length; i++) {
            let tmpLink = subGraph[i];
            let tmpLen = tmpLink.length;
            if (tmpLink[tmpLen - 1] === ' ') {
              tmpLink = tmpLink.slice(0, tmpLen - 1);
            }
            tmpLink = tmpLink.split(' ');
            for (let j = 0; j < tmpLink.length; j++, idx++) {
              let curWord = tmpLink[j];
              data.push({
                name: curWord,
                value: curWord.length,
                x: Math.random() * 30 + Math.random() * 5,
                y: Math.random() * 30 + Math.random() * 5,
                symbolSize: curWord.length * 5,
                id: idx,
                itemStyle: {
                  color: (j === tmpLink.length - 1) ?
                      '#3fb1e3' : (j === 0) ?
                      '#626c91' :
                      '#6be6c1',
                }
              })
              if (j !== 0) {
                edges.push({
                  source: idx - 1,
                  target: idx,
                })
              }
            }
          }
        }
      }

      console.log(data);
      console.log(edges)

      if (opt === 1) {
        option = {
          tooltip: {},
          animationDurationUpdate: 1500,
          animationEasingUpdate: 'quinticInOut',
          series:  [{
            type: 'graph',
            layout: 'none',
            roam: true,
            edgeLabel: {
              fontSize: 20
            },
            force: {
              edgeLength: 5,
              repulsion: 20,
              gravity: 0.2
            },
            draggable: true,
            edgeSymbol: ['circle', 'arrow'],
            emphasis: {
              focus: 'adjacency',
              label: {
                show: false
              },
              lineStyle: {
                width: 5,
              }
            },
            data: data.map(function (item) {
              return item;
            }),
            edges: edges.map(function (item) {
              return item;
            }),
            lineStyle: {
              color: '#626c91',
              width: 0.5,
              opacity: 0.7,
              curveness: 0.3
            }
          }]
        };
      } else {
        option = {
          tooltip: {},
          animationDurationUpdate: 1500,
          animationEasingUpdate: 'quinticInOut',
          series: [{
            type: 'graph',
            layout: 'none',
            roam: true,
            edgeLabel: {
              fontSize: 20
            },
            force: {
              edgeLength: 5,
              repulsion: 20,
              gravity: 0.2
            },
            draggable: true,
            edgeSymbol: ['circle', 'arrow'],
            emphasis: {
              focus: 'adjacency',
              label: {
                show: false
              },
              lineStyle: {
                width: 5,
              }
            },
            data: data,
            edges: edges,
            lineStyle: {
              color: '#626c91',
              width: 0.5,
              opacity: 0.7,
              curveness: 0.3
            }
          }]
        };
      }
      myChart.clear();
      myChart.setOption(option);
    }
  }
};
</script>

<style lang="css">
html {
  overflow: auto;
}
#main:before {
  content: "";
  background: url('assets/email-pattern.png') center fixed;
  position: absolute;
  top: 0;
  right: 0;
  left: 0;
  bottom: 0;
}
</style>
