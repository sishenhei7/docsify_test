# test

> :bulb:testfsfsdfsdfsadfsadfsadfsdf

## 二级标题

### 三级标题

```
    let fn = {
        setSwiper: function() {
            let swiper = new Swiper('.page4-swiper',{
                direction: 'horizontal',
                speed:500,
                prevButton: '.swiper-button-prev',
                nextButton: '.swiper-button-next',
                slidesPerView: 2,
                centeredSlides: true,
                spaceBetween: 90,
                loop: true
            });
        },
        //localstorge存user_id和login_token
        setLocalStorage: function(user_id, login_token) {
            localStorage.setItem('user_id', user_id);
            localStorage.setItem('login_token', login_token);
        },
        getLocalStorage: function() {
            let user_id = localStorage.getItem('user_id'),
                login_token = localStorage.getItem('login_token');
            if(user_id !=null && user_id != '') {
                userInfo.userID = user_id;
            }
            if(login_token !=null && login_token != '') {
                userInfo.loginToken = login_token;
            }
            return user_id !=null && user_id != '' && login_token !=null && login_token != '';
        },
        //打开页面初始化登录状态
        initLoginState: function() {
            let loginState = fn.getLocalStorage();
            if(loginState) {
                userInfo.isLogin = true;
                $('.btn-login').addClass('curr');
                //初始化礼包码和历史记录
                fn.initUserSN();
                fn.setDrawRecord();
            }
        },
        //打开XX弹窗，XX是类名
        openPop: function(selector) {
            $('.pop-item').removeClass('curr');
            $(selector).addClass('curr');
            $('.pop').addClass('curr');
        },
        //关闭XX弹窗，XX是类名
        closePop: function() {
            $('.pop').removeClass('curr');
            $('.pop-item').removeClass('curr');
        },
        //短信验证码倒计时60秒
        loginTimeout: function() {
            let count = 60,
                $loginText = $('.login-time');
            $('.login-block').hide();
            let setTim = setInterval(function() {
                if(count < 0) {
                    clearInterval(setTim);
                    $('.login-block').show();
                } else {
                    $loginText.html(count + '秒');
                    count--;
                }
            }, 1000);
        },
        //初始化用户信息——用户token无效或者登出时使用
        initUserInfoAndRecord: function() {
            fn.setLocalStorage('', '');
            record = [];
            userInfo.isLogin = false;
            userInfo.isRecord = false;
            userInfo.userID = '';
            userInfo.loginToken = '';
            userInfo.prizeType = 0;
            userInfo.sn = '';
            userInfo.drawTimes = 0;
            userInfo.addressSubmit = false;
            userInfo.packageCode = '';
            $('.btn-login').removeClass('curr');
        },
        //用户token无效的时候处理
        invalidToken: function(res) {
            if(res.msg === '用户token无效') {
                fn.initUserInfoAndRecord();
                alert('登录已过期，请重新登录！');
            } else {
                alert(res.msg);
            }
        },
        //登录
        initLogin: function() {
            //右上角登录
            $('#js_login_btn').on("click", function() {
                if(userInfo.isLogin) {
                    fn.initUserInfoAndRecord();
                } else {
                    fn.openPop('.pop-login');
                }
            });
            //关闭登录弹窗
            $('.login-close').on("click", function() {
                fn.closePop();
            });
            //发短信
            $('.login-block').on("click", function() {
                let mobileNum = $('#js_login_mobileNum').val();
                if(!fn.checkPN(mobileNum)) {
                    alert('请输入正确的手机号！');
                } else {
                    $('.login-block').hide();
                    fn.loginTimeout();
                    yutil.getAuthCode(mobileNum, function(res){
                        // console.log(res);
                        if(res.success) {
                            $('#js_login_code').focus();
                        } else {
                            alert(res.msg);
                        }
                    });
                }
            });
            //登录按钮
            $('.login-btn').on("click", function() {
                let mobileNum = $('#js_login_mobileNum').val(),
                    authCode = $('#js_login_code').val();
                if(fn.checkNull('#js_login_mobileNum') || fn.checkNull('#js_login_code')) {
                    alert('请将手机号和验证码填写完整！');
                } else if(!fn.checkPN(mobileNum)) {
                    alert('请输入正确的手机号码！');
                } else if(authCode.length != 6) {
                    alert('请输入6位验证码！');
                } else {
                    yutil.verifyLogin(mobileNum, authCode, function(res) {
                        // console.log(res);
                        if(res.success) {
                            userInfo.userID = res.user_id;
                            userInfo.loginToken = res.login_token;
                            fn.setLocalStorage(res.user_id, res.login_token);
                            userInfo.isLogin = true;
                            $('.btn-login').addClass('curr');
                            fn.closePop();
    
                            //清空数据
                            $('#js_login_mobileNum').val('');
                            $('#js_login_code').val('');
    
                            //初始化礼包码和历史记录
                            fn.initUserSN();
                            fn.setDrawRecord();
    
                            //登录成功
                            // alert('登录成功！');
                        } else {
                            alert(res.msg);
                        }
                    });
                }
            });
        },
        //初始化个人礼包码
        initUserSN: function(callback) {
            yutil.getPackage(userInfo.userID, userInfo.loginToken, function(res) {
                if(res.success) {
                    userInfo.packageCode = res.package_code;
                    $('#js_code_text').html(res.package_code);
                    if(typeof callback === 'function') {
                        callback();
                    }
                } else {
                    fn.invalidToken(res);
                }
            });
        },
        //把中奖记录数据复制进record里面去
        copyRecord: function(prizes) {
            if(prizes.length != 0) {
                for(let i=0; i<prizes.length; i++) {
                    record.push(prizes[i].prize_type);
                }
            }
        },
        //初始化中奖纪录
        setDrawRecord: function(callback) {
            yutil.prizeRecord(userInfo.userID, userInfo.loginToken, function(res) {
                if(res.success) {
                    // console.log(res);
                    fn.copyRecord(res.prizes);
                    userInfo.isRecord = true;
                    userInfo.drawTimes = res.draw_times;
                    userInfo.addressSubmit = res.address_submit;
                    fn.updateSN(res.prizes);
                    fn.updateDrawTimes();
                    fn.updateCode2Text();
                    fn.updatePopRecord();
                    if(typeof callback === 'function') {
                        callback();
                    }
                } else {
                    fn.invalidToken(res);
                }
            });
        },
        //找出record接口中的sn并更新
        updateSN: function(prizes) {
            if(prizes.length > 0) {
                for(let i=0; i<prizes.length; i++) {
                    if(prizes[i].sn != '') {
                        userInfo.sn = prizes[i].sn;
                    }
                }
            }
        },
        updateDrawTimes: function() {
            $('.times-remain').html(userInfo.drawTimes + '次');
        },
        //第一页领取礼包
        initPage1Code: function() {
            $('.page1 .btn-gift').on("click", function() {
                if(userInfo.isLogin) {
                    if(userInfo.packageCode != '') {
                        fn.openPop('.pop-code');
                    } else {
                        fn.initUserSN(function() {
                            fn.openPop('.pop-code');
                        });
                    }
                } else {
                    fn.openPop('.pop-login');
                }
            });
            $('.code-close').on("click", function() {
                fn.closePop();
            });

            //复制剪切板
            let clipboard_code = new ClipboardJS('#js_code_btn');
            clipboard_code.on('success', function(e) {
                alert('复制成功！');
            });
            clipboard_code.on('error', function(e) {
                alert('您的浏览器不支持自动复制，请手动输入！');
            });
        },
        initVideo: function() {
            let $video = $('#js_video');
            $('.page1 .btn-video').on("click", function() {
                let src = $(this).data('src');
                $video.attr('src', src);
                fn.openPop('.pop-video');
                $video[0].play();
            });
            $('.video-close').on('click', function() {
                $video[0].pause();
                fn.closePop();
                $video.attr('src', '');
            });
        },
        //抓蛋游戏
        initEgg: function() {
            let $eggMach = $('.egg-machine'),
                $rope = $('.machine-rope'),
                moveRange = [0.8, 4.8],
                moveStep = 0.02,
                moveTimeRange = 10,
                ropeRange = [0, 1.8],
                isMidClicked = 0,
                ropeTimeRange = 80,
                ropeStep = 0.1,
                leftInterval, rightInterval, midInterval;

            //初始化位置
            $eggMach.css('left', '2rem');
            $rope.height('0rem');

            $('#js_control_left').on('touchstart', function() {
                let eggLeft = $eggMach.css('left'),
                    eggLocation = eggLeft.substring(0, eggLeft.length - 3) - 0,
                    count = 1;
                leftInterval = setInterval(function() {
                    let tempLocation = eggLocation-moveStep*count;
                    if(tempLocation >= moveRange[0]) {
                        $eggMach.css('left', tempLocation+'rem');
                        count++;
                    } else {
                        clearInterval(leftInterval);
                    }
                }, moveTimeRange);
                return false;
            });
            $('#js_control_left').on('touchend', function() {
                clearInterval(leftInterval);
            });
            $('#js_control_right').on('touchstart', function() {
                let eggLeft = $eggMach.css('left'),
                    eggLocation = eggLeft.substring(0, eggLeft.length - 3) - 0,
                    count = 1;
                rightInterval = setInterval(function() {
                    let tempLocation = eggLocation + moveStep*count;
                    if(tempLocation <= moveRange[1]) {
                        $eggMach.css('left', tempLocation+'rem');
                        count++;
                    } else {
                        clearInterval(rightInterval);
                    }
                }, moveTimeRange);
                return false;
            });
            $('#js_control_right').on('touchend', function() {
                clearInterval(rightInterval);
            });
            //抓蛋弹窗的关闭按钮
            $('.page2 .pop-close').on("click", function() {
                fn.closeEggPop();
            });
            //抓蛋次数不够的确定按钮
            $('.eggTimes-btn').on('click', function() {
                fn.closePop();
            });
            $('#js_control_mid').on('click', function() {
                let $rope = $('.machine-rope'),
                    ropeHeight = $rope.css('height'),
                    ropeHeightNum = ropeHeight.substring(0, ropeHeight.length - 3) - 0,
                    count = 1,
                    loop = 0;
                $('.pop-close').removeClass('curr');
                if(userInfo.isLogin) {
                    if(userInfo.drawTimes > 0) {
                        if(!isMidClicked) {
                            // 监测 抓蛋 按钮的点击次数
                            fn.clickingStat();

                            userInfo.isEggSuccess = false;
                            userInfo.isEggError = false;
                            fn.drawEggGame();
                            isMidClicked = 1;
                            $('.machine-left').addClass('rotateLeft');
                            $('.machine-right').addClass('rotateRight');
                            midInterval = setInterval(function() {
                                let tempRopeHeight = ropeHeightNum + ropeStep*count;
                                if(tempRopeHeight >= ropeRange[0] && tempRopeHeight < ropeRange[1]) {
                                    $rope.height(tempRopeHeight+'rem');
                                    if(!loop) {
                                        count++;
                                    } else {
                                        count--;
                                    }
                                } else if(tempRopeHeight >= ropeRange[1]) {
                                    //如果接口返回成功则蛋出现
                                    if(userInfo.isEggSuccess) {
                                        loop = 1;
                                        count--;
                                        $('.machine-doll').show();
                                    }
                                    //如果接口返回失败则蛋不出现
                                    if(userInfo.isEggError) {
                                        loop = 1;
                                        count--;
                                    }
                                    //如果接口没有返回，则一直停留在底部
                                    $rope.height(tempRopeHeight+'rem');
                                } else {
                                    // console.log('结束了');
                                    clearInterval(midInterval);
                                    isMidClicked = 0;
                                    $('.machine-left').removeClass('rotateLeft');
                                    $('.machine-right').removeClass('rotateRight');
                                    $('.machine-doll').hide();
                                    if(userInfo.isEggSuccess) {
                                        fn.openEggPop();
                                    }
                                }
                            }, ropeTimeRange);
                        }
                    } else {
                        fn.openPop('.pop-eggTimes');
                    }

                } else {
                    fn.openPop('.pop-login');
                }
            });
        },
        //孵蛋前的动画
        eggPopAnimate: function(callback) {
            $('.pop-doll-item').addClass('openEgg');
            setTimeout(function() {
                $('.pop-doll-item').removeClass('curr');
                $('.pop-doll-born').addClass('curr');
                callback();
            }, 1000);
        },
        initEggPopAnimate: function() {
            $('.pop-doll-item').removeClass('openEgg');
            $('.pop-doll-item').addClass('curr');
            $('.pop-doll-born').removeClass('curr');
        },
        //初始化抓蛋弹窗
        initEggPop: function() {
            //关闭按钮
            $('.page2 .pop-close').on("click", function() {
                fn.closeEggPop();
            });
            //点击孵化
            $('.page2 .pop-btn').on("click", function() {
                fn.eggPopAnimate(function() {
                    $('.egg-pop .pop-btn').removeClass('curr');
                    $('.pop-close').addClass('curr');
                    fn.fillInPrizeAfter();
                });
            });
            //礼包码复制
            let clipboard_code = new ClipboardJS('#js_pop_paste');
            clipboard_code.on('success', function(e) {
                alert('复制成功！');
            });
            clipboard_code.on('error', function(e) {
                alert('您的浏览器不支持自动复制，请手动输入！');
            });

            //实物奖励弹窗
            $('.gift-btn').on('click', function() {
                fn.openPop('.pop-info');
            });
            $('.info-btn').on('click', function() {
                let name = $('.info-name').val(),
                    phone = $('.info-phone').val(),
                    address = $('.info-address').val();
                if($(this).hasClass('curr')) {
                    return;
                }
                if(fn.checkNull('.info-name') || fn.checkNull('.info-phone') || fn.checkNull('.info-address')) {
                    alert('请将信息填写完整！');
                } else if(!fn.checkPN(phone)) {
                    alert('请输入正确的手机号码！');
                } else {
                    yutil.submitInfo(userInfo.userID, userInfo.loginToken, name, phone, address, function(res) {
                        // console.log(res);
                        if(res.success) {
                            alert('信息提交成功！');
                            userInfo.addressSubmit = true;
                            fn.updatePopRecord();
                            fn.closePop();
                            $('.info-name').val('');
                            $('.info-phone').val('');
                            $('.info-address').val('');
                        } else {
                            fn.invalidToken(res);
                        }
                    });
                }
            });
            $('.info-close').on('click', function() {
                fn.closePop();
            });
        },
        checkPN: function(number) {
            const myreg=/^(13|14|15|16|17|18|19)\d{9}$/;
            return myreg.test(number);
        },
        checkNull: function(selector) {
            let str = $(selector).val();
            return str == '' || str == null;
        },
        updatePopRecord: function() {
            // console.log(record);
            let $recordItems = $('.record-items li');
            if(userInfo.addressSubmit) {
                $('.item-get').addClass('curr');
                $('.info-btn').addClass('curr');
            }

            if(record.length > 0) {
                $('.record-default').removeClass('curr');
                $('.record-items').addClass('curr');
            } else {
                $('.record-default').addClass('curr');
                $('.record-items').removeClass('curr');
            }

            if(record.indexOf(1) != -1) {
                $recordItems.eq(0).addClass('curr');
            }
            if(record.indexOf(2) != -1) {
                $recordItems.eq(1).addClass('curr');
            }
            if(record.indexOf(3) != -1) {
                $recordItems.eq(2).addClass('curr');
            }
            if(record.indexOf(4) != -1) {
                $recordItems.eq(3).addClass('curr');
            }
        },
        initEggRecord: function() {
            //抓蛋历史记录
            $('.egg-record').on("click", function() {
                if(userInfo.isLogin) {
                    if(userInfo.isRecord) {
                        fn.openPop('.pop-record');
                    } else {
                        fn.setDrawRecord(function() {
                            fn.openPop('.pop-record');
                        });
                    }
                } else {
                    fn.openPop('.pop-login');
                }
            });
            $('.record-close').on("click", function() {
                fn.closePop();
            });
            $('.item-see').on('click', function() {
                if(userInfo.sn !== '') {
                    fn.openPop('.pop-code2');
                } else {
                    fn.setDrawRecord(function() {
                        fn.openPop('.pop-code2');
                    });
                }
            });
            $('.item-get').on('click', function() {
                if(!$(this).hasClass('curr')) {
                    fn.openPop('.pop-info');
                }
            });

            //抓蛋激活码
            let clipboard_code = new ClipboardJS('.code2-btn');
            clipboard_code.on('success', function(e) {
                alert('复制成功！');
            });
            clipboard_code.on('error', function(e) {
                alert('您的浏览器不支持自动复制，请手动输入！');
            });
            $('.code2-close').on("click", function() {
                fn.closePop();
            });
        },
        updateCode2Text: function() {
            $('#js_code2_text').html(userInfo.sn);
        },
        //根据prizeType填充奖励(孵化前)
        fillInPrizeBefore: function() {
            let $textWrap = $('.text-wrap'),
                $popDollItem = $('.pop-doll-item'),
                $popDollBorn = $('.pop-doll-born'),
                $popTitle = $('.pop-title');
            $textWrap.removeClass('curr');
            $popDollItem.removeClass(drawInfo.dollArrayStr);
            $popDollBorn.removeClass(drawInfo.bornArrayStr);
            switch(userInfo.prizeType-0) {
                //礼包
                case 1:
                    $popDollItem.addClass('doll1');
                    $popDollBorn.addClass('born1');
                    $textWrap.eq(0).addClass('curr');
                    $popTitle.html(eggTitleBefore[0]);
                    break;
                //周边
                case 2:
                    $popDollItem.addClass('doll2');
                    $popDollBorn.addClass('born2');
                    $textWrap.eq(1).addClass('curr');
                    $popTitle.html(eggTitleBefore[1]);
                    break;
                //立牌
                case 3:
                    $popDollItem.addClass('doll3');
                    $popDollBorn.addClass('born3');
                    $textWrap.eq(2).addClass('curr');
                    $popTitle.html(eggTitleBefore[2]);
                    break;
                //键盘
                case 4:
                    $popDollItem.addClass('doll4');
                    $popDollBorn.addClass('born4');
                    $textWrap.eq(3).addClass('curr');
                    $popTitle.html(eggTitleBefore[3]);
                    break;
                //谢谢惠顾
                default:
                    $popDollItem.addClass('doll5');
                    $popDollBorn.addClass('born5');
                    $textWrap.eq(4).addClass('curr');
                    $popTitle.html(eggTitleBefore[4]);
            }
        },
        //根据prizeType填充奖励(孵化后)
        fillInPrizeAfter: function() {
            let $textWrap = $('.text-wrap'),
                $popDollItem = $('.pop-doll-item'),
                $popDollBorn = $('.pop-doll-born'),
                $popTitle = $('.pop-title');
            $('.pop-egg-item').removeClass('curr');
            $textWrap.removeClass('curr');
            $popDollItem.removeClass(drawInfo.dollArrayStr);
            $popDollBorn.removeClass(drawInfo.bornArrayStr);
            switch(userInfo.prizeType-0) {
                //礼包
                case 1:
                    $popDollItem.addClass('doll1');
                    $popDollBorn.addClass('born1');
                    $textWrap.eq(0).addClass('curr');
                    $('#js_pop_code').html(userInfo.sn);
                    $('.pop-code3').addClass('curr');
                    $popTitle.html(eggTitleAfter[0]);
                    break;
                //周边
                case 2:
                    $popDollItem.addClass('doll2');
                    $popDollBorn.addClass('born2');
                    $textWrap.eq(1).addClass('curr');
                    fn.setPopGift(0);
                    $('.pop-gift').addClass('curr');
                    $popTitle.html(eggTitleAfter[1]);
                    break;
                //立牌
                case 3:
                    $popDollItem.addClass('doll3');
                    $popDollBorn.addClass('born3');
                    $textWrap.eq(2).addClass('curr');
                    fn.setPopGift(1);
                    $('.pop-gift').addClass('curr');
                    $popTitle.html(eggTitleAfter[2]);
                    break;
                //键盘
                case 4:
                    $popDollItem.addClass('doll4');
                    $popDollBorn.addClass('born4');
                    $textWrap.eq(3).addClass('curr');
                    fn.setPopGift(2);
                    $('.pop-gift').addClass('curr');
                    $popTitle.html(eggTitleAfter[3]);
                    break;
                //谢谢惠顾
                default:
                    $popDollItem.addClass('doll5');
                    $popDollBorn.addClass('born5');
                    $textWrap.eq(4).addClass('curr');
                    $('.pop-thanks').addClass('curr');
                    $popTitle.html(eggTitleAfter[4]);
            }
        },
        //设置第n个实物奖励
        setPopGift: function(n) {
            $('.gift-pic').removeClass(drawInfo.picArrayStr);
            $('.gift-pic').addClass(drawInfo.picArray[n]);
            $('.gift-text').html(drawInfo.text[n]);
        },
        //打开抓蛋弹窗
        openEggPop: function() {
            $('.egg-pop .pop-btn').addClass('curr');
            $('.egg-pop').addClass('curr');
        },
        //关闭抓蛋弹窗
        closeEggPop: function() {
            $('.egg-pop').removeClass('curr');
            $('.egg-pop .pop-egg-item').removeClass('curr');
            $('.egg-pop .pop-btn').addClass('curr');
            fn.initEggPopAnimate();
        },
        //抓蛋抽奖
        drawEggGame: function() {
            yutil.drawGame(userInfo.userID, userInfo.loginToken, function(res) {
                // console.log("抓蛋抽奖");
                // console.log(res);
                userInfo.prizeType = 0;
                if(res.success) {
                    userInfo.prizeType = res.prize_type;
                    userInfo.drawTimes = res.draw_times;
                    record.push(res.prize_type);
                    if(userInfo.prizeType == 1) {
                        userInfo.sn = res.sn;
                        fn.updateCode2Text();
                    }
                    fn.updateDrawTimes();
                    fn.updatePopRecord();
                    fn.fillInPrizeBefore();
                    userInfo.isEggSuccess = true;
                } else {
                    alert(res.msg);
                    fn.updatePopRecord();
                    userInfo.isEggError = true;
                }
            }, function() {
                // console.log('hahahahahah');
                userInfo.isEggError = true;
            });
        },
        //点击统计代码
        clickingStat: function() {
            nie.config.stats.url.addto('click=draw', '抽奖按钮点击次数');
        },
        init: function() {
            fn.initLoginState();
            fn.setSwiper();
            fn.initVideo();
            fn.initLogin();
            fn.initPage1Code();
            fn.initEgg();
            fn.initEggPop();
            fn.initEggRecord();
        }
    };
```

#### 四级标题


##### 五级标题

### 又一个三级标题

### 还有一个三级标题

### 最后一个三级标题