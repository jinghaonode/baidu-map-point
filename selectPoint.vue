<template>
  <div>
    <div style="overflow: hidden;">
      <section>
        <!--工具条-->
        <el-col :span="24" class="toolbar" style="padding-bottom: 0;">
          <el-form :inline="true">
            <el-form-item>
              <el-input v-model="inputVal" placeholder="可输入城市+景区名称后点击搜索" @keyup.native="inputChange"
                        style="width: 250px;"></el-input>
            </el-form-item>
            <el-form-item>
              <el-button type="primary" @click="search" class="searchBtn">搜索地理位置</el-button>
            </el-form-item>
            <el-form-item>
              <el-input v-model="latlngCurrent" @change="inputLatlngChange" placeholder="可直接粘贴坐标值到此"
                        style="width: 300px;"></el-input>
            </el-form-item>
          </el-form>
        </el-col>
      </section>
    </div>
    <div class="mapWrap">
      <div id="bdmapCont" style="width: 720px; height: 600px;">
      </div>
      <div class="lngTips"></div>
    </div>

  </div>

</template>

<script>
export default {
  props: {
    mapCenter: {
      type: String,
      default: '113.629691,34.755813'
    },
    oldMarker: {
      type: String
    },
    inputDefault: {
      type: String,
      default: ''
    }
  },
  data: function () {
    return {
      bdmap: '',
      premarker: '',
      marker: '',
      inputVal: '',
      latlngCurrent: '',
      markerIco: null,
      tileLayer: null
    }
  },
  mounted () {
    if (this.oldMarker && this.oldMarker.split(',')[0]) {
      this.latlngCurrent = this.oldMarker
    }
    if (this.$store.state.common.scenicInfo.lng) {
      this.mapCenter = this.$store.state.common.scenicInfo.lng + ',' + this.$store.state.common.scenicInfo.lat
    }
    setTimeout(() => {
      this.createMap()
    }, 100)
  },
  watch: {
    oldMarker (newVal, oldVal) {
      if (newVal) {
        if (newVal.split(',')[0]) {
          this.latlngCurrent = newVal
          //  this.bdmap && this.createMarker()
        }
      }
    },
    inputDefault (newVal, oldVal) {
      // console.log('默认地址变成：', newVal)
      this.inputVal = newVal
    }
  },
  methods: {
    search () {
      let that = this
      let callback = function (point) {
        if (point) {
          that.bdmap.centerAndZoom(point, 16)
          that.bdmap.addOverlay(new BMap.Marker(point))
        } else {
          console.log('您输入的地址没有解析到结果!')
        }
      }
      if (this.inputVal) {
        // 创建地址解析器实例
        let myGeo = new BMap.Geocoder()
        // 将地址解析结果显示在地图上,并调整地图视野
        myGeo.getPoint(that.inputVal, callback, '郑州')
      }
    },
    inputChange () {
      this.$emit('addr', this.inputVal) // 地图点击坐标 传递给父组件
    },
    createMap () {
      // 百度地图API功能
      let mapIntance = new BMap.Map('bdmapCont')
      console.log(this.mapCenter)
      mapIntance.centerAndZoom(new BMap.Point(this.mapCenter.split(',')[0], this.mapCenter.split(',')[1]), 19) // 初始化地图,设置中心点坐标和地图级别
      mapIntance.enableScrollWheelZoom(true)
      mapIntance.disableContinuousZoom()
      this.bdmap = mapIntance
      setTimeout(() => {
        this.createMarker()
        this.bindMapEvent()
      }, 500)
    },
    createMarker () {
      let that = this
      if (that.premarker) {
        that.premarker.setMap(null)
      }
      if (this.oldMarker && this.oldMarker.split(',')[0]) {
        console.log('编辑模式：', this.oldMarker)
        let oldPoint = new BMap.Point(this.oldMarker.split(',')[0], this.oldMarker.split(',')[1])
        that.bdmap.clearOverlays()
        let marker = new BMap.Marker(oldPoint) // 创建标注
        that.bdmap.addOverlay(marker) // 将标注添加到地图中
        marker.setAnimation(2) // 跳动的动画
        that.bdmap.centerAndZoom(oldPoint, 18)
      } else {
      }
    },
    bindMapEvent () {
      let that = this
      // 单击获取点击的经纬度-
      this.bdmap.addEventListener('click', function (e) {
        that.bdmap.clearOverlays()
        let marker = new BMap.Marker(new BMap.Point(e.point.lng, e.point.lat)) // 创建标注
        that.bdmap.addOverlay(marker) // 将标注添加到地图中
        marker.setAnimation(2) // 跳动的动画
        that.latlngCurrent = e.point.lng + ',' + e.point.lat
        that.$emit('mapclick', {lng: e.point.lng, lat: e.point.lat}) // 地图点击坐标 传递给父组件
      })
      // 鼠标移动事件
      let $lngTipsBox = document.querySelector('.lngTips')
      this.bdmap.addEventListener('mousemove', function (e) {
        $lngTipsBox.style.display = 'block'
        $lngTipsBox.style.top = (e.pixel.y + 5) + 'px'
        $lngTipsBox.style.left = (e.pixel.x + 5) + 'px'
        $lngTipsBox.innerText = '点击选择此坐标：' + e.point.lng + ',' + e.point.lat
      })
      this.bdmap.addEventListener('mouseout', function (e) {
        $lngTipsBox.style.display = 'none'
      })
    },
    inputLatlngChange () {
      this.$emit('mapclick', {lng: this.latlngCurrent.split(',')[0], lat: this.latlngCurrent.split(',')[1]}) // 地图点击坐标 传递给父组件
      this.oldMarker = this.latlngCurrent
      this.createMarker()
    }
  }
}
</script>

<style lang="scss">
  .mapWrap { position:relative; width:100%; height:600px; margin-top:15px;}
  .lngTips { display:none; width:255px; height:40px; padding:5px 7px; overflow:hidden; position:absolute; left:0; top:0; z-index:123456; background:#ffffff; border-radius:5px;
    line-height:20px; box-shadow:#eeeeee 1px 1px 3px; border:#eeeeee 1px solid; font-size:12px; }
</style>
