# parser-audio
完善jyf-parser的audio插件

## 首先非常感谢
Parser是非常优秀的富文本解析插件，以前开发微信小程序用它，现在开放uniapp也用它，目前没发现比它更强大的，作者地址https://github.com/jin-yufeng/Parser

## 使用时遇到问题
- autopause属性无效，经过检查它是依据id来实现，而视频和音频的id默认是video1和audio1，除非html中已经给video和audio指定id，明显这不现实
- audio插件在uniapp编译后，微信小程序工作正常，而在APP中则不正常，当然官方已经申明了APP不支持audio

## 目前解决的
- 修改了audio插件，使用它可以支持微信小程序和APP
  - 为支持app，将this.setSrc(this.src)移到_buttonTap()方法中
  - 为实现app播放时自动暂停上一首，增加了onPause事件
  - 为实现微信小程序也支持自动暂停，引入了vuex，这里vuex是使用uView封装的https://www.uviewui.com/guide/globalVariable.html  
  通过数组压入和弹出来控制暂停上一首，需要增加vuex变量vuex_audio。
  - 原audio插件布局太不灵活，在平板适配时非常大，不好看，我对界面进行了自适应调整，适配微信小程序、手机和平板，比较人性化
