使用uni-app实现小程序特有功能
---
## uni-app部分功能手册

## uni-app

## map编写：获取位置并查看

### map组件

```HTML
<map
  ref="map1"
  id="map1"
  :latitude="latitude"
  :longitude="longitude"
  :markers="markers"
  :controls="controls"
  :polyline="polyline"
  :include-points="includePoints"
  style="width: 100%; height: 300px;"
  show-location
  show-compass
  enable-overlooking
  enable-traffic
  @controltap="ctr"
/>
<script>
onReady() {
  this.map = uni.createMapContext('map1', this);
},
</script>
```

### 属性

| 参数 | 类型 | 默认值 | 说明 | 备注 |
| ---- | ---- | ---- | ---- | ---- |
| latitude | String | | 中心经度 |
| longitude | String | | 	中心纬度 |
| markers | Array | | 标记点 |
| controls | Array | | 控件 |
| polyline | Array | | 路线 |
| include-points | Array | | 缩放视野以包含所有给定的坐标点 |
| show-location | Boolean | | 显示带有方向的当前定位点 |
| show-compass | Boolean | false | 是否显示指南针 |
| enable-overlooking | Boolean | false | 是否开启俯视 |
| enable-traffic  | Boolean | false | 是否开启实时路况 |
| @controltap | EventHandle | | 点击控件时触发，e.detail = {controlId} |

### 初始化

### 事件

- 获取当前用户位置信息

```JavaScript
<script>
getLocation() {
  uni.getLocation({
    type: 'gcj02',
    success: res => {
      this.latitude = res.latitude
      this.longitude = res.longitude
      this.testPolyline[0].points[0] = {
        latitude: res.latitude,
        longitude: res.longitude
      }
    }
  })
},
</script>
```

- 选择位置

```JavaScript
<script>
choose() {
  uni.chooseLocation({
    success: res => {
      console.log(res);
      this.hasLocation = true
      this.chooseLocation = res
      this.markers[0].latitude = res.latitude
      this.markers[0].longitude = res.longitude
      this.testPolyline[0].points[1] = {
        latitude: res.latitude,
        longitude: res.longitude
      }
      this.ctr(res.latitude, res.longitude)
    }
  })
},
</script>
```

- 打开位置

```JavaScript
<script>
open() {
  uni.openLocation(this.chooseLocation)
},
</script>
```

- 连接当前位置与所选位置，并缩放视野展示

```JavaScript
<script>
point() {
  this.polyline = Object.assign([], this.testPolyline)
  this.includePoints = this.polyline[0].points
},
</script>
```

- 移动至该地点，默认为移动至当前用户位置

```JavaScript
<script>
ctr(la,lo) {
  this.map.moveToLocation({
    latitude: typeof la === 'number' && la || this.latitude,
    longitude: lo || this.longitude
  })
},
</script>
```

- 清除页面位置信息

```JavaScript
<script>
clear() {
  this.hasLocation = false
  this.polyline = []
  this.includePoints = []
  this.markers[0].latitude = 0
  this.markers[0].longitude = 0
  this.ctr()
}
</script>
```

- 参考文档
> https://uniapp.dcloud.net.cn/component/map
>
> https://uniapp.dcloud.net.cn/api/location/map

## image编写：图片上传及下载保存至相册

### image组件

- 支持九宫格展示及拖拽调换位置

```HTML
<view @touchmove.stop.prevent="moveHandle">
  <movable-area  ref="areaBox" id="areaBox" class="areaBox">
    <view class="uni-img-box">
      <view class="img-list" v-for="(item,index) in scopeImgList" :key="index">
        <image
          :id="'img' + index"
          :ref="'img' + index"
          :src="item.url"
          :mode="mode"
          :class="( hoverImgIdx === 'img' + index ) ? 'select' : ''"
          class="img-list-image"
          @touchstart="touchstart(item, index, $event)"
          @touchmove="touchmove"
          @touchend="touchend"
          @click="preview(item.url)"
        />
        <icon class="img-list-icon" type="clear" size="18" @click="closeImg(index)" />
      </view>
      <view v-if="scopeImgList.length < 9" class="img-list img-list-add" @click="addImgBtn">
        <image class="img-list-image" src="/static/plus.png"></image>
      </view>
    </view>
    <movable-view v-if="moveSrc" :animation="false" class="moveV" :x="x" :y="y" direction="all">
      <image :src="moveSrc" :mode="mode" class="img-list-move-image"></image>
    </movable-view>
  </movable-area>
</view>
```
### 属性

