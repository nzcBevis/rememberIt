<template>
	<!-- <view> -->
		<!-- 状态栏(主要针对手机端) -->
		<view class="home">
			<view class="status_bar"></view>
				<!-- 头部导航部分 -->
				<view class="header">
					<view class="hdcontent" 
					:class="navIndex==0?'active':''" 
					@tap="clickNav(0)">
						记下
					</view>
					<view class="hdcontent" 
					:class="navIndex==1?'active':''" 
					@tap="clickNav(1)">
						待办
					</view>
				</view>
				<!-- 内容部分 -->
				<view class="content">
					<!-- 搜索框 -->
					<view class="searchPart">
						<uni-search-bar
							@confirm="search"
							@input="input"
							@blur="blur"
							radius="20"
							v-model="keyWord"
							:placeholder="navIndex==0?'记下':'待办'" />
					</view>
					<!-- 笔记部分 -->
					<swiper class="swiper"
						@change="changeSwiper"
						:current="swiperCurrent"
						:current-item-id="currentItemId"
						@animationfinish="aniFinSwiper"
						v-if="blobReady">
						<swiper-item class="lists" item-id="noteList">
							<scroll-view class="note" v-if="showNoteList.length" scroll-y>
								<view class="noteMsg" v-for="(item,index) in showNoteList" :key="index">
									<view @tap="toNote(item)">
										<view class="noteName">{{item.noteName}}</view>
										<view class="noteContent" v-if="item.notePic.length>0" v-html="item.noteContent"></view>
										<view class="noteContent" v-else v-html="item.noteContent"></view>
									</view>
									<view class="other">
										<view class="noteTime">{{item.noteTime}}</view>
										<image 
										src="/static/images/ellipsis.png" 
										mode="aspectFill"
										@tap="deleteIt(index)"></image><!--@tap.stop阻止事件继续传播-->
									</view>
								</view>
							</scroll-view>
							<!-- 无内容 -->
							<view class="nothing" v-else>
								<!-- <image src="" mode=""></image> -->
								<view class="text">╮(╯▽╰)╭搜索不到任何内容噢！</view>
							</view>
						</swiper-item>
							<!-- 待办部分 -->
						<swiper-item class="lists" item-id="toDoList" >
							<scroll-view class="toDo" v-if="toDoList.length" scroll-y>
								<view class="toDoMsg" v-for="(item,index) in toDoList" :key="index">
									<checkbox-group>
										<view class="toDoTime">{{item.toDoTime}}</view>
										<view class="msgContent" v-for="(iList,iIndex) in item.list" :key="iIndex">
											<label>
												<checkbox 
												:value="iList.toDoContent" 
												:checked="iList.checked"
												@tap="changeState(index,iIndex,iList.checked)" />
											</label>
											<view class="toDoContent" :class="toDoList[index].list[iIndex].checked==true?'changeColor':''">{{iList.toDoContent}}</view>
										</view>
									</checkbox-group>	
								</view>
							</scroll-view>
							<!-- 无内容 -->
							<view item-id="none" class="nothing" v-else>
								<!-- <image src="" mode=""></image> -->
								<view class="text">╮(╯▽╰)╭搜索不到任何内容噢！</view>
							</view>
						</swiper-item>
					</swiper>
					<!-- 加载中 -->
					<view class="loading" v-else>
						<!-- <image src="" mode=""></image> -->
						<view class="text">加载内容当中....别催别催!</view>
					</view>
				</view>
				<!-- 右下角添加选项 -->
				<view>
					<addIcon class="addicon" :navIndex="navIndex"></addIcon>
					<!-- <image src="" mode=""></image> -->
				</view>
			</view>
	<!-- </view> -->
</template>

