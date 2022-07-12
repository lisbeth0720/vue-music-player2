<template>
    <div id="playContent">
        <p class="musicTitle" v-if="itemname.length>9">
            <!-- 王雪芹 - 先暂时注销下面的 因为被当成了组件-报错 -->
            <!-- <marquee behavior="scroll" scrollamount="5" scrolldelay="10">{{itemname.split('_')[0]}}</marquee> -->
            {{itemname.split('_')[0]}}
        </p>
        <p class="musicTitle" v-else>{{itemname.split('_')[0]}}</p>
        <p class="author">{{author}}</p>
        <div class="picContent">
            <div class="out ">
                <div class="roll">
                    <div class="inner">
                        <img src="~assets/img/default.jpg" />
                        <!-- <img :src="picUrl" :onerror="defaultImg"/>    -->
                    </div>
                </div>
            </div>
            <div class="musicrod">
                <img :class="{'activeSing':activeSing}" src="~assets/img/musicrod.png" />
            </div>
        </div>
        <!-- 歌词 -->
        <div class="lrcContent">
            <ul ref="ulList">
                <li v-for="(item,index) in lrcContent" :key="index" :dataindex="index">
                    {{item}}
                </li>
            </ul>
            <!-- {{currentLrc}} -->
        </div>

        <div class="content-box">
            <div class="timeBlock">
                <div class="left-box played-progress">{{playedProgress}}</div>
                <div class="right-box all-progress">{{allProgress}}</div>
            </div>

            <div class="center-box">
                <div class="slider-bar" ref="silider" @click="clickSiderBar($event)">
                    <div class="slider-progress" ref="progress">
                        <div class="slider-dot-control" @touchstart="startTouchBar($event)"
                            @touchmove="moveTouchBar($event)" @touchend="endTouchBar($event)"></div>
                    </div>
                </div>
            </div>
        </div>
        <div class="playBar">
            <audio ref='audio' controls @play="playAudioMusic()" @pause="pauseAudioMusic()" @canplay="canplayMusic()"
                @timeupdate="timeUpdateMusic()" @ended="endMusic()">
                <!-- <source src="http://www.satall.cn/privatefiles/wisepeak/av/有没有 - 薛之谦.flac" type="audio/flac"> -->
                <source :src="misicurl" type="audio/flac">
                您的浏览器不支持 flac 元素
            </audio>
        </div>
        <div class="playBtn">
            <ul>
                <li>
                    <img :src="playCirclePic" @click="changePlayStatus()" />
                </li>
                <li>
                    <img src="~assets/img/before.png" @click="beforeMusic()" />
                </li>
                <li>
                    <img style="width:50%;max-width:70px;" :src="playSrc" class="playMusic" ref="playBtn"
                        @click="playMusic()" />
                </li>
                <li>
                    <img src="~assets/img/next.png" @click="nextMusic()" />
                </li>
                <li>
                    <img src="~assets/img/playList.png" @click="showList()" />
                </li>
            </ul>
        </div>
    </div>
</template>

<script>
import { request1 } from '@/network/request'

