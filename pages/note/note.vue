<template>
		<view class="container" @touchstart="handTouchStart" @touchend="handTouchEnd">
			<!-- #ifdef APP-PLUS -->
			<view class="status_bar"></view>
			<!-- #endif -->
				<!-- 自定义头部导航栏组件 -->
				<!-- <uni-nav-bar>
					<view v-slot="left" class="navLeft">
						<uni-icons type="left" size="20"></uni-icons>
					</view>
					<view v-slot="right" class="navRight">
						<uni-icons type="right" size="20"></uni-icons>
						<uni-icons type="right" size="20"></uni-icons>
						<uni-icons type="right" size="20"></uni-icons>
					</view>
				</uni-nav-bar> -->
				<view class="navDiy">
					<view class="navLeft">
						<uni-icons class="uniicon" type="left" size="20" @tap="back"></uni-icons>
					</view>
					<view class="navRight">
						<uni-icons class="uniicon" type="undo-filled" v-show="currentTextNum!=0" 
							size="20" @tap="undo"></uni-icons>
						<uni-icons class="uniicon" type="redo-filled" v-show="currentTextNum!=0 || currentTextContent=='<p><br></p>'" 
							size="20" @tap="redo"></uni-icons>
						<uni-icons class="uniicon" type="checkmarkempty"
							v-show="currentTextNum!=0 || noteName || currentTextContent!='' || notePic.length>0" 
							size="20" @tap="toNoteMsg"></uni-icons>
					</view>
				</view>
				<!-- 标题部分 -->
				<view class="noteHeader">
					<view class="noteName">
						<input 
							id = "noteName"
							placeholder="标题" 
							v-model="noteName" 
							maxlength="100" 
							placeholder-style="color:rgba(192, 196, 204, 0.8);">
					</view>
					<view class="noteOthers">
						<view class="timeAndNum" style="color: #CCCCCC;">
							<!-- 显示创建笔记的时间 -->
							<view class="time">{{noteTime || currentTime}}</view>
							<image 
								:showLoading="true" 
								src="/static/images/verticalBar.png" 
								style="width:40rpx;height:40rpx" 
								mode=aspectFill></image>
							<!-- 当前字数 -->
							<view class="num">{{ currentTextNum }}字</view>
						</view>
						<uni-icons type="calendar" @tap="insertDate" size="20"></uni-icons>
					</view>
				</view>
				<!-- 正文部分 -->
				<view class="editorContent">
					<editor v-if="isDataReady" clas="editor"
						id="editor"
						placeholder="开始输入..." 
						showImgSize
						showImgToolbar
						showImgResize
						:read-only="readOnly"
						@ready="onEditorReady"
						@statuschange="onStatusChange"
						@input="onEditorInput"
					></editor>
						<!-- @blur="lostFocus" -->
				</view>
				<!-- 颜色界面 -->
				<view class="colorMenu" v-if="selectName=='color'">
					<view class="selectColor" v-for="(item,index) in colors" :key="index">
						<text class='title'>{{item.title}}</text>
						<scroll-view class="scroll" scroll-x>
							<view class="list">
								<view class="color" @tap="select(color,item.list)"
									v-for="color in item.list"
									:key="color.id"
									:style="'background:'+color.color">
									<uni-icons
										v-show="formats[color.key]==color.color || color.isChecked"
										type="checkmarkempty" size="40rpx">
									</uni-icons>
								</view>	
							</view>
						</scroll-view>
					</view>
					<!-- <view class="moreColors">
					<button @tap="moreColors">更多颜色</button>
						<t-color-picker ref="colorPicker" :color="color" @confirm="select"></t-color-picker>
					</view> -->
				</view>
				<!-- 字号界面 -->
				<view class="fontSizeMenu" v-if="selectName=='fontSize'">
					<view class="selectFontSize">
						<text class='title'>{{textSize.title}}</text>
						<scroll-view class="scroll" scroll-x>
							<!-- <view class="list"> -->
								<view class="fontSize" @tap="select(item,textSize.list)"
									v-for="(item,index) in textSize.list"
									:key="item.id"
									:class="formats[item.key]==item.val || item.isChecked?'toSize':''">
									{{item.num}}
								</view>
							<!-- </view> -->
						</scroll-view>
					</view>
				</view>
				<!-- 工具栏 -->
				<view class="toolbar" @tap="format">
						<!-- 字体加粗 -->
						<view class="iconfont icon-zitijiacu" data-name="bold" :class="formats.bold?'activated':''"></view>
						<!-- 字体斜体 -->
						<view class="iconfont icon-zitixieti" data-name="italic" :class="formats.italic?'activated':''"></view>
						<!-- 字体下划线 -->
						<view class="iconfont icon-zitixiahuaxian" data-name="underline" :class="formats.underline?'activated':''"></view>
						<!-- 字体删除线 -->
						<view class="iconfont icon-zitishanchuxian" data-name="strike" :class="formats.strike?'activated':''"></view>
						<!-- 颜色 -->
						<view class="iconfont icon-yanse1" 
							:class="selectName == 'color'?'activated':''" 
							@tap="selectColorOrFontSize('color')"></view>
						<!-- 字号 -->
						<view class="iconfont icon-ziti" 
							:class="selectName == 'fontSize'?'activated':''"
							@tap="selectColorOrFontSize('fontSize')"></view>
						<!-- 分隔线 -->
						<view class="iconfont icon-fengexian" @tap="insertDivider"></view>
						<!-- 插入图片 -->
						<view class="iconfont icon-charutupian" @tap="insertImage"></view>
						<!-- 清除全部内容 -->
						<view class="iconfont icon-shanchu" @tap="clear"></view>
					</view>			
			</view>