<script>
	import {parseTime} from '../../utils/tool.js'
	export default {
		data() {
			return {
				navIndex:0,//默认显示笔记页面
				systemInfo:{},//当前设备信心
				keyWord:'',//搜索框输入内容
				swiperCurrent:0,//默认为笔记内容的滑块index
				currentItemId:'',//当前滑动项名称
				// previousItemId:'',//上一个滑动项名称
				blobReady:false,//默认临时地址没改动
				tempDiv:'',
				currentMaxId:0,//记录当前最大的id
				itemList:['删除'],
				defaultNote:[
					{
						id:1,
						noteName: '欢迎使用',
						noteContent: `第一次用uniapp做的项目`,
						noteTime: '2025-01-06 20:29',
						notePic:[]
					}
				],
				defaultToDo:[
					{
						toDoTime: '2025-01-12',
						list:[
							{
								id:1,
								toDoContent: `这里是待办`,
								checked:false
							}
						]
					}
				],
				noteList:[],
				toDoList:[]
			}
		},
		onReady(){
			// #ifdef H5
			//url无效时刷新
			this.$nextTick(()=>{
				// console.log("onready的",this.noteList);
				this.checkBlob();
			});
			// #endif
			// #ifndef H5
			this.blobReady = true;
			// #endif
			console.log("首页挂载完毕!");
		},
		onLoad() {
			this.getTheSystemInfo();
			//获取数据
			this.getData();
			// #ifdef H5
			this.tempDiv = document.createElement('view');//临时元素但不插入DOM
			this.tempDiv.className = 'tempDiv';
			// #endif
			// 新增或更新内容后跳转
			// this.noteOrToDo();
			// #ifdef H5
			uni.$on('msg',this.noteOrToDo);
			// #endif
			// #ifndef H5
			uni.$on('msg',this.noteOrToDo);
			// #endif
			console.log("onLoad");
		},
		onShow() {
			console.log("onShow");
		},
		onHide() {
			// uni.$off('msg',this.noteOrToDo);
			console.log('onhide');
		},
		onUnload() {
			uni.$off('msg',this.noteOrToDo);
			// #ifdef H5
			this.noteList.forEach(note=>{
				note.notePic.forEach(pic=>{
					URL.revokeObjectURL(pic.imgUrl);
					console.log("触发了移除");
				})
			})
			// #endif
			console.log("unload!");
		},
		
		computed:{
			showNoteList(){
				if(this.keyWord != ''){
					//filter是过滤函数:去除了不包含关键字的情况
					return this.noteList.filter(p=>{
						// let noteContent = p.noteContent || p.hasImgContent;
						let noteContent = p.noteContent;
						// 避免图片的url干扰搜索
						if(/<img([^>]*)>/g.test(noteContent)){
							noteContent =  noteContent.replace(/<img([^>]*)>/g,'');
						}
						//返回过滤后的数组
						if(noteContent || p.noteName){
							return noteContent.indexOf(this.keyWord) != -1 || p.noteName.indexOf(this.keyWord) != -1;
						}
						return [];
					});
				}else{
					return this.noteList;
				}
			},
			
			// showToDoList(){
			// 	//判断是否有输入内容
			// 	if(this.keyWord != ''){
			// 		let temp = [];
			// 		let msg = {
			// 			toDoTime : '',
			// 			list:[]
			// 		};
			// 		let pushId = 0;
			// 		this.toDoList.forEach(p=>{
			// 			p.list.filter(i=>{
			// 				if(i.toDoContent.indexOf(this.keyWord) != -1 && i.id != pushId){
			// 					//匹配的待办对应的时间且不是同一个待办笔记
			// 					msg.toDoTime = p.toDoTime;
			// 					//将匹配的插入到空数组中
			// 					msg.list.push(i);
			// 					temp.push(msg);
			// 					pushId = i.id;
			// 					return this.toDoList = temp;
			// 				}console.log("this.toDoList:",this.toDoList,i)
			// 				return this.toDoList = temp;
			// 			})
			// 		})
			// 	}else{
			// 		return this.toDoList = uni.getStorageSync('toDoList');
			// 	}
			// }
			
		},
		methods: {
			clickNav(e){
				this.keyWord = '';
				this.navIndex = e;
				this.swiperCurrent = e;
				this.currentItemId = e == 0?'noteList':'toDoList';
			},
			toNote(e){
				uni.navigateTo({
					url:"/pages/note/note",
					success:res=>{
						// 触发一个事件
						res.eventChannel.emit('noteValue',e);
					}
				})
			},
			deleteIt(e){
				uni.showActionSheet({
					itemList:this.itemList,
					success:res=>{
						if(res.tapIndex==0){
							//选中删除   splice(从指定索引开始,删除几个)
							let delList = this.noteList.splice(e,1);
							// 同步更新本地存储
							uni.setStorageSync('noteList',this.noteList);
						}
					},
					fail:err=>{
						console.warn(err);
					}
				})
				
			},
			// 获取当前勾选的索引值和修改状态
			changeState(index,iIndex,iListChecked){
				this.toDoList[index].list[iIndex].checked = !iListChecked;
				let currentChecked = this.toDoList[index].list[iIndex].checked;
				uni.setStorageSync('toDoList',this.toDoList);
			},
			// 获取本地存储数据或者默认数据
			getData(){
				// 检查是否有本地数据
				let getNoteList = uni.getStorageSync('noteList') || this.defaultNote;
				let getToDoList = uni.getStorageSync('toDoList') || this.defaultToDo;
				// 无本地数据，第一次进入时
				if(getNoteList.length == this.defaultNote.length &&
					getToDoList.length == this.defaultToDo.length){
					//加载时获取默认数组 
					this.noteList = this.defaultNote;
					this.toDoList = this.defaultToDo;
				}
				// 如果刷新时本地有数据 
				else if(getNoteList.length > this.defaultNote.length ||
					getToDoList.length > this.defaultToDo.length){
					this.noteList = getNoteList;
					this.toDoList = getToDoList;
					//console.log("新增或更新了",getNoteList)
				}else{
					uni.showToast({
						title:'读取错误!',
						icon:error,
						mask:true
					})
				}
				// 获取当前最大的id
				getNoteList.forEach(note=>{
					if(this.currentMaxId<note.id){
						this.currentMaxId = note.id;
					}
				})
			},
		
			// 判断并获取新增或更新的笔记或者待办
			noteOrToDo(res){
				// uni.$once('msg',res=>{
					console.log("$on的res",res);
					if(this.navIndex==0){
						//笔记页面
						let newNoteContent = {
							//接收新增或者更新的笔记内容
							id:res.id,
							noteName: res.noteName,
							noteContent: res.currentTextContent || res.noteContent || '',
							noteTime: res.noteTime,
							notePic:res.imgMsg || res.notePic || [],
							// hasImgContent:'',//默认所有笔记内容无图片
						};
						// 判断是否有图片
						if(newNoteContent.notePic.length>0){
							// newNoteContent.hasImgContent = res.currentTextContent;
							// 遍历后将base64转为临时编码给img标签
							/**
							 * 	.*和.*?模式不同
								1、．*：．*为贪婪匹配模式。
								2、．*?：．*?为最小匹配模式。
								.*和.*?匹配条件不同
								1、．*：．*的匹配条件为单个字符。
								2、．*?：．*?的匹配条件为多个字符组成的 字符串 。
								\1:匹配与开头相同的引号（通过反向引用第一个分组）
							*/
							// 设置显示图片的大小
							newNoteContent.noteContent = newNoteContent.noteContent
															.replace(/<img/g,
															'<img style="max-width:20vw;'+
															'max-height:20vh;display:inline-block;'+
															'vertical-align:bottom;"');
							// 								.replace(/data-local\s*=\s*(["']).*?\1/,'')
						}
						// 判断新增还是更新
						if(newNoteContent.id != 0){
							// 直接更新
							const oldDataSize = JSON.stringify(this.noteList).length; // 单位：字节
							let index = this.noteList.findIndex(item=>{
								return item.id == res.id;
							});
							if(index>=0){
								// 如果清空内容后直接返回或者提交时该空白笔记不加入数组中
								if(newNoteContent.noteName == '' && 
									/^<p><br><\/p>$/.test(newNoteContent.noteContent)){
									// console.log("当前修改的元素index是",index)
									this.noteList.splice(index,1);
									return;
								}
								this.noteList.splice(index,1);//把旧的删除
								this.noteList.unshift(newNoteContent);
								// 如果提交的所有内容为空或将所有内容清空后提交
								if(!res.noteName && (res.currentTextContent==''||res.currentTextContent=='<p><br></p>')){
									this.noteList.splice(0,1);
								}
								// #ifdef H5
								try{
									const newDataSize = JSON.stringify(this.noteList).length;
									uni.setStorageSync('noteList',this.noteList);
									if (newDataSize > 5 * 1024 * 1024) {
										console.warn('新数据长度超过 5MB 限制',newDataSize,'旧数据长度为',oldDataSize);
									    uni.showModal({
											content:`新增后超出本地缓存5MB限制，当前缓存为${oldDataSize}`,
											showCancel:false,
											success:res=>{
												this.blobReady = false;
												if(res.confirm){
													this.noteList=uni.getStorageSync('noteList');
													this.checkBlob();
												}
											}
										})
									}else{
										console.log("新数据长度:",newDataSize,"旧数据长度:",oldDataSize);
									}
								}catch(e){
									console.warn("失败了",e)
								}
								// #endif
								// #ifndef H5
								uni.setStorageSync('noteList',this.noteList);
								// #endif
								return;
							}
						}
						//新增笔记，重新赋予id值后直接插入数组前面
						newNoteContent.id = this.currentMaxId + 1;
						this.currentMaxId = newNoteContent.id;
						this.noteList.unshift(newNoteContent);
						// 如果提交的所有内容为空或将所有内容清空后提交
						if(!res.noteName && (res.currentTextContent==''||res.currentTextContent=='<p><br></p>')){
							this.noteList.splice(0,1);
						};
						uni.setStorage({
							key:'noteList',
							data:this.noteList
						});
						// console.log("新增后笔记list",this.noteList);
					}else{
						// 如果提交的所有内容为空
						if(!res.msg)return;
						// 待办页面
						let toDoMsg = {
							//接收待办的所有内容
							// 待办新增时间
							toDoTime: parseTime(res.time,'{y}-{m}-{d}'),//例，2025-01-14 17:47:16
							list:[
							{
								id:0, //默认id为0
								toDoContent: res.msg,//待办内容
								checked:false//默认未完成
							}]
						};
						if(this.toDoList != []){
							// 遍历原数组获取时间
							let el,index,newDay=false;
							for(let i = 0 ;i<this.toDoList.length;i++){
								el = this.toDoList[i];
								index = i;
								//判断是否为同一天内新增的
								if(toDoMsg.toDoTime == el.toDoTime){
									//在同一日期下的原数组前方插入新元素并按顺序给当前元素的id赋值
									toDoMsg.list[0].id = el.list.length;
									el.list.unshift(...toDoMsg.list);
									uni.setStorage({
										key : 'toDoList',
										data : this.toDoList
									});
									return;
								}
							};
							//遍历后均不是当天新增的
							this.toDoList.unshift(toDoMsg);
						}else{
							//原待办页面内容为空
							this.toDoList.unshift(toDoMsg);
						}
						uni.setStorage({
							key : 'toDoList',
							data : this.toDoList
						});
					}
				// })
			},
			getTheSystemInfo(){
				// 获取系统信息的异步接口。
				uni.getSystemInfo({
					success:res=>{
						this.systemInfo = res;
						console.log("设备为:",this.systemInfo.deviceType);//pc
					},
					fail:err=>{
						console.warn("err:",err)
					}
				});
			},
			search(e){
				console.log("search返回了",e);
			},
			input(e){
				//判断是否有输入内容
				if(e != ''){
					let temp = [];
					let pushId = 0;
					this.toDoList.forEach(p=>{
						let msg = {
							toDoTime : '',
							list:[]
						};
						p.list.filter(i=>{
							if(i.toDoContent.indexOf(e) != -1){
								msg.toDoTime = p.toDoTime;
								//匹配的待办对应的时间相同且不是同一个待办笔记
								if(i.id != pushId){
									//将匹配的插入到空数组中
									msg.list.push(i);
								}else{
									msg.list.push(i);
								}
							}
						});
						// 当数组不为空时插入到临时数组中
						if(msg.list.length>0){
							temp.push(msg);
						}
						return this.toDoList = temp;
					});
				}else{
					this.toDoList = uni.getStorageSync('toDoList') || this.defaultToDo;
				}
			},
			blur(e){
				console.log("失去焦点时，",e);
			},
			changeSwiper(e){
				// this.previousItemId = e.detail.currentItemId;
				this.currentItemId = e.detail.currentItemId;
				// console.log("changeSwiper:",e.detail,this.navIndex,this.currentItemId);
			},
			aniFinSwiper(e){
				this.navIndex = e.detail.currentItemId == 'noteList' ? 0 : 1;
				// this.previousItemId = e.detail.currentItemId;
				this.currentItemId = e.detail.currentItemId;
				// console.log("aniFinSwiper:",e.detail,this.navIndex,this.currentItemId)
			},
			// #ifdef H5
			checkBlob(){
				// 收集图片加载状态
				const loadReady = [];
				this.noteList.forEach(noteMsg=>{
					if(/<ima?ge?/g.test(noteMsg.noteContent)){
						this.tempDiv.innerHTML = noteMsg.noteContent;
						// 浏览器会自动将image改回img
						let temp = this.tempDiv.querySelectorAll('.noteImg');
						temp.forEach((img,index)=>{
							const  promise = new Promise((resolve,reject)=>{
								// 当img成功加载完成后
								const loadHandler =()=>{
									img.removeEventListener('error',errorHandler);
									resolve();
									// URL.revokeObjectURL(tempUrl);//如果清除会导致页面对应图片无法正常显示
								};
								//当img的临时地址报错时调用
								img.addEventListener('load',loadHandler);
								const errorHandler = ()=>{
									this.blobReady = false;
									// this.errImg(img,noteMsg,index).finally(resolve);
									this.errImg(noteMsg,index).finally(resolve);
								};
								img.addEventListener('error',errorHandler);
								// 直接调用虽然能正确替换URL并且显示图片，但会使失效url的报错提示放到最后
								// img.addEventListener('error',this.errImg(img,noteMsg,index));
							});
							loadReady.push(promise);
						});
						
						
						noteMsg.noteContent = noteMsg.noteContent
												.replace(/<img/g,'<img style="max-width:20vw;'+
												'max-height:20vh;display:inline-block;'+
												'vertical-align:bottom;" ');
												// .replace(/data-local\s*=\s*(["']).*?\1/,'')
					}
				});
				// 全部图片加载完成后再执行
				Promise.all(loadReady).then(()=>{
					this.blobReady = true;
					// uni.setStorageSync('noteList',this.noteList);
				}).catch(e=>{
					this.blobReady = true;
					uni.showToast({
						title:'部分图片有错误',
						icon:'error'
					})
				});
			},
			
			
			// 原临时地址缓存被清除后	
					
			async errImg(noteMsg,i){
				try{
					this.tempDiv.innerHTML = noteMsg.noteContent;
					let temp = this.tempDiv.querySelectorAll('.noteImg');
					let tempUrl='';
					URL.revokeObjectURL(temp[i].src);//释放旧的blob
					temp[i].src='';
					// Base64 字符串解码为二进制字符串
					let binary = atob(noteMsg.notePic[i].imgUrl);
					// 创建一个与二进制字符串长度匹配的 ArrayBuffer
					let imgArrayBuffer = new ArrayBuffer(binary.length);
					// 创建一个 Uint8Array 视图来操作 imgArrayBuffer(必须)
					// ArrayBuffer 本身是“只读容器”，不能直接修改数据。通过 Uint8Array 视图，可以逐个字节读写二进制数据。
					let bytes = new Uint8Array(imgArrayBuffer);
					// 遍历二进制字符串，将每个字符的 ASCII 码写入 bytes(只读 → 读写)
					for(let i = 0;i<binary.length;i++){
						bytes[i] = binary.charCodeAt(i);
					}
					//'data:image/jpeg;base64,'+noteMsg.notePic[i].imgUrl
					//因图片存储时为base64格式，转为临时地址加载提交效率
					// 直接使用原ArrayBuffer更高效
					let newBlob = new Blob([imgArrayBuffer],{type:noteMsg.notePic[i].imgType})
					tempUrl = URL.createObjectURL(newBlob);
					// 存储时顺序与显示的需一致
					temp[i].src = tempUrl;
					// ‌setAttribute‌是DOM（文档对象模型）中的一个内置方法，用于在HTML元素上设置属性tempNoteImgs[i]
					temp[i].setAttribute('data-local',tempUrl);
					noteMsg.noteContent = this.tempDiv.innerHTML;
					return Promise.resolve();
				}catch(err){
					console.warn("errImg有错误信息：",err);
					return Promise.reject();
				}
			}
					
			// async errImg(e,noteMsg,i){
			// 	try{
			// 		this.tempDiv.innerHTML = noteMsg.hasImgContent;
			// 		let temp = this.tempDiv.querySelectorAll('.noteImg');
			// 		let findNum = Array.from(temp).findIndex(node=>node.src===e.src);
			// 		let tempUrl='';
			// 		if(findNum !== -1){
			// 			URL.revokeObjectURL(temp[i].src);//释放旧的blob
			// 			URL.revokeObjectURL(e.src);
			// 			temp[i].src='';
			// 			e.src='';
			// 			// Base64 字符串解码为二进制字符串
			// 			let binary = atob(noteMsg.notePic[i].imgUrl);
			// 			// 创建一个与二进制字符串长度匹配的 ArrayBuffer
			// 			let imgArrayBuffer = new ArrayBuffer(binary.length);
			// 			// 创建一个 Uint8Array 视图来操作 imgArrayBuffer(必须)
			// 			// ArrayBuffer 本身是“只读容器”，不能直接修改数据。通过 Uint8Array 视图，可以逐个字节读写二进制数据。
			// 			let bytes = new Uint8Array(imgArrayBuffer);
			// 			// 遍历二进制字符串，将每个字符的 ASCII 码写入 bytes(只读 → 读写)
			// 			for(let i = 0;i<binary.length;i++){
			// 				bytes[i] = binary.charCodeAt(i);
			// 			}
			// 			//因图片存储时为base64格式，转为临时地址加载提交效率
			// 			// 直接使用原ArrayBuffer更高效
			// 			let newBlob = new Blob([imgArrayBuffer],{type:noteMsg.notePic[i].imgType})
			// 			tempUrl = URL.createObjectURL(newBlob);
			// 			// this.testUrl = tempUrl;//'data:image/jpeg;base64,'+noteMsg.notePic[i].imgUrl
			// 			// 存储时顺序与显示的需一致
			// 			// temp[findNum].src = tempUrl;
			// 			e.src=tempUrl;//由于传入了一个e，若e.src不重新赋值，则会触发error监听
			// 			// ‌setAttribute‌是DOM（文档对象模型）中的一个内置方法，用于在HTML元素上设置属性tempNoteImgs[i]
			// 			temp[findNum].setAttribute('data-local',tempUrl);
			// 		}
			// 		noteMsg.noteContent = this.tempDiv.innerHTML;
			// 		noteMsg.hasImgContent = this.tempDiv.innerHTML;
			// 		return Promise.resolve();
			// 	}catch(err){
			// 		console.warn("errImg有错误信息：",err);
			// 		return Promise.reject();
			// 	}
			// }
			// #endif
		}
	}
