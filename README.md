# parser-audio
完善jyf-parser的audio插件

## 首先非常感谢
Parser是非常优秀的富文本解析插件，以前开发微信小程序用它，现在开放uniapp也用它，目前没发现比它更强大的，作者地址https://github.com/jin-yufeng/Parser

## 使用时遇到问题
- autopause属性无效，经过检查它是依据id来实现，而视频和音频的id默认是video1和audio1，除非html中已经给video和audio指定id，明显这不现实
- audio插件在uniapp编译后，微信小程序工作正常，而在APP中则不正常，当然官方已经申明了APP不支持audio

## 目前解决的
- 修改了audio插件，使用它可以支持微信小程序和APP
