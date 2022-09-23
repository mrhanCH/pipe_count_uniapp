<template>
	<view class="content">
		<view class="button-item">
			<u-button type="success" icon="camera" :plain="true" text="拍照" class="button-area" @click="takePhoto">
			</u-button>
		</view>
		<view class="button-item">
			<u-button type="primary" icon="photo" :plain="true" text="相册" class="button-area" @click="takePhotos">
			</u-button>
		</view>
		<u-divider text="使用说明" textColor="black"></u-divider>
		<u--text type="info" lineHeight="22" size="14" class="details"
			text="点击拍照或相册，选择合适的钢管范围，可使用内置裁剪工具对照片进行裁剪，请保持需要检测的钢管主体位于图像中央部位，尽量不要有其余的干扰因素（如树木、杂草遮挡，主体缺失，逆光环境等）；"></u--text>
		<u--text type="info" lineHeight="22" size="14" class="details" text="裁剪后点击“开始识别”，等待片刻系统会自动钢管数量和标注框等参数；">
		</u--text>
		<u--text type="info" lineHeight="22" size="14" class="details"
			text="在新打开的识别结果页面中，页面下方将显示当前图片自动识别到的钢管数量和人工增加标注总数量，蓝色标注框为AI识别结果，绿色标注框为人工增加结果，点击图片可补点/去点，补点显示为绿色标注框，去点则将绿框删除或删除蓝色标注框；">
		</u--text>
		<u--text type="info" lineHeight="22" size="14" class="details"
			text="全部补点/去点操作完成后，请点击提交数据，图片和点位数据将回传到服务器做为新数据集优化更高的识别效果。"></u--text>
	</view>
</template>

<script>
	export default {
		data() {
			return {

			}
		},
		onLoad() {

		},
		methods: {
			takePhoto() {
				uni.chooseImage({
					count: 1,
					sizeType: ['original'],
					sourceType: ['camera', 'album'],
					success: function(res) {
						var img = res.tempFilePaths[0];
						console.log(img);
						uni.navigateTo({
							url: "/pages/cropp/cropp?url=" + img
						});
					}
				});
			},
			takePhotos() {
				uni.chooseImage({
					count: 1, //最多上传图片数量
					sizeType: ['original'], //图片质量，original原图
					sourceType: ['album'], //从相册选择
					success: function(res) {
						var img = res.tempFilePaths[0];
						console.log(img);
						uni.navigateTo({
							url: "/pages/cropp/cropp?url=" + img
						});
					}
				});
			}
		}
	}
</script>

<style>
	.details {
		text-indent: 20px;
	}

	.button-item {
		width: 100%;
		padding: 20px 0;
	}

	.button-area {
		width: 50%;
		display: flex;
		justify-content: center;
	}
</style>
