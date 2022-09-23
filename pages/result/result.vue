<template>
	<view>
		<view class="content">
			<view class="image">
				<canvas canvas-id="myCanvas"
					:style="{'border': '1px solid','width': cWidth + 'px','height': cHeight + 'px'}" @click="clickImg" id="canvasId" />
				<!-- API识别结果点位 -->
				<view class="draw-square-api" @click="clickImg" style="position:absolute;" :style="{'left':(Number(item.middle.xaxis)+Number(wlift))+'px','top':(Number(item.middle.yaxis)+Number(wtop))+'px','width':bboxW+'px','padding-bottom':bboxW+'px'}"  v-for="(item, index) in apiList" :key="item.id"  v-if="!item.hasDelete">
					<view class="draw-arc" style="position:absolute;" :style="{'left':bboxW/2+'px','top':bboxW/2+'px'}"></view>
				</view>
				<!-- 添加/删除新点位 -->
				<view @click="deleteAdd" class="draw-square" :data-index="index" style="position:absolute;" :style="{'left':(Number(item.leftTop.xaxis)+Number(wlift))+'px','top':(Number(item.leftTop.yaxis)+Number(wtop))+'px','width':bboxW+'px','padding-bottom':bboxW+'px'}" v-for="(item, index) in addList" :key="item.id" ></view>
			</view>
		</view>
		<view class="nums">
			<view class="nums_item">总根数 {{allCount}} </view>
			<view class="nums_item">识别数 {{recognitionCount}}</view>
		</view>
		<view class="footer">
			<u-button text="保存结果" type="success" class="btn" @click="saveList"></u-button>
		</view>
	</view>
</template>