export default {
    name:'Player',
    data(){
        return {
            musicInfo:[],
            itemname:"",
            author:"",
            picUrl:"",
            lrc:"",  
            filterLrc:[],
            lrcContent:[],
            misicurl:"",
            // baseURL:'http://www.satall.cn',
            baseURL: '',
            power:false,
            lastX:0,
            playedProgress:'00:00',
            allProgress:'00:00',
            playSrc:require('../assets/img/play.png'),
            lineNo:0,
            currenLine:0,
            showMusicList:false,
            //clicNoMusic:this.paramStr.iid==""?this.paramStr.pid:this.paramStr.iid,
            clicNoMusic: this.paramStr.iid == "" ? this.paramStr.pid : this.paramStr.iid,
            playCircleStatus:1,//0:单曲循环 1：列表顺序 2：列表随机
            playCirclePic:require('../assets/img/order.png'),
            activeSing:false,//设置属性来判断唱片杆的位置
            defaultImg:'this.src="musicplayer/' + require('../assets/img/default.jpg') + '"'
        }
    },
    props:{
        listNoid:String,
        //列表页正在播放的index
        listcid:Number,
        orderid:Number,
        randomid:Number,
        paramStr:Object
    },
    created() {
        this.getNowInfo(this.clicNoMusic)
    },
    methods: {
        //播放列表中选中的
         getNowInfo(id) {
            var that = this;
            var url = window.location.href;//网页地址
            var serverUrl = "http://" + (url.split("//")[1]).split("/")[0] + "/" + (url.split("//")[1]).split("/")[1];//服务器地址
            if (serverUrl.indexOf("#") >= 0) {
                serverUrl = serverUrl.split("#")[0]
            }
            if (serverUrl.indexOf("localhost") >= 0) {
                //serverUrl = "http://localhost:8080"
            } else {
                serverUrl = serverUrl + "/musicplayer/";
            }
            //模拟的数据
            if(id==""){//默认播放第一个
                id=1;
            }
            var newurl = serverUrl + "data/nowPlay_"+id+".json";
            request1({
                url: newurl,
                methods: 'get'
            }).then(function (res) {
                //console.log(res)
                //this.$refs.ulList.innerHTML="";
                that.musicInfo = res.detail;
                let newdata = res.detail;
                if (newdata.itemname.indexOf("-") >= 0) {
                    that.itemname = newdata.itemname.split("-")[1].trim();
                    //console.log(that.itemname)
                    //解决给audio标签设置src属性后音频不能播放的问题
                    that.author = newdata.itemname.split("-")[0].trim();
                    if (that.author.indexOf(".") >= 0) {
                        that.author = that.author.split(".")[0];
                    }
                }
                that.misicurl = that.baseURL + newdata.fileurl;
                that.$refs.audio.src = that.baseURL + newdata.fileurl;
                if (newdata.ct == "9") {
                    that.picUrl = that.baseURL + newdata.fileurl.replace("av", "pic").replace("flac", "jpg");
                    that.lrc = that.baseURL + newdata.fileurl.replace("av", "txt").replace("flac", "lrc");
                    console.log(that.lrc)
                    // 目前存在一个问题，本地请求歌词.lrc文件是能请求到的，但是发布到服务器上后歌词明明在，却说文件不存在，改成.json文件可以
                    let newlrcUrl='';
                    if (serverUrl.indexOf("localhost") >= 0) {
                        newlrcUrl =serverUrl + that.lrc
                    } else {
                        newlrcUrl =serverUrl + (that.lrc).split(".lrc")[0]+".json"
                    }
                    //console.log(newlrcUrl)
                    //获取歌词
                    request1({
                        url: newlrcUrl,
                        methods: 'get'
                    }).then(function (res) { 
                        if (res != undefined) {
                            that.lrc = res.split("\r\n");
                            that.filterLrcFun(res);
                        } else {
                            that.lrc = [];
                            that.filterLrcFun([]);
                        }
                    }, function (err) {
                        console.log(err)
                    })
                } else if (newdata.wpcontentlist[0].ct == "10") {
                    this.picUrl = this.baseURL + newdata.wpcontentlist[0].thumbnail;
                }
            }, function (err) {
                console.log(err)
            })

        },
        startTouchBar(e){
            console.log(e)
            var touch=e.touches[0];
            this.power=true;
            this.lastX=touch.clientX;
        },
        //拖动进度条
        moveTouchBar(e){  
            //alert("移动了")
            var touch=e.touches[0];
            if(this.power){
                var curX=touch.clientX;
                var addW=curX-this.lastX;
                var setW=this.$refs.progress.offsetWidth+addW;
                var maxW=this.$refs.silider.offsetWidth;
                if(setW>=0&&setW<=maxW){
                    this.lastX=curX;
                    var percen=setW/maxW;
                }else if(setW<0){
                    percen=0;
                }else {
                    percen=1;
                }
                this.$refs.progress.style.width=percen*100+"%";
            }
        },
        endTouchBar(e){
            if(this.power){
                var setW=this.$refs.progress.offsetWidth;
                var maxW=this.$refs.silider.offsetWidth;
                if(setW>=0&&setW<=maxW){
                    var percen=setW/maxW;
                }else if(setW<0){
                    percen=0;
                }else {
                    percen=1;
                }
                this.$refs.progress.style.width=percen*100+"%";
                this.upDateProgress(percen);
            }
            this.power=false;
        },
        clickSiderBar(e){
            var setW=e.clientX-e.offsetLeft;
            var maxW=this.$refs.silider.offsetWidth;
            if(setW>=0&&setW<=maxW){
                var percen=setW/maxW;
                this.$refs.progress.style.width=percen*100+"%";
                this.upDateProgress(percen);
            }
        },
        upDateProgress(percence) {
            this.$refs.audio.currentTime=percence*(this.$refs.audio).duration;
        },
        playMusic(){
           if(this.$refs.audio.paused){
                this.$refs.audio.play()
            }else {
                this.$refs.audio.pause()
            }
        },
        playAudioMusic(){
            this.$refs.playBtn.className='play_bt icon-zanting iconfont';
            this.playSrc=require('../assets/img/pause.png');
            this.activeSing=true;
        },
        pauseAudioMusic(){
            this.playSrc=require('../assets/img/play.png');
            this.activeSing=false;
        },
        canplayMusic(){
           this.allProgress=this.duration(parseInt(this.$refs.audio.duration)).replace("：",":");
            if(!this.power){
                var prence=this.$refs.audio.currentTime/this.$refs.audio.duration;
                this.$refs.progress.style.width=prence*100+"%";
            }
            if(this.$refs.audio.paused){
                this.$refs.playBtn.className='play_bt icon-bofang iconfont';
            }else {
                this.$refs.playBtn.className='play_bt icon-zanting iconfont';
            }
        },
        timeUpdateMusic(){
            this.playedProgress=this.duration(parseInt(this.$refs.audio.currentTime)).replace("：",":");
            var ulStr=this.$refs.ulList;
           
            for(var i=0;i<this.filterLrc.length;i++){
                var eleTime="";
                if(this.filterLrc[i][0].indexOf(".")>=0){
                    eleTime=this.filterLrc[i][0].replace("[","").replace("]","").split(".")[0];
                    if(this.changeTime(eleTime)<=this.changeTime1(this.playedProgress)){
                        
                        if(this.lineNo<i){
                            if(i>=1){
                                ulStr.getElementsByTagName("li")[this.lineNo ].removeAttribute("class");
                            }
                            this.lineNo = i;
                            ulStr.getElementsByTagName("li")[this.lineNo ].classList.add("activeClass");
                            var el = document.querySelector('.activeClass');
                            var index = [].indexOf.call(el.parentElement.children, el);
                            this.currenLine=index;
                        }else{
                            ulStr.getElementsByTagName("li")[this.lineNo ].removeAttribute("class");
                            this.lineNo = i;
                            ulStr.getElementsByTagName("li")[this.lineNo ].classList.add("activeClass");            
                        }         
                    }              
                }
            }
            if(!this.power){
                var prence=this.$refs.audio.currentTime/this.$refs.audio.duration;
                this.$refs.progress.style.width=prence*100+"%";
            }
        },
        //监听播放结束事件
        endMusic(){
            if(this.playCircleStatus=="0"){
                var that=this;
                setTimeout(function () {
                    var a=document.getElementsByClassName("lrcContent")[0];
                    that.$refs.audio.play();
                    a.scrollTo(0, 0);
                },200)
            }else if(this.playCircleStatus=="1"){
                //this.listcid;
                var nextindex=this.listcid+1;
                //this.$parent.playToList({nextindex,"add"});
                this.$emit('playToList',nextindex,"add");
            }else if(this.playCircleStatus=="2"){
                //var nextindex=this.listcid+1;
                var nextindex = this.randomid + 1;
                 this.$emit('playToList',nextindex,"add");
            }
            this.$emit('sendType', this.playCircleStatus);
        },
        //下一首
        nextMusic(){
            //由于listcid是父组件传过来的值在子组件里是不能修改的，所以下面代码不可行，可以通过发送事件让父组件修改
            //if(this.playCircleStatus==0||this.playCircleStatus==1){
                //this.listcid=this.orderid;      
           // }else if(this.playCircleStatus==2){
                //this.listcid=this.randomid;
            //}
            //var nextindex=this.listcid+1;
           // this.$emit('playToList',nextindex,"add");

           //更改后的代码
            let nextIndex = 1;
            if (this.playCircleStatus == 0 || this.playCircleStatus == 1){//单机循环、顺序循环
                nextIndex = this.listcid + 1;
            }else{//随机播放
                nextIndex = this.randomid + 1;
            }
            console.log(nextIndex)
            this.$emit('playToList', nextIndex, "add");
        },
        //上一首
        beforeMusic(){
            //由于listcid是父组件传过来的值在子组件里是不能修改的，所以下面代码不可行，可以通过发送事件让父组件修改
            // if(this.playCircleStatus==0||this.playCircleStatus==1){
            //     console.log(this.orderid)
            //     this.listcid=this.orderid;
                
            // }else if(this.playCircleStatus==2){
            //     this.listcid=this.randomid;
            // }
            // var nextindex=this.listcid-1;
            // this.$emit('playToList',nextindex,"atten");

            //更改后的代码
            let nextIndex2 = 1;
            if (this.playCircleStatus == 0 || this.playCircleStatus == 1) {//单机循环、顺序循环
                nextIndex2 = this.listcid - 1;
            } else {//随机播放
                nextIndex2 = this.randomid - 1;
            }
            this.$emit('playToList', nextIndex2, "atten");
        },
        //时间转换
        duration(time){
            var fen=parseInt(time/60);
            var miao=time%60;
            if(fen<=9){
                fen="0"+fen;
            }
            if(miao<=9){
                miao="0"+miao;
            }
            return fen+'：'+miao;
        },
        //过滤生成单独的歌词数组和时间数组
        filterLrcFun(values){
            var lrc={};
            var lrcics=[];
            if(values.indexOf("\r\n")>=0){
                lrcics=values.split("\r\n");
            }else if(values.indexOf("\n")>=0){
                lrcics=values.split("\n");
            } 
           // alert("lrcics_"+lrcics);
            var reg=/\[\d*:\d*(\.|:)\d*]/g;
            this.lrcContent=[];
            this.filterLrc=[];
            if(lrcics.length>0){
                for(var i=0;i<lrcics.length;i++){
                    var timeRegArr=lrcics[i].match(reg);
                    if(!timeRegArr){
                        continue;
                    }
                    var content=lrcics[i].replace(timeRegArr,"");
                    if(content!=""){
                        this.lrcContent.push(content);
                        this.filterLrc.push(timeRegArr);
                    }   
                }
            }else{
                 this.lrcContent.push("无内容")
            }
            
             //alert(this.lrcContent);
            var a=document.getElementsByClassName("lrcContent")[0];
            a.scrollTo({
                top:0,
                behavior:"smooth"
            })
        },
        //将分,秒转成秒
        changeTime(value){
            var mTime=parseInt(value.split(":")[0]);
            var sTime=parseInt(value.split(":")[1]);
            return mTime*60+sTime;

        },
        changeTime1(value){
            var mTime=parseInt(value.split(":")[0]);
            var sTime=parseInt(value.split(":")[1]);
            return mTime*60+sTime;

        },
        showList(){

            this.showMusicList=true;
            this.$emit('showList', this.showMusicList);
        },
        //改变切换音乐的方式，单曲循环、顺序播放、随机播放
        changePlayStatus(){
            if(this.playCircleStatus==0){
                this.playCircleStatus=1;
                this.playCirclePic=require('../assets/img/order.png');  
            }else if(this.playCircleStatus==1){
                this.playCircleStatus=2;
                this.playCirclePic=require('../assets/img/randPlay.png');
            }else if(this.playCircleStatus==2){
                this.playCircleStatus=0;
                this.playCirclePic=require('../assets/img/circle.png')
            }
            this.$emit('sendType', this.playCircleStatus);
        }
        
    },
    watch: {
        //设置监听属性，当歌词行数产生变化时，及时滚动
        currenLine(newValue, oldValue) {
            //debugger;
            var a=document.getElementsByClassName("lrcContent")[0];
            var el=document.getElementsByClassName("activeClass")[0]; 
            //算出中间行，在中间位置滚动，且保持当前的最新歌词始终在中间位置
            var middleIndex=Math.floor(parseInt(a.offsetHeight/document.getElementsByClassName("activeClass")[0].offsetHeight)/2);
            if(newValue>middleIndex){
               
                var index = [].indexOf.call(el.parentElement.children, el);
                //以下两种方式都实现滚动，显示的样式略有差异
                // a.scrollTo(0,(document.getElementsByClassName("activeClass")[0].offsetHeight*index));
               a.scrollTo({
                   top:document.getElementsByClassName("activeClass")[0].offsetHeight*(index-middleIndex),
                   behavior:"smooth"
               })
                //滚动效果不带动画
                //this.$refs.ulList.style.transform="translateY(-"+document.getElementsByClassName("activeClass")[0].offsetHeight*(index-3)+"px)";   
            }
           
        },
    },
    computed: {
       
    },
    mounted() {
         
    },
    
}
</script>

