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
					url: 'http://127.0.0.1:8000/finds/',
					filePath: that.imgSrc,
					name: "image",
					formData: {},
					method: 'POST',
					success: function(e) {
						var i = JSON.parse(e.data);
						// console.log(i)
						uni.hideLoading();
						var n = i.data;
						if (200 === i.code) {
							getApp().globalData.picResult = n.picResult;
							var newSize = that.imgInfo(n)
							uni.navigateTo({
								url: "/pages/result/result?resultCount=" + n.resultCount + "&resultUrl=" + n.picurl + "&picWidth=" + n.width + "&picHeight=" + n.height + "&canvasWidth=" + parseInt(newSize.nWidth) + "&canvasHeight=" + parseInt(newSize.nHeight) + "&name=" + n.name
							})
						} else {
							uni.showToast({
								title: i.msg,
								icon: "none",
								duration: 2e3
							});
						}

					},
					fail: function(t) {
						uni.hideLoading();
						uni.showToast({
							title: '识别失败，请查看服务是否启动',
							icon: "none",
							duration: 2e3
						});
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
			},
			imgInfo(n) {
				var sys = uni.$u.sys(),
					ratio = n.width / sys.windowWidth,
					cH = ratio > 1 ? n.height / ratio : n.height,
					cW = ratio > 1 ? sys.windowWidth : n.width;
				return {
					'nWidth': cW,
					'nHeight': cH
				}
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
