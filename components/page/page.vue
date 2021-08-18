<template>
	<view class="page" :style="{'background-color': bgColor}">
		<view class="box">
			<view class="content" :id="'page' + dataId" :prop="pageProp" :change:prop="page.pagePropChange"></view>
		</view>
		<!-- <scroll-view :scroll-top="scrollTop" scroll-anchoring @scrolltoupper="scrolltoupper" @scrolltolower="scrolltolower" :upper-threshold="upperThreshold" :lower-threshold="lowerThreshold" class="scroll" scroll-y :show-scrollbar="false" v-if="pageType == 'scroll'">
			<view class="scroll-box">
				<view class="scroll-item" v-for="(item, index) in pages" :key="index">
					<view class="scroll-text" :style="{color: color, 'font-size': fontsize + 'px', 'margin-top': lineHeight + 'px'}" v-for="(text, i) in item.text" :key="i" v-html="text || ' '"></view>
				</view>
			</view>
		</scroll-view> -->
		<view id="scroll-box" class="scroll-box">
		</view>
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
				default: 0
			},
			//距底部多远时（单位px），触发 scrolltolower 事件
			lowerThreshold: {
				type: Number | String,
				default: 0
			}
		},
		data () {
			return {
				contents: [],
				nowContent: '',
				current: 1,
				loading: false,//等待内容请求
				pages: [],
				scrollTop: 0,
				upper: false,//文章是否到第一章节
				lower: false//文章是否到最后章节
			}
		},
		computed: {
			pageProp () {
				return {
					content: this.nowContent,
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
			init (data) {
				this.contents = data.contents || this.contents;
				this.nowContent = this.contents[0];
				this.current = data.current || this.current;
				this.upper = data.upper || this.upper;
				this.lower = data.lower || this.lower;
			},
			finish (e) {
				// #ifdef H5
				let pages = e;
				// #endif
				// #ifndef H5
				let pages = e.pages;
				// #endif
				if ( pages[0].chapter < (this.pages.length > 0 ? this.pages[this.pages.length - 1].chapter : 0) ) {
					this.pages = pages.concat(this.pages);
				} else {
					this.pages = this.pages.concat(pages);
				}
				if ( this.contents.length < 2 ) {
					setTimeout(() => {
						this.scrolltoupper();
					}, 4000)
				}
			},
			scrolltoupper () {
				if ( this.upper ) {
					uni.showToast({
						title: '前面已经没有了',
						icon: 'none'
					})
					return;
				}
				if ( this.loading ) {
					return;
				}
				this.loading = true
				this.$emit('loadmore',
				this.current - 1,
				(content, isEnd = false) => {
					this.loading = false;
					this.lower = isEnd;
					this.nowContent = content;
					this.contents = this.contents.shift(content);
				});
			},
			scrolltolower (e) {
				if ( this.lower ) {
					uni.showToast({
						title: '后面已经没有了',
						icon: 'none'
					})
					return;
				}
				if ( this.loading ) {
					return;
				}
				this.loading = true
				this.$emit('loadmore',
				this.current + 1,
				(content, isEnd = false) => {
					this.loading = false;
					this.lower = isEnd;
					this.nowContent = content;
					this.contents = this.contents.push(content);
				});
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
				viewHeight: 0,
				viewWidth: 0
			}
		},
		mounted () {
			this.initDom.bind(this);
			if ( this.pageProp.content.content ) {
				this.computedText();
			}
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
				let myCanvas = this.createCanvas(0);
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
						chapter: this.pageProp.content.chapter,
						start: lastIndex,
						end: 0,
						text: []
					})
					while ( pageHeight <= this.viewHeight ) {
						strs.push('');
						let content = this.pageProp.content.content;
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
						if ( lastIndex >= content.length - 1 ) break;
					}
					pages[pages.length - 1].end = lastIndex;
					pages[pages.length - 1].text = strs;
					if ( lastIndex < this.pageProp.content.content.length - 1 ) {
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
						let myCanvas = this.createCanvas(i);
						let context = myCanvas.getContext('2d');
						for ( let j = 0; j < pages[i].text.length; j++ ) {
							context.font = this.pageProp.fontsize + 'px 微软雅黑';
							context.fillStyle = this.pageProp.color;
							context.fillText(pages[i].text[j], 0, (j + 1) * (this.pageProp.fontsize + this.pageProp.lineHeight));
						}
					}
				} else {
					for ( let i = 0; i < pages.length; i++ ) {
						let scrollDom = this.createScrollItem(pages[i], i);
						for ( let j = 0; j < pages[i].text.length; j++ ) {
							this.insetScrollText(scrollDom, pages[i].text[j]);
						}
					}
				}
				this.finish(pages);
			},
			createCanvas (value) {
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
			createScrollItem (info, value) {
				let parent = document.getElementById('scroll-box');
				let divDom = document.createElement('div');
				divDom.style.width = '100%';
				divDom.setAttribute('class', 'scroll-item scroll-chapter__' + info.chapter);
				divDom.setAttribute('chapter', info.chapter);
				divDom.setAttribute('start', info.start);
				divDom.setAttribute('end', info.end);
				parent.appendChild(divDom);
				return document.getElementsByClassName('scroll-chapter__' + info.chapter)[value]
			},
			insetScrollText (el, text) {
				let pDom = document.createElement('p');
				pDom.style.color = this.pageProp.color;
				pDom.style.fontSize = this.pageProp.fontsize + 'px';
				pDom.style.marginTop = this.pageProp.lineHeight + 'px';
				pDom.style.whiteSpace = 'pre-wrap';
				pDom.setAttribute('class', 'scroll-text');
				pDom.innerHTML = text || ' ';
				el.appendChild(pDom);
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
	.scroll-box {
		position: absolute;
		left: 20rpx;
		right: 20rpx;
		top: 0;
		bottom: 20rpx;
		overflow-anchor: auto;
		overflow-y: auto;
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