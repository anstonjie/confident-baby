<template>
	<view>
		<view style="text-align: center">AI数字人生成工具</view>

		<view style="margin-top: 20rpx">
			<label style="color: blue; font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif">请上传图片</label>
			<uni-file-picker
				file-mediatype="image"
				mode="grid"
				file-extname="jpeg,jpg,png"
				:limit="1"
				@progress="progress1"
				@success="success1"
				@fail="fail1"
				@delete="delete1"
				@select="select1"
			></uni-file-picker>
		</view>

		<view style="margin-top: 40rpx">
			<label style="color: blue; font-family: 'Segoe UI', Tahoma, Geneva, Verdana, sans-serif">请上传声音文件</label>
			<uni-file-picker
				file-mediatype="audio"
				mode="grid"
				file-extname="mp3,wav"
				:limit="1"
				@progress="progress"
				@success="success"
				@fail="fail"
				@delete="delete2"
				@select="select"
			/>
			<view>
				<view class="page-body">
					<view class="page-section page-section-gap" style="text-align: left">
						<audio
							:src="currentOne.src"
							:poster="currentOne.poster"
							:name="currentOne.name"
							:author="currentOne.author"
							:action="audioAction"
							:controls="currentOne.controlsFlag"
						></audio>
					</view>
				</view>
			</view>
		</view>

		<br />
		<br />
		<view>
			<label>设置其他参数</label>
			<br />
			Pose style:
			<slider value="0" min="0" max="46" @change="poseStypeChange" show-value activeColor="#FFCC33" backgroundColor="#000000" block-color="#8A6DE9" />
			<br />
			face model resolution:
			<view>
				<radio-group @change="sizeChange">
					<label class="radio">
						<radio value="256" checked="true" />
						256
					</label>
					<label class="radio">
						<radio value="512" />
						512
					</label>
				</radio-group>
			</view>
			<br />
			preprocess:
			<view>
				<radio-group @change="preprocessChange">
					<label class="radio">
						<radio value="crop" checked="true" />
						crop
					</label>
					<label class="radio">
						<radio value="resize" />
						resize
					</label>
					<label class="radio">
						<radio value="full" />
						full
					</label>
					<label class="radio">
						<radio value="extcrop" />
						extcrop
					</label>
					<label class="radio">
						<radio value="extfull" />
						extfull
					</label>
				</radio-group>
			</view>
			<br />

			<view>
				<checkbox value="false" :checked="stillChecked" @click="stillgo">
					<view class="uni-title uni-common-mt">Still Mode (fewer head motion, works with preprocess `full`)</view>
				</checkbox>
			</view>
			<br />
			<checkbox value="gfpgan" :checked="gfpganChecked" @click="gfpgango"><view class="uni-title uni-common-mt">GFPGAN as Face enhancer</view></checkbox>
			<br />
			<br />
			batch size in generation:
			<slider value="2" min="2" step="2" max="10" @change="batchSizeChange" show-value activeColor="#FFCC33" backgroundColor="#000000" block-color="#8A6DE9" />
			<br />

			<button type="primary" style="background-color: #ff4500" @click="generate">生成</button>
		</view>

		<view>
			<video id="myVideo" :src="videoSrc" enable-danmu danmu-btn controls></video>
		</view>
	</view>
</template>

