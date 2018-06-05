## 高德地图、百度地图 使用案例

效果图
![效果图](https://github.com/ec2ec/ec.snippet/raw/master/map.api/mapapi.gif)

### 本案例演示了【高德地图】的
[x] 坐标与地址
将经纬度转换成地址描述
[2] 标注点
在指定位置上标注点
搜索功能


### 首先要去高德开放平台注册成为开发者
### 然后到控制台创建应用
### 接着为创建好的应用添加新Key，以后引入js文件的时候使用此Key
> 以后可以到 数据分析 里面查看地图接口使用的统计数据

### 引入js
>`<script type="text/javascript" src='https://webapi.amap.com/maps?v=1.4.0&key=你的key&plugin=AMap.Autocomplete,AMap.PlaceSearch,AMap.Geocoder'></script>`

### 代码（vue）
``` Vuejs
toggleMap: function () {
    this.Map.show = !this.Map.show

    if (!this.Map.isLoaded) {
        this.Map.isLoaded = true;

        //地图加载
        var map = new AMap.Map("map", { resizeEnable: true });
        var geocoder = new AMap.Geocoder({ city: "010" });//城市，默认：“全国”
        var that = this;
        marker = NaN;

        //========== 标注已有坐标点 ===========
        //Latitude:"34.364621000"
        //Longitude:"114.34.."
        var __lat = this.model.EditShop.Store.Latitude;
        var __lng = this.model.EditShop.Store.Longitude;
        if (__lat != "" && __lng != "")
            marker = new AMap.Marker({ map: map, position: { N: __lng, Q: __lat, lng: __lng, lat: __lat } });

        //为地图注册click事件获取鼠标点击出的经纬度坐标
        var clickEventListener = map.on('click', function (e) {
            var l = String(e.lnglat.getLng()).split(".")[0]
            var r = String(e.lnglat.getLng()).split(".")[1]
            r += "000000000".substring(0, 9 - r.length)
            that.model.EditShop.Store.Longitude = l + '.' + r

            l = String(e.lnglat.getLat()).split(".")[0]
            r = String(e.lnglat.getLat()).split(".")[1]
            r += "000000000".substring(0, 9 - r.length)
            that.model.EditShop.Store.Latitude = l + '.' + r

            //Web服务
            geocoder.getAddress(e.lnglat, function (status, result) {
                if (status == 'complete') {
                    that.model.EditShop.Store.DistrictCode = result.regeocode.addressComponent.adcode
                    that.model.EditShop.Store.ProvinceCode = result.regeocode.addressComponent.adcode.substring(0, 2) + '0000'
                    that.model.EditShop.Store.CityCode = result.regeocode.addressComponent.adcode.substring(0, 4) + '00'
                    var _dpc = result.regeocode.addressComponent.province + result.regeocode.addressComponent.city + result.regeocode.addressComponent.district
                    that.model.EditShop.Store.Address = result.regeocode.formattedAddress.replace(_dpc, "")
                } else {
                    that.model.EditShop.Store.Address = '无法获取地址'
                }
            })

            //========== 标注点 ===========
            if (marker) marker.setMap(null)
            marker = new AMap.Marker({ map: map, position: e.lnglat });
            marker.setAnimation('AMAP_ANIMATION_BOUNCE');
        });

        //搜索功能  输入提示  Web端
        var autoOptions = { input: "map_tipinput" };
        var auto = new AMap.Autocomplete(autoOptions);
        var placeSearch = new AMap.PlaceSearch({ map: map }); //构造地点查询类
        AMap.event.addListener(auto, "select", select);     //注册监听，当选中某条记录时会触发
        function select(e) {
            placeSearch.setCity(e.poi.adcode);
            placeSearch.search(e.poi.name);                 //关键字查询查询
        }
    }
},
```
