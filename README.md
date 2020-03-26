# DGMovie
破解影视APP广告 可以看美剧（反编译学习）

[项目地址](https://e.coding.net/jqorz/DGMoive.git)

# 已破解包的地址
- [DG_1.1.1_c3.apk](https://jqorz.coding.net/s/1b9b16a7-cd22-484b-8676-f0f53a5b05e3)

# 工具
- [framework-res-28.apk](https://jqorz.coding.net/s/cdc6fc5f-8738-4639-8458-178583d4f705)
- [apktool_2.4.1.jar](https://jqorz.coding.net/s/ccff6d04-a986-41f3-8c07-e67a4539ca9c)
- [jd-gui-windows-1.6.6.zip](https://jqorz.coding.net/s/83fb3cc2-bac4-4d5b-adca-90e6d87c77bf)
- [baksmali_2.2.5.jar](https://jqorz.coding.net/s/51a8f52b-dc23-4362-bbc9-d912fd079975)
- [dex2jar 2.1.zip](https://jqorz.coding.net/s/d68f9414-caab-4fd9-850f-d821a8433446)
- [Android逆向助手2.2.zip](https://jqorz.coding.net/s/44b7a766-1ad5-4934-8955-6d1bd9b44c40)

### 冬瓜影视去广告步骤记录

使用环境 `AndroidKiller 1.3.1.0 `， `apktool-2.4.1 ` ，`dex2jar-2.1`，`framework-res-28.apk`，`JD-GUI—-1.6.6`

> 配置环境：在AK中打开APKTOOL管理器，指定为apktool-2.4.1
>
> AndroidStudio中安装java2smali插件，新建一个空项目，用于自己写想要的java代码，然后转成对应的smali添加到原smali中

>smali 语法教程1 https://blog.csdn.net/lpohvbe/article/details/7981386
>
>smali 语法教程2 https://www.cnblogs.com/lee0oo0/p/3728271.html

1. 使用AK编译，直接报错

   解决方法：替换apktool由默认的ShakeTools改为apktools2.4.1

2. 回编译报错，提示`No resource identifier found for attribute 'compileSdkVersion' in package 'android'`

   解决方法：提取AVD的28版本的系统镜像system.img/sytem/framework/framework-res.apk
   替换到C:\Users\用户名\apktool\framework\1.apk

   注意：最好用新版的，DG影视的sdk是28，用26的资源框架反编译会报错

3. 此时回编译打包成功，安装后打开，提示“您的安装包异常”

   解决方法：使用dex2jar将两个dex都转成jar，然后全局搜索"您的安装包异常"；或者使用AK搜索unicode转义字符

4. 定位到判断逻辑，位于com.waxgourd.wg.WaxgourdApp里，通过KS()方法判断，如果这个方法返回true，则会退出应用。

   ```
     public boolean KS()
     {
       String str1 = EncryptUtils.getSHA1FromJNI();
       String str2 = com.waxgourd.wg.utils.b.r(this, getPackageName(), "SHA1");
       StringBuilder localStringBuilder = new StringBuilder();
       localStringBuilder.append("sha1Now = ");
       localStringBuilder.append(str2);
       Log.d("WaxgourdApp", localStringBuilder.toString());
       return TextUtils.equals(str1, str2) ^ true;
     }
   ```

   因此，修改817行的`xor-int/lit8 v0, v0, 0x1`为`xor-int/lit8 v0, v0, 0x0`

5. 修改后，提示新版本，修改`AndroidManifest.xml`中的`platformBuildVersionCode`，改高点，改为119，打包发现还是提醒，然后修改`apktool.yml`文件里的`VersionCode`为100，解决。

6. 开始去广告。首先在AndroidManifest.xml的`application`标签里增加`android:debuggable="true"`，然后使用AndroidStudio的布局查看器，定位到列表里的一个广告，是个ImageView，id为`iv_cover`，全局检索此id，定位到布局文件为`bean_recycle_item_video_ad`，布局id为`0x7f0c0073`，修改`bean_recycle_item_video_ad`中的ImageView的高度为0dp

   ```
   <ImageView android:id="@id/iv_cover" android:background="#fff2f2f2" android:paddingTop="6.0dip" android:layout_width="fill_parent" android:layout_height="0dip"
   ```

7. 为了防止广告还是被点击到，在全局检索布局文件`bean_recycle_item_video_ad`的id`0x7f0c0073`，定位到代码类`com/waxgourd/wg/ui/viewbinder/b.smali`在`b.smali`中发现，广告信息都存放在`AdInfoBean`这个实体类中，进入`com/waxgourd/wg/javabean/AdInfoBean`，把`getAdUrl`返回值改为null，把`getAdPic`返回值改为`""`

   ```
   .method public getAdUrl()Ljava/lang/String;
       .locals 1
   
       .line 50
       const/4 v0, 0x0
   
       return-object v0
   .end method
   ```

   ```
   .method public getAdPic()Ljava/lang/String;
       .locals 1
   
       .line 42
       const-string v0, ""
   
       return-object v0
   .end method
   ```

   此时已经成功的去除了列表下部分中的一个广告。

   > 看日志提示图片加载失败，所以把`getAdPic`改回了
   >
   > ```
   > iget-object v0, p0, Lcom/waxgourd/wg/javabean/AdInfoBean;->adPic:Ljava/lang/String;
   > 
   > return-object v0
   > ```

8. 启动页的等待时间有点长，把时间缩短。计时代码在`com.waxgourd.wg.module.splash.SplashPresenter`的`countDownSplash()`

   ```
   .line 67
       sget-object v0, Ljava/util/concurrent/TimeUnit;->SECONDS:Ljava/util/concurrent/TimeUnit;
   
       const-wide/16 v1, 0x2
   
    invoke-static {v1, v2, v0}, La/a/m;->b(JLjava/util/concurrent/TimeUnit;)La/a/m;
   
   ```

   中的`0x2`修改为`0x0`

9. 继续去除启动页广告。查看`AdActivity`里代码，发现有个关闭界面的方法

   ```
    .line 65
          iget-object p1, p0, Lcom/waxgourd/wg/module/ad/AdActivity$b;->bMY:Lcom/waxgourd/wg/module/ad/AdActivity;
          
          invoke-virtual {p1}, Lcom/waxgourd/wg/module/ad/AdActivity;->finish()V
   ```

      查看`AdActivtiy`，发现它的父类BaseActivity中`onCreate()`最先执行的方法是`Lv()`，于是在`AdActivity`中的`Lv()`添加finish方法

   ```
   .line 56
           
          iget-object v0, p0, Lcom/waxgourd/wg/module/ad/AdActivity;->bWX:Lcom/waxgourd/wg/framework/BasePresenter;
   ```
   ```
   .line 56
   
    iget-object p1, p0, Lcom/waxgourd/wg/module/ad/AdActivity$b;->bMY:Lcom/waxgourd/wg/module/ad/AdActivity;
       
    invoke-virtual {p1}, Lcom/waxgourd/wg/module/ad/AdActivity;->finish()V
       
     iget-object v0, p0, Lcom/waxgourd/wg/module/ad/AdActivity;->bWX:Lcom/waxgourd/wg/framework/BasePresenter;
   ```

   发现这样会崩溃。

10. 然后准备修改倒计时的时间，`com.waxgourd.wg.module.ad.AdPresenter`中`countDownSplash`方法为倒计时方法，修改时间由5为0，运行后发现只是把计时器关了，还是需要手动在广告界面点击关闭按钮。

11. 发现广告界面有个`mLayoutSkip`，定位到.line63` v0`是`mLayoutSkip`的寄存器。模拟点击即可跳过广告界面

    ```
           invoke-virtual {v0, v1}, Landroid/widget/FrameLayout;->setOnClickListener(Landroid/view/View$OnClickListener;)V
        
            .line 67
    ```

    改为

    ```
     invoke-virtual {v0, v1}, Landroid/widget/FrameLayout;->setOnClickListener(Landroid/view/View$OnClickListener;)V
    
        invoke-virtual {v0}, Landroid/view/View;->performClick()Z
    
        .line 67
    ```

    广告启动页成功去除


12. 准备去除顶部轮播图的广告。发现顶部轮播图的广告是可以点击跳转的，普通视频则不会，所以想通过数据判断是否为正常视频，否则就从适配器的数据源中剔除。通过布局查看器，发现轮播图的控件是`Banner`，id为`banner_recommend_fragment`，全局检索，使用该控件的布局是`bean_fragment_recommend.xml`，id为`0x7f0c0046`，检索发现对应的类为`com.waxgourd.wg.module.videorecommend.VideoRecommendFragment`，发现里面往Banner塞数据的是`VideoRecommendFragment`的`a(List<String> paramList1, List<String> paramList2, List<BannerBean> paramList)`方法，发现里面有调用`k.i()`进行输出，所以定位到`com.waxgourd.wg.utils.k`，发现这应该是一个日志输出类，但是作者把Log日志去掉了。

13. 把Log日志加到`com.waxgourd.wg.utils.k`中的`e()`,`d()`,`i()`方法

    ```
    .method public static i(Ljava/lang/String;Ljava/lang/String;)V
        .locals 0
        
        return-void
    .end method
    ```

    改为

    ```
    .method public static i(Ljava/lang/String;Ljava/lang/String;)V
        .locals 0
        
        .param p0, "paramString1"    # Ljava/lang/String;
        .param p1, "paramString2"    # Ljava/lang/String;
    
        .prologue
        invoke-static {p0, p1}, Landroid/util/Log;->i(Ljava/lang/String;Ljava/lang/String;)I
    
                
        return-void
    .end method
    ```

    `e()`和`d()`类似，这样就把作者去除的日志都打印出来了

14. 通过查看数据，打印了解密后的网络请求数据

    ```
    {"bannerData":[{"id":"76","phone_type":"3","slide_pic":"https:\/\/storage.taifutj.com\/admin\/202002151315902new.jpg","target_name":"\u8425\u6551","type":"1","zt_id":null,"vod_id":"143606","web_url":"","android_router":"","ios_router":{},"banner_content":"\u6700\u65b0\u7535\u5f71\u2014\u89e3\u653e\u00b7\u7ec8\u5c40\u8425\u6551","pid_banner":"1","zt_router":{"zt_tag":"","zt_type":"","zt_pid":2}},{"id":"75","phone_type":"3","slide_pic":"https:\/\/storage.taifutj.com\/admin\/202002151313311new.jpg","target_name":"tibfd","type":"1","zt_id":null,"vod_id":"140522","web_url":"","android_router":"","ios_router":{},"banner_content":"\u52c7\u6562\u8005\u6e38\u620f2\u2014\u6700\u65b0\u9ad8\u6e05\u7248","pid_banner":"1","zt_router":{"zt_tag":"","zt_type":"","zt_pid":2}},{"id":"74","phone_type":"3","slide_pic":"https:\/\/storage.taifutj.com\/admin\/202002100945223new.jpg","target_name":"\u4e0a\u53e4\u6c90\u6708","type":"1","zt_id":null,"vod_id":"143539","web_url":"","android_router":"","ios_router":{},"banner_content":"\u5434\u78ca\u738b\u4fca\u51ef\u70ed\u8840\u96c6\u7ed3\uff01\u4e0a\u53e4\u5bc6\u7ea6\uff01","pid_banner":"1","zt_router":{"zt_tag":"","zt_type":"","zt_pid":2}},{"id":"73","phone_type":"3","slide_pic":"https:\/\/storage.taifutj.com\/admin\/202002100944487new.jpg","target_name":"xiayizhan","type":"1","zt_id":null,"vod_id":"143145","web_url":"","android_router":"","ios_router":{},"banner_content":"\u4e0b\u4e00\u7ad9\u662f\u5e78\u798f\u2014\u706b\u70ed\u8fde\u66f4\u4e2d\uff01","pid_banner":"1","zt_router":{"zt_tag":"","zt_type":"","zt_pid":2}},{"id":"70","phone_type":"3","slide_pic":"https:\/\/storage.taifutj.com\/admin\/20200204194039new.jpg","target_name":"\u5927\u4e3b\u5bb0","type":"1","zt_id":null,"vod_id":"143255","web_url":"","android_router":"","ios_router":{},"banner_content":"\u738b\u6e90\u6b27\u9633\u5a1c\u5a1c\u5c11\u5e74\u5f81\u9014","pid_banner":"1","zt_router":{"zt_tag":"","zt_type":"","zt_pid":2}},{"id":"37","phone_type":"3","slide_pic":"https:\/\/storage.taifutj.com\/admin\/20200129181220new.jpg","target_name":"679\u68cb\u724c","type":"13","zt_id":null,"vod_id":"","web_url":"http:\/\/679966.cn:679\/679.html?shareName=xg123119xg.com","android_router":"","ios_router":{},"banner_content":"679\u68cb\u724c\uff1a\u73b0\u5728\u4e0b\u8f7d\u6ce8\u518c\uff0c\u7acb\u9001679\u5143\u5927\u793c\uff01","pid_banner":"1","zt_router":{"zt_tag":"","zt_type":"","zt_pid":2}},{"id":"62","phone_type":"3","slide_pic":"https:\/\/storage.taifutj.com\/admin\/202001251850446new.jpg","target_name":"\u4e09\u751f\u4e09\u4e16","type":"1","zt_id":null,"vod_id":"140864","web_url":"","android_router":"","ios_router":{},"banner_content":"\u4e09\u751f\u4e09\u4e16\u518d\u7eed\u524d\u7f18","pid_banner":"1","zt_router":{"zt_tag":"","zt_type":"","zt_pid":2}},{"id":"46","phone_type":"3","slide_pic":"https:\/\/storage.taifutj.com\/admin\/202001311825415new.jpg","target_name":"one","type":"13","zt_id":null,"vod_id":"","web_url":"https:\/\/one6.app\/","android_router":"","ios_router":{},"banner_content":"ONE\uff1a\u6210\u4eba\u7684\u4e16\u754c\uff0c\u4e00\u4e2a\u5c31\u591f\u4e86\u3002","pid_banner":"1","zt_router":{"zt_tag":"","zt_type":"","zt_pid":2}}],"noticeData":{"id":0,"notice_name":"","notice_pic":"","notice_cont":"","type":"","vod_id":"","web_url":"","zt_id":"","android_router":"","ios_router":[],"zt_router":{"zt_tag":"","zt_type":"","zt_pid":2},"end_time":""},"noticeNewData":[]}
    2020-02-20 23:57:24.999 16641-16933/com.waxgourd.wg I/JsonConvertFactory: jsonResult : {"code":200,"msg":"succ","data":{"bannerData":[{"id":"76","phone_type":"3","slide_pic":"https://storage.taifutj.com/admin/202002151315902new.jpg","target_name":"营救","type":"1","vod_id":"143606","web_url":"","android_router":"","ios_router":{},"banner_content":"最新电影—解放·终局营救","pid_banner":"1","zt_router":{"zt_tag":"","zt_type":"","zt_pid":2}},{"id":"75","phone_type":"3","slide_pic":"https://storage.taifutj.com/admin/202002151313311new.jpg","target_name":"tibfd","type":"1","vod_id":"140522","web_url":"","android_router":"","ios_router":{},"banner_content":"勇敢者游戏2—最新高清版","pid_banner":"1","zt_router":{"zt_tag":"","zt_type":"","zt_pid":2}},{"id":"74","phone_type":"3","slide_pic":"https://storage.taifutj.com/admin/202002100945223new.jpg","target_name":"上古沐月","type":"1","vod_id":"143539","web_url":"","android_router":"","ios_router":{},"banner_content":"吴磊王俊凯热血集结！上古密约！","pid_banner":"1","zt_router":{"zt_tag":"","zt_type":"","zt_pid":2}},{"id":"73","phone_type":"3","slide_pic":"https://storage.taifutj.com/admin/202002100944487new.jpg","target_name":"xiayizhan","type":"1","vod_id":"143145","web_url":"","android_router":"","ios_router":{},"banner_content":"下一站是幸福—火热连更中！","pid_banner":"1","zt_router":{"zt_tag":"","zt_type":"","zt_pid":2}},{"id":"70","phone_type":"3","slide_pic":"https://storage.taifutj.com/admin/20200204194039new.jpg","target_name":"大主宰","type":"1","vod_id":"143255","web_url":"","android_router":"","ios_router":{},"banner_content":"王源欧阳娜娜少年征途","pid_banner":"1","zt_router":{"zt_tag":"","zt_type":"","zt_pid":2}},{"id":"37","phone_type":"3","slide_pic":"https://storage.taifutj.com/admin/20200129181220new.jpg","target_name":"679棋牌","type":"13","vod_id":"","web_url":"http://679966.cn:679/679.html?shareName\u003dxg123119xg.com","android_router":"","ios_router":{},"banner_content":"679棋牌：现在下载注册，立送679元大礼！","pid_banner":"1","zt_router":{"zt_tag":"","zt_type":"","zt_pid":2}},{"id":"62","phone_type":"3","slide_pic":"https://storage.taifutj.com/admin/202001251850446new.jpg","target_name":"三生三世","type":"1","vod_id":"140864","web_url":"","android_router":"","ios_router":{},"banner_content":"三生三世再续前缘","pid_banner":"1","zt_router":{"zt_tag":"","zt_type":"","zt_pid":2}},{"id":"46","phone_type":"3","slide_pic":"https://storage.taifutj.com/admin/202001311825415new.jpg","target_name":"one","type":"13","vod_id":"","web_url":"https://one6.app/","android_router":"","ios_router":{},"banner_content":"ONE：成人的世界，一个就够了。","pid_banner":"1","zt_router":{"zt_tag":"","zt_type":"","zt_pid":2}}],"noticeData":{"id":0,"notice_name":"","notice_pic":"","notice_cont":"","type":"","vod_id":"","web_url":"","zt_id":"","android_router":"","ios_router":[],"zt_router":{"zt_tag":"","zt_type":"","zt_pid":2},"end_time":""},"noticeNewData":[]}}
    ```

    格式化后发现，如果`web_url`参数不为空则是广告，调用`com.waxgourd.wg.module.videorecommend.VideoRecommendFragment`的`a(List<String> paramList1, List<String> paramList2, List<BannerBean> paramList)`方法的是`com.waxgourd.wg.module.videorecommend.VideoRecommendPresenter`的一个方法

    ```
      private void setBannerAndNotice(BannerAndNoticeListBean paramBannerAndNoticeListBean) {
        List list1 = paramBannerAndNoticeListBean.getBannerList();
        if (list1 != null) {
          int i = list1.size();
          ArrayList<String> arrayList1 = new ArrayList(i);
          ArrayList<String> arrayList2 = new ArrayList(i);
          for (BannerBean bannerBean : list1) {
            arrayList1.add(bannerBean.getSlidePic());
            arrayList2.add(bannerBean.getBanner_content());
            StringBuilder stringBuilder = new StringBuilder();
            stringBuilder.append("picUrls == ");
            stringBuilder.append(bannerBean.getSlidePic());
            stringBuilder.append(" vodId == ");
            stringBuilder.append(bannerBean.getVodId());
            stringBuilder.append(" bannerContent : ");
            stringBuilder.append(bannerBean.getBanner_content());
            k.i("VideoRecommendPresenter", stringBuilder.toString());
          } 
          ((VideoRecommendContract.b)this.mView).a(arrayList1, arrayList2, list1);
        } 
        List list2 = paramBannerAndNoticeListBean.getNoticeList();
        ((VideoRecommendContract.b)this.mView).ah(list2);
      }
    ```

    所以在for循环里做判断，如果`!TextUtil.isEmpty(bnnerBean.getWebUrl())`就continue；

    通过AndroidStudio运行java2smali插件，参考出`!TextUtil.isEmpty(bnnerBean.getWebUrl())`对应的代码。

    原smali

    ```
     	.line 53
        invoke-interface {v0}, Ljava/util/List;->iterator()Ljava/util/Iterator;
    
        move-result-object v1
    
        :goto_0
        invoke-interface {v1}, Ljava/util/Iterator;->hasNext()Z
    
        move-result v4
    
        if-eqz v4, :cond_0
    
        invoke-interface {v1}, Ljava/util/Iterator;->next()Ljava/lang/Object;
    
        move-result-object v4
    
        check-cast v4, Lcom/waxgourd/wg/javabean/BannerBean;
        
    ```

    修改为（#中为新加的）

    ```
     .line 53
        invoke-interface {v0}, Ljava/util/List;->iterator()Ljava/util/Iterator;
    
        move-result-object v1
        
        #1
        :cond_1a
        #1
        :goto_0
        invoke-interface {v1}, Ljava/util/Iterator;->hasNext()Z
    
        move-result v4
    
        if-eqz v4, :cond_0
    
        invoke-interface {v1}, Ljava/util/Iterator;->next()Ljava/lang/Object;
    
        move-result-object v4
    
        check-cast v4, Lcom/waxgourd/wg/javabean/BannerBean;
        
        #2
        invoke-virtual {v4}, Lcom/waxgourd/wg/javabean/BannerBean;->getWebUrl()Ljava/lang/String;
    
        move-result-object v5
    
        invoke-static {v5}, Landroid/text/TextUtils;->isEmpty(Ljava/lang/CharSequence;)Z
    
        move-result v5
    
        if-eqz v5, :cond_1a
        #2
                
    ```

    >:cond_1a 指定一个跳转，用于实现continue跳转
    >
    >v4寄存器中存的是for循环里的BannerBean实例
    >
    >#2
    >
    >调用bannerBean.getWebUrl()
    >
    >字符串结果存到v5寄存器
    >
    >调用TextUtils.isEmpty(v5)
    >
    >结果存到v5中
    >
    >if(v5==0) 跳转到cond_1a
    >
    >#2

    这样就实现了在for循环里做判断，如果`!TextUtil.isEmpty(bnnerBean.getWebUrl())`就continue；

    这样就成功的去除了顶部轮播图的广告

15. 接下来去除播放界面位于播放器下面的广告。通过布局查看，定位到广告是一个id为`iv_ad`的ImageView，在`bean_activity_player.xml`中，修改它的高度为0dip

    ```
    <ImageView android:id="@id/iv_ad" android:background="@color/colorDivider" android:paddingTop="3.0dip" android:paddingBottom="1.0dip" android:layout_width="fill_parent" android:layout_height="65.0dip" app:layout_constraintTop_toTopOf="parent" />
    ```

16. 接下来修改暂停和播放前的广告。暂停时的广告控件是一个id为`layout_pause_ad`的FrameLayout，使用该控件的布局id为`bean_video_player_land`和`bean_video_player_normal`，分别把这两个布局里id为`layout_pause_ad`的FrameLayout高度改为0dip。修改后发现暂停时虽然不显示广告，但是也没有显示暂停按钮，所以还原此改动。搜索发现`layout_pause_ad`的id为`0x7f09015f`，对应的十进制为`2131296607`,在`com.waxgourd.wg.ui.widget.LandLayoutVideo`中调用，成员变量名为`bZs`。

    发现`onVideoPause()`中间有个判断

    ```
    if (this.bZw.OO()) {
            this.bZr = true;
            changeAdUIState();
      } 
    ```

     `startPlayLogic()`中有个判断

    ```
    if (this.bZw != null && this.bZw.OR()) {
          this.bZn = true;
          changeAdUIState();
          return;
    } 
    ```

    所以应该把`bZw`的`OO()`和`OR()`都返回flase

    ```
      public final boolean OO() {
        if (this.bWw != null) {
          CharSequence charSequence;
          VideoAdBean videoAdBean = this.bWw;
          if (videoAdBean != null) {
            charSequence = videoAdBean.getAdType();
          } else {
            charSequence = null;
          } 
          if (!TextUtils.isEmpty(charSequence)) {
            a a1 = this.bWr;
            if (a1 != null) {
              VideoAdBean videoAdBean1 = this.bWw;
              if (videoAdBean1 == null)
                j.SU(); 
              a1.fo(videoAdBean1.getAdUrl());
            } 
            if (!this.bWq)
              OP(); 
            return true;
          } 
        } 
        return false;
      }
    ```

    所以把`com.waxgourd.wg.javabean.VideoAdBean`的`getAdType()`返回""，修改如下

    ```
    .method public final getAdType()Ljava/lang/String;
        .locals 1

        .line 8
         # iget-object v0, p0, Lcom/waxgourd/wg/javabean/VideoAdBean;->adType:Ljava/lang/String;
         const/4 v0, 0x0
           
        return-object v0
    .end method
    ```
    
    编译运行，发现播放界面的广告，包括启动播放和暂停播放的广告都已经成功隐藏
    
    ### 至此,1.1.1_c1版本已经完成
    
17. 发现顶部轮播图虽然不显示广告和文字，但是点击跳转的界面是不正确的，而且有可能跳转到广告，因此直接在布局中`banner_recommend_fragment`隐藏Banner，将Banner高度设置为0dip

    ```
    <com.waxgourd.wg.ui.widget.Banner android:id="@id/banner_recommend_fragment" android:layout_width="fill_parent" android:layout_height="0.0dip" app:indicator_drawable_selected="@drawable/shape_indicator_banner" app:layout_scrollFlags="scroll" app:title_height="40.0dip" app:title_textcolor="@color/colorWhite" app:title_textsize="@dimen/text_size_16sp" />
                        
    ```

    

