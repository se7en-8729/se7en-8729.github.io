<template>
  <div>
    <div class="job_wrapper vux-1px-b" v-for="(item, index) in listData" key="index" @click="goCompanyDetail(item.id)">
      <div class="job_box">
        <div class="logo vux-1px">
          <img class="logo_img" :src="item.logo"  v-if="item.logo">
          <img class="logo_img" src="~assets/image/defaultCorp.jpg"  v-else>
        </div>
        <div class="info">
          <h5 class="info_company">{{item.fullName}}</h5>
          <p class="info_jobs">在招职位：<span>{{item.pCount}}个</span></p>
          <div class="sub_info">
            <i class="iconfont icon-location" v-if="item.city"></i>
            <span v-if="item.city">{{item.city}}</span>
            <i class="iconfont icon-people" v-if="item.size"></i>
            <span v-if="item.size">规模：{{item.size}}</span>
          </div>
          <div class="send_com_group"><span class="send_industry" v-if="item.primaryIndustry">{{item.primaryIndustry}}</span><span class="send_nature" v-if="item.nature">{{item.nature}}</span></div>
        </div>
      </div>
    </div>
  </div>
</template>
<script>
export default {
  props: {
    data: {}
  },
  computed: {
    listData() {
      return this.data
    }
  },
  methods: {
    goCompanyDetail(id) {
      this.$router.push('/company/' + id)
    }
  }
}
</script>
<style scoped lang="less">
@import '~vux/src/styles/1px.less';
.job_wrapper {
  padding: 10px 10px;
}

.job_box {
  display: flex;
  width: 100%;
  padding: 0;
  justify-content: space-between;
  align-items: center;
}

.logo {
  display: flex;
  flex: 0 0 80px;
  width: 80px;
  height: 80px;
  margin-right: 10px;
  box-sizing: border-box;
  justify-content: center;
  align-items: center;
}

.logo_img {
  display: block;
  width: 100%;
  height: auto;
}

.info {
  flex-shrink: 1;
  flex-grow: 1;
  flex-basis: auto;
  font-size: 14px;
  overflow: hidden;
  color: #999;
}

.info_company {
  width: 100%;
  font-size: 16px;
  font-weight: normal;
  color: #464646;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
}

.info_jobs {
  max-width: 100%;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
  color: #666;
  span{
    color:#049cff;
  }
}

.sub_info {
  vertical-align: middle;
  margin-left: -3px;
}
.send_com_group{
  max-width: 100%;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
  span:last-child{
    border-left:1px solid #ccc;
    padding-left:8px;
    margin-left:8px;
  }
}
.send_industry,.send_nature {
  max-width: 100%;
  overflow: hidden;
  text-overflow: ellipsis;
  white-space: nowrap;
  color: #999;
}

.jobCounts {
  flex-shrink: 0;
  color: #f23023;
}
</style>

