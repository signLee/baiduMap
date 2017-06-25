<template>
  <div class="C-map">
    <header class="mui-bar mui-bar-nav">
      <router-link :to="{name: 'storeRecycling'}" class="mui-icon mui-action-back mui-icon-left-nav"></router-link>
      <h1 class="mui-title">iPhone7</h1>
      <router-link :to="{name: 'index.home'}"><img src="../../image/ico1_ac.png"/></router-link>
    </header>
    <div id="allmap"></div>
    <div id="r-result">
      <input id="longitude" type="hidden" style="width:100px; margin-right:10px;"/>
      <input id="latitude" type="hidden" style="width:100px; margin-right:10px;"/>
    </div>
  </div>
</template>

<script>
export default {
  data() {
    return {
      nowLat: 0,
      nowLng: 0,
      point: {},
      marker: {},
      map: {}
    }
  },
  methods: {},
  computed: {},
  created(){
    var _this = this;
    var geolocation = new BMap.Geolocation();
    //弹出地理授权 获取当前的经纬度
    geolocation.getCurrentPosition(function (r) {
        if (this.getStatus() == BMAP_STATUS_SUCCESS) {
          _this.nowLat = r.point.lat;
          _this.nowLng = r.point.lng;
          _this.useBaiduMap();
        } else {
          alert("baidu return failed");
        }
      },
      //获取失败时候的回调
      function (r) {
        alert('请允许百度地图获取当前位置信息');
        return {
          //设置高精度
          enableHighAccuracy: true
        };
      }
    );
  },
  mounted(){

  },
  methods: {
    //根据动态获取到的经纬度显示当前位置
    useBaiduMap: function () {
      var _this = this;
      // 百度地图API功能
      let Lat = this.nowLat;
      let Lng = this.nowLng;
      // 百度地图API功能*/
      _this.map = new BMap.Map("allmap");
      _this.point = new BMap.Point(Lng, Lat);
      _this.map.centerAndZoom(_this.point, 18);
      _this.map.enableScrollWheelZoom(true);   //启用滚轮放大缩小，默认禁用
      _this.map.enableContinuousZoom();    //启用地图惯性拖拽，默认禁用
      _this.marker = new BMap.Marker(_this.point); //创建marker对象
      _this.map.addOverlay(_this.marker);              // 将标注添加到地图中
      _this.marker.setAnimation(BMAP_ANIMATION_BOUNCE); //跳动的动画
      //用户确认重新选择位置坐标
      _this.map.addEventListener("click", function (e) {
        _this.disp_confirm(e);
      });
      //获取手机设备高度
      var deviceHeight = window.innerHeight;
      document.getElementById('allmap').style.height = deviceHeight + 'px';
    },
    //重新选择位置信息
    disp_confirm(e) {
      var _this = this;
      this.point = new BMap.Point(e.point.lng, e.point.lat);
      var r = confirm("确定将位置修改到此处？点击定位小点可以获取详细位置信息")
      if (r == true) {
        _this.point = new BMap.Point(e.point.lng, e.point.lat);
        _this.marker.setPosition(_this.point);//设置覆盖物位置
      }
      _this.getAddressName();
    },
    // 获取具体位置信息名称
    getAddressName() {
      var _this = this;
      var gc = new BMap.Geocoder();
      gc.getLocation(_this.point, function (rs) {
        var addComp = rs.addressComponents;
        // 百度地图API功能
        _this.marker = new BMap.Marker(_this.point);  // 创建标注
        _this.remove_overlay();
        _this.map.addOverlay(_this.marker);              // 将标注添加到地图中
        _this.map.centerAndZoom(_this.point, 18);
        var opts = {
          width: 200,     // 信息窗口宽度
          height: 100,     // 信息窗口高度
          title: "", // 信息窗口标题
          enableMessage: true,//设置允许信息窗发送短息
          message: ""
        }
        var infoWindow = new BMap.InfoWindow("地址：" + addComp.province + addComp.city + addComp.district + addComp.street + addComp.streetNumber, opts);  // 创建信息窗口对象
        _this.marker.addEventListener("click", function (e) {
          e.domEvent.cancelBubble=true;//点击标注时显示当前位置信息  阻止地图的点击事件
          _this.map.openInfoWindow(infoWindow, _this.point); //开启信息窗口

        });
        console.log(addComp)
      });
    },
    //清除覆盖物
    remove_overlay(){
      this.map.clearOverlays();
    }
  }
}
</script>

<style>
.C-map {
  width: 100%;
}

#allmap {
  height: 500px;
  width: 100%;
}

#r-result {
  width: 100%;
  font-size: 14px;
}
</style>
