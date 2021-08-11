<template>
	<view class="page" :style="{'background-color': bgColor}">
		<view class="box">
			<view class="content" :id="'page' + dataId" :prop="pageProp" :change:prop="page.pagePropChange"></view>
		</view>
		<scroll-view :scroll-top="scrollTop" @scrolltoupper="scrolltoupper" @scrolltolower="scrolltolower" :upper-threshold="upperThreshold" :lower-threshold="lowerThreshold" class="scroll" scroll-y :show-scrollbar="false" v-if="pageType == 'scroll'">
			<view id="scroll">
				<view class="scroll-item" v-for="(item, index) in pages" :key="index">
					<view class="scroll-text" :style="{color: color, 'font-size': fontsize + 'px', 'margin-top': lineHeight + 'px'}" v-for="(text, i) in item.text" :key="i" v-html="text || ' '"></view>
				</view>
			</view>
		</scroll-view>
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
			//传入文本
			content: {
				type: String,
				default: ''
			},
			//章节定位
			chapter: {
				type: String | Number,
				default: 0
			},
			//阅读位置定位
			start: {
				type: String | Number,
				default: 0
			},
			//字体颜色
			color: {
				type: String,
				default: '#333333'
			},
			//字体大小（单位px）
			fontsize: {
				type: String | Number,
				default: 20
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
				default: 5
			},
			//距顶部多远时（单位px），触发 scrolltoupper  事件
			upperThreshold: {
				type: Number | String,
				default: 100
			},
			//距底部多远时（单位px），触发 scrolltolower 事件
			lowerThreshold: {
				type: Number | String,
				default: 100
			}
		},
		data () {
			return {
				pages: [],
				scrollTop: 0,
				prevChapter: false,
				nextChapter: false,
				upper: false,//文章是否到第一章节
				lower: false//文章是否到最后章节
			}
		},
		computed: {
			pageProp () {
				return {
					content: this.content,
					start: this.start,
					dataId: this.dataId,
					color: this.color,
					bgColor: this.bgColor,
					fontsize: parseInt(this.fontsize),
					pageType: this.pageType,
					lineHeight: parseInt(this.lineHeight)
				};
			}
		},
		methods: {
			finish (e) {
				console.log(e);
				// #ifdef H5
				let pages = e;
				// #endif
				// #ifndef H5
				let pages = e.pages;
				// #endif
				if ( this.prevChapter ) {
					this.pages = pages.concat(this.pages);
					this.prevChapter = false;
				} else {
					this.pages = this.pages.concat(pages);
					this.nextChapter = false;
				}
			},
			async scrolltoupper () {
				if ( this.upper ) {
					uni.showToast({
						title: '前面已经没有了',
						icon: 'none'
					})
					return;
				}
				let scrollHeight = await this.getScrollHeight();
				this.prevChapter = true;
			},
			scrolltolower (e) {
				if ( this.lower ) {
					uni.showToast({
						title: '后面已经没有了',
						icon: 'none'
					})
					return;
				}
				this.nextChapter = true;
			},
			getScrollHeight () {
				return new Promise((resolve) => {
					const query = uni.createSelectorQuery().in(this);
					query.select('#scroll').boundingClientRect(data => {
					  resolve(data.height)
					}).exec();
				})
			}
		},
		watch: {
			
		}
	}
</script>

