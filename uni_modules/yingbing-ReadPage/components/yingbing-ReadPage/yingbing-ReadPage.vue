<template>
	<view class="page">
		<!-- 翻页模式 -->
		<!-- <view class="box">
			<view class="content"
			:id="'content' + dataId"
			v-if="pageType != 'scroll'"
			@touchstart="page.pageTouchstart"
			@touchmove="page.pageTouchmove"
			@touchend="page.pageTouchend"></view>
			<view class="content" style="z-index: -1000;" :id="'computed' + dataId"></view>
		</view> -->
		<!-- 滚动模式 -->
		<!-- <view
		:id="'scroll-box' + dataId"
		class="scroll-box"
		:style="{
		'color': color,
		'padding-left': slide + 'px',
		'padding-right': slide + 'px',
		'border-top': `${topGap}px solid ${bgColor}`,
		'padding-bottom': bottomGap + 'px',
		'background': bgColor}"
		v-if="pageType == 'scroll'"></view> -->
		
		<scroll-page
		ref="scroll"
		class="scroll"
		:topGap="topGap"
		:bottomGap="bottomGap"
		:fontSize="fontSize"
		:lineHeight="lineHeight"
		:pages="pages"
		@scrolltoUpper="scrolltoUpper"
		@scrolltoLower="scrolltoLower"
		@scrollEnd="scrollEnd"
		:style="{
		'color': color,
		'padding-left': slide + 'px',
		'padding-right': slide + 'px',
		'padding-top': `${topGap}px solid ${bgColor}`,
		'padding-bottom': bottomGap + 'px',
		'background': bgColor}">
		</scroll-page>
		
		<computed-page
		ref="computedPage"
		:pageType="pageType"
		:fontSize="fontSize"
		:lineHeight="lineHeight"
		:slide="slide"
		:topGap="topGap"
		:bottomGap="bottomGap"></computed-page>
	</view>
</template>

<script>
	export default {
		props: {
			//传入唯一标识动态命名ID用于获取dom对象（可选）默认已经生成
			dataId: {
				type: String,
				default () {
					let mydate = new Date();
					return 'cms' + mydate.getMinutes() + mydate.getSeconds() + mydate.getMilliseconds() + Math.round(Math.random() * 10000);
				}
			},
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
			}
		},
		data () {
			return {
				contents: [],
				loading: false,//等待内容请求
				upper: false,//文章是否到最前面
				lower: false,//文章是否到最后面
				restart: false,//是否重绘页面
				preLoading: false,//等待预加载请求
				pages: []
			}
		},
		computed: {
			pageProp () {
				return {
					contents: this.contents,
					dataId: this.dataId,
					color: this.color,
					bgColor: this.bgColor,
					slide: this.slide > 0 ? parseInt(this.slide) : 0,
					topGap: this.topGap > 0 ? parseInt(this.topGap) : 0,
					bottomGap: this.bottomGap > 0 ? parseInt(this.bottomGap) : 0,
					fontSize: this.fontSize >= 12 ? parseInt(this.fontSize) : 12,//字体大小最小只能到12px，因为谷歌浏览器最小只支持12px
					pageType: this.pageType || 'none',
					lineHeight: this.lineHeight >= 5 ? parseInt(this.lineHeight) : 5,
					restart: this.restart
				};
			}
		},
		methods: {
			scrolltoUpper (e) {
				// this.pages.unshift({
				// 	type: 'loading',
				// 	text: '正在加载内容',
				// 	chapter: this.pages[0].chapter,
				// 	start: 0,
				// 	end: 0,
				// 	dataId: this.pages[0].chapter * 100000 - 1
				// })
				// setTimeout(() => {
					// this.pages.shift();
					// setTimeout(() => {
						let pages = JSON.parse(JSON.stringify(this.pages));
						for ( let i in pages ) {
							pages[i].chapter = 1
							pages[i].dataId = pages[i].chapter * 100000 + pages[i].start
						}
						this.pages = pages.concat(this.pages);
					// }, 500)
					
				// }, 3000)
			},
			scrolltoLower (e) {
				let pages = JSON.parse(JSON.stringify(this.pages));
			},
			scrollEnd (e) {
				console.log(e);
			},
			//初始化
			init (data) {
				// this.contents = data.contents || this.contents;
				// this.restart = true;
				// this.getCatalog(this.contents[0].content);
				this.$refs.computedPage.computed({
					content: data.contents[1].content,
					chapter: data.contents[1].chapter,
					start: data.start
				}).then((pages) => {
					this.pages = pages;
				})
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
			//抛出阅读页面改变事件
			currentChange (e) {
				this.$emit('currentChange', e.currentInfo);
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
	.scroll {
		width: 100%;
		height: 100%;
		position: absolute;
		left: 0;
		top: 0;
		box-sizing: border-box;
	}
	.scroll-item {
		width: 100%;
		box-sizing: border-box;
		padding: 1px 0;
	}
	.scroll-loading {
		display: flex;
		align-items: center;
		justify-content: center;
		width: 100%;
		height: 100%;
	}
	.scroll-text {
		font-family: "Microsoft YaHei", 微软雅黑;
		white-space: pre-wrap;
	}
</style>