</template>

<script>
	import {parseTime} from '../../utils/tool.js'
	import tColorPicker from '@/components/t-color-picker/t-color-picker.vue'
	export default {
		components:{
			tColorPicker
		},
		data() {
			return {
				checkToIndex:false,//安卓侧滑返回时
				readOnly: false,
				isDataReady:false,
				noteStatus:'',//当为新增笔记时赋值
				// 格式状态
				formats: {},
				// 跳转页面传递过来的笔记所有信息
				noteValue: '',
				// 当前文本的内容
				currentTextContent: '',
				// 当前文本字数
				currentTextNum: 0,
				// 富文本对象
				editorCtx: {},
				noteName: '',
				noteTime: '',
				noteContent:'',
				notePic:[],
				currentTime:'',
				hasImage:false,//默认没有图片
				imgMsg:[],
				selectName:'',//选中的是颜色color 还是 字号fontSize
				// 选择文字颜色
				colors: [
					{
						title: '文字颜色',
						list: [
							{
								id: 0,
								color: '#333333',
								key: 'color',
								val: '#333333',
								isChecked: false
							},
							{
								id: 1,
								color: '#666666',
								key: 'color',
								val: '#666666',
								isChecked: false
							},
							{
								id: 2,
								color: '#999999',
								key: 'color',
								val: '#999999',
								isChecked: false
							},
							{
								id: 3,
								color: '#ff0000',
								key: 'color',
								val: '#ff0000',
								isChecked: false
							},
							{
								id: 4,
								color: '#ff5f2d',
								key: 'color',
								val: '#ff5f2d',
								isChecked: false
							},
							{
								id: 5,
								color: '#f1b000',
								key: 'color',
								val: '#f1b000',
								isChecked: false
							},
							{
								id: 6,
								color: '#00c878',
								key: 'color',
								val: '#00c878',
								isChecked: false
							},
							{
								id: 7,
								color: '#09abff',
								key: 'color',
								val: '#09abff',
								isChecked: false
							},
							{
								id: 8,
								color: '#0963ff',
								key: 'color',
								val: '#0963ff',
								isChecked: false
							},
							{
								id: 9,
								color: '#6b5af7',
								key: 'color',
								val: '#6b5af7',
								isChecked: false
							}
						]
					},
					{
						title: '文字背景颜色',
						list: [
							{
								id: 0,
								color: '#ffcfcf',
								key: 'backgroundColor',
								val: '#ffcfcf',
								isChecked: false
							},
							{
								id: 1,
								color: '#ffd6c9',
								key: 'backgroundColor',
								val: '#ffd6c9',
								isChecked: false
							},
							{
								id: 2,
								color: '#fff0c8',
								key: 'backgroundColor',
								val: '#fff0c8',
								isChecked: false
							},
							{
								id: 3,
								color: '#c2ffe6',
								key: 'backgroundColor',
								val: '#c2ffe6',
								isChecked: false
							},
							{
								id: 4,
								color: '#bee8ff',
								key: 'backgroundColor',
								val: '#bee8ff',
								isChecked: false
							},
							{
								id: 5,
								color: '#c5daff',
								key: 'backgroundColor',
								val: '#c5daff',
								isChecked: false
							},
							{
								id: 6,
								color: '#c9c2ff',
								key: 'backgroundColor',
								val: '#c9c2ff',
								isChecked: false
							}
						]
					}
				],
				// 选择字号
				textSize: {
					title: '字号',
					list: []
				},
				color: {r: 255,g: 0,b: 0,a: 0.6}
			};
		},
		async onLoad(options) {
			this.noteStatus = options.status;
			try{
				await this.getNote();
				this.isDataReady = true;
			}catch(err){
				console.log("错误:",err)
				uni.showToast({
					title:"读取错误",
					icon:"error"
				})
			}
		},
		onBackPress(options)  {
			// console.log("hide",options)
			if(!this.checkToIndex){
				uni.showModal({
					content:'确定保存返回吗?',
					success:res=>{
						if(res.confirm){
							this.back();
						}
					}
				});
				// 返回 true 表示阻止默认返回行为
				return true;
			}
		},
		onUnload() {
			// uni.$off('msg');//避免有多个on没有关闭
			console.log("note-unload");
		},
		mounted() {
			for(let i = 12;i<=36;i++){
				this.textSize.list.push({
					id:i-12,
					num:i,
					key:'fontSize',
					val:i+'px',
					isChecked:false
				})
			}
			// console.log("挂载完了！this.noteValue",this.noteValue,"noteContent:",this.noteContent)
		},
		methods:{
			// #ifdef APP-PLUS || MP-WEIXIN
			handTouchStart(e){
			  console.log("手机滑动起点",e)
			},
			handTouchEnd(e){
			  console.log("手机滑动终点",e)
			},
			// #endif
			// 获取当前笔记内容
			getNote(){
				return new Promise((resolve,reject)=>{
					// getOpenerEventChannel用于页面间通信的一个功能，主要用于父子页面间的通信。
					const eventChannel = this.getOpenerEventChannel();
					//相当于this.currentTime==''
					if(!this.currentTime){
						this.currentTime = parseTime(Date.now());
					}
					// 设置超时防止永久等待
					const timeoutId = setTimeout(() => {
					  reject(new Error('等待数据超时'));
					}, 3000); // 3秒超时
					// 如果为新增笔记
					if(this.noteStatus=='add'){
						clearTimeout(timeoutId);
						resolve();
					}
					// 如果为更新笔记
					// 注意不能用$on、$once等，因为在没有定义完成后，因此不能能接收到emit传过来的数据
					eventChannel.once('noteValue',res=>{
						clearTimeout(timeoutId);
						this.noteValue = res;
						this.noteTime = res.noteTime;
						this.noteName = res.noteName;
						this.noteContent = res.noteContent;
						this.notePic = res.notePic;
						// #ifdef MP-WEIXIN
						// 参数replacement若为函数,则第一个参数是匹配模式的字符串。接下来的参数是与模式中的子表达式匹配的字符串，可以有 0 个或多个这样的参数。接下来的参数是一个整数，声明了匹配在 string 中出现的位置。最后一个参数是 string 本身。
						// filter() 方法创建一个新的数组，新数组中的元素是通过检查指定数组中符合条件的所有元素。
						// ([^>]*)提取0或多个除>的内容
						this.noteContent = this.noteContent.replace(/<img([^>]*)>+/g,(match)=>{//match:匹配的字符串部分
							const attrs = match.replace(/<img|>/g,'')//将匹配的标签头尾先移除
												.split(/\s+/)//按找空格拆分  注：不能写成split('')→每个字符间分割
												// filter用于对数组进行过滤(不会改变原始数组)。filter(function(currentValue, index, arr))currentValue:必需,当前元素的值
												.filter(attr=>{
													// string.startsWith(searchString, position)searchString：必需，要查找的子字符串
													return !attr.startsWith('style=');// 清理重复style
												}).join(' ');//join()将数组转为一个字符串
							return `<img${attrs}>`;
						})
						// #endif
						resolve();
					});
				});
			},
			// 完成编辑提交并跳转
			async toNoteMsg(){
				let noteMsg = {
					id:0,//新增默认id为0
					noteName: this.noteName,
					currentTextContent:this.currentTextContent || this.noteContent,
					noteTime: parseTime(Date.now()),
					imgMsg:this.imgMsg || this.notePic
				};
				this.checkToIndex = true;
				/**
				 * 1.原内容有图片不删除，写入新内容后提交
				 * 2.原有图片删除，其他内容不改动，提交
				 * 3.原内容全不改动，提交
				*/
				// 判断是否为原有笔记内容的更新
				if(this.noteValue){
					noteMsg.id = this.noteValue.id;
					// 当原笔记中含有图片时
					if(this.notePic.length>0){
						if(/img/.test(noteMsg.currentTextContent)){
							// #ifdef H5
							// 获取原正文内容里的img信息遍历后以base64编码再存储(因图片插入位置不固定)
							noteMsg.imgMsg = await this.blobToBase64(noteMsg.currentTextContent)
							// #endif
							// #ifndef H5
							const getImgs = await this.getEditorImages();
							getImgs.forEach(getImg=>{
								let tempUrlMsg = {
									imgUrl:getImg,
									imgType:getImg.split('.')[1]
								};
								this.imgMsg.push(tempUrlMsg);
							});
							noteMsg.imgMsg = this.imgMsg;
							// #endif
						}else{
							//如果提交时内容无图片 → 原图片已删除
							noteMsg.imgMsg = [];//清空图片url
						}
					}
				}
				// #ifdef MP-WEIXIN
				// 小程序提交时会有多个p标签 
				const txc = noteMsg.currentTextContent.replace(/<p>|<\/p>/g,'');
				noteMsg.currentTextContent = `<p>${txc}</p>`;
				// #endif
				// 无输入
				// if(this.noteName == ''  && this.currentTextContent == '' ){
				// 	noteMsg = this.noteValue;
				// }
				// 判断原有笔记是否有图片 
				// uni.redirectTo({//关闭当前页面，跳转到应用内的某个页面。
				uni.reLaunch({//关闭所有页面，打开到应用内的某个页面。
					url:"/pages/index/index",
					success:res=>{
						setTimeout(()=>{
							// console.log("提交时msg：",noteMsg);
							uni.$emit('msg',noteMsg);//延迟触发，避免on还没注册定义
						},200);
					}
				})
			},
			// #ifndef H5
			// 新增通用图片获取方法
			getEditorImages() {
			    try{
					return new Promise((resolve) => {
						this.editorCtx.getContents({
							success:res=>{
								const getHtml = res.html;
								const imgRegex = /<img[^>]+src="([^">]+)"[^>]*>/g;
								const hasImgs = [];
								let match;
								/**
								 * 这个返回数组的第 0 个元素就是与表达式相匹配的文本。
								 * 第 1 个元素是与 regexp 的第一个子表达式相匹配的文本（如果存在）。
								 * 第 2 个元素是与 regexp 的第二个子表达式相匹配的文本，以此类推。
								 * 子表达式：通过括号来实现 如([^">]+)
								 * */
								while((match = imgRegex.exec(getHtml))!== null){
									if(match[1]){//match[1]为src
										hasImgs.push(match[1]);
									}
								}
								resolve(Array.from(new Set(hasImgs)));//去重
							}
						});
					});
				}catch(e){
					return [];
				}
			},
			// #endif
			readOnlyChange() {
				this.readOnly = !this.readOnly
			},
			onEditorReady(){
				// 获取editor 组件对应的 editorContext 实例
				uni.createSelectorQuery().in(this).select('#editor').context((res) => {
					this.editorCtx = res.context;
					if(this.noteValue != ''){//跳转到该页面时携带数据
						this.currentTextNum = this.removeHTMLTag(this.noteContent).length;
						this.editorCtx.setContents({
							html:this.noteContent
						})
					}
				}).exec();//执行所有的请求
			},
			// 获取文本内容
			onEditorInput(){
				this.editorCtx.getContents({
					success:res=>{
						this.currentTextContent = res.html;
						this.currentTextNum = this.removeHTMLTag(res.html).length;
						if(this.currentTextContent=='<p><br></p>' && !this.noteName){
							this.currentTextContent='';
						}
					}
				})
			},
			//失去焦点时触发获取当前内容
			// lostFocus(){
			// 	this.editorCtx.getContents({
			// 		success:res=>{
			// 			console.log("失去焦点时",res)
			// 		}
			// 	})
			// },
			//定义移除标签
			removeHTMLTag(str) {
				str = str.replace(/<\/?[^>]*>/g, ''); // 去除HTML tag
				str = str.replace(/[ | ]*\n/g, '\n'); // 去除行尾空白
				str = str.replace(/\n[\s| | ]*\r/g, '\n'); //去除多余空行
				// str = str.replace(/ /gi, ''); // 去掉空格
				const arrEntities = { lt: '<', gt: '>', nbsp: ' ', amp: '&', quot: '"' }; // 转义符换成普通字符
				str = str.replace(/&(lt|gt|nbsp|amp|quot);/gi, function(all, t) {
					return arrEntities[t];
				});
				return str;
			},
			// 返回上一页并保存
			async back(){
				let noteMsg = {
					id: this.noteValue.id || 0,
					noteName: this.noteName,
					currentTextContent:this.currentTextContent,
					noteTime: parseTime(Date.now()),
					imgMsg:this.imgMsg || this.notePic
				};
				this.checkToIndex = true;
				if(this.noteValue){
					// 修改原有笔记时获取其id
					noteMsg.id = this.noteValue.id;
					// 若正文内容无改动则返回原先内容
					if(!noteMsg.currentTextContent){
						noteMsg.currentTextContent=this.noteContent;
					}
					// 当原笔记中含有图片时
					if(this.notePic.length>0){
						if(/img/.test(noteMsg.currentTextContent)){
							// #ifdef H5
							// 获取原正文内容里的img信息
							// 获取原正文内容里的img信息遍历后以base64编码再存储(因图片插入位置不固定)
							noteMsg.imgMsg = await this.blobToBase64(noteMsg.currentTextContent);
							// #endif
							// #ifndef H5
							const getImgs = await this.getEditorImages();
							getImgs.forEach(getImg=>{
								let tempUrlMsg = {
									imgUrl:getImg,
									imgType:getImg.split('.')[1]
								};
								this.imgMsg.push(tempUrlMsg);
							});
							noteMsg.imgMsg = this.imgMsg;
							// #endif
						}else{
							//如果提交时内容无图片 → 原图片已删除
							noteMsg.imgMsg = [];//清空图片url
						}
					}
					// 当原笔记内容全部清空<p><br><\/p>
					if(!noteMsg.noteName && (!noteMsg.noteContent 
						|| noteMsg.currentTextContent == '<p><br></p>')){
						noteMsg.currentTextContent = '<p><br></p>';
					}
				}
				// console.log("返回时内容：",noteMsg,/^\s*$/.test(this.currentTextContent))
				// #ifdef MP-WEIXIN
				// 小程序返回时会有多出<p><br></p>(添加图片后出现的) 
				const txc = noteMsg.currentTextContent.replace(/<p>|<\/p>/g,'');
				noteMsg.currentTextContent = `<p>${txc}</p>`;
				// #endif
				// 无输入 /^\s*$/.test(this.currentTextContent) 
				// \s* 表示零个或多个空白字符（包括空格、制表符、换行符等）。
				if(this.noteName==this.noteValue.noteName && /^\s*$/.test(this.currentTextContent)){
					uni.$emit('msg',noteMsg);
					uni.navigateBack();
					return;
				}else if(this.checkToIndex  && !/^\s*$/.test(this.currentTextContent)){
					// 手势返回时存在更新内容
					uni.$emit('msg',noteMsg);
					uni.navigateBack();
				}else{
					uni.reLaunch({
						url:"/pages/index/index",
						success:res=>{
							setTimeout(()=>{
								uni.$emit('msg',noteMsg);
							},200);
						}
					});
				}
			},
			// 撤销
			undo() {
				this.editorCtx.undo();
			},
			// 恢复
			redo() {
				this.editorCtx.redo();
			},
			// 修改编辑器的对应样式
			format(e) {
				let {
					name,
					value
				} = e.target.dataset
				if (!name) return
				this.editorCtx.format(name, value);//设置格式
			},
			
			//获取对应样式的状态
			onStatusChange(e){
				const formats = e.detail;
				this.formats = formats;
				console.log("this.formats:",this.formats)
			},
			// 插入分割线
			insertDivider() {
				this.editorCtx.insertDivider({
					success: function() {
						console.log('insert divider success')
					}
				})
			},
			// 清空编辑器内容
			clear() {
				uni.showModal({
					title: '清空编辑器',
					content: '确定清空编辑器全部内容？',
					success: res => {
						if (res.confirm) {
							this.editorCtx.clear({
								success: function(res) {
									console.log("clear success")
								}
							})
						}
					}
				})
			},
			// 清除当前选区的样式
			removeFormat() {
				this.editorCtx.removeFormat()
			},
			// 覆盖当前选区，设置一段文本
			insertDate() {
				const date = new Date()
				const formatDate = `${date.getFullYear()}/${date.getMonth() + 1}/${date.getDate()}`
				this.editorCtx.insertText({
					text: formatDate
				})
			},
			// 插入图片
			insertImage() {
				uni.chooseImage({
					count: 1,
					success: res => {
						// #ifdef H5
						let currentFile = res.tempFiles;
						console.log("插入图片后res为",res)
						this.editorCtx.insertImage({
							alt: '图像',
							//不加上这串字符'data:image/jpeg;base64,'，在页面无法显示
							src: currentFile[0].path,//'data:image/jpeg;base64,' + base64Url
							extClass:'noteImg',
							success: function() {
								console.log('insert image success',res);
								uni.showToast({//避免添加图片成功但未写入编辑器中时点击提交
									title:'插入图片成功',
									mask:true,
									duration:800//默认1500,
								})
							}
						});
						// 原内容或新内容中无图片时插入
						if(this.notePic.length==0){
							this.toRequest(currentFile[0].path,currentFile[0].type)
							.then(val=>{
								this.imgMsg.push(val);
								// console.log("第一张图片",val,this.imgMsg)
							})
						}
						// #endif
						// #ifndef H5 || APP-PLUS
						uni.getFileSystemManager().saveFile({
							tempFilePath: res.tempFilePaths[0],
							success: saveRes => {
								this.editorCtx.insertImage({
									src: saveRes.savedFilePath,
									alt: '图像',
									extClass:'noteImg',
									success: function() {
										console.log('insert image success',res);
										uni.showToast({//避免添加图片成功但未写入编辑器中时点击提交
											title:'插入图片成功',
											mask:true,
											duration:800//默认1500,
										})
									}
								});
								this.imgMsg.push({
									imgUrl:saveRes.savedFilePath,
									imgType:''
								});
							}
						});
						// #endif
						
						// #ifdef APP-PLUS
						uni.saveFile({
							tempFilePath: res.tempFilePaths[0],
							success: saveRes => {
								this.editorCtx.insertImage({
									src: saveRes.savedFilePath,
									alt: '图像',
									extClass:'noteImg',
									success: function() {
										console.log('insert image success',res);
										uni.showToast({//避免添加图片成功但未写入编辑器中时点击提交
											title:'插入图片成功',
											mask:true,
											duration:800//默认1500,
										})
									}
								});
								this.imgMsg.push({
									imgUrl:saveRes.savedFilePath,
									imgType:''
								});
								// console.log("saveFile:",saveRes,res,this.imgMsg)
							}
						});
						// #endif
						this.hasImage = true;
					}
				})
			},
			// 点击了画板或者字号
			selectColorOrFontSize(name){
				// 取消高亮
				if(this.selectName == name){
					this.selectName = '';
					return;
				}
				this.selectName = name;
			},
			//选择颜色或者字号
			select(e,list){
				const {key,val} = e;
				for(let i=0;i<list.length;i++){
					let el = list[i];
					// 当前点击
					if(e.id==el.id){
						e.isChecked = !e.isChecked;
					}else{
						el.isChecked = false;
					}
				}
				// 设置样式
				this.editorCtx.format(key,val);
			},
			//打开颜色选择器
			moreColors(item){
				this.$refs.colorPicker.open();
			},
			errImg(e){
				console.log("图片获取错误!",e.detail)
			},
			// #ifdef H5
			// 临时地址转为base64(原内容中含有图片)
			blobToBase64(currentTextContent){
				return new Promise(async (resolve,reject)=>{
					let tempDiv = document.createElement('view');//临时元素但不插入DOM
					tempDiv.className = 'tempDiv';
					tempDiv.innerHTML = currentTextContent;
					let tempNotePic = tempDiv.querySelectorAll('.noteImg');
					let imgReqs=[]
					for(let i = 0;i<tempNotePic.length;i++){
						let tempUrl = tempNotePic[i].src;
						const imgMess = await this.toRequest(tempUrl);
						imgReqs.push(imgMess);
						// console.log("临时的url为",tempUrl,imgMess,imgReqs)
					}
					resolve(imgReqs);
				});
			},
			toRequest(tempUrl,type){
				return new Promise((resolve,reject)=>{
					if(!tempUrl){
						reject(new Error("URL为空"));
						return;
					}
					uni.request({
						url:tempUrl,//currentFile[0].path
						responseType: 'arraybuffer',
						success:response=>{
							let imgReqMes='';
							//临时地址转为base64后插入imgUrl数组中存储this.imgMsg
							imgReqMes={
								imgUrl:uni.arrayBufferToBase64(response.data),
								// imgArrayBuffer:response.data,//无法保存
								//判读content-type:jpg/jpeg/png
								imgType:type || response.header['content-type'],//currentFile[0].type
							};
							resolve(imgReqMes);
						},
						fail:error=>{
							console.log("获取请求失败！", error);
							reject(error);
						}
					})
				}).catch(err=>{
					console.log("无法编码",err);
				});
			}
			// #endif
		}
	}
