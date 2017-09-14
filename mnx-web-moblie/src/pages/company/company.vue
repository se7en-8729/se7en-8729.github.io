<template>
  <div class="view companys">
    <Mheader ref="header" :tipText="searchContent"></Mheader>
    <flexbox :style="{marginTop: filterTop}" :gutter="0" class="filter_wrapper vux-1px-b">
      <flexbox-item>
        <div class="flex-demo vux-1px-r" @click="showOption(0)">
          <span>地区</span>
          <i class="iconfont icon-down"></i>
        </div>
      </flexbox-item>
      <flexbox-item>
        <div class="flex-demo vux-1px-r" @click="showOption(1)">
          <span>行业</span>
          <i class="iconfont icon-down"></i>
        </div>
      </flexbox-item>
      <flexbox-item>
        <div class="flex-demo vux-1px-r" @click="showOption(2)">
          <span>性质</span>
          <i class="iconfont icon-down"></i>
        </div>
      </flexbox-item>
    </flexbox>
  
    <transition name="salary-show">
      <div class="option" v-show="option[0]">
        <div class="option_box" ref="optionBox">
          <group v-for="(item, index) in addressRadio" key="index" :title="item.name" :id="item.name">
            <radio :options="item.list" @on-change="addressChange"></radio>
          </group>
        </div>
        <div class="letter_list">
          <ul>
            <li v-for="(item, index) in addressRadio" key="index" class="letter_item vux-1px-l" @click="selLetter(item.name)">{{item.name}}</li>
          </ul>
        </div>
      </div>
    </transition>
  
    <transition name="trade-show">
      <div class="option option_box" v-show="option[1]">
        <group>
          <radio :options="tradeRadio" @on-change="tradeChange" value="不限"></radio>
        </group>
      </div>
    </transition>
  
    <transition name="nature-show">
      <div class="option" v-show="option[2]">
        <group>
          <radio :options="natureRadio" @on-change="natureChange" value="不限"></radio>
        </group>
      </div>
    </transition>
  
    <transition name="fade">
      <div class="option_cover" v-show="option[0] || option[1] || option[2]" @click="cancelOption"></div>
    </transition>
  
    <div class="companys_list_view" ref="CompanyListView" @scroll="loadMore">
      <CompanyList :data="CompanyList"></CompanyList>
      <load-more :tip="loadTip" :show-loading="showLoading"></load-more>
    </div>
  </div>
</template>
<script>
import { Flexbox, FlexboxItem, LoadMore, Radio, Group } from 'vux'
import Mheader from '@/components/m-header/m-header'
import CompanyList from '@/components/company-list/company-list'
import { cityByLetter } from '@/assets/cityByLetter.js'