<style scoped>
* {
    /* margin: 0px;
    padding: 0px; */
    box-sizing: border-box;
    border: none;
}
ul,li{
    list-style: none;
}
/* body {
    margin: 0;
    padding: 0;
        border:1px solid red;
} */
#playContent{
    width:100%;
    height: 100vh;
    display: flex;
    flex-direction: column;
    background: url('../assets/img/playBg.jpg');
    background-size: 100% 100%;
    overflow: hidden;
}
@keyframes spin {

    0% {
        transform: rotateZ(0deg);
    }
    100% {
        transform: rotateZ(360deg);
    }
}

.inner {
    width: 100%;
    height: 100%;
    border-radius: 50%;
    overflow: hidden;

}

.inner img {
    width: 100%;
    height: 100%;
    border-radius: 50%;
    /* -webkit-filter: blur(1px); */
    transition-delay: 1.5s;
    transition: all 0.005s ease-in-out;
}

.out {
    width: 160px;
    height: 160px;
    /* border: 1px dashed red; */
    border-radius: 50%;
    /* padding: 5px; */
    margin: 0px auto;
    transform: rotateX(0deg);
    /* transform-origin: center; */
    /* box-shadow: -6px -5px 0px #aaa; */
    -webkit-box-shadow: #666 0px 0px 10px;
    -moz-box-shadow: #666 0px 0px 10px;
    box-shadow: #666 0px 0px 10px;
    margin-top:20px;   
}
.roll{
    animation: spin 10s linear infinite; 
    width: 100%;
    height: 100%;
}
.inner:hover img {
    transform: scale(1.1, 1.1);
    -webkit-filter: blur(0px);
}

