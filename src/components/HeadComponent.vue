<template>

  <div class="head-box">
    <el-menu class="el-menu"
             mode="horizontal"
             background-color="#f5f5f5"
             active-text-color="#409EFF"
             text-color="#333333"
             :default-active="activeIndex"
             @select="handleMenuSelect"
             router>
      <el-menu-item index="Index"><i class="el-icon-s-home"></i>主 页</el-menu-item>
      <el-submenu index="ProblemSub">
        <template slot="title"><i class="el-icon-menu"></i>题 库</template>
        <el-menu-item index="LocalProblem">本地题库</el-menu-item>
        <el-menu-item index="VJudgeProblem">Virtual Judge题库</el-menu-item>
      </el-submenu>
      <el-menu-item index="Status"><i class="el-icon-s-order"></i>评 测</el-menu-item>
      <el-submenu index="ContestSub">
        <template slot="title"><i class="el-icon-s-platform"></i>比 赛</template>
        <el-menu-item index="Contest?kind=0">练习</el-menu-item>
        <el-menu-item index="Contest?kind=1">积分</el-menu-item>
        <el-menu-item index="Contest?kind=2">趣味</el-menu-item>
        <el-menu-item index="Contest?kind=3">正式</el-menu-item>
        <el-menu-item index="Contest?kind=5">自定义</el-menu-item>
      </el-submenu>
      <!-- <el-submenu index="">
        <template slot="title"><i class="el-icon-menu"></i>考 试</template>
        <el-menu-item index="1">编程题考试</el-menu-item>
      </el-submenu> -->
      <el-submenu index="RankSub">
        <template slot="title"><i class="el-icon-s-data"></i>排 名</template>
        <el-menu-item index="MainRank">积分榜</el-menu-item>
        <el-menu-item index="HonorRank">荣誉榜</el-menu-item>
        <el-menu-item index="9-2">活跃榜</el-menu-item>
        <el-menu-item index="9-3">现役榜</el-menu-item>
        <el-menu-item index="9-4">组队榜</el-menu-item>
      </el-submenu>
      <el-menu-item index="Challenge" v-show="this.isLogin"><i class="el-icon-star-on"></i>挑战模式</el-menu-item>
      <el-menu-item index="Discuss"><i class="el-icon-s-comment"></i>讨 论</el-menu-item>
      <!-- <el-menu-item index="VideoLesson">视频课堂</el-menu-item> -->
      <el-menu-item index="Mall"><i class="el-icon-s-shop"></i>商 城</el-menu-item>
      <el-menu-item v-if="this.isAdmin"
                    index="Admin">管理员</el-menu-item>
      <!-- 方便调试暂时设置，正式部署时需要去除 -->
      <!-- <el-menu-item index="Test">测试</el-menu-item> -->
      <el-menu-item class="el-menu-item-right"
                    v-if="!this.isLogin"
                    index="Login">登录</el-menu-item>
      <el-menu-item class="el-menu-item-right"
                    v-if="!this.isLogin"
                    index="Register">注册</el-menu-item>
      <div class="menu-rightside">
        <el-button class="clockin-button"
                   type="primary"
                   v-if="this.isLogin && !this.isClockIn"
                   size="medium"
                   round
                   @click="clockin">点我签到</el-button>
        <el-button class="clockin-button"
                   type="info"
                   v-if="this.isLogin && this.isClockIn"
                   size="medium"
                   round
                   @click="toClockIn">你已签到</el-button>
        <el-dropdown class="el-menu-item-userinfo"
                     v-if="this.isLogin"
                     @command="handleCommand">
          <router-link to="User"
                       class="router-link">
            <el-avatar :size="25"
                       :src="$store.getters.getAvatarUrl"></el-avatar>
            <el-badge is-dot
                      class="badge-dot"
                      v-if="unReadMsgCount > 0">
              <i>{{$store.getters.getUsername}}</i>
              <i class="el-icon-arrow-down"></i>
            </el-badge>
            <i v-else>
              {{$store.getters.getUsername}}
              <i class="el-icon-arrow-down"></i>
            </i>
          </router-link>
          <el-dropdown-menu slot="dropdown">
            <el-dropdown-item command="toMessage">
              <el-badge :value="unReadMsgCount"
                        :max="99"
                        class="mark"
                        v-if="this.unReadMsgCount > 0">
                <span>
                  <i class="el-icon-message">消息</i>
                </span>
              </el-badge>
              <span v-else>
                <i class="el-icon-message">消息</i>
              </span>
            </el-dropdown-item>
            <el-dropdown-item command="toEditUser"
                              divided>
              <i class="el-icon-edit">编辑</i>
            </el-dropdown-item>
            <el-dropdown-item command="toVerify"
                              divided>
              <i class="el-icon-document-checked">认证</i>
            </el-dropdown-item>
            <el-dropdown-item command="logout"
                              divided>
              <i class="el-icon-switch-button">退出</i>
            </el-dropdown-item>
          </el-dropdown-menu>
        </el-dropdown>
      </div>
    </el-menu>
  </div>
