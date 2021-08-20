<template>
	<view class="page" :id="'page' + dataId" :prop="pageProp" :change:prop="page.pagePropChange">
		<view class="box">
			<view class="content" :id="'content' + dataId" v-if="pageType != 'scroll'"></view>
			<view class="content" style="z-index: -999;" :id="'computed' + dataId"></view>
		</view>
		<view :id="'scroll-box' + dataId" class="scroll-box" :style="{'color': color, 'padding': `0px ${slide}px`, 'background-color': bgColor}" v-if="pageType == 'scroll'"></view>
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
			//页面左右边距（单位px）
			slide: {
				type: Number | String,
				default: 10
			},
			
		},
		data () {
			return {
				contents: [],
				nowContent: '',
				loading: false,//等待内容请求
				upper: false,//文章是否到第一章节
				lower: false,//文章是否到最后章节
				restart: false
			}
		},
		computed: {
			pageProp () {
				return {
					content: this.nowContent,
					dataId: this.dataId,
					color: this.color,
					bgColor: this.bgColor,
					slide: this.slide,
					fontsize: parseInt(this.fontsize),
					pageType: this.pageType,
					lineHeight: parseInt(this.lineHeight),
					restart: this.restart
				};
			}
		},
		methods: {
			//初始化
			init (data) {
				this.restart = true;
				this.contents = data.contents || this.contents;
				let index = this.indexOf(this.contents, 'chapter', data.current);
				this.nowContent = this.contents[index > -1 ? index : 0];
				this.upper = this.nowContent.chapter == 1;
				this.lower = this.nowContent.isEnd;
			},
			//跳转
			change (data) {
				this.restart = true;
				let arr = [];
				let len = data.contents.length;
				for ( let i = 0; i < len; i++ ) {
					let index = this.indexOf(this.contents, 'chapter', data.contents[i].chapter);
					if ( index == -1 ) {
						arr.push(data.contents[i])
					}
				}
				this.contents = this.contents.concat(arr);
				let index = this.indexOf(this.contents, 'chapter', data.current);
				this.nowContent = this.contents[index > -1 ? index : 0];
				this.upper = this.nowContent.chapter == 1;
				this.lower = this.nowContent.isEnd;
			},
			/**
			 * 数组查找符合条件元素并返回下标
			 * @param {Array} arr 传入数组
			 * @param {String} value 条件元素
			 * @param {String} query 对比key值
			*/
			indexOf (arr, query, value) {
				let len = arr.length;
				for ( let i = 0; i < len; i++ ) {
					if ( arr[i][query] == value ) {
						return parseInt(i);
					}
				}
				return -1;
			},
			getPrevContent (e) {
				if ( this.loading ) {
					return;
				}
				this.upper = e.chapter == 1;
				if ( this.upper ) {
					if ( e.isTop ) {
						uni.showToast({
							title: '前面已经没有了',
							icon: 'none'
						})
					}
					return;
				}
				this.loading = true
				let index = this.indexOf(this.contents, 'chapter', parseInt(e.chapter) - 1);
				if ( index > -1 ) {
					this.loading = false;
					this.lower = this.contents[index].isEnd;
					this.nowContent = this.contents[index];
				} else {
					this.$emit('loadmore',
					parseInt(e.chapter) - 1,
					(content) => {
						this.loading = false;
						this.lower = content.isEnd;
						this.nowContent = content;
						this.contents.push(content);
					});
				}
				
			},
			getNextContent (e) {
				if ( this.loading ) {
					return;
				}
				if ( this.lower ) {
					if ( e.isBottom ) {
						uni.showToast({
							title: '后面已经没有了',
							icon: 'none'
						})
					}
					return;
				}
				this.loading = true
				let index = this.indexOf(this.contents, 'chapter', parseInt(e.chapter) + 1);
				if ( index > -1 ) {
					this.loading = false;
					this.lower = this.contents[index].isEnd;
					this.nowContent = this.contents[index];
				} else {
					this.$emit('loadmore',
					parseInt(e.chapter) + 1,
					(content) => {
						this.loading = false;
						this.lower = content.isEnd;
						this.nowContent = content;
						this.contents.push(content);
					});
				}
			},
			//预加载上下章节
			preload (e) {
				if ( this.loading ) {
					return;
				}
				if ( this.lower ) {
					return;
				}
				let prevIndex = e.chapter > 1 ? this.indexOf(this.contents, 'chapter', parseInt(e.chapter) - 1) : 0;
				let nextIndex = this.indexOf(this.contents, 'chapter', parseInt(e.chapter) + 1);
				let arr = [];
				if ( prevIndex == -1 ) arr.push(parseInt(e.chapter) - 1);
				if ( nextIndex == -1 ) arr.push(parseInt(e.chapter) + 1);
				if ( prevIndex == -1 || nextIndex == -1 ) {
					this.$emit('preload', arr,
					(contents) => {
						this.loading = false;
						this.contents = this.contents.concat(contents);
					});
				}
			},
			//抛出阅读页面改变事件
			currentChange (e) {
				this.$emit('readPageChange', e.currentInfo);
			},
			resetRestart () {
				this.restart = false;
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
				updownloading: false,
				currentInfo: {
					chapter: 0,
					start: 0,
					end: 0
				}
			}
		},
		mounted () {
			this.initDom.bind(this);
			let scrollBox = document.getElementById('scroll-box' + this.pageProp.dataId);
			if ( scrollBox ) {
				scrollBox.onscroll = () => {
					this.scroll(scrollBox)
				};
			}
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
				if ( Math.ceil(el.scrollTop + el.offsetHeight) >= el.scrollHeight ) {//触底
					let args = {'chapter': el.lastChild.getAttribute('chapter'), 'isBottom': Math.ceil(el.scrollTop + el.offsetHeight) >= el.scrollHeight};
					this.scrollToLower(args);
				}
				if ( el.scrollTop <= 0 ) {//触顶
					let args = {'chapter': el.firstChild.getAttribute('chapter'), 'isTop': el.scrollTop == 0};
					this.scrollToUpper(args);
				}
			},
			scrollToLower (args) {
				// #ifndef H5
				UniViewJSBridge.publishHandler('onWxsInvokeCallMethod', {
					cid: this._$id,
					method: 'getNextContent',
					args: args
				})
				// #endif
				// #ifdef H5
				this.getNextContent(args)
				// #endif
			},
			scrollToUpper (args) {
				// #ifndef H5
				UniViewJSBridge.publishHandler('onWxsInvokeCallMethod', {
					cid: this._$id,
					method: 'getPrevContent',
					args: args
				})
				// #endif
				// #ifdef H5
				this.getPrevContent(args)
				// #endif
			},
			preloadContent (chapter) {
				// #ifndef H5
				UniViewJSBridge.publishHandler('onWxsInvokeCallMethod', {
					cid: this._$id,
					method: 'preload',
					args: {'chapter': chapter}
				})
				// #endif
				// #ifdef H5
				this.preload({'chapter': chapter})
				// #endif
			},
			//计算页面显示文字
			computedText () {
				let parent = document.getElementById('computed' + this.pageProp.dataId);
				this.viewWidth = parent.offsetWidth;
				this.viewHeight = parent.offsetHeight;
				let computedCanvas = this.createComputedCanvas(parent);
				let context = computedCanvas.getContext('2d');
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
							} else if ( lineWidth >= this.viewWidth - (2 * this.pageProp.slide) ) {
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
					let parent = document.getElementById('content' + this.pageProp.dataId);
					for ( let i = 0; i < pages.length; i++ ) {
						let myCanvas = this.createPageItem(parent, pages[i]);
						let context = myCanvas.getContext('2d');
						for ( let j = 0; j < pages[i].text.length; j++ ) {
							console.log()
							context.font = this.pageProp.fontsize + 'px 微软雅黑';
							context.fillStyle = this.pageProp.color;
							context.fillText(pages[i].text[j], this.pageProp.slide, (j + 1) * (this.pageProp.fontsize + this.pageProp.lineHeight));
						}
					}
				} else {
					let scrollBox = document.getElementById('scroll-box' + this.pageProp.dataId);
					let type = scrollBox.firstChild ? pages[0].chapter < scrollBox.firstChild.getAttribute('chapter') ? 'prev' : 'next' : 'init';
					let scrollChapter = this.createScrollChapter(scrollBox, pages[0].chapter);
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
						// #ifndef H5
						UniViewJSBridge.publishHandler('onWxsInvokeCallMethod', {
							cid: this._$id,
							method: 'resetRestart'
						})
						// #endif
						// #ifdef H5
						this.resetRestart()
						// #endif
						this.scroll(scrollBox);
					}
					setTimeout(() => {
						this.preloadContent(this.pageProp.content.chapter);
					}, 500)
				}
			},
			//创建一个独立的canvas画板，用于计算文字布局
			createComputedCanvas (el) {
				if ( document.getElementsByClassName('computedCanvas')[0] ) return document.getElementsByClassName('computedCanvas')[0];
				let canvasDom = document.createElement('canvas');
				canvasDom.width = this.viewWidth;
				canvasDom.height = this.viewHeight;
				canvasDom.style.position = 'absolute';
				canvasDom.style.top = 0;
				canvasDom.style.left = 0;
				canvasDom.setAttribute('class', 'computedCanvas');
				el.appendChild(canvasDom);
				return document.getElementsByClassName('computedCanvas')[0];
			},
			createPageItem (el, info) {
				let pageItem = document.createElement('div');
				pageItem.style.width = '100%';
				pageItem.style.height = '100%';
				pageItem.style.position = 'absolute';
				pageItem.style.top = 0;
				pageItem.style.left = 0;
				pageItem.style.zIndex = -(info.chapter + info.start);
				pageItem.setAttribute('class', 'page-item page-item-chapter__' + (info.chapter + info.start));
				pageItem.setAttribute('chapter', info.chapter);
				pageItem.setAttribute('start', info.start);
				pageItem.setAttribute('end', info.end);
				let canvas = document.createElement('canvas');
				canvas.width = this.viewWidth;
				canvas.height = this.viewHeight;
				canvas.style.position = 'absolute';
				canvas.style.top = 0;
				canvas.style.left = 0;
				canvas.style.backgroundColor = this.pageProp.bgColor;
				canvas.setAttribute('class', 'page-item-canvas');
				pageItem.appendChild(canvas);
				let pageBg = document.createElement('div');
				pageBg.style.width = '100%';
				pageBg.style.height = '100%';
				pageBg.style.position = 'absolute';
				pageBg.style.top = 0;
				pageBg.style.left = 0;
				pageBg.style.transform = 'translateX(100%)';
				pageBg.style.backgroundColor = this.pageProp.bgColor;
				pageBg.setAttribute('class', 'page-item-bg');
				pageItem.appendChild(pageBg);
				el.appendChild(pageItem);
				return document.getElementsByClassName('page-item-chapter__' + (info.chapter + info.start))[0].getElementsByClassName('page-item-canvas')[0];
			},
			//创建滚动布局下的章节盒子
			createScrollChapter (el, chapter) {
				let divDom = document.createElement('div');
				divDom.style.width = '100%';
				divDom.setAttribute('class', 'scroll-chapter-box scroll-box-chapter__' + chapter);
				divDom.setAttribute('chapter', chapter);
				if ( chapter > (el.lastChild ? el.lastChild.getAttribute('chapter') : 0) ) {
					el.appendChild(divDom);
				}
				if ( chapter < (el.firstChild ? el.firstChild.getAttribute('chapter') : 0) ) {
					el.insertBefore(divDom, el.firstChild);
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
				if ( newValue.restart != oldValue.restart ) {//内容改变
					this.restartChange(newValue.restart, oldValue.restart);
				}
			},
			//重绘页面
			restartDrawText () {
				if ( this.pageProp.pageType == 'scroll' ) {
					document.getElementById('scroll-box' + this.pageProp.dataId).innerHTML = '';
					this.computedText();
				} else {
					document.getElementById('content' + this.pageProp.dataId).innerHTML = '';
					this.computedText();
				}
			},
			//文本内容改变时
			contentChange (newValue, oldValue) {
				if ( this.pageProp.restart ) {
					return;
				}
				if ( !this.currentInfo.chapter ) {
					this.currentInfo.chapter = newValue.chapter;
					this.currentInfo.start = newValue.start;
				}
				if ( newValue?.content ) {
					this.computedText();
				}
			},
			restartChange (newValue) {
				if ( newValue ) {
					this.currentInfo.chapter = this.pageProp.content.chapter;
					this.currentInfo.start = this.pageProp.content.start;
					this.restartDrawText();
				}
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
		left: 0;
		right: 0;
		top: 0;
		bottom: 0;
		box-sizing: border-box;
		/* overflow-anchor: auto; */
		overflow-y: auto;
	}
	.box {
		width: 100%;
		height: 100%;
		box-sizing: border-box;
		position: absolute;
		left: 0;
		top: 0;
		overflow: hidden;
	}
	.content {
		position: absolute;
		left: 0;
		top: 0;
		width: 100%;
		height: 100%;
	}
</style>