/*音乐播放器进度*/
/*滑动时间的样式*/
.slider-bar{
    height:4px;
    width: 100%;
    background-color: #6d6d6d;
    border-radius: 2px;
}
.slider-progress{
    position: relative;
    height: 100%;
    width: 0;
    background-color: #fff;
    border-radius: 2px;
}
.slider-dot-control{
    position: absolute;
    width: 6px;
    height: 6px;
    border-radius: 50%;
    left: 100%;
    background-color: #ff7471;
    border: 6px solid #fff;
    top: 50%;
    margin-top: -7px;
    margin-left: -9px;
}
/*滑动时间的样式*/
/*播放时间的样式*/
.played-progress,.all-progress{
    display: flex;
    -webkit-justify-content: center;
    justify-content: center;
    padding: 0 10px;
    font-size: 14px;
}
.content-box{
    display: flex;
    height: 20px;
    color: #eee;
    margin-top: 20px;
    align-items: center;
    flex-direction: column;
}
.center-box{
    width: 80%;
    height: 100%;
    display: flex;
    align-items: center;
    margin-bottom: 20px;
}
.timeBlock{
    display: block;
    width:80%;
    margin-bottom: 10px;
}
.left-box{
    width: 70px;
    height: 100%;
    text-align: center;
    font-size: 14px;
    flex: 1;
    float: left;
}
.right-box{
    width: 70px;
    height: 100%;
    text-align: center;
    font-size: 14px;
    flex: 1;
    float: right;
}
#playContent{
    overflow: hidden;
}
.musicTitle{
    font-size: 4vh;
    margin: auto;
    margin-top: 30px;
    color: #c6c6c6;
    flex: 1;
    width: 70%;
}
.author{
    color:#c6c6c6;
    margin-top:10px;
    font-size: 20px;
    flex: 1;
}
.picContent{   
    flex: 3;
    position: relative;
}
.musicrod{
    width: 100px;
    height: 100px;
    position: absolute;
    top: 0;
    left: calc(50% + 28px);
    margin-top: 10px;
}
.musicrod img{
    height:100%;
    height: 100%;
    /* transform-origin: right top; */
    transform-origin: 30px 10px;
    transform: rotate(-40deg);
}
.lrcContent{
    /* flex:10;
    overflow: auto; */
    margin-top: 20px;
    overflow: hidden;
    height:230px;
}
.content-box{
    flex:2;
}
audio{
    display: none;
}
.playBtn{
    flex: 2;
    margin-bottom: 10px;
    margin-top:10px;
}
.playBtn ul{
    display: flex;
    align-items: center;
}
.playBtn ul li{
    flex: 1;
}
.playBtn ul li:nth-child(3n+3){
    flex: 2;
}
.playBtn ul li img{
    width:40%;
    max-width:25px;
}
.lrcContent li{
    line-height: 30px;
}
.activeClass{
    color: red;
    font-size: 20px;
    text-shadow: 1px 1px 4px #717171;
}
.activeSing{
    transform: rotate(0deg) !important;
}
/* .play_bt{
    background: url('../assets/img/play.png') no-repeat;
} */
/*播放时间的样式*/
</style>