<script>
	var i = Object.assign || function(t) {
		for (var i = 1; i < arguments.length; i++) {
			var e = arguments[i];
			for (var a in e) Object.prototype.hasOwnProperty.call(e, a) && (t[a] = e[a]);
		}
		return t;
	}
	export default {
		data() {
			return {
				cWidth: '',
				cHeight: '',
				pWidth: '',
				pHeight: '',
				allCount: 0,
				recognitionCount: 0,
				apiList: [],
				addList: [],
				ctx: '',
				bboxW: 20,
				wlift: 0,
				wtop: 0,
				name: ''
			}
		},
		onLoad(e) {
			this.cWidth = e.canvasWidth
			this.cHeight = e.canvasHeight
			this.pWidth = e.picWidth
			this.pHeight = e.picHeight
			this.allCount = e.resultCount
			this.recognitionCount = e.resultCount
			this.name = e.name
			this.createImg(e)
			// console.log(e);
		},
		onReady() {
			var that = this;
			uni.createSelectorQuery().select("#canvasId").boundingClientRect(function(i) {
				that.wlift = i.left
				that.wtop = i.top
			}).exec();
		},
		methods: {
			clickImg(t) {
				var that = this;
				if ("click" === t.type) {
					//点击位置-页面尺寸xy-10 = 方框左上角x,y
					var clickPosition = {
							x: t.touches[0].pageX - that.wlift,
							y: t.touches[0].pageY - that.wtop
						},
						clickList = that.apiList.concat(that.addList).filter(function(item) {
							var i = item.middle || {};
							//返回点击坐标在标注框内部的标注框集合
							//circler和其他坐标均为适配屏幕大小后的坐标
							return that.getDistances(clickPosition.x, clickPosition.y, i.xaxis, i.yaxis) <=
								item.circler;
						}),
						d = 0;
					console.log(JSON.stringify(clickList));
					if (clickList.length > 0) {
						//点击位置在api标注框内-重写apiList对象-前端删除蓝框
						for (var l = 0; l < that.apiList.length; l++) {
							if (that.apiList[l].middle.xaxis === clickList[0].middle.xaxis && that.apiList[l].middle.yaxis === clickList[0].middle.yaxis) {
								that.apiList[l].hasDelete = !that.apiList[l].hasDelete, 
								d = l
							}
						}
					} else {
						//点击位置在api标注框外-新增addList对象-前端渲染绿框
						var addRectMiddle = {
							x: Number(clickPosition.x) - Number(that.bboxW / 2),
							y: Number(clickPosition.y) - Number(that.bboxW / 2)
						};
						that.addList.push({
							leftTop: {
								xaxis: addRectMiddle.x,
								yaxis: addRectMiddle.y
							},
							middle: {
								xaxis: clickPosition.x,
								yaxis: clickPosition.y
							},
							rightDown: {
								xaxis: Number(addRectMiddle.x) + Number(that.bboxW),
								yaxis: Number(addRectMiddle.y) + Number(that.bboxW)
							},
							circler: parseInt((Number(clickPosition.x) - Number(addRectMiddle.x)) *
								that.cWidth / that.pWidth),
							isadd: 1
						});
					}
					console.log("ADD方框xy", JSON.stringify(that.addList))
					var allPort = that.apiList.concat(that.addList).filter(function(port) {
						return !port.hasDelete;
					})
					that.allCount = allPort.length
				}
			},
			createImg(e) {
				var that = this;
				const ctx = uni.createCanvasContext('myCanvas')
				that.ctx = ctx
				var img = new Image();
				img.src = e.resultUrl;
				img.onload = function() {
					ctx.clearRect(0, 0, parseInt(e.canvasWidth), parseInt(e.canvasHeight))
					ctx.drawImage(e.resultUrl, 0, 0, parseInt(e.canvasWidth), parseInt(e.canvasHeight))
					ctx.save()
					// 遍历画出标注框 isadd=0是AI结果 =1是人工添加结果
					var apiList = that.getPicResult(getApp().globalData.picResult, e.picWidth, e.canvasWidth),
						bbox_w = apiList[0].rightDown.xaxis - apiList[0].leftTop.xaxis;
					that.bboxW = bbox_w;
					that.apiList = apiList;
					ctx.draw(true)
				};
			},
			//保存结果
			saveList() {
				var that = this;
				uni.showModal({
				  title: '提示',
				  content: '确定保存结果？',
				  success(res) {
					if (res.confirm) {
					  uni.showLoading({
						title: "加载中..."
					  });
					  var allList = that.apiList.concat(that.addList).filter(function (port) {
						return !port.hasDelete;
					  })
					  var data = {
						addPort: that.restorePicResult(that.addList, that.pWidth, that.cWidth),
						aiPort: that.restorePicResult(that.apiList, that.pWidth, that.cWidth),
						allPort: that.restorePicResult(allList, that.pWidth, that.cWidth),
						name: that.name,
						pWidth: that.pWidth,
						pHeight: that.pHeight
					  };
					  console.log("POST data", data);
					  uni.request({
					      url: 'http://127.0.0.1:8000/save/', 
					      data: data,
						  method: 'POST',
					      success: (res) => {
					          // console.log(res.data);
							  var i = res.data;
							  if(200 === i.code){
							    uni.hideLoading()
							    uni.showToast({
							      title: '保存成功',
							      icon: 'success',
							      duration: 2000
							    })
							    setTimeout(function () {
							      uni.reLaunch({
							        url: '../index/index'
							      })
							    }, 2100)
							  }else{
							    uni.showToast({
							      title: i.msg,
							      icon: 'error',
							      duration: 2000
							    })
							    uni.hideLoading()
							  }
					      }
					  });
					}
				  }
				})
			  },
			//根据屏幕大小获取适配后坐标
			//参数 API结果 图片原宽 canvas宽
			getPicResult(t, i, e) {
				var that = this;
				return t.map(function(t) {
					return that.pointScall(t, i, e);
				});
			},
			//还原原始图片宽高坐标
			//参数 结果 图片原宽 canvas宽
			restorePicResult(t, i, e) {
			    var a = this;
			    return t.map(function (t) {
			      return a.pointRestore(t, i, e);
			    });
			},
			//标注框坐标适配屏幕大小
			//参数 API结果 图片原宽 canvas宽
			pointScall(t, e, a) {
				return i({}, t, {
					circler: parseInt((Number(t.middle.xaxis) - Number(t.leftTop.xaxis)) * a / e),
					leftTop: {
						//2375*340/3008 x坐标*裁剪图片宽/原始图片宽
						xaxis: parseInt(Number(t.leftTop.xaxis) * a / e),
						yaxis: parseInt(Number(t.leftTop.yaxis) * a / e)
					},
					middle: {
						xaxis: parseInt(Number(t.middle.xaxis) * a / e),
						yaxis: parseInt(Number(t.middle.yaxis) * a / e)
					},
					rightDown: {
						xaxis: parseInt(Number(t.rightDown.xaxis) * a / e),
						yaxis: parseInt(Number(t.rightDown.yaxis) * a / e)
					},
					isadd: 0
				});
			},
			//标注框坐标还原原始图像大小
			pointRestore: function (t, e, a) {
			    return i({}, t, {
			      leftTop: {
			        //268.4507978723404*3008/340 x坐标*原始图片宽/裁剪图片宽
			        xaxis: parseInt(Number(t.leftTop.xaxis) * e / a),
			        yaxis: parseInt(Number(t.leftTop.yaxis) * e / a)
			      },
			      middle: {
			        xaxis: parseInt(Number(t.middle.xaxis) * e / a),
			        yaxis: parseInt(Number(t.middle.yaxis) * e / a)
			      },
			      rightDown: {
			        xaxis: parseInt(Number(t.rightDown.xaxis) * e / a),
			        yaxis: parseInt(Number(t.rightDown.yaxis) * e / a)
			      },
			      hasDelete: t.hasDelete ? '1' : '0',
			      circler: 1
			    });
			},
			// 计算两点距离
			getDistances(t, i, e, a) {
				//abs 减
				var n = Math.abs(e - t),
					s = Math.abs(a - i);
				//pow n s的2次方
				return Math.sqrt(Math.pow(n, 2) + Math.pow(s, 2));
			},
			//删除绿色方框
			deleteAdd(t){
				var n = t.currentTarget.dataset.index;
				this.addList.splice(n, 1);
				this.allCount = this.allCount - 1
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

	.nums {
		display: flex;
		position: absolute;
		bottom: 50px;
		width: 100%;
	}

	.nums .nums_item {
		width: 50%;
		text-align: center;
		height: 35px;
		line-height: 35px;
		color: #545454;
		font-weight: bold;
	}
	
	/* 画绿色正方形 */
	.draw-square {
	  height: 0;
	  border: 1px solid #59e013;
	  background-color: #59e013 ;
	  opacity: 0.4;
	}
	
	/* 篮框 */
	.draw-square-api {
	  height: 0;
	  border: 1px solid #1119FC;
	  background-color: #1119FC ;
	  opacity: 0.5;
	  transform: translate(-50%,-50%);
	  display: flex;
	  align-items: center;
	}
	.draw-square-api .draw-arc {
	  width: 7px;
	  height: 7px;
	  border-radius: 50%;
	  background: #ff0000;
	  transform: translate(-50%,-50%);
	  display: flex;
	  align-items: center;
	  opacity: 1;
	}
</style>
