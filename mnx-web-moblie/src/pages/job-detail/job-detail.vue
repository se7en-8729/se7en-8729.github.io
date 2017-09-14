<template>
  <div class="view">
    <x-header class="header" :left-options="{backText: ''}">{{job.title}}</x-header>
  
    <div class="main_view" ref="scroll">
      <div>
        <div class="content_view">
          <div class="title_wrapper">
            <h3 class="title">{{job.title}}</h3>
            <!-- <i class="iconfont icon-collection" :class="{active: isCollect}" @click="collect"></i> -->
          </div>
          <p class="salary">{{job.salary}}</p>
          <div class="info">
            <i class="iconfont icon-location"></i>
            <span>{{job.jobArea}}</span>
            <i class="iconfont icon-university"></i>
            <span>{{job.careerLevel}}</span>
            <i class="iconfont icon-people"></i>
            <span>{{job.headcount}}</span>
          </div>
          <div class="job_end">网申时间：{{job.applyBeginDate | dateformat}}-{{job.applyEndDate | dateformat}}</div>
        </div>
  
        <div class="company_wrapper">
          <div class="company_box" @click="showCompany">
            <div class="img_wrapper vux-1px">
              <img class="img" :src="job.logo">
            </div>
            <div class="company_info">
              <h5 class="company_name">{{job.fullName}}</h5>
              <p class="company_category">{{job.primaryIndustry}}</p>
              <p class="company_info_group">
                <span class="company_category" v-if="job.nature">{{job.nature}}</span>
                <span class="company_size" v-if="job.size">{{job.size}}</span>
              </p>
            </div>
            <i class="iconfont icon-right"></i>
          </div>
        </div>
  
        <div class="job_info">
          <div class="job_box">
            <h5 class="job_title">
              <span class="job_text">职位描述</span>
            </h5>
            <pre class="job_des" v-html="job.description"></pre>
          </div>
        </div>
      </div>
    </div>
  
    <div class="apply_btn" :class="{disabled: isApply}" @click="apply">{{jobBtnStr}}</div>
  </div>
</template>
<script>
import { XHeader, Cell, Group } from 'vux'
export default {
  name: 'JobDetail',
  data() {
    return {
      isCollect: false,
      isApply: false,
      user: {},
      job: {},
      jobBtnStr: '申请该职位'
    }
  },
  components: { XHeader, Cell, Group },
  created() {
    this.$root.eventHub.$emit('hideToolBar')
    // 查询该职位是否收藏
    if (localStorage.getItem('userInfo')) {
      this.user = JSON.parse(localStorage.getItem('userInfo'))
      this.$http.get(`/user/${this.user.id}/favorite/jobposting/${this.$route.params.jobId}`).then((res) => {
        if (res.data) {
          this.isCollect = res.data.success
        }
      })
    }
  },
  mounted() {
    // 跳转的时候带过来的jobid
    this.$http.get(`/jobposting/${this.$route.params.jobId}?uid=${this.user.id ? this.user.id : ''}`).then((res) => {
      if (res.data) {
        this.job = res.data.message
        let isHadOffLine = this.job.hadOffLine
        let isHadDeliver = this.job.hadDeliver
        let endTimeStr = this.job.applyEndDate.replace(/(\d{4})(\d{2})(\d{2})/, '$1,$2,$3')
        let endTime = new Date(endTimeStr)
        let newTime = new Date()
        if (endTime < newTime + 8.64e7 || isHadOffLine) {
          this.isApply = true
          this.jobBtnStr = '已下线'
        } else if (isHadDeliver) {
          this.isApply = true
          this.jobBtnStr = '已投递'
        }
      }
    })
    let url = window.location.href
    let xieIndex = url.lastIndexOf('/') + 1
    let wenIndex = url.lastIndexOf('?')
    let urlJob = url.substring(xieIndex, wenIndex)
    let flag = localStorage.getItem('pandaFLag')
    if (url.indexOf('linkinfo=outlink') > -1) {
      if (!this.user.id) {
        localStorage.setItem('pandaFLag', 'pandaTv')
        this.pandaTvApply()
      } else if (this.user.id.length > 0) {
        if (flag === 'pandaTv') {
          localStorage.setItem('pandaFLag', '')
          this.pandaTvApply(urlJob)
        } else {
          return false
        }
      }
    } else {
      if (flag === 'pandaTv') {
        localStorage.setItem('pandaFLag', '')
        setTimeout(() => {
          this.apply()
        }, 500)
      }
    }
  },
  filters: {
    dateformat: (val) => {
      if (!val) return ''
      return val.replace(/(\d{4})(\d{2})(\d{2})/, '$1.$2.$3')
    }
  },
  methods: {
    // 添加收藏
    collect() {
      if (this.user.id) {
        if (this.isCollect) {
          // 取消收藏
          this.$http.delete(`/user/${this.user.id}/favorite/jobposting/${this.job.id}`).then((res) => {
            if (res.data) {
              this.isCollect = !res.data.success
            }
          })
        } else {
          // 点击添加收藏
          this.$http.post(`/user/${this.user.id}/favorite/jobposting`, {
            stuId: this.user.id,
            objId: this.job.id,
            objType: 1,
            name: this.job.title,
            url: null,
            descr: this.job.fullName + '-' + this.job.title,
            status: 0
          }).then((res) => {
            if (res.data) {
              this.isCollect = res.data.success
              this.jobBtnStr = '已申请该职位'
            }
          })
        }
      } else {
        this.$vux.toast.text('您还未登录')
        setTimeout(() => {
          this.$router.push('/login')
        }, 1200)
      }
    },
    apply() {
      if (this.user.id) {
        if (!this.isApply) {
          this.$http.post(`/user/${this.user.id}/application?jobId=${this.job.id}`).then((res) => {
            if (res.data) {
              this.$router.push({
                path: '/application',
                query: {appId: res.data.message}
              })
              this.isApply = res.data.success
            }
          })
        }
      } else {
        this.$vux.toast.text('您还未登录')
        setTimeout(() => {
          this.$router.push('/login')
        }, 1200)
      }
    },
    pandaTvApply(jobId) {
      var self = this
      console.log('this.job.hadDeliver:' + self.isApply)
      if (this.user.id) {
        if (!this.isApply) {
          this.$http.post(`/user/${this.user.id}/application?jobId=${jobId}`).then((res) => {
            if (res.data) {
              this.$router.push({
                path: '/application',
                query: {appId: res.data.message}
              })
              this.isApply = res.data.success
            }
          })
        }
      } else {
        this.$vux.toast.text('您还未登录')
        this.$router.push('/login')
      }
    },
    // 跳转公司详情
    showCompany() {
      this.$router.push(`/company/${this.job.recId}`)
    }
  }
}
</script>
<style scoped lang="less">
@import '~vux/src/styles/1px.less';
.main_view {
  position: absolute;
  top: 46px;
  bottom: 50px;
  left: 0;
  right: 0;
  overflow: scroll;
  -webkit-overflow-scrolling: touch;
}

