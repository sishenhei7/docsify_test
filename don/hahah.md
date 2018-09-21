# test

> 第三页:bar_chart:

## 二级标题

### 三级标题

```
  var _initShare = function() { //初始化分享组件
    var share = nie.require("nie.util.mobiShare2");
    var title = $("#share_title").html();

    MobileShare.init({
      title: title, //分享标题
      desc: $("#share_desc").html(), //分享正文
      url: location.href, //分享URL
      imgurl: $("#share_pic").attr("src"), //分享图片
      circleTitle: title, //分享到朋友圈的标题。不填则与title一致
      guideText: "点击右上角按钮<br/>分享到朋友圈", //微信中点分享按钮显示的分享引导语
      qrcodeIcon: __uri("https://webinput.nie.netease.com/img/mumu/icon.png"), //二维码图标
      shareCallback: function(res) { //微信易信分享回调res=｛type:0,res:[微信返回的提示]｝res.type：0表示取消，-1：分享失败，1：分享到朋友圈，2：分享给好友，3：QQ，4：微博。易信只返回1或2两种情况。
      },
      wxSdkCallback: function() { //微信sdk加载完成后回调，可在此回调中调用微信JS-SDK的其他API,如上传图片等。
      }
    });
    $("#btn_share").click(function(e) { //分享按钮绑定点事件，显示分享弹层
      MobileShare.showShare();
    })
  };
  var _initAttention = function() { //初始化关注公众号弹层

    $("#btn_attention").bind("click", function(e) {

      $("#md_attention").show();
      var st = setTimeout(function() {
        $("#md_attention").addClass("show");
      }, 50)
    });
    $("#md_attention").bind("click", function(e) {
      $("#md_attention").removeClass("show");
      var st = setTimeout(function() {
        $("#md_attention").hide();
      }, 300);
    })
    $("#md_attention")[0].addEventListener("touchmove", function(e) {
      e.preventDefault();
      //e.stopPropagation()
    }, false);
  };
  var _addEvent = function() {
    window.addEventListener("onorientationchange" in window ? "orientationchange" : "resize", function(e) {
      _onorientationchange(e);
    }, false);
    $(".btn_toTop")[0].addEventListener("click", function(e) {
      window.scrollTo(0, 0);
    }, false);
  };
  var _onorientationchange = function(e) {
    if (window.orientation == 90 || window.orientation == -90) {
      $("#forhorview").css("display", "flex"); //显示竖屏浏览提示框
    } else { //竖屏下恢复默认显示效果
      var st = setTimeout(_initScreen, 300);
      $("#forhorview").css("display", "none");
    }
  };
```

#### 四级标题


##### 五级标题

### 又一个三级标题

### 还有一个三级标题

### 最后一个三级标题