</script>

<style lang="scss">
// page 相当于 body 节点
page{
	height: 100%;
}
.container{
	height: 100%;
	display: flex;
	flex-direction: column;
	.status_bar{
		height: var(--status-bar-height);
		border-bottom: 1px solid rgba(0, 0, 0, 0.5);
		width: 100%;
	}
	.navDiy{
		display: flex;
		justify-content: space-between;
		border-bottom: 1px solid rgba(0, 0, 0, 0.5);
		background-color: #f5f5f5;
		.navLeft,.navRight{
			margin: 20rpx 30rpx;
			display: flex;
			align-items: center;
			.uniicon{
				color:#999 !important;
			}
		}
		.navLeft{
			
		}
		.navRight{
			.uniicon{
				padding-left:30rpx;
			}
		}
	}
	.noteHeader{
		margin-top: 10rpx;
		.noteName{
			padding: 10rpx 30rpx;
			font-size: 30rpx;
			font-weight: bold;
			input{
				
			}
		}
		.noteOthers{
			display: flex;
			justify-content: space-between;
			align-items: center;
			padding: 10rpx 30rpx;
			.timeAndNum{
				display: flex;
				height: 40rpx;
				line-height: 40rpx;
				font-size: 20rpx;
				.time{
					
				}
				.num{
					
				}
			}
			.calendar{
				
			}
		}
	}
	.editorContent{
		flex: 1;
		overflow: hidden;
		padding:30rpx;
		editor{
			height: 100%;
			font-size: 28rpx;
			color:#666666;
		}
	}
	.toolbar{
		height: 80rpx;
		overflow-y: auto;
		border:1px solid lightskyblue;
		display: flex;
		justify-content: center;
		align-items: center;
		.iconfont{
			padding: 0 10rpx;
			&.activated{
				color: #0000ff;
			}
		}
	}
	.colorMenu,.fontSizeMenu{
		width: 678rpx;
		margin: 0 auto;
		border-radius: 10rpx;
		box-shadow: 4rpx 2rpx 12rpx rgba(0, 0, 0, 0.2);
		display: flex;
		flex-direction: column;
		justify-content: space-between;
		padding: 10rpx 0;
		.selectColor,.selectFontSize{
			display: flex;
			flex-direction: column;
			justify-content: space-between;
			margin:0 20rpx;
			.title{
				font-weight: bold;
				padding: 10rpx 0;
			}
		}
	}
	.colorMenu{
		.selectColor{
			.list{
				display: flex;
				justify-content: space-between;
				.color{
					width: 40rpx;
					height: 40rpx;
					border-radius: 10rpx;
					margin: 10rpx 0;
					// display: inline-block;
				}
			}
		}
	}
	.fontSizeMenu{
		.selectFontSize{
			.scroll{
				white-space: nowrap;
				overflow-x: hidden;
				// deep主要针对H5(隐藏滚动条)
				// /deep/ ::-webkit-scrollbar{
				// 	width: 4px !important;
				// 	height: 1px !important;
				// 	overflow: auto !important;
				// 	background: transparent !important;//transparent透明
				// 	-webkit-appearance: auto !important;
				// 	display: block;
				// }
				// .list{
				// 	white-space: nowrap;
				// 	overflow-x: hidden;
					.fontSize{
						background-color: rgba(107, 90, 247, 0.2);
						text-align: center;
						width: 60rpx;
						height: 60rpx;
						line-height: 60rpx;
						// margin-right: 20rpx;
						border-radius: 10rpx;
						display: inline-block;
						margin: 10rpx;
						&.toSize{
							font-weight: bold;
						}
					}
				// }
			}
		}
	}
}
</style>