| 参数 | 类型 | 默认值 | 说明 | 备注 |
| ---- | ---- | ---- | ---- | ---- |
| src | String | | 图片资源地址 |
| mode | String | 'scaleToFill' | 图片裁剪、缩放的模式 |
| @touchstart |	EventHandle | | 手指触摸动作开始
| @touchmove | EventHandle | | 手指触摸后移动
| @touchend |	EventHandle | | 手指触摸动作结束

### 事件

- 根据条件选择图片

```JavaScript
<script>
choose() {
  uni.chooseImage({
    count: this.imgList.length + this.countArr[this.count] > 9 ? 9 - this.imgList.length : this.countArr[this.count],
    sizeType: this.size,
    sourceType: this.source,
    success: res => {
      this.imgList = this.imgList.concat(res.tempFilePaths)
      this.scopeImgList = this.imgList.map(val => {
        return { url: val }
      })
      this.$nextTick(_ => {
        this.$refs.dragImage.dom()
      })
    }
  });
},
</script>
```

- 预览图片

```JavaScript
<script>
preview(url) {
  uni.previewImage({
    current: url,
    urls: this.imgList,
    longPressActions: {
      itemList: ['发送给朋友', '保存图片', '收藏', '删除'],
    }
  });
}
</script>
```

- 删除图片并更新

```JavaScript
<script>
closeImg(index) {
  this.scopeImgList.splice(index, 1)
  this.dom()
  this.$emit('updateImg', this.scopeImgList)
},
updateImg(list) {
  this.imgList = list.map(val => val.url)
  this.scopeImgList = list.concat([])
},
</script>
```

- 展示图片选择条件：图片来源、图片质量、数量限制、图片裁剪缩放
```HTML
<view class="uni-list">
  <checkbox-group @change="sourceChange">
    <label class="uni-list-left">图片来源</label>
    <label class="uni-list-right" v-for="item in sourceType" :key="item.value">
      <checkbox :value="item.value" :name="item.value" checked="true" />
      <label :for="item.value" style="margin-left: 8px">{{ item.name }}</label>
    </label>
  </checkbox-group>
</view>
<view class="uni-list">
  <checkbox-group @change="sizeChange">
    <label class="uni-list-left">图片质量</label>
    <label class="uni-list-right"  v-for="item in sizeType" :key="item.value">
      <checkbox :value="item.value" :name="item.value" checked="true" />
      <label :for="item.value" style="margin-left: 8px">{{ item.name }}</label>
    </label>
  </checkbox-group>
</view>
<view class="uni-list">
  <label class="uni-list-left">数量限制</label>
  <view class="uni-list-right">
    <picker @change="countChange" :value="count" :range="countArr" mode="selector">
      <label>{{ countArr[count] }}</label>
    </picker>
  </view>
</view>
<view class="uni-list">
  <label class="uni-list-left" style="line-height: 20px;">图片裁剪缩放</label>
  <view class="uni-list-right">
    <picker @change="modeChange" :value="mode" :range="modeNameArr" mode="selector">
      <label>{{ modeArr[mode] + '：' + modeNameArr[mode] }}</label>
    </picker>
  </view>
</view>
```

```JavaScript
<script>
sourceChange(e) {
  this.source = e.detail.value
},
sizeChange(e) {
  this.size = e.detail.value
},
countChange(e) {
  this.count = Number(e.detail.value)
},
modeChange(e) {
  this.mode = e.detail.value
},
</script>
```

- 参考文档
> https://uniapp.dcloud.net.cn/component/image
>
> https://uniapp.dcloud.net.cn/api/media/image

## video编写：视频的播放及播放模式

### 初始化video组件

```HTML
<video
  id="myVideo"
  v-if="vedioSrc"
  :src="vedioSrc"
  :danmu-list="danmuList"
  class="video"
  autoplay
  loop
  controls
  enable-danmu
  show-mute-btn
  enable-play-gesture
  @longpress="open"
/>
<script>
onReady() {
	this.videoContext = uni.createVideoContext('myVideo')
},
</script>
```	
### 属性

