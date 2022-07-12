<template>
  <div id="wrap">
    <player @showList="showMusic($event)" @playToList="playToList" @sendType="sendType" :listNoid="noid" :listcid="cid"
      :orderid="orderid" :randomid="randomid" :playType="playType" ref="playerref" :paramStr="paramStr"></player>
    <list :showContent="showMusicList" v-show="showMusicList" @changShow="changeShowShadow($event)"
      @sendId="sendId($event)" @sendcid="sendcid($event)" ref="playListRef" :paramStr="paramStr"></list>
  </div>
</template>

<script>
//http://192.168.1.10:8080/?cp=wisepeak&cn=%E9%9F%B3%E4%B9%90%E6%92%AD%E6%94%BE%E5%B1%8F&tag=&mn=&mid=9396&pid=9382&ct=&uid=&rn=715804
var url = window.location.search;
var itemid = "";
var contenttype = "";
var Tag = "";
var mid = "";
var uid = "";
var mn = "";
var cn = "";
var iid = "";
var cp = "";
var pid = "";
var ct = "";
if (url.indexOf("iid") >= 0) {
  iid = url.split("iid=")[1];
  if (iid.indexOf("&") >= 0) {
    iid = iid.split("&")[0]
  }
}
if (url.indexOf("cp") >= 0) {
  cp = url.split("cp=")[1];
  if (cp.indexOf("&") >= 0) {
    cp = cp.split("&")[0]
  }
}
if (url.indexOf("itemid") >= 0) {
  itemid = url.split("itemid=")[1];
  if (itemid.indexOf("&") >= 0) {
    itemid = itemid.split("&")[0]
  }
}
if (url.indexOf("ct") >= 0) {
  contenttype = url.split("ct=")[1];
  if (contenttype.indexOf("&") >= 0) {
    contenttype = contenttype.split("&")[0]
  }
}

if (url.indexOf("cn") >= 0) {
  cn = url.split("cn=")[1];
  if (cn.indexOf("&") >= 0) {
    cn = cn.split("&")[0]
  }
}
if (url.indexOf("ct") >= 0) {
  ct = url.split("ct=")[1];
  if (ct.indexOf("&") >= 0) {
    ct = ct.split("&")[0]
  }
}
if (url.indexOf("uid") >= 0) {
  uid = url.split("uid=")[1];
  if (uid.indexOf("&") >= 0) {
    uid = uid.split("&")[0]
  }
}
if (url.indexOf("pid") >= 0) {
  pid = url.split("pid=")[1];
  if (pid.indexOf("&") >= 0) {
    pid = pid.split("&")[0]
  }
}
if (iid == "") {
  if (url.indexOf("mn") >= 0) {
    mn = url.split("mn=")[1];
    if (mn.indexOf("&") >= 0) {
      mn = decodeURI(mn.split("&")[0])
    }
  }
  if (url.indexOf("mid") >= 0) {
    mid = url.split("mid=")[1];
    if (mid.indexOf("&") >= 0) {
      mid = mid.split("&")[0]
    }
  }
  if (url.indexOf("tag") >= 0) {
    Tag = url.split("tag=")[1];
    if (Tag.indexOf("&") >= 0) {
      Tag = Tag.split("&")[0]
    }
  }

}
document.documentElement.addEventListener('touchstart', function (event) {
  if (event.touches.length > 1) {
    event.preventDefault();
  }
}, false);

var lastTouchEnd = 0;
document.documentElement.addEventListener('touchend', function (event) {
  var now = Date.now();
  if (now - lastTouchEnd <= 300) {
    event.preventDefault();
  }
  lastTouchEnd = now;
}, false);
import Player from './views/Player'
import List from './views/List'
export default {
  name: "App",
  data() {
    return {
      showMusicList: false,
      noid: itemid,
      cid: 0,//歌曲列表中数组中的索引。默认是第一个
      orderid: 0,
      randomid: 0,
      playType: "0",
      paramStr: {
        "cp": cp,
        "cn": cn,
        "tag": Tag,
        "mn": mn,
        "mid": mid,
        "iid": iid,
        "pid": pid,
        "ct": ct,
        "uid": uid
      }
    }
  },
  components: {
    Player,
    List
  },
  methods: {
    //从播放页读取是否需要展示列表页
    showMusic(res) {
      this.showMusicList = res
    },
    //从列表页读取是否需要关闭列表页
    changeShowShadow(res) {
      this.showMusicList = res;
    },

    //接受点击播放的音频id
    sendId(res) {
      this.noid = res;
      this.$refs.playerref.getNowInfo(this.noid);
    },

    //接受点击播放的音频index
    sendcid(res) {
      //console.log(res)
      this.cid = res[0];
      this.orderid = res[1];
      this.randomid = res[2];
      setTimeout(() => {
        this.$refs.playerref.playMusic()
      }, 1000)
      //this.$refs.playerref.getNowInfo(this.noid);
    },
    //点击上一首-下一首触发的事件
    playToList(playIndex, str) {
      //需要研究下面这个事件
      this.$refs.playListRef.switchPlayToList(playIndex, str);
    },
    //播放的类型（单曲循环还是列表循环还是随机循环
    sendType(res) {
      this.playType = res;
      this.$refs.playListRef.changeListOrder(res);
    }
  },
}
</script>
<!-- 目前存在一个问题，本地请求歌词.lrc文件是能请求到的，但是发布到服务器上后歌词明明在，确说文件不存在，改成.json文件可以 -->
<style>
 @import "assets/css/base.css";
#wrap{
  font-family: Avenir, Helvetica, Arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  text-align: center;
  color: #c6c6c6;
  width: 100%;
  height: 100vh;
  overflow: hidden;
}
</style>
