<template>
	<view class="content">
		<button class="mini-btn" type="primary" plain="true" @tap="chooseImage">选择图片</button>
		<view class="picture-list">
			<view v-for="(item,index) in upload_picture_list" :key="index" class="sunsin_picture_item">
				<image class="pic" :data-src="item" :src="item" mode="aspectFill" :data-idx="index"
					@tap="previewImg(index)"></image>
			</view>
		</view>

	</view>
</template>

<script>
	function getLocalImgDataPromise(localId) {
		return new Promise((resolve, reject) => {
			wx.getLocalImgData({
				localId: localId,
				success: function(res) {
					resolve(res)
				},
				fail: function(err) {
					reject(err)
				}
			});
		})
	}

	function base64ToBlob(dataurl) {
		let arr = dataurl.split(',');
		let mime = arr[0].match(/:(.*?);/)[1];
		let bstr = atob(arr[1]);
		let n = bstr.length;
		let u8arr = new Uint8Array(n);
		while (n--) {
			u8arr[n] = bstr.charCodeAt(n);
		}
		return new Blob([u8arr], {
			type: mime
		});
	}
	import wx from '@/common/js/jweixin-module/index.js'
	export default {
		data() {
			return {
				upload_picture_list: []
			}
		},
		onLoad() {
			//获取微信公众号的配置
			uni.request({
				url: 'http://192.168.120.25:4000/sig',
				data: {
					url: location.href.split('#')[0]
				},
				success: res => {
					var s = res.data;
					wx.config({
						debug: false, // 开启调试模式,调用的所有api的返回值会在客户端alert出来，若要查看传入的参数，可以在pc端打开，参数信息会通过log打出，仅在pc端时才会打印。
						appId: s.data.appId, // 必填，公众号的唯一标识
						timestamp: s.data.timestamp, // 必填，生成签名的时间戳
						nonceStr: s.data.noncestr, // 必填，生成签名的随机串
						signature: s.data.signature.toLowerCase(), // 必填，签名，见附录1
						jsApiList: [
							'chooseImage',
							'previewImage',
							"getLocalImgData",
							'uploadImage',
						]
					});
					if (typeof callback === 'function') {
						callback(res.data);
					}
					wx.ready(() => {
						wx.checkJsApi({
							// 需要检测的JS接口列表
							jsApiList: ['chooseImage'],
							success: function(res) {
								console.log("res: " + res);
							}
						});
					})
				},
				fail: err => {
					console.log('request fail', err);
				}
			});

		},
		methods: {
			previewImg(index) {
				let that = this
				uni.previewImage({
					current: index,
					urls: that.upload_picture_list
				});
			},
			chooseImage() {
				let _this = this
				wx.chooseImage({
					success: async (res) => {
						let localIds = res.localIds;
						for (let i = 0, len = localIds.length; i < len; i++) {
							let imgDataRes = await getLocalImgDataPromise(localIds[i])
							const localData = imgDataRes.localData
							let imageBase64 = '';
							if (localData.indexOf('data:image') == 0) {
								//苹果的直接赋值，默认生成'data:image/jpeg;base64,'的头部拼接
								imageBase64 = localData;
							} else {
								//此处是安卓中的唯一得坑！在拼接前需要对localData进行换行符的全局替换
								//此时一个正常的base64图片路径就完美生成赋值到img的src中了
								imageBase64 = 'data:image/jpeg;base64,' + localData.replace(
									/\n/g, '');
							}
							// let blob = base64ToBlob(imageBase64)
							_this.upload_picture_list.push(imageBase64);
						}
					}
				});
			}
		}
	}
</script>

<style>
	.content {
		display: flex;
		flex-direction: column;
		align-items: center;
		justify-content: center;
		padding: 20px 0;
	}

	/* 宽度 */
	.pic {
		width: 159upx !important;
		height: 159upx !important;
		line-height: 159upx !important;
		margin: 10px;
	}

	/* 循环列表样式 */
	.picture-list {
		/* width: 100%; */
		padding: 20rpx;
		display: flex;
		flex-wrap: wrap;
		text-align: center;
		/* justify-content: space-between; */
		background-color: #FFFFFF;
	}

	.title {
		font-size: 36rpx;
		color: #8f8f94;
	}
</style>