| 参数 | 类型 | 默认值 | 说明 | 备注 |
| ---- | ---- | ---- | ---- | ---- |
| src | String | | 要播放视频的资源地址 |
| danmu-list | Object Array | | 弹幕列表 |
| autoplay | Boolean | false | 是否自动播放 |
| loop | Boolean | false | 是否循环播放 |
| controls | Boolean | true | 是否显示默认播放控件（播放/暂停按钮、播放进度、时间） |
| enable-danmu | Boolean | false | 是否展示弹幕，只在初始化时有效，不能动态变更 |
| show-mute-btn | Boolean | false | 是否显示静音按钮 |
| enable-play-gesture | Boolean | false | 是否开启播放手势，即双击切换播放/暂停 |
| @longpress | EventHandle |  | 用于监听长按事件 |


### 事件
- 添加视频，限制视频最小时长

```JavaScript
<script>
addVideo() {
	if (this.duration < 4) {
		this.$modal('视频时长不可小于3秒', '请重新选择')
		return
	}
	uni.chooseVideo({
		sourceType: this.source,
		maxDuration: this.duration,
		camera: this.camera,
		success: res => {
			this.vedioSrc = res.tempFilePath    // 视频路径
		}
	})
},
</script>
```

- 保存视频

```JavaScript
<script>
save() {
	uni.saveVideoToPhotosAlbum({
		filePath: this.vedioSrc,
		success: _ => {
			this.close()
		}
	})
},
</script>
```
- 可移动弹幕，动画展开输入框

```HTML
<movable-area id="videoBox" class="video-box" :style="{'height': videoBoxHeight + 'px'}">
	<movable-view direction="vertical" class="uni-danmu">
		<view v-if="vedioSrc" class="uni-danmu-content"  :style="{ width: showDanmu ? windowWidth + 'px' : '55px', height: '55px' }">
			<input v-if="showDanmu" v-model="danmuValue" class="uni-input" type="text" placeholder="在此处输入弹幕内容" />
			<view class="uni-danmu-circle" @click="show">{{ showDanmu ? '发' : '弹'}}</view>
		</view>
	</movable-view>
</movable-area>
```
- 发送弹幕

```JavaScript
<script>
show() {
	if (this.showDanmu && this.danmuValue) this.sendDanmu()
	this.showDanmu = !this.showDanmu
},
sendDanmu() {
	this.videoContext.sendDanmu({
		text: this.danmuValue,
		color: '#fff'
	});
	this.$toast('已发送')
	this.danmuValue = ''
}
</script>
```
- 展示视频选择条件：视频来源、摄像头位置、拍摄时间

```HTML
<view class="uni-list">
	<label class="uni-list-left">视频来源</label>
	<checkbox-group class="uni-list-right" @change="sourceChange">
		<label style="margin-right: 20px;" v-for="item in sourceType" :key="item.value">
			<checkbox :value="item.value" :name="item.value" checked="true" />
			<label :for="item.value">{{ item.name }}</label>
		</label>
	</checkbox-group>
</view>
<view class="uni-list">
	<label class="uni-list-left" style="line-height: 20px;">摄像头位置</label>
	<radio-group class="uni-list-right" @change="cameraChange">
		<label style="margin-right: 20px;" v-for="item in cameraType" :key="item.value">
			<radio :value="item.value" :name="item.value" :checked="item.checked" />
			<label :for="item.value">{{ item.name }}</label>
		</label>
	</radio-group>
</view>
<view class="uni-list">
	<label class="uni-list-left" style="line-height: 20px;">最长拍摄时间(秒)</label>
	<label class="uni-list-right">
		<slider min="1" max="60" show-value :value="duration" @change="sliderChange" />
	</label>
</view>
```

```JavaScript
<script>
sourceChange(e) {
	this.source = e.detail.value
},
cameraChange(e) {
	this.camera = e.detail.value
},
sliderChange(e) {
	this.duration = e.detail.value
},
</script>
```

- 参考文档
> https://uniapp.dcloud.net.cn/component/video
>
> https://uniapp.dcloud.net.cn/api/media/video-context



## camera编写：相机的使用：拍照及录像

### 初始化camera组件