.content_view {
  position: relative;
  padding: 0 16px 10px;
  background-color: #fff;
  font-weight: normal;
  border-bottom: 10px solid #eee;
}

.title {
  min-height: 20px;
  line-height: 1.4;
  font-weight: normal;
  font-size: 18px;
  padding:17px 22px 5px 0;
}

.icon-collection {
  position: absolute;
  right: 10px;
  top: 10px;
  padding: 8px;
  font-size: 22px;
}

.salary {
  height: 22px;
  line-height: 22px;
  font-size: 16px;
  color: #fd4545;
}

.info {
  margin-left: -3px;
  height: 36px;
  line-height: 36px;
  font-size: 16px;
  color: #8f8f8f;
}

.info .iconfont {
  font-size: 18px;
}

.job_end {
  padding-bottom: 10px;
  font-size: 14px;
  color: #666;
}

.company_wrapper {
  padding: 16px 16px;
  border-bottom: 10px solid #eee;
  background-color: #fff;
}

.company_box {
  display: flex;
  width: 100%;
  justify-content: space-between;
  align-items: center;
}

.img_wrapper {
  display: flex;
  flex: 0 0 80px;
  width: 80px;
  height: 80px;
  margin-right: 10px;
  box-sizing: border-box;
  justify-content: center;
  align-items: center;
}

.img {
  display: block;
  width: 100%;
  height: auto;
}

.company_info {
  flex-shrink: 1;
  flex-grow: 1;
  overflow: hidden;
  color: #999;
}

.company_name {
  display: block;
  max-width: 100%;
  padding-bottom: 10px;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
  font-weight: normal;
  font-size: 16px;
  color: #464646;
}

.company_info_group{
  span:last-child{
    padding-left:8px;
    margin-left:8px;
    border-left:1px solid #ccc;
  }
}
.company_category,
.company_size {
  padding-bottom: 10px;
  font-size: 14px;
}

.job_info {
  background-color: #fff;
}

.icon-right {
  flex-shrink: 0;
}

.job_box {
  padding: 0 16px;
}

.job_title {
  padding: 18px 0 0;
}

.job_text {
  display: inline-block;
  padding-left: 10px;
  font-size: 18px;
  border-left: 3px #049cff solid;
  font-weight: normal;
}

.job_des {
  line-height: 24px;
  font-size: 14px;
  color: #666;
  white-space: pre-wrap;
}

.apply_btn {
  position: absolute;
  bottom: 0;
  width: 100%;
  line-height: 50px;
  z-index: 11;
  height: 50px;
  color: #fff;
  text-align: center;
  font-size: 18px;
  background-color: rgba(4, 156, 255, 1);
}

.active {
  // animation: scale 0.5s;
  color: #ffd912;
}

@keyframes scale {
  0% {
    transform: scale(1)
  }
  50% {
    transform: scale(1.5)
  }
  100% {
    transform: scale(1)
  }
}

.active::before {
  content: "\E6EA";
}

.scale-enter,
.scale-leave-to {
  transform: scale(1);
}

.apply_btn{
  &.disabled {
    background-color: rgba(211, 211, 211, 1);
  }
}
</style>