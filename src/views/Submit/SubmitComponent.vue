// FIXME: 本页面有严重BUG，需重做
<template>
  <div class="submitinfo-body"
       v-loading="loading"
       element-loading-text="正在提交中">
    <div class="left-box"
         id="left-box">
      <el-scrollbar style="height:100%">
        <div class="title-box">
          <h1 class="title-font">
            {{this.dataProblemDetail.title?this.dataProblemDetail.title:'题目'}}
          </h1>
          时间限制: <el-tag type="info">{{this.dataProblemMain.timeLimit?this.dataProblemMain.timeLimit:'0MS'}}</el-tag>
          内存限制:<el-tag type="info">{{this.dataProblemMain.memoryLimit?this.dataProblemMain.memoryLimit:'0KB'}}</el-tag>
          <br />64位Integer的IO类型:<el-tag type="info">{{this.dataProblemMain.intFormat?this.dataProblemMain.intFormat:'-'}}</el-tag>
        </div>
        <div v-if="this.$store.getters.getIsLogin">
          <el-link v-if="this.isSolved"
                   type="success"
                   icon="el-icon-check">已解决</el-link>
          <el-link v-else
                   type="danger"
                   icon="el-icon-close">未解决</el-link>
          | <el-link v-if="!this.problemIsStar"
                   type="primary"
                   icon="el-icon-star-off">点击收藏</el-link>
          <el-link v-else
                   icon="el-icon-star-on">已收藏</el-link>
          | <el-link type="warning"
                   v-if="this.dataProblemDetail.judgeOption===0"> 本地判题</el-link>
          <el-link type="danger"
                   v-else> 第三方判题</el-link>
          | <el-link type="danger"
                   v-if="this.dataProblemMain.spj===1">特判</el-link>
          <el-link type="success"
                   v-else>结果匹配</el-link>
        </div>
        <el-card id="problem-card"
                 class="problem-detail-card">
          <div slot="header">题目描述</div>
          <div class="detail-card-body"
               v-html="this.dataProblemMain.description?this.dataProblemMain.description:'【没有描述】'"></div>
        </el-card>
        <el-card id="problem-card"
                 class="problem-detail-card">
          <div slot="header">输入</div>
          <div class="detail-card-body"
               v-html="this.dataProblemMain.input?this.dataProblemMain.input:'【没有输入】'"></div>
        </el-card>
        <el-card id="problem-card"
                 class="problem-detail-card">
          <div slot="header">输出</div>
          <div class="detail-card-body"
               v-html="this.dataProblemMain.output?this.dataProblemMain.output:'【没有输出】'"></div>
        </el-card>
        <div v-for="item in this.dataProblemSamples"
             :key="item">
          <el-card id="problem-card"
                   class="problem-detail-card">
            <div slot="header">输入样例 {{item.caseOrder===0?null:item.caseOrder+1}}</div>
            <div class="detail-card-body"
                 v-html="item.inputCase?item.inputCase:'【没有输入样例】'"></div>
          </el-card>
          <el-card id="problem-card"
                   class="problem-detail-card">
            <div slot="header">输出样例 {{item.caseOrder===0?null:item.caseOrder+1}}</div>
            <div class="detail-card-body"
                 v-html="item.outputCase?item.outputCase:'【没有输出样例】'"></div>
          </el-card>
        </div>
        <!-- <el-card id="problem-card"
                 class="problem-detail-card">
          <div slot="header">题目统计信息
            <el-link type="info"
                     class="elcard-showmore-link">查看更多</el-link>
          </div>
          <div class="detail-card-body">
            <div class="echarts-box"
                 id="submit-echarts"></div>
            <div class="echarts-box"
                 id="acuser-echarts"></div>
          </div>
        </el-card> -->
      </el-scrollbar>
    </div>

    <div class="right-box">
      <el-collapse id="submit-collapse"
                   v-model="this.activeIndex">
        <el-collapse-item title="答题"
                          name="1">
          <div class="code-editor-box">
            请选择语言：
            <el-select class="language-select"
                       size="medium"
                       v-model="compileLanguage"
                       @change="this.handleChangeLanguage"
                       :disabled="!this.$store.getters.getIsLogin">
              <el-option v-for="item in languageType"
                         :key="item.value"
                         :label="item.label"
                         :value="item.value"></el-option>
            </el-select>
            <!-- 全屏显示编辑框 -->
            <el-button icon="el-icon-full-screen"
                       type="primary"
                       size="mini"
                       round
                       plain
                       @click="fullEditor = true">全屏</el-button>
            <el-dialog :visible.sync="fullEditor"
                       custom-class="dialog-body"
                       :fullscreen="true">
              <aceEditor class="full-code-editor"
                         :language="this.compileLanguage"
                         :readOnly="!this.$store.getters.getIsLogin"
                         :isFull="true"
                         @input="getCode"
                         :value="this.code"></aceEditor>
            </el-dialog>
            <aceEditor id="ace-editor"
                       class="code-editor"
                       :language="this.compileLanguage"
                       :readOnly="!this.$store.getters.getIsLogin"
                       @input="getCode"
                       :value="this.code"></aceEditor>
            <el-button type="primary"
                       @click="handleSubmit"
                       :disabled="!this.$store.getters.getIsLogin">提交代码</el-button>
          </div>
        </el-collapse-item>
        <el-collapse-item title="题解"
                          name="2">
          <el-card>最佳题解：
            无
          </el-card>
          <el-card>添加题解：
            <el-divider></el-divider>
            <div v-if="this.isSolved">
              <markdownEditor :text="answerText"
                              @inputValue="getText"></markdownEditor>
              {{this.answerText}}
              <el-button type="primary"
                         size="mini">提交</el-button>
            </div>
            <div v-else>解答该题后才能添加题解哦</div>
          </el-card>
        </el-collapse-item>
      </el-collapse>
    </div>
  </div>