```HTML
<camera
  :flash="flash"
  :device-position="position"
  style="width: 100%; height: 300px;"
  @error="error"
/>
<script>
onShow() {
	this.camera = uni.createCameraContext()
},
</script>
```
### 属性

| 参数 | 类型 | 默认值 | 说明 | 备注 |
| ---- | ---- | ---- | ---- | ---- |
| flash | String | auto | 闪光灯，值为auto, on, off |
| device-position | String | back | 前置或后置摄像头，值为front, back |
| @error | EventHandle | | 用户不允许使用摄像头时触发 |

### 事件

- 拍摄图片

```JavaScript
<script>
getImage() {
	this.camera.takePhoto({
		quality: 'high',
		success: res => {
			this.imgSrc = res.tempImagePath
		}
	})
},
</script>
```
- 开始录像

```JavaScript
<script>
startVideo() {
	this.camera.startRecord({
		success: res => {}
	})
},
</script>
```
- 结束录像

```JavaScript
<script>
endVideo() {
	this.camera.stopRecord({
		success: res => {
			this.vedioSrc = res.tempVideoPath   // 视频路径
			this.vSrc = res.tempThumbPath       // 视频封面图片路径
		}
	})
},
</script>
```
- 展示相机选择条件：摄像头位置、闪光灯

```HTML
<view class="uni-list">
	<label class="uni-list-left" style="line-height: 20px;">摄像头位置</label>
	<radio-group class="uni-list-right" @change="positionChange">
		<label style="margin-right: 20px;" v-for="item in positionType" :key="item.value">
			<radio :value="item.value" :name="item.value" :checked="item.checked" />
			<label :for="item.value">{{ item.name }}</label>
		</label>
	</radio-group>
</view>
<view class="uni-list">
	<label class="uni-list-left">闪光灯</label>
	<radio-group class="uni-list-right" @change="flashChange">
		<label style="margin-right: 20px;" v-for="item in flashType" :key="item.value">
			<radio :value="item.value" :name="item.value" :checked="item.checked"/>
			<label :for="item.value">{{ item.name }}</label>
		</label>
	</radio-group>
</view>
```
```JavaScript
<script>
flashChange(e) {
	this.flash = e.detail.value
},
positionChange(e) {
	this.position = e.detail.value
},
</script>
```

- 参考文档
> https://uniapp.dcloud.net.cn/component/camera
>
> https://uniapp.dcloud.net.cn/api/media/camera-context

## audio编写：音频的播放及播放模式

### 初始化
- uni.createInnerAudioContext()
创建并返回内部 audio 上下文 innerAudioContext 对象。

```JavaScript
<script>
createAudio() {
  var audioContext = this.audioContext = uni.createInnerAudioContext()
  audioContext.autoplay = false
  audioContext.src = this.current.src
  audioContext.onPlay(_ => {})
  audioContext.onTimeUpdate(e => {
    if (this.isChanging) return
    this.currentTime = audioContext.currentTime || 0
    this.duration = audioContext.duration || 0
    this.time = this.formatSeconds(Math.floor(this.currentTime))
  })
  audioContext.onEnded(_ => {
    this.currentTime = 0
    this.isPlaying = false
    this.isPlayEnd = true
    this.next()
  })
  audioContext.onError(res => {
    this.isPlaying = false
    this.next()
  })
  return audioContext
},
</script>
```

### 属性

| 参数 | 类型 | 说明 | 只读 | 备注 |
| ---- | ---- | ---- | ---- | ---- |
| src | String | 音频的数据链接，用于直接播放。 | 否 | 
| autoplay | Boolean | 是否自动开始播放，默认 false | 否 | H5端部分浏览器不支持
| duration | Number | 当前音频的长度（单位：s），只有在当前有合法的 src 时返回 | 是 | 
| currentTime | Number | 当前音频的播放位置（单位：s），只有在当前有合法的 src 时返回，时间不取整，保留小数点| 后 6 位 | 是 | 

### 方法

| 方法 | 参数 | 说明
| ---- | ---- | ---- |
| play |	|	播放（H5端部分浏览器需在用户交互时进行）
| pause |  | 暂停
| stop | | 停止
| onPlay | callback | 音频播放事件
| onPause | callback | 音频暂停事件
| onStop | callback | 音频停止事件
| onEnded | callback | 音频自然播放结束事件
| onTimeUpdate | callback | 音频播放进度更新事件
| onError | callback | 音频播放错误事件

