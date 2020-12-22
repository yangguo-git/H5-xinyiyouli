<template>
	<div class="main">
		
		<header-vue title='心意有礼周边'></header-vue>
		<div id="app">
			<div id="main1" style="width: 600px;height:400px;"></div>
		</div>
		<div v-if="isLoading" class="loadingWrap">
            <van-loading type="spinner" size="20px" />
		</div>
		<div class="mainContent">
			<!-- 轮播图 -->
			<van-swipe class="my-swipe"  indicator-color="#fc894c" :autoplay="4000">
				<van-swipe-item v-for="(image,index) in imgList" :key="image.categoryId" >
					<a href="#" @click="swiperGo(image.id)"><img :src="swiperArrs[index].url"></a>
				</van-swipe-item>
			</van-swipe>
			<!--推荐商品  -->
			<div class="recommend">
				<p v-if="isShowProvince">推荐商品</p>

				<van-card :key="objs.id" v-for="objs in goodList" :price="(objs.salePrice/100)|postNum" :origin-price="(objs.marketPrice/100)|postNum" :title="objs.productName" :thumb="objs.thumbnailImgUrl" @click="goDetails(objs.id)">
					<template #desc>
						<van-tag plain type="default">{{objs.productSubtitle}}</van-tag>
						<van-tag plain type="default">{{objs.intro}}</van-tag>
					</template>

					<template #tags>
						<div>
							<!-- <van-tag plain type="default" class="yunfei">{{objs.transportPrice == 0?'免运费':objs.transportPrice}}</van-tag> -->
						</div>
					</template>
				</van-card>

			</div>

		</div>
		<footer-vue></footer-vue>
	</div>
</template>

<script>
    import utils from "../../assets/js/common.js"
	import Header from '../../components/header'
	import Footer from '../../components/footer'
	export default {
		name: "home",
		data() {
			return {
				imgList: [],
				goodList:[],
				swiperArrs:[],
				isLoading:true,
				isShowProvince:false
			}
		},
		components: {
			headerVue: Header,
			footerVue: Footer
		},
		methods:{
			goDetails(id){
				this.$router.push({
					name:'productdetail',
					params:{
						id:id
					}
				})
				//存储ID
				sessionStorage.setItem('proId',id);
			},
			swiperGo(id){
				this.goDetails(id);
			},
			drawChart() {
      // 基于准备好的dom，初始化echarts实例
      let myChart = this.$echarts.init(document.getElementById("main"));
      // 指定图表的配置项和数据
      let option = {
        title: {
          text: "ECharts 入门示例"
        },
        tooltip: {},
        legend: {
          data: ["销量"]
        },
        xAxis: {
          data: ["衬衫", "羊毛衫", "雪纺衫", "裤子", "高跟鞋", "袜子"]
        },
        yAxis: {},
        series: [
          {
            name: "销量",
            type: "bar",
            data: this.year[this.nowYear]
          }
        ]
      };
      myChart.setOption(option);
    },
			getUrlParam(name){
				var reg = new RegExp('(^|&)' + name + '=([^&]*)(&|$)')
				let url = window.location.href.split('#')[0]
				let search = url.split('?')[1]
				if (search) {
					var r = search.substr(0).match(reg)
					if (r !== null) return unescape(r[2])
					return null
				} else {
					return null
				}
			}
		},
		filters:{
			postNum(value){
               return utils.hasDot(value);
			}
		},
		created(){//获取推荐商品
			var substrUrl = location.search.substring(13);
			localStorage.setItem("surUrl",substrUrl);//存
			if(substrUrl.indexOf("==")!= -1){
				this.$get('/client/info',{
				   infoInside:substrUrl
				}).then(res=>{
					var res = res.data.data;
					if(res.nickName){
						var postUsername = decodeURIComponent(res.nickName);
						localStorage.setItem("userName",postUsername);
					}
					
				})
			}

			//查询所有订单
			this.$get('/client/product/list',{
               isRecommend:true
			}).then((res)=> {
				if(res.data.status == 0){
				   this.goodList = res.data.data.list;
				   if(this.goodList.length > 0){
					   this.isShowProvince = true;
					   this.isLoading = false;
				   }else{
					   this.isShowProvince = false;
					   this.isLoading = false;
				   }
				   var swiperArr = this.goodList.slice(0,3);
				   this.imgList = swiperArr;
				   var imgArr = [];
				   swiperArr.forEach(function(item){
					   imgArr.push(JSON.parse(item.coverImgUrl)[0]);
				   })
				   this.swiperArrs = imgArr;
				   
				}

			})
		},
		mounted() {
		this.drawChart();
	}
		
	}
</script>

<style scoped>
.mainContent .van-hairline--surround::after {
		border: none;
	}
.recommend>p{
	font-size: 30px;
	margin: 10px 0px;
	padding: 15px 15px 15px 30px;
}
/* 轮播图 */
img {
	width: 100%;
	height: 100%;
}
.mainContent{
	margin-top: 100px;
	margin-bottom: 100px;
}
.recommend{
	margin: 15px;
	border-radius: 10px;
	background-color: white;
}
.main{
	background-color: #f6f6f6;
}
.yunfei{
	color: #fb6003;
}
.my-swipe{
	margin-left: 15px;
	margin-right: 15px;
}
.van-card__title {
	font-size: 13px;
}
.van-tag{
	font-size: 11px;
}
.loadingWrap{
	text-align: center;
}
</style>
