<template>
  <div class="detail">
    <div class="dt-head">
      <div class="dt-h-back" @click="goback"></div>
      <span class="dt-h-title">优惠商品</span>
      <div class="dt-h-mark" @click="mark" :class="{markFull: marked}"></div>
    </div>
    <div class="dt-body">
      <div class="dt-item">
        <img class="dt-i-img" v-show="imgLoaded" :src="iprop.imgUrl" @load="imgLoaded = true">
        <div class="loading" v-if="!imgLoaded">
          <i></i>
          <i></i>
          <i></i>
        </div>
        <div class="dt-i-name">{{iprop.name}}</div>
        <div class="dt-i-price">{{iprop.price}}</div>
        <div class="dt-i-directlybuy" @click="go">直接购买</div>
      </div>
      <div class="dt-coupon">
        <img class="dt-c-img" :src="couponBg">
        <div class="dt-c-info">
          <div class="dt-c-discount">￥<span style="fontSize: 1.5rem">{{iprop.discount}}</span><br>满29减{{iprop.discount}}元</div><div class="dt-c-share">分享<br>优惠券</div><div @click="openUrl" class="dt-c-purchase"  >领券<br>购买</div>
          <a href="//m.taobao.com" data-type="bc_applink" data-appkey="23082328" data-itemid="44777395102" data-params='{"bc_test":"01"}'>去详情</a>
        </div>
        <div class="watermark">{{watermark}}</div>
      </div>
      <div class="recommend">
        <div class="r-head">达人推荐</div>
        <advice></advice>
      </div>
    </div>
  </div>
</template>

<script>
import Advice from './Advice'
import BC_SDK from './initialize.js'

export default {
  name: 'detail',
  data () {
    return {
      iprop:this.$root.eventHub.$on('toDetailDate'),
//      iprop: this.$route.query.iprop,
      couponBg: require('./assets/Coupon.png'),
      watermark: '专',
      imgLoaded: false,
      marked: false
    }
  },
  components: {
    'advice': Advice
  },
  methods: {
    openUrl: ()=>{
      console.log(BC_SDK)
      console.log(event)
      window.BC_SDK.goDetail({
        itemId : '521376186545',
        params : {}
      })
      },
    mark: function () {
      this.marked = !this.marked
      if (this.marked) {
        if (localStorage.mark === undefined || localStorage.mark === '') {
          localStorage.mark = '[]'
        }
        let markAll = JSON.parse(localStorage.mark)
        markAll.push(this.iprop)
        localStorage.mark = JSON.stringify(markAll)
        this.$root.eventHub.$emit('showNotification', '已收藏(๑•ㅂ•́)و✧')
      } else {
        new Promise((resolve, reject) => {
          let markedAll = JSON.parse(localStorage.mark)
          for (let i = 0; i < markedAll.length; i++) {
            if (markedAll[i].name === this.iprop.name) {
              markedAll.splice(i, 1)
              localStorage.mark = JSON.stringify(markedAll)
              resolve(localStorage.mark)
              break
            }
          }
        }).then(mark => {
          console.log(mark)
        })
      }
    },
    go: function () {
      let text = '复制这条信息，打开「手机淘宝」领券下单 ￥mkpa02TWO8q￥'
      if (window.co !== undefined) {
        window.co.plugins.clipboard.copy(text)
        location.href = 'taobao://'
        setTimeout(function () {
          alert('超时')
        }, 250)
        setTimeout(function () {
          location.reload()
        }, 1000)
      }
      // var ifr = document.createElement('iframe')
      // ifr.src = 'taobao://'
      // ifr.style.display = 'none'
      // document.body.appendChild(ifr)
      // var openTime = +new Date()
      // window.setTimeout(function () {
      //   document.body.removeChild(ifr)
      //   if ((+new Date()) - openTime > 2500) {
      //     window.location = ''
      //   }
      // }, 2000)
    },
    goback: function () {
      history.go(-1)
    },
    fetchdata: function () {
      let b = document.getElementsByClassName('dt-body')[0]
      if (b !== undefined) {
        document.getElementsByClassName('dt-body')[0].scrollTop = 0
        this.iprop = this.$route.query.iprop
        this.addBrowseHistory()
      }
    },
    addBrowseHistory: function () {
      if (this.iprop === null || this.iprop === [null]) {
        return
      }
      let thisbh = []
      let lsbh = localStorage.browseHistory
      if (lsbh !== undefined && lsbh !== '') {
        thisbh = JSON.parse(lsbh)
      }
      let i = 0
      for (; i < thisbh.length; i++) {
        let name = thisbh[i].name
        if (name === this.iprop.name) {
          thisbh.unshift(thisbh.splice(i, 1)[0])
          break
        }
      }
      if (i === thisbh.length) {
        thisbh.unshift(this.iprop)
      }
      localStorage.browseHistory = JSON.stringify(thisbh)
    },
    initMark: function () {
      if (localStorage.mark !== undefined && localStorage !== '') {
        let markAll = JSON.parse(localStorage.mark)
        for (let i = 0; i < markAll.length; i++) {
          if (markAll[i].name === this.iprop.name) {
            this.marked = true
          }
        }
      }
    }
  },
//
  mounted () {
    this.addBrowseHistory()

  },
  beforeRouteEnter (to, from, next) {
    next(vm => {
      vm.initMark()
    })
  },
  beforeRouteLeave (to, from, next) {
    this.marked = false
    next()
  }
}
</script>
<!-- Add "scoped" attribute to limit CSS to this component only -->
<style lang="scss" scoped rel="stylesheet/scss">
h1, h2 {
  font-weight: normal;
}