### 事件

```HTML
<view class="audio-scroll">
  <scroll-view>
    <view v-for="(item, idx) in list" :key="item.id" :class="{ active: index === idx }" class="audio-list" @click="change(item, idx, $event)">
      <view class="index">{{ idx + 1 }}</view>
      <view class="text">
        <view class="title">{{ item.name }}</view>
        <view class="creator">{{ item.author }}</view>
      </view>
      <image id="dot" class="dot" src="/static/dots.png" @click="more(item)"></image>
    </view>
    <view v-if="list.length > 12" style="height:65px;width: 100%;"></view>
  </scroll-view>
</view>
```

```HTML
<view class="audio-fixed">
  <view id="myAudio" class="myAudio">
    <view class="audio-left" @click="play">
      <image class="bg" :src="current.picUrl"></image>
      <image class="bt" :src="playImage"></image>
    </view>
    <swiper class="audio-right" :style="{ background: bgColor }" circular @touchstart="start" @touchend="end">
      <swiper-item v-for="(val, idx) in bgColorArr" :key="idx" class="audio-swiper-content">
        <view class="text">
          <view class="title">{{ current.name }}</view>
          <view class="creator">{{ current.author }}</view>
        </view>
        <view class="time">{{ time }}</view>
      </swiper-item>
    </swiper>
  </view>
</view>
```

```JavaScript
<script>
play() {
  if (this.isPlaying) {
    this.pause()
    return
  }
  this.isPlaying = true
  this.audioContext.play()
  this.isPlayEnd = false
},
pause() {
  this.audioContext.pause()
  this.isPlaying = false
},
stop() {
  this.audioContext.stop()
  this.isPlaying = false
},
</script>
```

- 点击列表切换音频、背景色
若为右侧点图则不进入
若为当前音频，则暂停/播放

```JavaScript
<script>
change(item, index, e) {
  if (e && e.target.id === 'dot') return
  if (this.index === index ) {
    this.play()
    return
  }
  this.index = index
  this.current = item
  // console.log(this.audioContext, this.audioContext.src, this.current)
  this.audioContext.src = item.src
  this.audioContext.autoplay = true
  this.isPlaying = true
  const num = Math.floor(Math.random() * 8)
  console.log('change', num)
  this.bgColor = this.bgColor === this.bgColorArr[num] ? this.bgColorArr[Math.abs(num - 1)] : this.bgColorArr[num]
},
</script>
```

- 格式化当前音频，以播放时间的格式展示

```JavaScript
<script>
formatBit(val) {
  val = +val
  return val > 9 ? val : '0' + val
},
// 秒转时分秒，求模很重要，数字的下舍入
formatSeconds(time) {
  let min = Math.floor(time % 3600)
  let val = this.formatBit(Math.floor(min / 60)) + ':' + this.formatBit(time % 60)
  return val
},
</script>
```

- 切换上一首/下一首

```JavaScript
<script>
next() {
  const idx = this.index === this.list.length -1 ? 0 : this.index + 1
  this.change(this.list[idx], idx)
},
prev() {
  const idx = this.index === 0 ? this.list.length -1 : this.index - 1
  this.change(this.list[idx], idx)
},
</script>
```

- 滑动底部固定音频时获取位置，与滑动后的值比较判断切换音频

```JavaScript
<script>
start(e) {
  this.clientX = e.changedTouches[0].clientX
},
end(e) {
  const subX = e.changedTouches[0].clientX - this.clientX
  if (subX > 100) {
    this.prev()
  } else if (subX < -100) {
    this.next()
  }
},
</script>
```

- 分享：打开弹窗并赋值

```JavaScript
<script>
// 和 onLoad 等生命周期函数同级
onShareAppMessage() {
  return {
    title: this.shareData.name,
    path: this.$mp.page.route,
    imageUrl: this.shareData.picUrl
  }
},
more(item) {
  this.$refs.popup.open()
  this.shareData = item
},
close() {
  this.$refs.popup.close()
},
</script>
```

- 参考文档
> https://uniapp.dcloud.io/api/media/audio-context
