# vue_qqmap

> 百度地图坐标拾取器vue组件，改善后台应用坐标输入体验，支持按城市名称搜索。
搜索框样式依赖elementUI,不需要可删除顶部的elementUI组件标签，地图部分无依赖项，使用前请在html模板文件中自行引入百度地图sdk的js外链文件


----------


![此处输入图片的描述][1]

          <mapselect :inputDefault="form.address" :oldMarker="oldMarker" @mapclick="pointChange"
                     @addr="addrChange" ref="mapsec"></mapselect>
         

 - mapcenter：地图初始化时的中心坐标
 - oldMarker：可选项，用于坐标点位编辑场景，显示已有的marker坐标，点击其他地方后自动消失
 - @mapclick：点击地图后的回调，参数为地图的坐标值

  [1]: https://static-1251225286.cos.ap-shanghai.myqcloud.com/chayashan/demo.png