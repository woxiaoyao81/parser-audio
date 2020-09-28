<!-- 修改说明
      Author：woxiaoyao
 Description: 
 1、修复APP上只能播放一个音频文件的错误，将this.setSrc(this.src)移到_buttonTap()方法中
 2、修复APP播放一个音频文件时自动暂停上一个音频文件，增加onPause事件函数，这个对微信小程序无效。
 3、修复微信小程序音频播完后再播放时间条不更新的问题，就是将条件编译MP-ALIPAY改为MP-WEIXIN
 4、修复小程序多个音频自动播放问题，播放下一首时自动暂停上一首，是通过vuex实现
 5、界面进行调整
 -->
<template>
	<view class="container">
		<view v-if="controls" class="contain">

			<view class="poster" :style="'background-image:url('+poster+')'">
				<view class="button" @tap="_buttonTap">
					<view :class="playing?'pause':'play'" />
				</view>
			</view>
			<view class="info">
				<view class="name"><text>北京九思自然教育出品</text></view>
				<view class="slide_time">
					<slider class="slider" activeColor="#585959" block-size="12" handle-size="12" :disabled="error" :value="value"
					 @changing="_seeking" @change="_seeked" />
					<view class="time">{{time||'00:00:00'}}</view>
				</view>
			</view>

		</view>
	</view>
</template>

<script>
	/**
	 * audio 扩展包
	 * @tutorial https://github.com/jin-yufeng/Parser
	 * @author JinYufeng
	 */
	const app = getApp();
	export default {
		data() {
			return {
				error: false,
				playing: false,
				time: '00:00',
				value: 0
			}
		},
		props: {
			author: String,
			autoplay: null,
			controls: null,
			loop: null,
			name: String,
			poster: String,
			src: String
		},
		watch: {
			src(src) {
				this.setSrc(src);
			}
		},
		// #ifdef MP-WEIXIN
		beforeDestroy() {
			clearInterval(this._timer);
		},
		// #endif
		mounted() {
			this._ctx = uni.createInnerAudioContext();
			this._ctx.onError((err) => {
				this.error = true;
				this.$emit('error', err);
			})
			// #ifdef MP-WEIXIN
			this._timer = setInterval(() => {
				if (this._ctx.paused) return;
				// #endif
				// #ifndef MP-WEIXIN
				this._ctx.onTimeUpdate(() => {
					// #endif
					var time = this._ctx.currentTime,
						min = parseInt(time / 60),
						sec = Math.ceil(time % 60);
					this.time = (min > 9 ? min : '0' + min) + ':' + (sec > 9 ? sec : '0' + sec);
					if (!this.lastTime) this.value = time / this._ctx.duration * 100; // 不在拖动状态下
					// #ifndef MP-WEIXIN
				})
				// #endif
				// #ifdef MP-WEIXIN
			}, 1000);
			// #endif
			this._ctx.onEnded(() => {
				//停止时若存有音乐则弹出
				if (!this.$u.test.isEmpty(this.vuex_audio)) {
					this.vuex_audio.pop();
				}
				this.playing = false;
			});
			this._ctx.onPause(() => {
				//暂停时若存有音乐则弹出
				if (!this.$u.test.isEmpty(this.vuex_audio)) {
					this.vuex_audio.pop();
				}
				this.playing = false;
			});
			this._ctx.onPlay(() => {
				//保存当前播放音乐，当暂停、播放完或播放下一首时使用。
				this.vuex_audio.push(this._ctx);
			});
		},
		onPageShow() {
			if (this.playing && this._ctx.paused)
				this._ctx.play();
		},
		methods: {
			// 设置源
			setSrc(src) {
				this._ctx.autoplay = Boolean(this.autoplay);
				this._ctx.loop = Boolean(this.loop);
				this._ctx.src = src;
				if (this.autoplay && !this.playing)
					this.playing = true;
			},
			// 播放
			play() {
				this._ctx.play();
				this.playing = true;
				this.$emit('play');
			},
			// 暂停
			pause() {
				this._ctx.pause();
				this.playing = false;
				this.$emit('pause');
			},
			// 移动进度条
			seek(sec) {
				this._ctx.seek(sec);
			},
			// 内部方法
			_buttonTap() {
				//播放前检查是否有播放音乐，若有则暂停
				if (!this.$u.test.isEmpty(this.vuex_audio)) {
					this.tempctr = this.vuex_audio.pop();
					this.tempctr.pause();
				}
				this.setSrc(this.src);
				if (this.playing) this.pause();
				else this.play();
			},
			_seeking(e) {
				if (e.timeStamp - this.lastTime < 200) return;
				var time = Math.round(e.detail.value / 100 * this._ctx.duration),
					min = parseInt(time / 60),
					sec = time % 60;
				this.time = (min > 9 ? min : '0' + min) + ':' + (sec > 9 ? sec : '0' + sec);
				this.lastTime = e.timeStamp;
			},
			_seeked(e) {
				this.seek(e.detail.value / 100 * this._ctx.duration);
				this.lastTime = void 0;
			}
		}
	}
</script>

<style>
	/* #ifdef MP-WEIXIN */
	/* :host是定义#shadow-root的样式，这里我增加了100%宽度 */
	:host {
		display: inline-block;
	    width: 100%;
	}
	/* #endif */

	.container {
		/* #ifdef MP-WEIXIN */
		display: flex;
		/* #endif */
		/* #ifndef MP-WEIXIN */
		display: inline-flex;
		/* #endif */
		width: 100%;
		height: 70px;
		justify-content: center;
	}

	.contain {
		display: inherit;
		width: 100%;
		height: 100%;
		max-width: 650px;
		background-color: #fcfcfc;
		border: 1px solid #e0e0e0;
		border-radius: 2px;
	}

	.poster {
		display: flex;
		align-items: center;
		justify-content: center;
		width: 50px;
		line-height: 100%;
		background-size: contain;
	}

	.button {
		display: flex;
		align-items: center;
		justify-content: center;
		width: 47px;
		height: 47px;
		overflow: hidden;
		background-color: rgb(0, 0, 0, 0.2);
		border: 3.5px solid white;
		border-radius: 50%;
		opacity: 0.9;
	}

	.pause {
		width: 20px;
		height: 20px;
		background-color: white;
	}

	.play {
		margin-left: 8px;
		border-top: 15px solid transparent;
		border-bottom: 15px solid transparent;
		border-left: 27px solid white;
	}

	.info {
		flex: 1;
		line-height: 100%;
		padding: 5px;
		display: flex;		
		flex-direction: column;
	}

	.name {	
		text-align: center;
		font-size: 15px;
		line-height: 20px;
		overflow: hidden;
		text-overflow: ellipsis;
		white-space: nowrap;
	}

	.slide_time {
		display: flex;
	}

	.slider {
		flex: 1;
	}

	.time {
		width: 60px;
		display: flex;
		flex-direction: column;
		justify-content: center;
		align-items: center;
		font-size: 15px;
		color: #888;
	}
</style>