ul {
  list-style-type: none;
  padding: 0;
}

li {
  display: inline-block;
  margin: 0 10px;
}

a {
  color: #42b983;
}

.detail {
  position: absolute;
  top: 0;
  left: 0;
  right: 0;
  bottom: 3rem;
  background-color: #F1F1F1;
  overflow: scroll;
  -webkit-overflow-scrolling: touch;
}

$headH: 3rem;
.dt-head {
  position: absolute;
  top: 0;
  left: 0;
  width: 100%;
  height: $headH;
  background-color: #FB5776;

  $headTextH: 1.5rem;
  & > div {
    margin-top: ($headH - $headTextH) / 2;
  }

  .dt-h-back {
    float: left;
    margin-left: 0.5rem;
    width: 1.5rem;
    height: $headTextH;
    background-image: url(./assets/back.png);
    background-size: contain;
    background-repeat: no-repeat;
  }

  .dt-h-title {
    line-height: 3rem;
    width: 4rem;
    color: white;
    font-size: 1.1rem;
  }

  .dt-h-mark {
    float: right;
    margin-right: 0.5rem;
    width: 1.5rem;
    height: $headTextH;
    background-image: url(./assets/mark-empty.png);
    background-size: contain;
    background-repeat: no-repeat;
  }

  .markFull {
    background-image: url(./assets/mark-full.png);
  }
}

.dt-body {
  position: absolute;
  top: 3rem;
  width: 100%;
  bottom: 0;
  overflow: scroll;
  -webkit-overflow-scrolling : touch;
}
.dt-item {
  position: relative;
  height: 8rem;
  padding: 0.5rem;
  background-color: white;
  text-align: left;

  .dt-i-img {
    margin-right: 0.5rem;
    width: 8rem;
    float: left;
  }

  .dt-i-price {
    font-size: 1.3rem;
    text-align: left;
    color: #FB5776;
    &:before {
      content: '￥';
      font-size: 1rem;
    }
  }

  .dt-i-directlybuy {
    position: absolute;
    right: 1.5rem;
    bottom: 0.5rem;
    text-align: right;
    color: #FB5776;

    &:after {//&是父亲
      content: '';
      position: absolute;
      width: 1.4rem;
      height: 1.4rem;
      background-image: url(./assets/foward.png);
      background-size: contain;
      background-repeat: no-repeat;
    }
  }
}

.dt-coupon {
  position: relative;
  margin-top: 0.5rem;
  padding: 0.5rem 1rem;
  background-color: white;
  color: #908BDE;

  .dt-c-img {
    width: 100%;
    vertical-align: bottom;
  }

  .dt-c-info {
    position: absolute;
    left: 1rem;
    right: 1rem;
    top: 0;
    bottom: 0;
    display: flex;
    flex-flow: row wrap;
    align-items: center;
    font-size: 0.875rem;

    .dt-c-discount {
      width: 52%;
    }
    .dt-c-share {
      width: 24%;
    }
    .dt-c-purchase {
      width: 24%;
    }
  }

  .watermark {
    position: absolute;
    left:50%;
    top:50%;
    transform: translateX(-50%) translateY(-50%);
    font-size: 3rem;
    opacity: 0.5;
  }
}

.loading {
  position: relative;
  width: 8rem;
  height: 8rem;
  margin-right: 0.5rem;
  float: left;
  text-align: center;

  i {
    position: absolute;
    top: calc(50% - 1.75rem);
    left: calc(50% - 0.75rem);
    width: 1.5rem;
    height: 1.5rem;
    border-radius: 100%;
    border: 1px solid rgba(0, 0, 0,.8);
    opacity: 0;
    animation: loading 1.25s 0s infinite cubic-bezier(.21,.53,.56,.8);
    -webkit-animation: loading 1.25s 0s infinite cubic-bezier(.21,.53,.56,.8);
    &:nth-child(1) {
      animation-delay: 0s;
      -webkit-animation-delay: 0s;
    }
    &:nth-child(2) {
      animation-delay: .2s;
      -webkit-animation-delay: .2s;
    }
    &:nth-child(3) {
      animation-delay: .4s;
      -webkit-animation-delay: .4s;
    }
  }
  @keyframes loading {
    0% {
      transform: scale(.1);
      opacity: 1;
    }
    70% {
      transform: scale(1);
      opacity: .7;
    }
    100% {
      opacity: 0;
    }
  }
}

.recommend {
  margin-top: .5rem;

  .r-head {
    height: 2rem;
    padding-left: 0.5rem;
    line-height: 2rem;
    text-align: left;
    background-color: white;
  }
}
</style>