</script>

<style lang="scss" scoped>
	*{
		border: 0;
		padding: 0;
		margin: 0;
	}
	page{
		height: 100%;
	}
	.home{
		box-sizing: border-box;
		height: 100%;
		display: flex;
		flex-direction: column;
		.status_bar{
			/* #ifdef APP-PLUS || H5 */
			height: var(--status-bar-height);
			/* #endif */
			background: rgba(135, 206, 235, 0.2);
			width: 100%;
		}
		.header{
			background: rgba(135, 206, 235, 0.2);
			display: flex;
			font-size: 40rpx;
			text-align: center;
			.hdcontent{
				padding: 20rpx;
				padding-bottom: 15rpx;
				&.active{
					font-weight: bolder;
					position: relative;
					&::before{
						position: absolute;
						content:'';
						left: 0;
						bottom: 0;
						width: 100%;
						border-bottom:  5px solid skyblue;
						border-radius: 32rpx;
					}
				}
			}
		}
		.content{
			background: #E8F0F2;
			flex: 1;
			.searchPart{
				padding: 0 10rpx;
			}
			.swiper{
				margin: 30rpx;
				/* #ifdef H5 || APP-PLUS */
				height: calc(100vh - (129px + var(--status-bar-height))); 
				/* #endif */
				/* #ifndef APP-PLUS || H5 */
				height: calc(100vh - 129px); //43px(导航)+56px(搜索)+30px(上下内边框)
				/* #endif */
				box-shadow: 0 0 30rpx 2rpx rgba(0, 0, 0, 0.4); /* 盒子阴影增加凸出感 */
				background-color: rgba(255, 255, 255, 0.6); /* 半透明背景 */
				border-radius:30rpx;
				.lists{
					.note,.toDo{
						padding: 20rpx 30rpx;
						width: auto;
						height: calc(100vh - 149px);
						// deep主要针对H5(隐藏滚动条)
						/deep/ ::-webkit-scrollbar{
							width: 4px !important;
							height: 1px !important;
							overflow: auto !important;
							background: transparent !important;//transparent透明
							-webkit-appearance: auto !important;
							display: block;
						}
					}
					.note{
						.noteMsg{
							border:1px solid black;
							border-radius:10rpx;
							padding: 10rpx 16rpx;
							margin: 20rpx 10rpx;
							.noteName{
								font-weight:bold;
								margin-bottom: 10rpx;
							}
							.noteContent{
								color:#3e3e3e;
								// 两行或以上隐藏溢出内容
								text-overflow: -o-ellipsis-lastline;
								overflow: hidden;//溢出内容隐藏
								text-overflow: ellipsis;//文本溢出部分用省略号表示
								display: -webkit-box;//特别显示模式
								-webkit-line-clamp: 2;//行数
								line-clamp: 2;
								-webkit-box-orient: vertical;//盒子中内容竖直排列
							}
							.other{
								display: flex;
								justify-content: space-between;
								align-content: center;
								margin-top: 16rpx;
								.noteTime{
									border:1px solid yellowgreen;
									border-radius:10rpx;
									color:#787878;
								}
								image{
									width: 40rpx;
									height: 40rpx;
								}
							}
						}
					}
					.toDo{
						// backdrop-filter: blur(20px); /* uniapp中无效? */
						// filter: blur(10px); /* 创建毛玻璃效果,但后代会继承该属性 */
						.toDoMsg{
							padding: 10rpx;
							.toDoTime{
								border:1px solid yellowgreen;
								margin-bottom: 16rpx;
								font-weight:bold;
								border-radius:10rpx;
							}
							.msgContent{
								display: flex;
								padding: 10rpx 0;
								.toDoContent{
									// flex:0.5;//缩放至剩余空间长度的0.5倍
									flex:1;//占用剩余空间长度
									border:1px solid black;
									border-radius:10rpx;
									// 两行或以上隐藏溢出内容
									text-overflow: -o-ellipsis-lastline;
									overflow: hidden;//溢出内容隐藏
									text-overflow: ellipsis;//文本溢出部分用省略号表示
									display: -webkit-box;//特别显示模式
									-webkit-line-clamp: 1;//行数
									line-clamp: 1;
									-webkit-box-orient: vertical;//盒子中内容竖直排列
								}
							}
							.changeColor{
								color: #808080;
								text-decoration: line-through;
							}
						}
					}
					.nothing{
						text-align: center;
						padding:30rpx;
					}
				}
			}
			.loading{
				text-align: center;
				padding:30rpx;
			}
		}
	}
</style>
