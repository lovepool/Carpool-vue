<template>
  <div class="page-view cp-page-history-select">
    <title-bar  :left-options="{showBack: true}" >
      <span >{{$t("message['carpool.addtrip.commonroute']")}}</span>
    </title-bar>
    <div class="page-view-main"   >
      <cp-scroller :position="{top:'46px'}" :on-refresh="onRefresh"  :dataList="scrollData" :enableInfinite="false">
        <ul>
          <li v-for="(item,index) in listDatas" class="cp-item btn-ripple" @click="onSelect(index)">
            <i class="cp-icon fa fa-history"></i>
            <span class="cp-time">{{item.time.split(' ')[1]}}</span>
            <span class="cp-point">{{item.start_info.addressname}}</span>
            <span class="cp-arrow"><i class="fa fa-long-arrow-right"></i></span>
            <span class="cp-point">{{item.end_info.addressname}}</span>
            <!-- <cp-trip-box :start_name="item.start_info.addressname" :end_name="item.end_info.addressname">
            </cp-trip-box> -->
          </li>
        </ul>
       <span slot="loading-text"><spinner type="dots" size="60px"></spinner></span>
       <div class="text-center">
         <div class="cp-nodata-tips" v-show="noData">
           {{$t("message.noMore")}} ⁽⁽ƪ(ᵕ᷄≀ ̠˘᷅ )ʃ⁾⁾
         </div>
         <spinner type="dots" size="60px" v-show="page==1 && isLoading"></spinner>
       </div>
      </cp-scroller>
    </div>
  </div>
</template>

<script>
import config from '../config'
import cFuns from '@/utils/cFuns'

import CpTripCard from '../components/CpTripCard'
import CpTripBox from '../components/CpTripBox'
export default {
  components: {
    CpTripCard,CpTripBox
  },
  data () {
    return {
      page    : 1,
      pageCount:1,
      isLoading : 0,
      listDatas :{},
      noData:0,
      scrollData: {
          noFlag: false //暂无更多数据显示
      },

    }
  },
  methods :{
    init (){


    },
    /**
     * [goHome 返回首页]
     */
    goHome (){
      this.$router.push({name:'carpool'})
    },

    /**
     * 取得列表
     * @param  {function} success 取得列表成功后执行。
     */
    getList (success){
      // console.log(config.urls.checkLogin)
      // alert(1)
      let params = {limit:20};

      this.isLoading = 1;
      this.noData = 0;
      this.$http.get(config.urls.getMytrip,{params:params}).then(res => {

        let data = res.data.data;
          this.isLoading = 0;
          if(res.data.code === 0) {
              this.listDatas = data.lists;
          }else{

          }
          if(typeof(success)==="function"){
            success(res);
          }
        })
        .catch(error => {
          this.isLoading = 0;
          console.log(error)
        })
    },
    /**
     * 下接刷新
     */
    onRefresh(done) {
      this.getList();
      done(); // call done
    },

    onSelect (index){
      let data =  this.listDatas[index]

      let formData = this.$store.state.tripFormData;
      let today = cFuns.formatDayItemData(new Date).value;
      let hm = data.time.split(' ')[1]
      formData.start = data.start_info;
      formData.end = data.end_info;

      formData.seat_count = parseInt(data.seat_count);
      formData.time = [  today ,hm.split(':')[0] ,hm.split(':')[1] ];
        // console.log(formData);
        this.$store.commit('setTripFormData',formData);
        this.$router.back();

    }

  },
  mounted () {

  },
  created () {
    this.init();
    this.getList();
    // this.$nextTick(function () {
    //  this.$refs['j-herblist-scrollBox'].addEventListener('scroll', this.listScroll); //监听滚动加载更多
    // })
  },
  activated (){
    this.$el.querySelector('.load-more').style.display = 'none';
  }
}

</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>

</style>