<script lang="renderjs" module="page" type="module">
	let myPageDom;
	export default {
		data () {
			return {
				//根据容器高度，行高计算出的最大高度
				viewHeight: 0,
				viewWidth: 0,
				contentSync: ''
			}
		},
		mounted () {
			this.initDom.bind(this);
			setTimeout(() => {
				if ( this.pageProp.content ) {
					this.contentSync = this.pageProp.content;
					this.computedText();
				}
			}, 30)
		},
		methods: {
			initDom () {
				myPageDom = page.init(document.getElementById('page' + this.pageProp.dataId));
				// 观测更新的数据在 view 层可以直接访问到
				myPageDom.setOption(this.pageProp);
			},
			//计算页面显示文字
			computedText () {
				let parent = document.getElementById('page' + this.pageProp.dataId);
				this.viewWidth = parent.offsetWidth;
				this.viewHeight = parent.offsetHeight;
				let myCanvas = this.createView(0);
				let context = myCanvas.getContext('2d');
				context.font = this.pageProp.fontsize + 'px 微软雅黑';
				context.fillStyle = this.pageProp.color;
				context.lineWidth = 1;
				let lastIndex = 0;
				let pages = [];
				const dowhile = () => {
					let pageHeight = this.pageProp.fontsize + this.pageProp.lineHeight;
					let strs = [];
					pages.push({
						start: lastIndex,
						end: 0,
						text: []
					})
					while ( pageHeight <= this.viewHeight ) {
						strs.push('');
						let content = this.contentSync;
						let lineWidth = 0;
						for ( let i = lastIndex; i < content.length; i++ ) {
							lineWidth += context.measureText(content[i]).width;
							if ( JSON.stringify(content[i]) == JSON.stringify('\r') || JSON.stringify(content[i]) == JSON.stringify('\n') ) {
								lastIndex = i + 1;
								break;
							} else if ( lineWidth >= myCanvas.width ) {
								lastIndex = i;
								break;
							} else {
								strs[strs.length - 1] += content[i];
								lastIndex = i;
							}
						}
						pageHeight += this.pageProp.fontsize + this.pageProp.lineHeight;
						if ( lastIndex == content.length - 1 ) break;
					}
					pages[pages.length - 1].end = lastIndex;
					pages[pages.length - 1].text = strs;
					if ( lastIndex < this.contentSync.length - 1 && pages.length < 2 ) {
						dowhile();
					} else {
						this.drawText(pages);
					}
				}
				dowhile();
			},
			drawText (pages) {
				if ( this.pageProp.pageType != 'scroll' ) {
					for ( let i = 0; i < pages.length; i++ ) {
						let myCanvas = this.createView(i);
						let context = myCanvas.getContext('2d');
						for ( let j = 0; j < pages[i].text.length; j++ ) {
							context.font = this.pageProp.fontsize + 'px 微软雅黑';
							context.fillStyle = this.pageProp.color;
							context.fillText(pages[i].text[j], 0, (j + 1) * (this.pageProp.fontsize + this.pageProp.lineHeight));
						}
					}
				}
				this.finish(pages);
			},
			createView (value) {
				if ( document.getElementsByClassName('myCanvas')[value] ) return document.getElementsByClassName('myCanvas')[value];
				let parent = document.getElementById('page' + this.pageProp.dataId);
				let canvasDom = document.createElement('canvas');
				canvasDom.width = this.viewWidth;
				canvasDom.height = this.viewHeight;
				canvasDom.style.position = 'absolute';
				canvasDom.style.top = 0;
				canvasDom.style.left = 0;
				canvasDom.style.backgroundColor = this.pageProp.bgColor;
				canvasDom.style.zIndex = -value;
				canvasDom.setAttribute('class', 'myCanvas');
				parent.appendChild(canvasDom);
				return document.getElementsByClassName('myCanvas')[value];
			},
			//清除画布
			clearCanvas () {
				let myCanvas = document.getElementById('myCanvas' + this.pageProp.dataId);
				let context = myCanvas.getContext('2d');
				context.clearRect(0, 0, myCanvas.width, myCanvas.height);
			},
			//绘制完成
			finish (pages) {
				// #ifndef H5
				UniViewJSBridge.publishHandler('onWxsInvokeCallMethod', {
					cid: this._$id,
					method: 'finish',
					args: {'pages': pages}
				})
				// #endif
			},
			//重绘制文字
			restartDrawText (value) {
				this.clearCanvas();
				this.drawText();
			},
			//参数改变
			pagePropChange (newValue, oldValue) {
				if ( newValue.fontsize != oldValue.fontsize ) {//字体大小改变
					this.restartDrawText();
				}
				if ( newValue.color != oldValue.color ) {//字体颜色改变
					this.restartDrawText();
				}
				if ( newValue.content != oldValue.content ) {//内容改变
					this.contentChange(newValue.content, oldValue.content);
				}
			},
			contentChange (newValue, oldValue) {
				this.contentSync = newValue.replace(oldValue, '');
				this.computedText();
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
	}
	#scroll {
		width: 100%;
	}
	.scroll-item {
		width: 100%;
		padding: 0 20rpx;
		box-sizing: border-box;
	}
	.scroll-text {
		white-space: pre-wrap;
	}
	.box {
		width: 100%;
		height: 100%;
		padding: 0 20rpx 20rpx 20rpx;
		box-sizing: border-box;
		position: absolute;
		left: 0;
		top: 0;
	}
	.content {
		position: relative;
		width: 100%;
		min-height: 100%;
	}
</style>