</template>

<script>
import aceEditor from '@/components/AceEditor'
import markdownEditor from '@/components/MarkDownEditor'
import echarts from 'echarts'
import echartStyle from '../../utils/echarts/shine.json'
export default {
  components: {
    aceEditor,
    markdownEditor
  },
  props: {
    pid: String
  },
  data () {
    return {
      loading: false,
      myChartSubmit: '',
      myChartsAcUser: '',
      dataProblemMain: '',
      dataProblemDetail: '',
      dataProblemSamples: [],
      isSolved: false,
      problemIsStar: false,
      code: '',
      // 题解内容
      answerText: '# 第 ' + this.pid +
        ' 题题解\n ##\n ##\n ##\n::: hljs-right \n#### Editor by :  ' +
        this.$store.getters.getUsername + '\n:::',
      fullEditor: false,
      compileLanguage: 'G++',
      languageType: [
        {
          value: 'G++',
          label: 'G++'
        },
        {
          value: 'GCC',
          label: 'GCC'
        },
        {
          value: 'JAVA',
          label: 'JAVA'
        },
        {
          value: 'Python',
          label: 'Python2'
        }
      ],
      activeIndex: ['1']
    }
  },
  watch: {
    pid (val) {
      this.getProblem()
    }
  },
  mounted () {
    // this.initEcharts()
    this.getProblem()
    if (!this.$store.getters.getIsLogin) {
      // this.activeIndex = []
      this.$notify({
        title: '提示',
        message: '登录后才能作答哦',
        type: 'warning',
        offset: 100,
        duration: 3000
      })
    } else {
      this.activeIndex.push('1')
    }
  },
  methods: {
    handleFullScreen () {
      this.fullEditor = true
    },
    initEcharts () {
      let obj = echartStyle
      echarts.registerTheme('shine', obj)
      this.myChartSubmit = echarts.init(document.getElementById('submit-echarts'), 'shine')
      this.myChartSubmit.showLoading()
      this.myChartsAcUser = echarts.init(document.getElementById('acuser-echarts'))
      this.myChartsAcUser.showLoading()
    },
    async getProblem () {
      let params = new URLSearchParams()
      let pidTemp = this.pid
      params.append('problemId', pidTemp)
      params.append('username', this.$store.getters.getUsername)
      let dataProblem = await this.$http
        .get('/problem/info/get', params)
      this.dataProblemDetail = dataProblem.datas[0]
      this.dataProblemMain = dataProblem.datas[1]
      this.dataProblemSamples = dataProblem.datas[2]
      this.isSolved = dataProblem.datas[3]
      if (this.dataProblemSamples.length === 0) {
        this.dataProblemSamples[0] = {
          caseOrder: 0,
          inputCase: '',
          outputCase: ''
        }
      }
      // this.loadChartsSubmit()
      // this.loadChartsAcUser()
    },
    loadChartsSubmit () {
      let totalAc = this.dataProblemDetail.totalAc
      let others = this.dataProblemDetail.totalSubmit - this.dataProblemDetail.totalAc
      let option = {
        title: {
          text: '提交AC占比',
          x: 'center'
        },
        tooltip: {
          trigger: 'item',
          formatter: '{a} <br/>{b} : {c} ({d}%)'
        },
        legend: {
          orient: 'vertical',
          left: 'left',
          data: ['总AC数', '其他']
        },
        series: [
          {
            name: '总AC数/总提交数',
            type: 'pie',
            radius: '55%',
            center: ['50%', '60%'],
            data: [
              { value: totalAc, name: '总AC数' },
              { value: others, name: '其他' }
            ],
            itemStyle: {
              emphasis: {
                shadowBlur: 10,
                shadowOffsetX: 0,
                shadowColor: 'rgba(0, 0, 0, 0.5)'
              }
            }
          }
        ]
      }
      this.myChartSubmit.setOption(option)
      this.myChartSubmit.hideLoading()
    },
    loadChartsAcUser () {
      let totalAcUser = this.dataProblemDetail.totalAcUser
      let others = this.dataProblemDetail.totalSubmitUser - this.dataProblemDetail.totalAcUser
      let option = {
        title: {
          text: '提交人数占比',
          x: 'center'
        },
        tooltip: {
          trigger: 'item',
          formatter: '{a} <br/>{b} : {c} ({d}%)'
        },
        legend: {
          orient: 'vertical',
          left: 'left',
          data: ['通过数', '其他']
        },
        series: [
          {
            name: '通过人数/尝试次数',
            type: 'pie',
            radius: '55%',
            center: ['50%', '60%'],
            data: [
              { value: totalAcUser, name: '通过数' },
              { value: others, name: '其他' }
            ],
            itemStyle: {
              emphasis: {
                shadowBlur: 10,
                shadowOffsetX: 0,
                shadowColor: 'rgba(0, 0, 0, 0.5)'
              }
            }
          }
        ]
      }
      this.myChartsAcUser.setOption(option)
      this.myChartsAcUser.hideLoading()
    },
    async onSubmit () {
      let params = new URLSearchParams()
      params.append('pid', this.pid)
      params.append('username', this.$store.getters.getUsername)
      params.append('code', this.code)
      params.append('language', this.compileLanguage)
      // FIXME: 暂时固定时间和内存限制
      params.append('timeLimit', 1000)
      params.append('memoryLimit', 128000)
      // 如果有本地判题标记，优先提交到本地
      let url = '/judge_status/submit/post'
      if (this.dataProblemDetail.judgeOption === 1) {
        url = '/judge_status/submit/post'
      } else {
        // url = '/submit/submitProblem'
      }
      let dataSubmitProblem = await this.$http.post(url, params)
      this.logger.i(dataSubmitProblem.datas)
      if (dataSubmitProblem.code === 10000) {
        this.$message.success('提交成功！')
      } else {
        this.$message.error(dataSubmitProblem.msg)
      }
      this.$router.push({ path: '/Status' })
    },
    getCode (code) {
      this.code = code
    },
    getText (text) {
      this.answerText = text
    },
    handleSubmit () {
      if (this.code.length <= 50) {
        this.$message.warning('提交空白代码是要打屁股的')
      } else {
        this.loading = true
        this.onSubmit()
      }
    },
    handleChangeLanguage (val) {
      this.compileLanguage = val
    }
  }

}
</script>

