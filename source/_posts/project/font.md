---
title: 本机字体检测
categories:
- project
---
前端时间做过一个关于字体检测项目，现在做下总结，主要用于在本机检测某字体是否安装。
<!--more--> 
### 一、介绍
```
function isSupportFontFamily(f) {
  if (typeof f !== 'string') {
    return false
  }
  var h = 'Arial'
  if (f.toLowerCase() == h.toLowerCase()) {
    return true
  }
  var e = 'a'
  var d = 100
  var a = 100; var i = 100
  var c = document.createElement('canvas')
  var b = c.getContext('2d')
  c.width = a
  c.height = i
  b.textAlign = 'center'
  b.fillStyle = 'black'
  b.textBaseline = 'middle'
  var g = function (j) {
    b.clearRect(0, 0, a, i)
    b.font = d + 'px ' + j + ', ' + h
    b.fillText(e, a / 2, i / 2)
    var k = b.getImageData(0, 0, a, i).data
    return [].slice.call(k).filter(function (l) { return l != 0 })
  }
  return g(h).join('') !== g(f).join('')
}
```
直接上代码，很简单一个方法，传入字体名称，判断本机是否已经安装该字体。
### 二、说明
#### 2.1、语法
```
isSupportFontFamily(fontFamily)
```
其中 fontFamily 参数是必须的，为 CSS 中 font-family 设置的 web 可识别的字体名称，例如宋体 'simsun'，微软雅黑 'Microsoft Yahei' 等。
例如，我们要判断用户的操作系统是否安装了微软雅黑字体，可以这么处理：
```
// isSupportMicrosoftYahei 为 true 或者 false
var isSupportMicrosoftYahei = isSupportFontFamily('Microsoft Yahei')
```
#### 2.2、原理
根据用户设置的字体将某一个字符 'a' 绘制在 canvas 上，并提取像素信息，然后和默认字体 'Arial' 进行比对。如果像素不一致，说明字体生效，反之说明字体不生效。
兼容 IE9+，以及其他所有现代浏览器。
#### 2.3、部分字体中英文名称
```
[
  { 'cn_name': 'Times New Roman', 'en_name': 'Times New Roman' },
  { 'cn_name': 'Zhiyong Elegant', 'en_name': 'Zhiyong Elegant' },
  { 'cn_name': 'ZhiyongWrite', 'en_name': 'ZhiyongWrite' },
  { 'cn_name': '仿宋', 'en_name': 'FangSong' },
  { 'cn_name': '汉仪贤二体', 'en_name': 'XianErTi' },
  { 'cn_name': '黑体', 'en_name': 'SimHei' },
  { 'cn_name': '楷体', 'en_name': 'KaiTi' },
  { 'cn_name': '濑户字体', 'en_name': 'SetoFont' },
  { 'cn_name': '隶书', 'en_name': 'LiSu' },
  { 'cn_name': '沐瑶软笔手写体', 'en_name': 'Muyao-Softbrush' },
  { 'cn_name': '庞门正道标题体', 'en_name': 'PangMenZhengDao' },
  { 'cn_name': '锐字真言体', 'en_name': 'ZhenyanGB' },
  { 'cn_name': '手书体', 'en_name': 'ShouShuti' },
  { 'cn_name': '思源黑体', 'en_name': 'Source Han Sans CN' },
  { 'cn_name': '思源柔黑体', 'en_name': 'Gen Jyuu Gothic' },
  { 'cn_name': '思源宋体', 'en_name': 'Source Han Serif CN' },
  { 'cn_name': '思源真黑体', 'en_name': 'Gen Shin Gothic' },
  { 'cn_name': '宋体', 'en_name': 'SimSun' },
  { 'cn_name': '新宋体', 'en_name': 'NSimSun' },
  { 'cn_name': '杨任东竹石体', 'en_name': 'YRDZST' },
  { 'cn_name': '幼圆', 'en_name': 'YouYuan' },
  { 'cn_name': '源流明体', 'en_name': 'GenRynMin TW' },
  { 'cn_name': '源漾明体', 'en_name': 'GenYoMin TW' },
  { 'cn_name': '源云明体', 'en_name': 'GenWanMin TW' },
  { 'cn_name': '站酷高端黑', 'en_name': 'huxiaobo-gdh' },
  { 'cn_name': '站酷酷黑', 'en_name': 'HuXiaoBo-KuHei' },
  { 'cn_name': '站酷快乐体', 'en_name': 'HappyZcool' },
  { 'cn_name': '站酷快乐体新版', 'en_name': 'HappyZco6' },
  { 'cn_name': '站酷庆科黄油体', 'en_name': 'zcoolqingkehuangyouti' },
  { 'cn_name': '站酷文艺体', 'en_name': 'zcoolwenyiti' },
  { 'cn_name': '站酷小薇logo体', 'en_name': 'xiaowei' },
  { 'cn_name': '装甲明朝体', 'en_name': 'SoukouMincho' },
  { 'cn_name': 'Abril', 'en_name': 'Abril' },
  { 'cn_name': 'Abril Titling', 'en_name': 'Abril Titling' },
  { 'cn_name': 'Adelle', 'en_name': 'Adelle' },
  { 'cn_name': 'Adelle Sans', 'en_name': 'Adelle Sans' },
  { 'cn_name': 'Adelle Sans DEV', 'en_name': 'Adelle Sans' },
  { 'cn_name': 'Adobe Gothic Std B', 'en_name': 'Adobe Gothic Std B' },
  { 'cn_name': 'Adobe Myungjo Std M', 'en_name': 'Adobe Myungjo Std M' },
  { 'cn_name': 'Alize', 'en_name': 'Alize' },
  { 'cn_name': 'Arial Unicode MS', 'en_name': 'Arial' },
  { 'cn_name': 'Athelas', 'en_name': 'Athelas' },
  { 'cn_name': 'AwanZaman', 'en_name': 'AwanZaman' },
  { 'cn_name': 'Bely', 'en_name': 'BELY' },
  { 'cn_name': 'Bree', 'en_name': 'Bree' },
  { 'cn_name': 'Bree Serif', 'en_name': 'Bree Serif' },
  { 'cn_name': 'Capitolium2', 'en_name': 'Capitolium2' },
  { 'cn_name': 'Century Gothic', 'en_name': 'Century Gothic' },
  { 'cn_name': 'Cora', 'en_name': 'Cora' },
  { 'cn_name': 'Coranto 2', 'en_name': 'Coranto 2' },
  { 'cn_name': 'Crete', 'en_name': 'Crete' },
  { 'cn_name': 'Ebony', 'en_name': 'Ebony' },
  { 'cn_name': 'Edita', 'en_name': 'Edita' },
  { 'cn_name': 'Eskapade', 'en_name': 'Eskapade' },
  { 'cn_name': 'Essay', 'en_name': 'Essay Text' },
  { 'cn_name': 'Fino', 'en_name': 'FINO' },
  { 'cn_name': 'Fino Stencil', 'en_name': 'FINO STENCIL' },
  { 'cn_name': 'Garalda', 'en_name': 'Garalda' },
  { 'cn_name': 'Givry', 'en_name': 'Givry' },
  { 'cn_name': 'Helvetica', 'en_name': 'Helvetica' },
  { 'cn_name': 'Iro Sans', 'en_name': 'IRO SANS' },
  { 'cn_name': 'Iskra', 'en_name': 'Iskra' },
  { 'cn_name': 'Karmina', 'en_name': 'Karmina' },
  { 'cn_name': 'Karmina Sans', 'en_name': 'Karmina Sans' },
  { 'cn_name': 'LFT Etica', 'en_name': 'LFT Etica' },
  { 'cn_name': 'Lipa Agate', 'en_name': 'Lipa Agate' },
  { 'cn_name': 'Lisbeth', 'en_name': 'Lisbeth' },
  { 'cn_name': 'Maiola', 'en_name': 'Maiola' },
  { 'cn_name': 'Marco', 'en_name': 'Marco' },
  { 'cn_name': 'Noam', 'en_name': 'NOAM' },
  { 'cn_name': 'Noort', 'en_name': 'NOORT' },
  { 'cn_name': 'Pollen', 'en_name': 'Pollen' },
  { 'cn_name': 'Protipo', 'en_name': 'PROTIPO' },
  { 'cn_name': 'Ronnia', 'en_name': 'Ronnia' },
  { 'cn_name': 'Sanserata', 'en_name': 'Sanserata' },
  { 'cn_name': 'Serifa®', 'en_name': 'Serifa' },
  { 'cn_name': 'Serpentine™', 'en_name': 'Serpentine' },
  { 'cn_name': 'Sirba', 'en_name': 'Sirba' },
  { 'cn_name': 'Soleil', 'en_name': 'Soleil' },
  { 'cn_name': 'Tablet Gothic', 'en_name': 'Tablet Gothic' },
  { 'cn_name': '冬青黑体简', 'en_name': 'Hiragino Sans GB' },
  { 'cn_name': '方正报宋简体', 'en_name': 'FZBaoSong4S' },
  { 'cn_name': '方正本墨绪圆体', 'en_name': 'FZBMXY JW' },
  { 'cn_name': '方正兵马俑体', 'en_name': 'FZBingMaYongTiS' },
  { 'cn_name': '方正博雅宋简体', 'en_name': 'FZBoYaSongS-GB' },
  { 'cn_name': '方正彩云简体', 'en_name': 'FZCaiYun9S' },
  { 'cn_name': '方正藏意汉体简体', 'en_name': 'FZZangYiHanTiS-R-GB' },
  { 'cn_name': '方正超粗黑简体', 'en_name': 'FZChaoCuHei-S' },
  { 'cn_name': '方正超重要体', 'en_name': 'FzCHAOZYTJW' },
  { 'cn_name': '方正粗活意简体', 'en_name': 'FZCuHuoYi-M25S' },
  { 'cn_name': '方正粗倩简体', 'en_name': 'FZCuQian7S' },
  { 'cn_name': '方正粗宋简体', 'en_name': 'FZCuSong9S' },
  { 'cn_name': '方正粗谭黑简体', 'en_name': 'FZTanHeiS-B-GB' },
  { 'cn_name': '方正粗雅宋简体', 'en_name': 'FZYaSongS-B-GB' },
  { 'cn_name': '方正粗圆简体', 'en_name': 'FZCuYuan3S' },
  { 'cn_name': '方正大标宋简体', 'en_name': 'FZDaBiaoSong6S' },
  { 'cn_name': '方正大黑简体', 'en_name': 'FZDaHei2S' },
  { 'cn_name': '方正大魏体简体', 'en_name': 'FZDaWeiTiS-R-GB' },
  { 'cn_name': '方正邓黑隶', 'en_name': 'FZDengHLJ' },
  { 'cn_name': '方正方俊黑', 'en_name': 'FZFangJunHeiS' },
  { 'cn_name': '方正仿宋简体', 'en_name': 'FZFangSong2S' },
  { 'cn_name': '方正飞跃体', 'en_name': 'FZFeiYueTiS' },
  { 'cn_name': '方正复古粗宋', 'en_name': 'FOUNDERTYPE' },
  { 'cn_name': '方正古隶简体', 'en_name': 'FZGuLi2S' },
  { 'cn_name': '方正管峻楷书', 'en_name': 'FZGUANJUNKAISHUS' },
  { 'cn_name': '方正汉真广标简体', 'en_name': 'FZHanZhenGuangBiaoS-GB' },
  { 'cn_name': '方正行楷简体', 'en_name': 'FZXingKai4S' },
  { 'cn_name': '方正黑体简体', 'en_name': 'FZHS' },
  { 'cn_name': '方正琥珀简体', 'en_name': 'FZHuPo4S' },
  { 'cn_name': '方正华隶简体', 'en_name': 'FZHuaLi4S' },
  { 'cn_name': '方正黄草简体', 'en_name': 'FZHuangCao9S' },
  { 'cn_name': '方正活龙体', 'en_name': 'FZHuoLTJ' },
  { 'cn_name': '方正记忆的碎片体', 'en_name': 'FZJiYiDeSuiPianTiS' },
  { 'cn_name': '方正剪纸简体', 'en_name': 'FZJianZhi-M23S' },
  { 'cn_name': '方正静蕾简体', 'en_name': 'FZJingLeiS-R-GB' },
  { 'cn_name': '方正卡通简体', 'en_name': 'FZKaTong9S' },
  { 'cn_name': '方正楷体简体', 'en_name': 'FZKai3S' },
  { 'cn_name': '方正康体简体', 'en_name': 'FZKangTi7S' },
  { 'cn_name': '方正克书皇榜体', 'en_name': 'FZKeShuHuangBang' },
  { 'cn_name': '方正兰亭超细黑简体', 'en_name': 'FZLanTingHeiS-UL-GB' },
  { 'cn_name': '方正兰亭粗黑简体', 'en_name': 'FZLTCHJW' },
  { 'cn_name': '方正兰亭黑简体', 'en_name': 'FZLanTingHeiS-R-GB' },
  { 'cn_name': '方正兰亭特黑简体', 'en_name': 'FZLanTingHeiS-H-GB' },
  { 'cn_name': '方正兰亭细黑_GBK', 'en_name': 'FZLanTingHei-L-GBK' },
  { 'cn_name': '方正兰亭细黑_GBK-M', 'en_name': 'FZLanTingHei-L-GBK-M' },
  { 'cn_name': '方正兰亭纤黑繁体', 'en_name': 'FZLanTingHeiT-EL-GB' },
  { 'cn_name': '方正兰亭纤黑简体', 'en_name': 'FZLanTingHeiS-EL-GB' },
  { 'cn_name': '方正兰亭圆简体', 'en_name': 'FZLanTingYuanS-R-GB' },
  { 'cn_name': '方正兰亭圆简体_纤', 'en_name': 'FZLanTingYuanS-EL-GB' },
  { 'cn_name': '方正隶变简体', 'en_name': 'FZLiBian2S' },
  { 'cn_name': '方正隶二简体', 'en_name': 'FZLiShu II6S' },
  { 'cn_name': '方正隶书简体', 'en_name': 'FZLiSS' },
  { 'cn_name': '方正流行体简体', 'en_name': 'FZLiuXingTi-M26S' },
  { 'cn_name': '方正美黑简体', 'en_name': 'FZMeiHei7S' },
  { 'cn_name': '方正萌软体', 'en_name': 'FZMengRuanTiS' },
  { 'cn_name': '方正萌艺体', 'en_name': 'FZMengYiTiS' },
  { 'cn_name': '方正喵呜简体', 'en_name': 'FZMiaoWuS-R-GB' },
  { 'cn_name': '方正喵呜体', 'en_name': 'FZMiaoWuS-R-GB' },
  { 'cn_name': '方正胖头鱼简体', 'en_name': 'FZPangTouYu-M24S' },
  { 'cn_name': '方正胖娃简体', 'en_name': 'FZPangWa8S' },
  { 'cn_name': '方正品尚粗黑简体', 'en_name': 'FZPinShangHeiS-B-GB' },
  { 'cn_name': '方正品尚黑简体', 'en_name': 'FZPinShangHeiS-R-GB' },
  { 'cn_name': '方正平和简体', 'en_name': 'FZPingHeS' },
  { 'cn_name': '方正奇妙体', 'en_name': 'FZQiMTJ' },
  { 'cn_name': '方正启笛简体', 'en_name': 'FZQiDiS-R-GB' },
  { 'cn_name': '方正启体简体', 'en_name': 'FZQiTi4S' },
  { 'cn_name': '方正清仿宋系列', 'en_name': 'FZQingFangSongS' },
  { 'cn_name': '方正晴朗体', 'en_name': 'FZQINGLANGTIS' },
  { 'cn_name': '方正趣黑', 'en_name': 'FZQuHJW' },
  { 'cn_name': '方正锐正圆', 'en_name': 'FZRuiZhengYuanS' },
  { 'cn_name': '方正尚酷简体', 'en_name': 'FZShangKuS-R-GB' },
  { 'cn_name': '方正尚艺体', 'en_name': 'FZShangYiTiS' },
  { 'cn_name': '方正少儿简体', 'en_name': 'FZShaoErS' },
  { 'cn_name': '方正瘦金书简体', 'en_name': 'FZShouJinShu-S' },
  { 'cn_name': '方正书宋', 'en_name': 'FZSS' },
  { 'cn_name': '方正书宋简体', 'en_name': 'FZShuSoS' },
  { 'cn_name': '方正舒体', 'en_name': 'FZShuTi' },
  { 'cn_name': '方正舒体简体', 'en_name': 'FZShuTi5S' },
  { 'cn_name': '方正水黑简体', 'en_name': 'FZShuiHei-S' },
  { 'cn_name': '方正水云简体', 'en_name': 'FZShuiYunS-EB-GB' },
  { 'cn_name': '方正水柱简体', 'en_name': 'FZShuiZhu8S' },
  { 'cn_name': '方正宋黑简体', 'en_name': 'FZSongHei7S' },
  { 'cn_name': '方正宋三简体', 'en_name': 'FZSong' },
  { 'cn_name': '方正宋一简体', 'en_name': 'FZSongYi3S' },
  { 'cn_name': '方正苏新诗小爨', 'en_name': 'FZSuXinShiXiaoCuanS' },
  { 'cn_name': '方正孙拥声简体', 'en_name': 'FZSunYongShengS-R-GB' },
  { 'cn_name': '方正淘乐简体', 'en_name': 'FZTLJW' },
  { 'cn_name': '方正玩伴体系列', 'en_name': 'FZWanBTJ' },
  { 'cn_name': '方正魏碑简体', 'en_name': 'FZWeiBei3S' },
  { 'cn_name': '方正细等线简体', 'en_name': 'FZXiDengXian6S' },
  { 'cn_name': '方正细黑一简体', 'en_name': 'FZXiHei I8S' },
  { 'cn_name': '方正细倩简体', 'en_name': 'FZXiQian5S' },
  { 'cn_name': '方正细珊瑚简体', 'en_name': 'FZXiShanHu3S' },
  { 'cn_name': '方正细圆简体', 'en_name': 'FZXiYuS' },
  { 'cn_name': '方正仙阵体', 'en_name': 'FZXianZhenTiS' },
  { 'cn_name': '方正祥隶简体', 'en_name': 'FZXiangLi7S' },
  { 'cn_name': '方正小标宋简体', 'en_name': 'FZXiaoBiaoSong5S' },
  { 'cn_name': '方正小篆体', 'en_name': 'FZXiaoZhuanTi3T' },
  { 'cn_name': '方正新报宋简体', 'en_name': 'FZNew BaoSong2S' },
  { 'cn_name': '方正新舒体简体', 'en_name': 'FZNew ShuTi8S' },
  { 'cn_name': '方正秀麗', 'en_name': 'FZXiuLiB3' },
  { 'cn_name': '方正雅士黑', 'en_name': 'FZYaShiHeiS' },
  { 'cn_name': '方正颜宋简体_粗', 'en_name': 'FZYanSongS-B-GB' },
  { 'cn_name': '方正姚体', 'en_name': 'FZYaoti' },
  { 'cn_name': '方正姚体简体', 'en_name': 'FZYaoTi6S' },
  { 'cn_name': '方正艺黑简体', 'en_name': 'FZYiHei-S' },
  { 'cn_name': '方正硬笔行书简体', 'en_name': 'FZYingBiXingShu6S' },
  { 'cn_name': '方正硬笔楷书简体', 'en_name': 'FZYingBiKaiShu5S' },
  { 'cn_name': '方正悠宋简可变', 'en_name': 'FZYouSJ VF WT' },
  { 'cn_name': '方正幼线简体', 'en_name': 'FZYouXian9S' },
  { 'cn_name': '方正韵动特黑简体', 'en_name': 'FZYunDongHeiS-H-GB' },
  { 'cn_name': '方正赞美体', 'en_name': 'FZZANMTJ' },
  { 'cn_name': '方正毡笔黑简体', 'en_name': 'FZZhanBiHei-M22S' },
  { 'cn_name': '方正正粗黑简体', 'en_name': 'FZZhengHeiS-B-GB' },
  { 'cn_name': '方正正大黑简体', 'en_name': 'FZZhengHeiS-EB-GB' },
  { 'cn_name': '方正正纤黑简体', 'en_name': 'FZZhengHeiS-EL-GB' },
  { 'cn_name': '方正正准黑简体', 'en_name': 'FZZhengHeiS-M-GB' },
  { 'cn_name': '方正稚艺简体', 'en_name': 'FZZhiYi2S' },
  { 'cn_name': '方正中等线简体', 'en_name': 'FZZhongDengXian7S' },
  { 'cn_name': '方正中倩简体', 'en_name': 'FZZhongQian6S' },
  { 'cn_name': '方正准圆简体', 'en_name': 'FZZhunYuan2S' },
  { 'cn_name': '方正字迹-曾柏求排笔', 'en_name': 'FZZJ-ZBQPBJW' },
  { 'cn_name': '方正字迹-陈光池楷书', 'en_name': 'FZZJ-CGCKSJW' },
  { 'cn_name': '方正字迹-顾建平楷书', 'en_name': 'FZZJ-GJPKSJW' },
  { 'cn_name': '方正字迹-顾建平隶书', 'en_name': 'FZZJ-GJPLSJF' },
  { 'cn_name': '方正字迹-顧建平篆書', 'en_name': 'FZZJ-GJPZSFU' },
  { 'cn_name': '方正字迹-豪放行书简体', 'en_name': 'FZZJ-HFXSJW' },
  { 'cn_name': '方正字迹-李凤武行书', 'en_name': 'FZZJ-LFWXSJW' },
  { 'cn_name': '方正字迹-刘宏楷书', 'en_name': 'FZZJ-LHKSJW' },
  { 'cn_name': '方正字迹-刘毅硬笔行书', 'en_name': 'FZZJ-LYYBXSJW' },
  { 'cn_name': '方正字迹-刘毅硬笔楷书', 'en_name': 'FZZJ-LYYBKSJW' },
  { 'cn_name': '方正字迹-柳正枢行楷', 'en_name': 'FZZJ-LZSXKJW' },
  { 'cn_name': '方正字迹-清代碑体', 'en_name': 'FZZJ-QDBTJW' },
  { 'cn_name': '方正字迹-陶建华魏碑', 'en_name': 'FZZJ-TJHWBJW' },
  { 'cn_name': '方正字迹-童体硬笔字体', 'en_name': 'FZZJ-TTYBFONT' },
  { 'cn_name': '方正字迹-吴进行书', 'en_name': 'FZZJ-WJXSJW' },
  { 'cn_name': '方正字迹-严祖喜行楷', 'en_name': 'FZZJ-YZXXKJW' },
  { 'cn_name': '方正字迹-颜振东楷', 'en_name': 'FZZJ-YZDKJW' },
  { 'cn_name': '方正字迹-叶根友特楷', 'en_name': 'YEGENYOUTEKAI' },
  { 'cn_name': '方正字迹-张二魁硬楷', 'en_name': 'FZZJ-ZEKYKJW' },
  { 'cn_name': '方正字迹-张亮硬笔行书', 'en_name': 'FZZJ-ZLYBXSJW' },
  { 'cn_name': '方正字迹-长江行书', 'en_name': 'FZZJ-CJXSJW' },
  { 'cn_name': '方正字迹-周密行楷', 'en_name': 'FZZJ-ZMXKJW' },
  { 'cn_name': '方正字迹-禚效锋行草', 'en_name': 'FZZJ-ZXFXCJW' },
  { 'cn_name': '方正字迹-自强魏楷体', 'en_name': 'FZZJ-ZQWKJW' },
  { 'cn_name': '方正综艺简体', 'en_name': 'FZZongYi5S' },
  { 'cn_name': '汉仪PP体简', 'en_name': 'HYPPTiJ' },
  { 'cn_name': '汉仪程行简', 'en_name': 'HYChengXingJ' },
  { 'cn_name': '汉仪大黑简', 'en_name': 'HYDaHeiJ' },
  { 'cn_name': '汉仪大宋简', 'en_name': 'HYDaSongJ' },
  { 'cn_name': '汉仪蝶语简体', 'en_name': 'HYDieYuJ' },
  { 'cn_name': '汉仪方叠体检', 'en_name': 'HYFangDieJ' },
  { 'cn_name': '汉仪仿宋简', 'en_name': 'HYFangSongJ' },
  { 'cn_name': '汉仪刚艺体', 'en_name': 'HYGangYiTi' },
  { 'cn_name': '汉仪行楷简', 'en_name': 'HYXingKaiJ' },
  { 'cn_name': '汉仪黑荔枝', 'en_name': 'HYHeiLiZhiTiJ' },
  { 'cn_name': '汉仪家书简', 'en_name': 'HYJiaShuJ' },
  { 'cn_name': '汉仪楷体', 'en_name': 'HYKaiti' },
  { 'cn_name': '汉仪楷体简', 'en_name': 'HYKaiTiJ' },
  { 'cn_name': '汉仪乐喵体简', 'en_name': 'HYLeMiaoTi' },
  { 'cn_name': '汉仪立黑简', 'en_name': 'HYLiHeiJ' },
  { 'cn_name': '汉仪菱心体简', 'en_name': 'HYLingXinJ' },
  { 'cn_name': '汉仪漫步体间', 'en_name': 'HYManBuJ' },
  { 'cn_name': '汉仪旗黑', 'en_name': 'HYQiheiS' },
  { 'cn_name': '汉仪旗黑', 'en_name': 'HYQiheiS' },
  { 'cn_name': '汉仪旗黑', 'en_name': 'HYQiheiS' },
  { 'cn_name': '汉仪尚巍手书W', 'en_name': 'HYShangWeiShouShuW' },
  { 'cn_name': '汉仪书魂体检', 'en_name': 'HYShuHunJ' },
  { 'cn_name': '汉仪娃娃纂简', 'en_name': 'HYWaWaZhuanJ' },
  { 'cn_name': '汉仪小麦体', 'en_name': 'HYXiaoMaiTiJ' },
  { 'cn_name': '汉仪醒示体简', 'en_name': 'HYXingShiJ' },
  { 'cn_name': '汉仪秀英体简', 'en_name': 'HYXiuYingJ' },
  { 'cn_name': '汉仪雪峰体简', 'en_name': 'HYXueFengJ' },
  { 'cn_name': '汉仪雪君体简', 'en_name': 'HYXueJunJ' },
  { 'cn_name': '汉仪丫丫体简', 'en_name': 'HYYaYaJ' },
  { 'cn_name': '汉仪雅酷黑W', 'en_name': 'HYYaKuHeiW' },
  { 'cn_name': '汉仪长美黑简', 'en_name': 'HYChangMeiHeiJ' },
  { 'cn_name': '汉仪中等线简', 'en_name': 'HYZhongDengXianJ' },
  { 'cn_name': '汉仪中黑简', 'en_name': 'HYZhongHeiJ' },
  { 'cn_name': '汉仪中隶书简', 'en_name': 'HYZhongLiShuJ' },
  { 'cn_name': '汉仪中宋简', 'en_name': 'HYZhongSongJ' },
  { 'cn_name': '汉仪综艺体简', 'en_name': 'HYZongYiJ' },
  { 'cn_name': '华康翩翩体', 'en_name': 'Hanzipen SC' },
  { 'cn_name': '华康手札体', 'en_name': 'Hannotate SC' },
  { 'cn_name': '华文彩云', 'en_name': 'STCaiyun' },
  { 'cn_name': '华文仿宋', 'en_name': 'STFangsong' },
  { 'cn_name': '华文行楷', 'en_name': 'STXingkai' },
  { 'cn_name': '华文黑体', 'en_name': 'STHeiti' },
  { 'cn_name': '华文琥珀', 'en_name': 'STHupo' },
  { 'cn_name': '华文楷体', 'en_name': 'STKaiti' },
  { 'cn_name': '华文隶书', 'en_name': 'STLiti' },
  { 'cn_name': '华文宋体', 'en_name': 'STSong' },
  { 'cn_name': '华文细黑', 'en_name': 'STXihei' },
  { 'cn_name': '华文新魏', 'en_name': 'STXinwei' },
  { 'cn_name': '华文中宋', 'en_name': 'STZhongsong' },
  { 'cn_name': '兰亭黑-简', 'en_name': 'Lantinghei SC' },
  { 'cn_name': '苹方', 'en_name': 'PingFang SC' },
  { 'cn_name': '微软雅黑', 'en_name': 'Microsoft Yahei' },
  { 'cn_name': '微软正黑体', 'en_name': 'Microsoft JhengHei' },
  { 'cn_name': '小美体', 'en_name': 'XIAOMEI JW' },
  { 'cn_name': '造字工房版', 'en_name': 'RTWS BanHe' },
  { 'cn_name': '造字工房博黑', 'en_name': 'MF BoHei' },
  { 'cn_name': '造字工房典黑', 'en_name': 'MF DianHei' },
  { 'cn_name': '造字工房鼎黑体', 'en_name': 'MF DingHei' },
  { 'cn_name': '造字工房黄金时代', 'en_name': 'MF TheGoldenEra' },
  { 'cn_name': '造字工房佳黑', 'en_name': 'MF JiaHei' },
  { 'cn_name': '造字工房坚黑', 'en_name': 'MF JianHei' },
  { 'cn_name': '造字工房锦宋体', 'en_name': 'MF JinSong' },
  { 'cn_name': '造字工房劲', 'en_name': 'RTWS JinHe' },
  { 'cn_name': '造字工房郎', 'en_name': 'RTWS LangSon' },
  { 'cn_name': '造字工房力黑', 'en_name': 'MF LiHei' },
  { 'cn_name': '造字工房力黑体', 'en_name': 'MF LiHei' },
  { 'cn_name': '造字工房凌黑', 'en_name': 'MF LingHei' },
  { 'cn_name': '造字工房明黑', 'en_name': 'MF MingHei' },
  { 'cn_name': '造字工房品宋', 'en_name': 'MF PinSong' },
  { 'cn_name': '造字工房启黑体', 'en_name': 'MF QiHei' },
  { 'cn_name': '造字工房形黑', 'en_name': 'MF XingHei' },
  { 'cn_name': '造字工房形黑体', 'en_name': 'MF XingHei' },
  { 'cn_name': '造字工房雅', 'en_name': 'RTWS YaYua' },
  { 'cn_name': '造字工房元黑体', 'en_name': 'MF YuanHei' },
  { 'cn_name': '造字工房云宋体', 'en_name': 'MF YunSong' },
  { 'cn_name': '造字工房哲黑', 'en_name': 'MF ZheHei' },
  { 'cn_name': '造字工房臻宋体', 'en_name': 'MF ZhenSong' },
  { 'cn_name': '造字工房卓黑体', 'en_name': 'MF ZhuoHei' },
  { 'cn_name': 'Tahoma', 'en_name': 'tahoma' },
  { 'cn_name': '行楷-简', 'en_name': 'Xingkai SC' },
  { 'cn_name': '宋体-简', 'en_name': 'Songti SC' },
  { 'cn_name': '娃娃体-简', 'en_name': 'Wawati SC' },
  { 'cn_name': '魏碑-简', 'en_name': 'Weibei SC' },
  { 'cn_name': '雅痞-简', 'en_name': 'Yapi SC' },
  { 'cn_name': '圆体-简', 'en_name': 'Yuanti SC' }
]
```