</template>

<script>
export default {
  name: 'Head',
  data () {
    return {
      picSrc: require('../assets/image/logo.png'),
      type: false,
      datas: [],
      avatarUrl: '',
      activeIndex: ''
    }
  },
  created () { },
  mounted () {
    if (this.isLogin) {
      this.checkUnReadMsgCount()
    }
    this.activeIndex = this.$store.getters.getActiveIndex
  },
  computed: {
    isLogin () {
      return this.$store.getters.getIsLogin
    },
    isAdmin () {
      return this.$store.getters.getIsAdmin
    },
    isClockIn () {
      return this.$store.getters.getIsClockIn
    },
    unReadMsgCount () {
      return this.$store.getters.getUnReadMsgCount
    }
  },
  methods: {
    handleMenuSelect (key) {
      this.activeIndex = key
      this.$store.commit('setActiveIndex', key)
    },
    // add by axiang [20190628] 统一处理用户名下拉框的下拉内容
    handleCommand (command) {
      if (command === 'toEditUser') {
        this.toEditUser()
      } else if (command === 'toMessage') {
        this.toMessage()
      } else if (command === 'toVerify') {
        this.toVerify()
      } else if (command === 'logout') {
        this.logout()
      }
    },
    logout () {
      let username = this.$store.getters.getUsername
      let params = new URLSearchParams()
      params.append('username', username)
      // 请求后端登出
      this.$http.post('/user/logout', params)
      this.$store.commit('LOGOUT')
      this.toIndex()
      // this.$message.warning('您已退出登录！')
      this.$notify.warning({
        title: '您已退出登录',
        duration: 2000,
        type: 'success',
        position: 'top-left',
        dangerouslyUseHTMLString: true,
        offset: 40,
        showClose: false,
        message: '欢迎继续回来刷题哦'
      })
    },
    honorRank () {
      this.$router.push({ path: 'HonorRank' })
    },
    toIndex () {
      this.$router.push({ path: '/' })
    },
    toEditUser () {
      this.$router.push({ path: 'EditUser' })
    },
    toClockIn () {
      this.$store.commit('setIndex', '')
      this.$router.push({ path: 'CheckIn' })
    },
    toMessage () {
      this.$router.push({ path: 'Message' })
    },
    toVerify () {
      this.$router.push({ path: 'Verify' })
    },
    // add by axiang [20190613] 签到
    async clockin () {
      let username = this.$store.getters.getUsername
      let params = new URLSearchParams()
      params.append('username', username)
      let dataSetClockIn = await this.$http
        .post('/check_in/check', params)
        .catch(() => { })
      if (dataSetClockIn.code !== 10000) {
        this.$message({
          message: '签到失败:' + dataSetClockIn.msg,
          type: 'error'
        })
        this.$store.commit('setIsClockIn', false)
      } else {
        this.$message({ message: '签到成功！', type: 'success' })
        this.$store.commit('setIsClockIn', true)
        this.toClockIn()
      }
    },
    async checkUnReadMsgCount () {
      this.logger.ms('UnReadMsgCount', '未读消息数量')
      let username = this.$store.getters.getUsername
      let params = new URLSearchParams()
      params.append('username', username)
      let dataUnReadMsgCount = await this.$http.get('/user_message/unread/count/get', params)
      let unReadMsgCount = dataUnReadMsgCount.datas[0]
      this.$store.commit('setUnReadMsgCount', unReadMsgCount)
      this.logger.me('UnReadMsgCount', '未读消息数量')
    }
  }
}
</script>

<style scoped >
.head-box {
  width: 90%;
  margin: auto;
}

.logo-menu-item {
  padding: 0;
  margin: 0;
  width: 100px;
}

.logo-pic {
  width: 100%;
  height: 100%;
}

.el-menu {
  width: 100%;
  border-left: #eeeeee 1px solid;
  border-right: #eeeeee 1px solid;
}

.el-menu-item-right {
  float: right !important;
  border-right: 0;
  border-left: #eeeeee 1px solid;
}

.router-link {
  text-decoration: none;
  color: black;
}

.el-menu-item-userinfo {
  line-height: 4;
  min-width: 120px;
  width: auto;
  border-left: 1px solid #eeeeee;
}

.menu-rightside {
  float: right;
}

.clockin-button {
  margin-right: 20px;
}

.badge-dot {
  line-height: 0px;
  margin-right: 10px;
}
</style>