<style scoped>
.submitinfo-body {
  width: 100%;
  margin: auto;
  margin-top: 0px;
  min-height: 700px;
  font-size: 15px;
}

/* .full-box{
    width: 100%;
} */

.left-box {
  float: left;
  text-align: left;
  width: 50%;
  height: 680px;
  margin-bottom: 30px;
}

.right-box {
  float: right;
  margin-top: 20px;
  width: 50%;
  min-height: 700px;
}

.echarts-box {
  margin: 0;
  padding: 0;
  float: left;
  height: 320px;
  width: 320px;
}

#problem-card {
  margin-bottom: 10px;
  margin-right: 14px;
}

.code-editor-box {
  text-align: center;
  background-color: #eeeeee;
}

.language-select {
  margin-bottom: 10px;
  width: 20%;
  margin-right: 2%;
  z-index: 1;
}

.full-code-editor {
  width: 100%;
  height: 100%;
}

.code-editor {
  width: 100%;
  margin-bottom: 10px;
}

.title-box {
  width: 100%;
  min-height: 120px;
  text-align: center;
}

.title-font {
  font-size: 31px;
}

.el-tag {
  font-size: 14px;
  margin-right: 20px;
  margin-left: 5px;
}

.detail-card-body {
  font-size: 16px;
  margin: 0;
}

.el-scrollbar__wrap {
  overflow-x: hidden;
}
</style>
