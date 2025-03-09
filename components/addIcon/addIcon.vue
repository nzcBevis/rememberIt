<template>
	<view>
		<!-- <view class="iconfont icon-jia" @click="toNote(navIndex)"> -->
		<view class="iconfont icon-jia" @tap="toNote(navIndex)">
		</view>
		<!-- #ifdef APP-PLUS -->
		<view 
			v-if="appShowModal" 
			class="custom-modal" 
			:style="`bottom: ${windowHeight - keyboardHeight + 20}rpx`">
			<view class="modal-content">
				<text class="title">待办</text>
				<input
					class="content"
					v-model="appToMsg"
					placeholder="待办的内容"
					auto-blur focus></input>
				<view class="button-group">
				  <button @click="handleConfirm">确定</button>
				  <button @click="appShowModal = false">取消</button>
				</view>
			</view>
		</view>
		<!-- #endif -->
	</view>
</template>

<script>
	export default {
		name:"addIcon",
		props:["navIndex"],
		data() {
			return {
				appShowModal:false,
				keyboardHeight: 0,
				windowHeight:0,
				appToMsg:''
			};
		},
		mounted() {
			// #ifdef APP-PLUS
			this.getwindowHeight();
			// #endif
		},
		methods:{
			toNote(e){console.log("点击加号时navIndex为",e)
				// 0代表笔记页 1代表待办页
				if(e == 0){
					uni.navigateTo({
						url:'/pages/note/note?status=add'
					})
				}else if(e == 1){
					// #ifdef APP-PLUS
					// 监听键盘高度变化
					uni.onKeyboardHeightChange(res => {
						this.keyboardHeight = res.height;
					});
					this.appShowModal = true;
					this.appToMsg = '';
					// #endif
					// #ifndef APP-PLUS
					uni.showModal({
						title:"待办",
						editable:true,
						placeholderText:"待办的内容",
						success:res=>{
							// 按下确认
							if(res.confirm){
								const toDoMsg = {
										time:Date.now(),
										msg:res.content
									}
								uni.$emit('msg',toDoMsg);
							}else{
								// 按下取消
							}
						}
					})
					// #endif
				}else{
					// 显示消息提示框
					uni.showToast({
						icon:'fail',
						// 文本长度有限制
						title:`读取错误！导航索引值为${e}`,
						mask:true  //防止触摸穿透
					})
				}
			},
			// #ifdef APP-PLUS
			closeModal() {
			      this.appShowModal = false;
			    },
			handleConfirm() {
			  // 处理确认逻辑
			  this.appShowModal = false;
			  const toDoMsg = {
			  		time:Date.now(),
			  		msg:this.appToMsg
			  	}
			  uni.$emit('msg',toDoMsg);
			},
			getwindowHeight(){
				uni.onWindowResize(res=>{
					this.windowHeigh = res.windowHeight;//获取当前窗口高度
				})
			}
			// #endif
		}
	}
</script>

<style lang="scss">
// @import '../../static/font/iconfont.css';
.icon-jia{
	position: fixed;
	right: 50rpx;
	bottom: 50rpx;
	font-size: 100rpx;
	color:skyblue
}
/* #ifdef APP-PLUS */
.custom-modal{
	position: fixed;
	top: 0;
	left: 0;
	right: 0;
	bottom: 0;
	background-color: rgba(0, 0, 0, 0.5);
	display: flex;
	justify-content: center;
	align-items: center;
	z-index: 9999;
}
.modal-content{
	background: white;
	padding: 20rpx;
	border-radius: 20rpx;
	width: 80%;
	text-align: center;
	z-index: 10000;
	.title,.content{
		font-size: 18px;
		padding: 10rpx 0; 
	}
  .content{
	width: 100%;
	border: 1px solid rgba(0, 0, 0, 0.2);
	border-radius: 15rpx;
	background-color: rgba(150, 150, 150, 0.2);
	margin: 20rpx 0;
  }
}
.button-group{
	display: flex;
	justify-content: center;
	align-items: center;
	button{
		font-size: 16px;
		width: 35%;
	}
}
.custom-modal{
	animation: showIt 0.5s;
}
@keyframes showIt{
	from{ 
		opacity: 0; 
	}
	to{ 
		opacity: 1;
	}
}
/* #endif */
</style>