<script>
export default {
	data() {
		return {
			currentOne: {
				controlsFlag: false,
				poster: 'https://qiniu-web-assets.dcloud.net.cn/unidoc/zh/music-a.png',
				name: '请上传声音文件',
				author: '暂无',
				src: ''
			},
			audioAction: {
				method: 'pause'
			},
			stillChecked: false,
			stillValue: false,
			gfpganChecked: false,
			gfpganValue: '',

			source_image: '',
			driven_audio: '',
			pose_style: 0,
			size: 256,
			still: '',
			preprocess: 'crop',
			enhancer: 'gfpgan',
			batch_size: 2,

			videoSrc: ''
		};
	},

	onReady: function (res) {
		// #ifndef MP-ALIPAY
		this.videoContext = uni.createVideoContext('myVideo');
		// #endif
	},
	onShow() {},
	onLoad() {},
	methods: {
		async generate() {
			console.log('this.source_image', this.source_image);
			await uni.request({
				url: 'http://127.0.0.1:8000/generate',
				timeout: 240000,
				sslVerify: false,
				method: 'POST',
				data: {
					/* seaName:'https://mp-1fecd741-6a1b-4984-8a41-b0526ec15758.cdn.bspapp.com/cloudstorage/7d127ece-8b09-480d-92b9-5b8e36e12c64.jpg',
						drivenAudio2: 'https://mp-1fecd741-6a1b-4984-8a41-b0526ec15758.cdn.bspapp.com/cloudstorage/a25e7de9-197f-4e21-9dae-1976e1cfd624.wav',
					source_image:this.source_image */

					driven_audio: this.driven_audio,
					source_image: this.source_image,
					pose_style: this.pose_style,
					size: this.size,
					result_dir: '',
					still: this.stillValue,
					preprocess: this.preprocess,
					enhancer: this.enhancer,
					batch_size: this.batch_size
				},
				success: (res) => {
					console.log('res.data', res.data);
					this.videoSrc = res.data.result_path;
					console.log('this.videoSrc 》》》》', this.videoSrc);
					//this.videoContext.play()
				}
			});
		},

		gfpgango() {
			console.log('gfpganChecked ', this.gfpganChecked);

			this.gfpganChecked = !this.gfpganChecked;
			console.log('gfpganChecked 333 ', this.gfpganChecked);
			if (this.gfpganChecked) {
				this.enhancer = 'gfpgan';
			} else {
				this.enhancer = 'RestoreFormer';
			}
			console.log('enhancer ', this.enhancer);
		},

		/**
			 * @param {Object} e
			 * pose_style:'',
				size:'',
				preprocess:'',
			 * still
			 * 
			 */
		poseStypeChange(e) {
			console.log(' styleChange value 发生变化：' + e.detail.value);
			this.pose_style = e.detail.value;
			console.log('this.pose_style：' + this.pose_style);
		},

		sizeChange(e) {
			console.log('change value 发生变化：' + e.detail.value);
			this.size = e.detail.value;
			console.log('this.size：' + this.size);
		},

		preprocessChange(e) {
			console.log('change value 发生变化：' + e.detail.value);
			this.preprocess = e.detail.value;
			console.log('this.preprocess：' + this.preprocess);
		},
		stillgo() {
			console.log('stillChecked ', this.stillChecked);

			this.stillChecked = !this.stillChecked;
			console.log('stillChecked 333 ', this.stillChecked);
			if (this.stillChecked) {
				this.stillValue = true;
			} else {
				this.stillValue = false;
			}
			console.log('stillValue ', this.stillValue);
		},

		batchSizeChange(e) {
			console.log('batchSizeChange value 发生变化：' + e.detail.value);
			this.batch_size = e.detail.value;
			console.log('this.batch_size：' + this.batch_size);
		},

		// 获取上传状态
		select(e) {
			console.log('选择音频文件：', e);
		},
		// 获取上传进度
		progress(e) {
			console.log('上传音频进度：', e);
		},

		select1(e) {
			console.log('选择图片文件：', e);
		},
		// 获取上传进度
		progress1(e) {
			console.log('上传图片进度：', e);
		},

		// 上传图片成功
		success1(e) {
			console.log('上传成功', e);
			this.source_image = e.tempFiles[0].url;
			console.log('source_image >>>>> ', e.tempFiles[0].url);
		},

		delete1(e) {
			console.log('删除图片', e);
			this.source_image = '';
			console.log('source_image >>>>> ', this.source_image);
		},

		// 上传音频成功
		success(e) {
			console.log('上传成功', e);
			this.currentOne.controlsFlag = true;
			this.currentOne.src = e.tempFiles[0].url;
			this.currentOne.name = '已有';
			this.currentOne.author = '请播放';
			this.driven_audio = this.currentOne.src;
			console.log('audioSrc>>>>>', this.currentOne.src);

			console.log('driven_audio', this.driven_audio);
		},

		delete2(e) {
			console.log('删除图片', e);
			this.currentOne.controlsFlag = false;
			this.currentOne.src = '';
			this.currentOne.name = '请上传声音文件';
			this.currentOne.author = '暂无';
			this.driven_audio = '';
			console.log('audioSrc>>>>>', this.currentOne.src);
			console.log('driven_audio', this.driven_audio);
		},

		// 上传失败
		fail(e) {
			console.log('上传音频失败：', e);
		},
		// 上传失败
		fail1(e) {
			console.log('上传图片失败：', e);
		}
	}
};
</script>

<style>
/* #ifndef APP-NVUE */
page {
	display: flex;
	flex-direction: column;
	box-sizing: border-box;
	background-color: #fff;
	min-height: 100%;
	height: auto;
}
view {
	font-size: 14px;
	line-height: inherit;
}
.example-body {
	/* #ifndef APP-NVUE */
	display: flex;
	/* #endif */
	flex-direction: row;
	flex-wrap: wrap;
	justify-content: center;
	padding: 0;
	font-size: 14px;
	background-color: #ffffff;
}
/* #endif */

.section-box {
	display: flex;
	flex-direction: row;
	align-items: center;
	padding: 20rpx;
}
.decoration {
	width: 4px;
	height: 12px;
	border-radius: 10px;
	background-color: #2979ff;
}
.section-text {
	color: #333;
	margin-left: 15rpx;
}

/* #ifdef APP-NVUE */
.warp {
	background-color: #fff;
}
/* #endif */

.example-body {
	flex-direction: column;
	padding: 15px;
	background-color: #ffffff;
}

.image {
	width: 50rpx;
	height: 50rpx;
}

.big-number {
	font-size: 50rpx;
	font-weight: 700;
	font-stretch: condensed;
	font-style: oblique;
}

.text {
	text-align: center;
	font-size: 26rpx;
	margin-top: 10rpx;
}

.example-body {
	/* #ifndef APP-NVUE */
	display: block;
	/* #endif */
}

.grid-item-box {
	flex: 1;
	/* #ifndef APP-NVUE */
	display: flex;
	/* #endif */
	flex-direction: column;
	align-items: center;
	justify-content: center;
	padding: 15px 0;
}

.banner-image {
	width: 750rpx;
	height: 400rpx;
}

.swiper-box {
	height: 400rpx;
}

.search-icons {
	padding: 16rpx;
}

.search-container-bar {
	/* #ifndef APP-NVUE */
	display: flex;
	/* #endif */
	flex-direction: row;
	justify-content: center;
	align-items: center;
	position: fixed;
	left: 0;
	right: 0;
	z-index: 10;
	background-color: #fff;
}

/* #ifndef APP-NVUE || VUE3*/
::v-deep
	/* #endif */
	.uni-searchbar__box {
	border-width: 0;
}

/* #ifndef APP-NVUE || VUE3 */
::v-deep
	/* #endif */
	.uni-input-placeholder {
	font-size: 28rpx;
}
</style>
