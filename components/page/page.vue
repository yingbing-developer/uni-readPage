<template>
	<view class="page" :style="{'background-color': bgColor}" :id="'page' + dataId" :prop="pageProp" :change:prop="page.pagePropChange">
		<view class="box">
			<view class="content" :id="'content' + dataId"></view>
		</view>
		<view id="scroll-box" class="scroll-box" :style="{'color': color}" v-if="pageType == 'scroll'"></view>
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
				this.upper = data.upper || this.upper;
				this.lower = data.lower || this.lower;
			},
			scrolltoupper (e) {
				this.upper = e.currentInfo.chapter == 1;
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
				parseInt(e.currentInfo.chapter) - 1,
				(content) => {
					this.loading = false;
					this.nowContent = content;
					this.contents.shift(content);
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
				parseInt(e.currentInfo.chapter) + 1,
				(content, isEnd = false) => {
					this.loading = false;
					this.lower = isEnd;
					this.nowContent = content;
					this.contents.push(content);
				});
			},
			getScrollHeight () {
				return new Promise((resolve) => {
					const query = uni.createSelectorQuery().in(this);
					query.select('#scroll').boundingClientRect(data => {
					  resolve(data.height)
					}).exec();
				})
			},
			//抛出阅读页面改变事件
			currentChange (e) {
				this.$emit('readPageChange', e.currentInfo);
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
				viewWidth: 0,
				currentInfo: {
					chapter: 0,
					start: 0,
					end: 0
				}
			}
		},
		mounted () {
			this.initDom.bind(this);
			let scrollBox = document.getElementById('scroll-box');
			scrollBox.onscroll = () => {
				this.scroll(scrollBox)
			};
			if ( this.pageProp.content.content ) {
				this.currentInfo.chapter = this.pageProp.content.chapter;
				this.currentInfo.start = this.pageProp.content.start;
				this.computedText();
			}
		},
		methods: {
			initDom () {
				myPageDom = page.init(document.getElementById('page' + this.pageProp.dataId));
				// 观测更新的数据在 view 层可以直接访问到
				myPageDom.setOption(this.pageProp);
			},
			scroll (el) {
				let scrollItems = document.getElementsByClassName('scroll-item');
				let scrollBottom = el.scrollTop + el.offsetHeight;
				for ( let i = 0; i < scrollItems.length; i++ ) {
					let offsetBottom1 = scrollItems[i].offsetTop + scrollItems[i].offsetHeight;
					let offsetBottom2 = i < scrollItems.length - 1 ? scrollItems[i+1].offsetTop + scrollItems[i+1].offsetHeight : offsetBottom1 + 2;
					if ( scrollBottom >= offsetBottom1 &&  scrollBottom < offsetBottom2 ) {
						let chapter = scrollItems[i].getAttribute('chapter');
						let start = scrollItems[i].getAttribute('start');
						let end = scrollItems[i].getAttribute('end');
						if ( this.currentInfo.chapter != chapter || this.currentInfo.start != start ) {
							this.currentInfo.chapter = chapter;
							this.currentInfo.start = start;
							this.currentInfo.end = end;
							// #ifndef H5
							UniViewJSBridge.publishHandler('onWxsInvokeCallMethod', {
								cid: this._$id,
								method: 'currentChange',
								args: {'currentInfo': this.currentInfo}
							})
							// #endif
							// #ifdef H5
							this.currentChange({'currentInfo': this.currentInfo})
							// #endif
						} else {
							this.currentInfo.chapter = chapter;
							this.currentInfo.start = start;
							this.currentInfo.end = end;
						}
					}
				}
				if ( Math.floor(el.scrollTop + el.offsetHeight) == el.scrollHeight ) {//触底
					// #ifndef H5
					UniViewJSBridge.publishHandler('onWxsInvokeCallMethod', {
						cid: this._$id,
						method: 'scrolltolower',
						args: {'currentInfo': this.currentInfo}
					})
					// #endif
					// #ifdef H5
					this.scrolltolower({'currentInfo': this.currentInfo})
					// #endif
				}
				if ( el.scrollTop <= 0 ) {//触顶
					// #ifndef H5
					UniViewJSBridge.publishHandler('onWxsInvokeCallMethod', {
						cid: this._$id,
						method: 'scrolltoupper',
						args: {'currentInfo': this.currentInfo}
					})
					// #endif
					// #ifdef H5
					this.scrolltoupper({'currentInfo': this.currentInfo})
					// #endif
				}
			},
			//计算页面显示文字
			computedText () {
				let parent = document.getElementById('content' + this.pageProp.dataId);
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
					let scrollBox = document.getElementById('scroll-box');
					let type = scrollBox.firstChild ? pages[0].chapter < scrollBox.firstChild.getAttribute('chapter') ? 'prev' : 'next' : 'init';
					let scrollChapter = this.createScrollChapter(pages[0].chapter);
					for ( let i = 0; i < pages.length; i++ ) {
						let scrollDom = this.createScrollItem(scrollChapter, pages[i], i);
						for ( let j = 0; j < pages[i].text.length; j++ ) {
							this.insetScrollText(scrollDom, pages[i].text[j]);
						}
					}
					if ( type == 'prev' ) {
						scrollBox.scrollTop = scrollChapter.offsetHeight;
						if ( document.getElementsByClassName('scroll-chapter-box').length > 3 ) scrollBox.removeChild(scrollBox.lastChild);
					} else if ( type == 'next' ) {
						if ( document.getElementsByClassName('scroll-chapter-box').length > 3 ) scrollBox.removeChild(scrollBox.firstChild);
						scrollBox.scrollTop = scrollBox.scrollHeight - scrollBox.lastChild.offsetHeight - scrollBox.offsetHeight + 50;
					} else {
						//初始化时，定位阅读位置
						let scrollItems = document.getElementsByClassName('scroll-item')
						let offsetHeight = 0;
						for ( let i = 0; i < scrollItems.length; i++ ) {
							offsetHeight += i > 0 ? scrollItems[i].offsetHeight : 0;
							if ( this.currentInfo.start >= scrollItems[i].getAttribute('start') && this.currentInfo.start <= scrollItems[i].getAttribute('end') ) {
								scrollBox.scrollTop = offsetHeight;
							}
						}
					}
				}
			},
			createCanvas (value) {
				if ( document.getElementsByClassName('myCanvas')[value] ) return document.getElementsByClassName('myCanvas')[value];
				let parent = document.getElementById('content' + this.pageProp.dataId);
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
			//创建滚动布局下的章节盒子
			createScrollChapter (chapter) {
				let parent = document.getElementById('scroll-box');
				let divDom = document.createElement('div');
				divDom.style.width = '100%';
				divDom.setAttribute('class', 'scroll-chapter-box scroll-box-chapter__' + chapter);
				divDom.setAttribute('chapter', chapter);
				if ( chapter > (parent.lastChild ? parent.lastChild.getAttribute('chapter') : 0) ) {
					parent.appendChild(divDom);
				}
				if ( chapter < (parent.firstChild ? parent.firstChild.getAttribute('chapter') : 0) ) {
					parent.insertBefore(divDom, parent.firstChild);
				}
				return document.getElementsByClassName('scroll-box-chapter__' + chapter)[0]
			},
			//创建滚动布局下的的页面盒子
			createScrollItem (el, info, value) {
				let divDom = document.createElement('div');
				divDom.style.width = '100%';
				divDom.setAttribute('class', 'scroll-item scroll-chapter__' + info.chapter);
				divDom.setAttribute('chapter', info.chapter);
				divDom.setAttribute('start', info.start);
				divDom.setAttribute('end', info.end);
				el.appendChild(divDom);
				return document.getElementsByClassName('scroll-chapter__' + info.chapter)[value]
			},
			//创建滚动布局下的的文字盒子
			insetScrollText (el, text) {
				let pDom = document.createElement('p');
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
			//重绘页面
			restartDrawText () {
				if ( this.pageProp.pageType == 'scroll' ) {
					document.getElementById('scroll-box').innerHTML = '';
					this.computedText();
				}
			},
			//文本内容改变时
			contentChange (newValue, oldValue) {
				if ( !this.currentInfo.chapter ) {
					this.currentInfo.chapter = newValue.chapter;
					this.currentInfo.start = newValue.start;
				}
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