export default {
  name: 'Companys',
  data() {
    return {
      isSearch: false,
      searchContent: '搜索公司',
      // 高度
      viewHeight: '',
      canLoad: true,
      filterTop: '46px',
      CompanyList: [],
      loadTip: '正在加载',
      showLoading: true,
      option: [false, false, false],
      addressRadio: cityByLetter,
      tradeRadio: [{ value: '不限', key: '' }, { value: '互联网·游戏·软件', key: '互联网·游戏·软件' }, { value: '金融', key: '金融' }, { value: '电子·通信·硬件', key: '电子·通信·硬件' }, { value: '服务·咨询·外包', key: '服务·咨询·外包' }, { value: '生产·机械·制造', key: '生产·机械·制造' }, { value: '房地产·建筑·物业', key: '房地产·建筑·物业' }, { value: '制药·医疗', key: '制药·医疗' }, { value: '贸易·消费', key: '贸易·消费' }, { value: '能源·原材料', key: '能源·原材料' }, { value: '教育·培训', key: '教育·培训' }, { value: '政府·非赢利机构·其他', key: '政府·非赢利机构·其他' }, { value: '物流·运输', key: '物流·运输' }, { value: '广告·媒体', key: '广告·媒体' }, { value: '其他', key: '其他' }],
      natureRadio: [{ value: '不限', key: '' }, { value: '私营/民营企业', key: '私营/民营企业' }, { value: '中外合资企业', key: '中外合资企业' }, { value: '外商独资企业', key: '外商独资企业' }, { value: '上市公司', key: '上市公司' }, { value: '国有企业', key: '国有企业' }, { value: '政府/事业单位', key: '政府/事业单位' }, { value: '非营利性组织', key: '非营利性组织' }],
      // 职位列表条件参数
      cityName: '',
      trade: '',
      nature: '',
      search: '',
      pageSize: 10,
      pn: 1
    }
  },
  components: { Mheader, Flexbox, FlexboxItem, CompanyList, LoadMore, Radio, Group },
  created() {
    // 在vue根上生命全局变量为了第一次进入判断是否是点击搜索进来
    this.$root.eventHub.$emit('changeTabBar', 2)
    if (this.$root.searchCompany) {
      this.search = this.$root.searchCompany
      this.searchContent = this.$root.searchCompany
      this.getData()
    } else {
      this.getData()
    }
    // 接受search事件  搜索职位
    this.$root.eventHub.$on('searchCompany', (data) => {
      if (data) {
        this.searchContent = data
      } else {
        this.searchContent = '搜索公司'
      }
      this.search = data
      this.pn = 1
      this.getData()
    })
  },
  mounted() {
    this.$nextTick(() => {
      this.viewHeight = this.$refs.CompanyListView.clientHeight
    })
  },
  methods: {
    getData() {
      this.$http.get(`/recruiter?ad=${this.cityName}&tr=${this.trade}&na=${this.nature}&kw=${this.search}&pn=${this.pn}&size=${this.pageSize}`)
        .then((res) => {
          if (res.data.data.length) {
            this.CompanyList = res.data.data
            if (res.data.totalPage === 1) {
              this.loadTip = '暂无其他公司'
              this.showLoading = false
            }
          } else {
            this.CompanyList = []
            this.loadTip = '未找到符合公司'
            this.showLoading = false
          }
        })
    },
    resetData() {
      this.option = [false, false, false]
      this.$refs.CompanyListView.scrollTop = 0
      this.pn = 1
      this.canLoad = true
      this.showLoading = true
      this.CompanyList = []
      this.loadTip = '正在加载'
      this.getData()
    },
    loadMore() {
      // 上拉刷新
      if (this.$refs.CompanyListView.scrollTop < -10) {
      }
      // 下拉加载更多
      if (this.$refs.CompanyListView.scrollTop === this.$refs.CompanyListView.scrollHeight - this.viewHeight && this.canLoad) {
        this.canLoad = false
        this.pn++
        this.$http.get(`/recruiter?ad=${this.cityName}&tr=${this.trade}&na=${this.nature}&kw=${this.search}&page=${this.pn}&size=${this.pageSize}`).then((res) => {
          if (res.data.data.length) {
            res.data.data.forEach((item) => {
              this.$set(this.CompanyList, this.CompanyList.length, item)
              this.canLoad = true
            })
          } else {
            this.canLoad = false
            this.loadTip = '暂无更多职位'
            this.showLoading = false
          }
        }).catch(() => {
          this.loadTip = '加载失败'
          this.showLoading = false
        })
      }
    },
    // 选择城市首字母
    selLetter(val) {
      this.$refs.optionBox.scrollTop = document.getElementById(val).offsetTop
    },
    showOption(val) {
      if (this.option[val]) {
        this.option = [false, false, false]
      } else {
        this.option = [false, false, false]
        this.option[val] = true
      }
    },
    cancelOption() {
      this.option = [false, false, false]
    },
    addressChange(val) {
      if (val === '不限') {
        this.cityName = ''
      } else {
        this.cityName = val
      }
      this.resetData()
    },
    tradeChange(val) {
      this.trade = val
      this.resetData()
    },
    natureChange(val) {
      this.nature = val
      this.resetData()
    }
  }
}
</script>
<style scoped lang="less">
@import '~vux/src/styles/1px.less';
.filter_wrapper {
  position: relative;
  left: 0;
  top: 0;
  z-index: 9;
  height: 40px;
  background-color: #fff;
}

.flex-demo {
  line-height: 40px;
  text-align: center;
  font-size: 14px;
  color: #666;
}

.flex-demo .iconfont {
  font-size: 12px;
}

.companys_list_view {
  position: absolute;
  top: 85px;
  bottom: 50px;
  z-index: 6;
  left: 0;
  right: 0;
  overflow: scroll;
  -webkit-overflow-scrolling: touch;
}

.option {
  position: absolute;
  top: 86px;
  left: 0;
  z-index: 8;
  width: 100%;
  transition: all .25s ease-in-out;
}

.letter_list {
  position: absolute;
  right: 0;
  top: 0;
  width: 44px;
  height: 300px;
  text-align: center;
  background-color: #fafafa;
  overflow: scroll;
  -webkit-overflow-scrolling: touch;
}

.letter_item {
  padding: 10px 0;
}

.option_cover {
  position: absolute;
  top: 0;
  left: 0;
  bottom: 0;
  right: 0;
  z-index: 7;
  background-color: rgba(0, 0, 0, 0.5);
  transition: 0.3s;
}

.option_box {
  height: 300px;
  overflow: scroll;
  -webkit-overflow-scrolling: touch;
  background-color: #fff;
}

.salary-show-enter,
.salary-show-leave-to {
  transform: translateY(-400px);
}

.degree-show-enter,
.degree-show-leave-to {
  transform: translateY(-200px);
}

.order-show-enter,
.order-show-leave-to {
  transform: translateY(-120px);
}

.fade-enter,
.fade-leave-to {
  opacity: 0;
}
</style>
