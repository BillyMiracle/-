# 写在前面

最近学习了一下高德地图的SDK的使用，确实挺神奇的，下面进行一个小总结：

# 准备工作
首先，我们要把需要的库导入项目，我使用 CocoaPods 安装了 SDK：
Podfile内容（详情参见[官方说明](https://lbs.amap.com/api/ios-sdk/summary)）：
```objectivec
platform :ios, '7.0'
target 'MapTest' do
pod 'AMap3DMap' 
pod 'AMapSearch'
pod 'AMapLocation'
pod 'AMapTrack'
end
```
其次，我们要为我们的项目获取一个key，它是在[高德开放控制平台](https://console.amap.com/dev)申请的。[如何获取](https://lbs.amap.com/api/ios-sdk/guide/create-project/get-key)。

# 开始肝活
## 添加权限：
![请添加图片描述](https://img-blog.csdnimg.cn/676c63ed8ce946deab1adf7a8c0fa95e.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAQmlsbHkgTWlyYWNsZQ==,size_20,color_FFFFFF,t_70,g_se,x_16)
开启定位权限。
在 TARGETS->Singings & Capabilities 中：
![请添加图片描述](https://img-blog.csdnimg.cn/513a29072bf34342b68ad3eeb1715222.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAQmlsbHkgTWlyYWNsZQ==,size_20,color_FFFFFF,t_70,g_se,x_16)
开启后台定位。
## 把地图显示到自己的view上：

```objectivec
- (BOOL)application:(UIApplication *)application didFinishLaunchingWithOptions:(NSDictionary *)launchOptions {
    // Override point for customization after application launch.
    [AMapServices sharedServices].apiKey = @"one’s key";
    return YES;
}
```

```objectivec
[AMapServices sharedServices].enableHTTPS = YES;
_mapView = [[MAMapView alloc] initWithFrame: CGRectMake(0, 100, self.view.bounds.size.width, self.view.bounds.size.height - 200)];
//把地图添加至view
[self.view addSubview:_mapView];
```
这样地图就显示出来了。
此外，地图还可以有一系列的附加属性可供自定义，下面给出一些示例：

```objectivec
_mapView.showsIndoorMap = YES;	//设置显示室内地图
_mapView.zoomLevel = 18;	//设置缩放比例
_mapView.zoomEnabled = YES;    //NO表示禁用缩放手势，YES表示开启
_mapView.rotateEnabled= NO;    //NO表示禁用旋转手势，YES表示开启
[_mapView setShowsCompass:NO];	//是否显示自带的指南针：NO
```
（PS：iOS虚拟机的双指缩放手势是按住 option 键，[参考博客](https://www.jianshu.com/p/3682048009f0)）
## 显示自己的位置：

```objectivec
//如果您需要进入地图就显示定位小蓝点，则需要下面两行代码
_mapView.showsUserLocation = YES;
_mapView.userTrackingMode = MAUserTrackingModeFollow;
```
![请添加图片描述](https://img-blog.csdnimg.cn/e3c01620d94a4395806d230f75cb5587.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAQmlsbHkgTWlyYWNsZQ==,size_13,color_FFFFFF,t_70,g_se,x_16)
效果如图，我们已经出现在地图上啦，嘻嘻😁。
接下来，如果对于我们的代表：小蓝点 不太满意的话，我们可以对它进行自定义：

```objectivec
#pragma mark - 自定义UserLocationRepresentation
    MAUserLocationRepresentation *r = [[MAUserLocationRepresentation alloc] init];
    r.showsAccuracyRing = NO;//精度圈是否显示，默认YES
    r.showsHeadingIndicator = NO;//是否显示方向指示(MAUserTrackingModeFollowWithHeading模式开启)。默认为YES
    r.fillColor = [UIColor redColor];//精度圈 填充颜色, 默认 kAccuracyCircleDefaultColor
    r.strokeColor = [UIColor blueColor];//精度圈 边线颜色, 默认 kAccuracyCircleDefaultColor
    r.lineWidth = 2;//精度圈 边线宽度，默认0
    r.enablePulseAnnimation = NO;//内部蓝色圆点是否使用律动效果, 默认YES
    r.locationDotBgColor = [UIColor greenColor];//定位点背景色，不设置默认白色
    r.locationDotFillColor = [UIColor grayColor];//定位点蓝色圆点颜色，不设置默认蓝色
    r.image = [UIImage imageNamed:@"你的图片"]; //定位图标, 与蓝色原点互斥
    [_mapView updateUserLocationRepresentation: r];
```
对它肆意妄为😈。
## 添加标记点：

```objectivec
#pragma mark - 添加标记点
	MAPointAnnotation *pointAnnotation = [[MAPointAnnotation alloc] init];
    pointAnnotation.coordinate = CLLocationCoordinate2DMake(34.154305,108.90217);
    pointAnnotation.title = @"西安邮电大学";
    pointAnnotation.subtitle = @"好地方";
    [_mapView addAnnotation:pointAnnotation];
```
为了找到你心仪❤️的点，可以使用：[高德坐标拾取器](https://lbs.amap.com/tools/picker)
接下来，我们可以对大头针进行自定义：

```objectivec
- (MAAnnotationView *)mapView:(MAMapView *)mapView viewForAnnotation:(id <MAAnnotation>)annotation {
    if ([annotation isKindOfClass:[MAPointAnnotation class]] && ![annotation isKindOfClass:[MAUserLocation class]]) {
        static NSString *pointReuseIndentifier = @"pointReuseIndentifier";
        MAPinAnnotationView *annotationView = (MAPinAnnotationView*)[mapView dequeueReusableAnnotationViewWithIdentifier:pointReuseIndentifier];
        if (annotationView == nil) {
            annotationView = [[MAPinAnnotationView alloc] initWithAnnotation:annotation reuseIdentifier:pointReuseIndentifier];
        }
        annotationView.canShowCallout= YES;       //设置气泡可以弹出，默认为NO
        annotationView.animatesDrop = YES;        //设置标注动画显示，默认为NO
        annotationView.draggable = YES;        //设置标注可以拖动，默认为NO
        annotationView.pinColor = MAPinAnnotationColorPurple;
        return annotationView;
    }
    return nil;
}
```
其实，我们的小蓝点也可能被修改，所以才需要加上`if ([annotation isKindOfClass:[MAPointAnnotation class]] && ![annotation isKindOfClass:[MAUserLocation class]])`条件来筛选。效果如下：

![请添加图片描述](https://img-blog.csdnimg.cn/644d7d46b59b486fb3f517bc4b9a86c5.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAQmlsbHkgTWlyYWNsZQ==,size_13,color_FFFFFF,t_70,g_se,x_16)

## 运动轨迹：
我们在学校经常会被要求跑步，还记入成绩，那么我们今天也来模仿，写一下这种功能，记录跑步距离，与配速。
我们的需求有三个条件：记录跑步经过各点，记录跑步时间，记录跑步距离。
实时更新位置：
```objectivec
- (void)location {
    if([CLLocationManager locationServicesEnabled]){
        locationManager = [[AMapLocationManager alloc]init];
        [locationManager setDelegate:self];
        //是否允许后台定位。默认为NO。只在iOS 9.0及之后起作用。设置为YES的时候必须保证 Background Modes 中的 Location updates 处于选中状态，否则会抛出异常。由于iOS系统限制，需要在定位未开始之前或定位停止之后，修改该属性的值才会有效果。
        [locationManager setAllowsBackgroundLocationUpdates:NO];
        //指定定位是否会被系统自动暂停。默认为NO。
        [locationManager setPausesLocationUpdatesAutomatically:NO];
        //设定定位的最小更新距离。单位米，默认为 kCLDistanceFilterNone，表示只要检测到设备位置发生变化就会更新位置信息
        [locationManager setDistanceFilter:5];
        //设定期望的定位精度。单位米，默认为 kCLLocationAccuracyBest
        [locationManager setDesiredAccuracy:kCLLocationAccuracyBest];
        //开始定位服务
        [locationManager startUpdatingLocation];
    }
}
```
实现MAMapViewDelegate的函数：
```objectivec
#pragma mark - MAMapViewDelegate
- (void)mapView:(MAMapView *)mapView didUpdateUserLocation:(MAUserLocation *)userLocation updatingLocation:(BOOL)updatingLocation {
    
    _currentLocation = userLocation;
//    NSLog(@"位置更新");
//    NSLog(@"当前位置：%f,%f", userLocation.location.coordinate.latitude, userLocation.location.coordinate.longitude);
    if (_longitudeArray.count == 0) {
    	//更新距离信息
        [self updateLength];
        /*添加点信息*/
    } else {
        /*取轨迹最后一点信息，计算距离*/
        //判断两个点的距离  大于等于1米时才画下一条路径
        if (distance >= 1) {
            _trackLength = [NSNumber numberWithDouble:_trackLength.doubleValue + distance];
            [self updateLength];
            //构造commonPolylineCoords
            //构造折线对象
            MAPolyline *commonPolyline = [MAPolyline polylineWithCoordinates:commonPolylineCoords count:2];
            //在地图上添加折线对象
            _isRoute = NO;
            [_mapView addOverlay: commonPolyline];
        }
        
    }
}
```
此外，我还添加了定时器。让配速一秒钟刷新一次：

```objectivec
//更新配速
- (void)updateSpeed {
    NSDate *currentDate = [NSDate date];
    // 时间2与时间1之间的时间差（秒
    double intervalTime = [currentDate timeIntervalSinceDate:_startDate];
    if (_trackLength.doubleValue < 1) {
        _speedLabel.text = @"0分0秒";
    } else {
        _speedLabel.text = [NSString stringWithFormat:@"%d分%d秒", (int)(intervalTime / 60 / _trackLength.doubleValue * 1000), (int)((intervalTime / 60 / _trackLength.doubleValue * 1000 - (int)(intervalTime / 60 / _trackLength.doubleValue * 1000)) * 60)];
    }
}
```
效果如图：
![在这里插入图片描述](https://img-blog.csdnimg.cn/ed1b0ae41079438780526983d3716a75.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAQmlsbHkgTWlyYWNsZQ==,size_16,color_FFFFFF,t_70,g_se,x_16#pic_center)
## 搜索功能：
搜索可以通过当前位置搜索附近内容，也可以通过输入关键字查找位置。
可以使用提供的 searchController 的 searchBar。
```objectivec
    _searchController = [[UISearchController alloc] initWithSearchResultsController:nil];
    _searchController.searchResultsUpdater = self;
    _searchController.searchBar.delegate = self;
```

也可以使用自己创建的 UITextField。

```objectivec
//搜索框激活时，使用提示搜索
- (void)updateSearchResultsForSearchController:(UISearchController *)searchController {
    //发起输入提示搜索
    AMapInputTipsSearchRequest *tipsRequest = [[AMapInputTipsSearchRequest alloc] init];
    //关键字
    tipsRequest.keywords = _searchController.searchBar.text;
    //城市
    tipsRequest.city = _currentCity;
    //执行搜索
    [_search AMapInputTipsSearch: tipsRequest];
}

//实现输入提示的回调函数
- (void)onInputTipsSearchDone:(AMapInputTipsSearchRequest*)request response:(AMapInputTipsSearchResponse *)response {
    if(response.tips.count == 0) {
        return;
    }
    //通过AMapInputTipsSearchResponse对象处理搜索结果
    //先清空数组
    [self.searchList removeAllObjects];
    for (AMapTip *p in response.tips) {    
        //把搜索结果存在数组
        [self.searchList addObject:p];
    }
    _isSelected = NO;
    //刷新表视图
    [self.searchTableView reloadData];
}
```
这样就可以实现搜索功能并把数据显示在屏幕上啦：
![在这里插入图片描述](https://img-blog.csdnimg.cn/9e134e9a2d934766aace5608bb3f2586.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAQmlsbHkgTWlyYWNsZQ==,size_15,color_FFFFFF,t_70,g_se,x_16#pic_center)
除了输入搜索目的地，我们还可以进行周边搜索。

```objectivec
- (void)searchAction {
    //初始化检索对象
    _search = [[AMapSearchAPI alloc] init];
    _search.delegate = self;
    
    //构造AMapPOIAroundSearchRequest对象，设置周边请求参数
    AMapPOIAroundSearchRequest *request = [[AMapPOIAroundSearchRequest alloc] init];
    //当前位置
    request.location = [AMapGeoPoint locationWithLatitude:_currentLocation.coordinate.latitude longitude:_currentLocation.coordinate.longitude];
    
    //关键字
//    request.keywords =
    // types属性表示限定搜索POI的类别，默认为：餐饮服务|商务住宅|生活服务
    // POI的类型共分为20种大类别，分别为：
    // 汽车服务|汽车销售|汽车维修|摩托车服务|餐饮服务|购物服务|生活服务|体育休闲服务|
    // 医疗保健服务|住宿服务|风景名胜|商务住宅|政府机构及社会团体|科教文化服务|
    // 交通设施服务|金融保险服务|公司企业|道路附属设施|地名地址信息|公共设施
//    request.types = @"餐饮服务|生活服务";
    request.radius =  5000;//查询半径，范围：0-50000，单位：米 [default = 3000]
    request.sortrule = 0;
    request.requireExtension = YES;
    //发起周边搜索
    [_search AMapPOIAroundSearch:request];
}
//实现POI搜索对应的回调函数
- (void)onPOISearchDone:(AMapPOISearchBaseRequest *)request response:(AMapPOISearchResponse *)response {
    if(response.pois.count == 0) {
        return;
    }
    //通过 AMapPOISearchResponse 对象处理搜索结果
    [self.dataList removeAllObjects];
    for (AMapPOI *p in response.pois) {
        //搜索结果存在数组
        [self.dataList addObject:p];
    }
    _isSelected = YES;
    [self.searchTableView reloadData];
}
```
这样以来，我们就可以对周边特定范围内的特定种类的地点进行搜索，效果如下：
![在这里插入图片描述](https://img-blog.csdnimg.cn/9aa063cdef06481f838476410b363efa.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAQmlsbHkgTWlyYWNsZQ==,size_15,color_FFFFFF,t_70,g_se,x_16#pic_center)
## 路线的搜索与显示：
为了搜索路线，我们要给出起始点与终点。用它们来查找路线。

```objectivec
AMapDrivingRouteSearchRequest *navi = [[AMapDrivingRouteSearchRequest alloc] init];
/* 出发点. */
navi.origin = [AMapGeoPoint locationWithLatitude:self.currentLocation.coordinate.latitude longitude:self.currentLocation.coordinate.longitude];
/* 目的地. */
navi.destination = [AMapGeoPoint locationWithLatitude:poi.location.latitude longitude:poi.location.longitude];
navi.requireExtension = YES;
//发起路线规划
[self.searchAPI AMapDrivingRouteSearch:navi];
```
搜索完成调用函数：

```objectivec
- (void)onRouteSearchDone:(AMapRouteSearchBaseRequest *)request response:(AMapRouteSearchResponse *)response {
    if (response.route == nil) {
        return;
    }
    if (response.count > 0) {
        //直接取第一个方案
        AMapPath *path = response.route.paths[0];
        //移除旧折线对象
        [_mapView removeOverlay:_polyline];
        //构造折线对象
        _polyline = [self polylinesForPath:path];
        //添加新的遮盖，然后会触发代理方法(- (MAOverlayRenderer *)mapView:(MAMapView *)mapView rendererForOverlay:(id<MAOverlay>)overlay)进行绘制
        _isRoute = YES;
        [_mapView addOverlay:_polyline];
        [_mapView showOverlays:@[_polyline] animated:NO];
    }
}
```
其中，`[_mapView showOverlays:@[_polyline] animated:NO];`这句话是为了能够让地图大小自适应，显示整条路线。
路线解析与坐标解析方法：

```objectivec
//路线解析
- (MAPolyline *)polylinesForPath:(AMapPath *)path {
    if (path == nil || path.steps.count == 0){
        return nil;
    }
    NSMutableString *polylineMutableString = [@"" mutableCopy];
    for (AMapStep *step in path.steps) {
        [polylineMutableString appendFormat:@"%@;",step.polyline];
    }
    NSUInteger count = 0;
    CLLocationCoordinate2D *coordinates = [self coordinatesForString:polylineMutableString coordinateCount:&count parseToken:@";"];
    MAPolyline *polyline = [MAPolyline polylineWithCoordinates:coordinates count:count];
    free(coordinates);
    coordinates = NULL;
    return polyline;
}

//解析经纬度
- (CLLocationCoordinate2D *)coordinatesForString:(NSString *)string coordinateCount:(NSUInteger *)coordinateCount parseToken:(NSString *)token {
    if (string == nil){
        return NULL;
    }
    if (token == nil){
        token = @",";
    }
    NSString *str = @"";
    if (![token isEqualToString:@","]){
        str = [string stringByReplacingOccurrencesOfString:token withString:@","];
    } else {
        str = [NSString stringWithString:string];
    }
    NSArray *components = [str componentsSeparatedByString:@","];
    NSUInteger count = [components count] / 2;
    if (coordinateCount != NULL){
        *coordinateCount = count;
    }
    CLLocationCoordinate2D *coordinates = (CLLocationCoordinate2D*)malloc(count * sizeof(CLLocationCoordinate2D));
    for (int i = 0; i < count; i++) {
        coordinates[i].longitude = [[components objectAtIndex:2 * i]     doubleValue];
        coordinates[i].latitude  = [[components objectAtIndex:2 * i + 1] doubleValue];
    }
    return coordinates;
}
```

在绘制方法中自定义折线形状：

```objectivec
MAPolylineRenderer *polylineRenderer = [[MAPolylineRenderer alloc] initWithPolyline:overlay];
if (_isRoute) {
    polylineRenderer.lineWidth = 8.f;
    polylineRenderer.strokeColor = [UIColor systemGreenColor];
} else {
    polylineRenderer.lineWidth = 5.f;
    polylineRenderer.strokeColor = [UIColor colorWithRed:arc4random()%256/255.0 green:arc4random()%256/255.0 blue:arc4random()%256/255.0 alpha:0.6];
}
polylineRenderer.lineJoinType = kMALineJoinRound;
polylineRenderer.lineCapType = kMALineCapRound;
```
效果：
![在这里插入图片描述](https://img-blog.csdnimg.cn/e175fd7082d84756a6f56f301757f434.png?x-oss-process=image/watermark,type_d3F5LXplbmhlaQ,shadow_50,text_Q1NETiBAQmlsbHkgTWlyYWNsZQ==,size_14,color_FFFFFF,t_70,g_se,x_16#pic_center)

# 小结：
博主萌新在写这个学习小demo时遇到过很多问题，但都有努力的去解决，希望我的博客可以帮助到大家，谢谢大家💖，喜欢的话可以点个赞和关注吧，我们一起学习，一起进步。
