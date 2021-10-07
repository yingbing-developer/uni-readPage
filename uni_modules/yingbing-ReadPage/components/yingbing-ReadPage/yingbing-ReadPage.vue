<template>
	<view class="page">
		
		<!-- 翻页模式 -->
		<flip-page
		v-if="pageType != 'scroll'"
		ref="filpPage"
		:page-type="pageType"
		:font-size="prop.fontSize"
		:line-height="prop.lineHeight"
		:color="color"
		:bg-color="bgColor"
		:slide="prop.slide"
		:topGap="prop.topGap"
		:bottomGap="prop.bottomGap"
		:enablePreload="enablePreload"
		@loadmore="loadmore"
		@preload="preload"
		@scrollEnd="scrollEnd">
		</flip-page>
		<!-- 翻页模式 -->
		
		<!-- 滚动模式 -->
		<scroll-page
		v-if="pageType == 'scroll'"
		ref="scrollPage"
		:page-type="pageType"
		:font-size="prop.fontSize"
		:line-height="prop.lineHeight"
		:color="color"
		:bg-color="bgColor"
		:slide="prop.slide"
		:topGap="prop.topGap"
		:bottomGap="prop.bottomGap"
		:enablePreload="enablePreload"
		@loadmore="loadmore"
		@preload="preload"
		@scrollEnd="scrollEnd">
		</scroll-page>
		<!-- 滚动模式 -->
	</view>
</template>

<script>
	export default {
		props: {
			//字体颜色
			color: {
				type: String,
				default: '#333333'
			},
			//字体大小（单位px）
			fontSize: {
				type: String | Number,
				default: 15
			},
			//背景颜色
			bgColor: {
				type: String,
				default: '#fcd281'
			},
			//翻页方式
			pageType: {
				type: String,
				default: 'scroll'
			},
			//行间距（单位px）
			lineHeight: {
				type: Number | String,
				default: 15
			},
			//页面左右边距（单位px）
			slide: {
				type: Number | String,
				default: 40
			},
			//页面上边距（单位px）
			topGap: {
				type: Number | String,
				default: 10
			},
			//页面下边距（单位px）
			bottomGap: {
				type: Number | String,
				default: 10
			},
			//开启预加载
			enablePreload: {
				type: Boolean,
				default: false
			}
		},
		data () {
			return {
				pageInfo: {
					dataId: -1
				}
			}
		},
		computed: {
			prop () {
				return {
					slide: this.slide > 0 ? parseInt(this.slide) : 0,
					topGap: this.topGap > 0 ? parseInt(this.topGap) : 0,
					bottomGap: this.bottomGap > 0 ? parseInt(this.bottomGap) : 0,
					fontSize: this.fontSize >= 12 ? parseInt(this.fontSize) : 12,//字体大小最小只能到12px，因为谷歌浏览器最小只支持12px
					lineHeight: this.lineHeight >= 5 ? parseInt(this.lineHeight) : 5
				}
			}
		},
		methods: {
			loadmore (chapter, callback) {
				this.$emit('loadmore', chapter, callback);
			},
			preload (chapters, callback) {
				this.$emit('preload', chapters, callback);
			},
			scrollEnd (e) {
				if ( e.dataId != this.pageInfo.dataId ) this.$emit('currentChange', e) //抛出阅读页面改变事件
				this.pageInfo = e;
			},
			//初始化
			init (data) {
				if ( this.pageType == 'scroll' ) {
					this.$refs.scrollPage.init(data)
				} else {
					this.$refs.filpPage.init(data)
				}
			},
			//跳转
			change (data) {
				this.contents[0].start = data.position;
				this.restart = true;
			},
			showToast (e) {
				uni.showToast({
					title: e.title,
					icon: 'none'
				})
			},
			//重置部分变量，方便下次使用
			resetPageProp () {
				this.restart = false;
				this.isNewChapter = false;
			},
			//使用正则获取章节目录 并抛出事件
			getCatalog (content) {
				const reg = new RegExp(/(第?[一二两三四五六七八九十○零百千万亿0-9１２３４５６７８９０※✩★☆]{1,6}[章回卷节折篇幕集部]?[、.-\s][^\n]*)[_,-]?/g);
				let match = '';
				let catalog = [];
				while ((match = reg.exec(content)) != null) {
					catalog.push({
						title: match[0],
						position: match.index
					})
				}
				this.$emit('setCatalog', catalog);
			}
		}
	}
</script>

<style scoped>
	.page {
		width: 100vw;
		height: 100vh;
		position: relative;
	}
</style>