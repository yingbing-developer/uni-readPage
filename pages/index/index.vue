<template>
	<view>
		<yingbing-ReadPage
		ref="page"
		:page-type="pageType"
		:font-size="fontsize"
		:line-height="lineHeight"
		:color="color"
		:bg-color="bgColor"
		:slide="slide"
		:enablePreload="enablePreload"
		:noChapter="noChapter"
		@loadmore="loadmoreContent"
		@preload="preloadContent"
		@currentChange="currentChange"
		@setCatalog="setCatalog"></yingbing-ReadPage>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				pages: [],
				pageType: 'real',
				scrollTop: 400,
				fontsize: 20,
				lineHeight: 15,
				color: '#333',
				slide: 40,
				bgColor: '#fcd281',
				enablePreload: true,
				noChapter: false
			}
		},
		onReady() {
			let contents = [{
				chapter: 3,
				content: this.getContent(3),
				title: '第三章',
				isStart: false,
				isEnd: false
			}]
			const { page } = this.$refs;
			if ( this.noChapter ) {
				page.init({
					content: this.getContent(2),
					start: 0
				})
			} else {
				page.init({
					contents: contents,
					start: 0,
					currentChapter: 3
				})
			}
		},
		methods: {
			currentChange (e) {
				console.log(e);
			},
			setCatalog (e) {
				console.log(e);
			},
			addFontsize () {
				this.fontsize += 4;
			},
			changePageType () {
				this.pageType = this.pageType == 'real' ? 'scroll' : 'real';
			},
			reduceFontSize () {
				this.fontsize -= 4;
			},
			changeLineHeight () {
				this.lineHeight += 4;
			},
			changeSkin () {
				this.color = '#f5f5f5';
				this.bgColor = '#999';
			},
			changeChapter () {
				if ( this.noChapter ) {
					page.change({
						start: 100
					})
				} else {
					setTimeout(() => {
						let contents = [{
							chapter: 3,
							content: this.getContent(3),
							title: '第三章',
							isStart: false,
							isEnd: false
						},{
							chapter: 4,
							content: this.getContent(4),
							title: '第四章',
							isStart: false,
							isEnd: false
						},{
							chapter: 5,
							content: this.getContent(5),
							title: '第五章',
							isStart: false,
							isEnd: false
						}]
						const { page } = this.$refs;
						page.change({
							contents: contents,
							start: 0,
							currentChapter: 5
						})
					}, 2000)
				}
			},
			loadmoreContent (chapter, callback) {
				setTimeout(() => {
					callback('success', {
						chapter: chapter,
						content: this.getContent(chapter),
						title: '第' + chapter + '章',
						isStart: chapter == 1,
						isEnd: chapter == 7
					});
					// callback('fail');
					// callback('timeout');
				}, 2000)
			},
			preloadContent (chapters, callback) {
				setTimeout(() => {
					let contents = []
					for ( let i in chapters ) {
						contents.push({
							chapter: chapters[i],
							start: 0,
							content: this.getContent(chapters[i]),
							title: '第' + chapters[i] + '章',
							isStart: chapters[i] == 1,
							isEnd: chapters[i] == 7
						})
					}
					callback('success', contents);
					// callback('fail');
					// callback('timeout');
				}, 2000)
			},
			getContent (chapter = 1) {
return `第${chapter}章
                你们好啊你们好啊你们好啊你们好啊你们好啊你们好啊你们好啊
你们好啊你们好啊你们好啊你们好啊你们好啊你们好啊你们好啊
你们好啊你们好啊你们好啊你们好啊你们好啊你们好啊你们好啊
你们好啊你们好啊你们好啊你们好啊你们好啊你们好啊你们好啊
你们好啊你们好啊你们好啊你们好啊你们好啊你们好啊你们好啊
你们好啊你们好啊你们好啊你们好啊你们好啊你们好啊你们好啊
你们好啊你们好啊你们好啊你们好啊你们好啊你们好啊你们好啊
你们好啊你们好啊你们好啊你们好啊你们好啊你们好啊你们好啊
你们好啊你们好啊你们好啊你们好啊你们好啊你们好啊你们好啊
你们好啊你们好啊你们好啊你们好啊你们好啊你们好啊你们好啊
你们好啊你们好啊你们好啊你们好啊你们好啊你们好啊你们好啊
你们好啊你们好啊你们好啊你们好啊你们好啊你们好啊你们好啊
你们好啊你们好啊你们好啊你们好啊你们好啊你们好啊你们好啊
你们好啊你们好啊你们好啊你们好啊你们好啊你们好啊你们好啊
你们好啊你们好啊你们好啊你们好啊你们好啊你们好啊你们好啊

神经病啊神经病啊神经病啊神经病啊神经病啊神经病啊神经病啊
神经病啊神经病啊神经病啊神经病啊神经病啊神经病啊神经病啊
神经病啊神经病啊神经病啊神经病啊神经病啊神经病啊神经病啊
神经病啊神经病啊神经病啊神经病啊神经病啊神经病啊神经病啊
神经病啊神经病啊神经病啊神经病啊神经病啊神经病啊神经病啊
神经病啊神经病啊神经病啊神经病啊神经病啊神经病啊神经病啊
神经病啊神经病啊神经病啊神经病啊神经病啊神经病啊神经病啊
神经病啊神经病啊神经病啊神经病啊神经病啊神经病啊神经病啊
神经病啊神经病啊神经病啊神经病啊神经病啊神经病啊神经病啊
神经病啊神经病啊神经病啊神经病啊神经病啊神经病啊神经病啊
神经病啊神经病啊神经病啊神经病啊神经病啊神经病啊神经病啊

疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊啊
疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊啊
疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊啊
疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊啊
疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊啊
疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊啊
疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊啊
疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊啊
疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊啊
疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊啊
疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊啊
疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊啊
疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊啊
疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊啊

傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊啊
傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊啊
傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊啊
傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊啊
傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊啊
傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊啊
傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊啊
傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊啊
傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊啊
傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊啊
傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊啊
傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊啊
傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊啊
傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊啊

神经病啊神经病啊神经病啊神经病啊神经病啊神经病啊神经病啊
神经病啊神经病啊神经病啊神经病啊神经病啊神经病啊神经病啊
神经病啊神经病啊神经病啊神经病啊神经病啊神经病啊神经病啊
神经病啊神经病啊神经病啊神经病啊神经病啊神经病啊神经病啊
神经病啊神经病啊神经病啊神经病啊神经病啊神经病啊神经病啊
神经病啊神经病啊神经病啊神经病啊神经病啊神经病啊神经病啊
神经病啊神经病啊神经病啊神经病啊神经病啊神经病啊神经病啊
神经病啊神经病啊神经病啊神经病啊神经病啊神经病啊神经病啊
神经病啊神经病啊神经病啊神经病啊神经病啊神经病啊神经病啊
神经病啊神经病啊神经病啊神经病啊神经病啊神经病啊神经病啊
神经病啊神经病啊神经病啊神经病啊神经病啊神经病啊神经病啊

疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊啊
疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊啊
疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊啊
疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊啊
疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊啊
疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊啊
疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊啊
疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊啊
疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊啊
疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊啊
疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊啊
疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊啊
疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊啊
疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊疯子啊啊

傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊啊
傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊啊
傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊啊
傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊啊
傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊啊
傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊啊
傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊啊
傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊啊
傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊啊
傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊啊
傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊啊
傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊啊
傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊啊
傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊傻子啊啊
`
			}
		}
	}
</script>

<style scoped>
</style>
