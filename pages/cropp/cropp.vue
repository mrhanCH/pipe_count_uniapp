<template>
	<view>
		<view class="content">
			<view class="image">
				<u--image :src="imgSrc" mode="aspectFit"></u--image>
			</view>
		</view>
		<view class="footer">
			<u-button text="裁剪图片" type="primary" class="btn" @click="croppImg"></u-button>
			<u-button text="开始识别" type="success" class="btn" @click="updateImg"></u-button>
		</view>
		<ksp-cropper mode="free" :url="imgSrc" @cancel="oncancel" @ok="onok" v-if="state==1"></ksp-cropper>
	</view>
</template>

<script>
	export default {
		data() {
			return {
				imgSrc: '',
				state: 0
			}
		},
		onLoad(e) {
			this.imgSrc = e.url;
			// uni.getImageInfo({
			// 	src:e.url,
			// 	success(res) {
			// 		console.log(res);
			// 	}
			// })
		},
		methods: {
			croppImg() {
				this.state = 1
			},
			updateImg: function() {
				var that = this;
				uni.showLoading({
					title: "识别中，请稍等"
				});
				uni.uploadFile({
					url: 'api.findNums',
					filePath: that.imgSrc,
					name: "image",
					formData: {},
					method: 'POST',
					success: function(e) {
						var i = JSON.parse(e.data);
						console.log(i)
						uni.hideLoading();
						// var n = i.data;
						// 200 === i.code ? (t.globalData.picRecognizeds = n.picRecognizeds, wx
						// 	.navigateTo({
						// 		url: "../operationResult/operationResult?recognitionCount=" + n
						// 			.recognitionCount + "&recognitionId=" + n.recognitionId +
						// 			"&resultUrl=" + that.data.src + "&width=" + n.width +
						// 			"&height=" + n.height
						// 	})) : wx.showToast({
						// 	title: i.msg,
						// 	icon: "none",
						// 	duration: 2e3
						// });
					},
					fail: function(t) {
						uni.hideLoading();
					}
				});
			},
			onok(ev) {
				if (ev.path != '') {
					this.state = 0;
					this.imgSrc = ev.path;
				}
			},
			oncancel() {
				this.state = 0
			}
		}
	}
</script>

<style>
	.image {
		padding-top: 20px;
		display: flex;
		justify-content: center;
	}
</style>
