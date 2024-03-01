<template>
  <div class="container">
    <!-- 上半部分 -->
    <div class="section" v-for="hIndex in highQuantity" :key="hIndex">
      <div class="qrcodeBox" v-for="wIndex in widthQuantity" :key="wIndex" ref="qrcodeBoxDiv"
           :style="`width: ${qrcodeSize}px; height: ${qrcodeSize}px; border-color: ${borderColor};`">
        <img v-show="qrcodeExit" alt="Image" ref="qrCodeDiv" class="vue-qrcode" src=""/>
      </div>
    </div>
  </div>
</template>

<script>
import config from '@/config.js';

export default {

  data() {
    return {
      imgList: [
        // {
        //   disposable: true,
        //   url: '/img/code/A000000001_1_116.jpg'
        // }
      ],
      intervalList: [],
      borderColor: 'red',
      borderColorList: ['red', 'blue', 'purple'],
      authorizationCode: null,
      lastGenerator: null,
      allGenerator: null,
      query: {},
      qrcodeSize: 300,
      highQuantity: 2,
      widthQuantity: 2,
      imgDiv: [],
      socket: null,
      qrcodeExit: false,
      content1: "",
      qNo: "Q000000",
      qText: "暂无问题",
      qrCode: null,
      qrcodeNo: null,
      questionClipsList: [],
      currentIndex: 0,
      codeNum: this.highQuantity * this.widthQuantity,
    }
  },

  async mounted() {
    this.Init_websocket();
    this.changeTitle();
    this.startLoopShowQRCode(this.highQuantity * this.widthQuantity)
  },
  unmounted() {
    this.socket.close()
  },

  created() {
    const size = this.getQueryParam('qrcodeSize')
    const high = this.getQueryParam('highQuantity')
    const width = this.getQueryParam('widthQuantity')
    this.qrcodeSize = size ? size : this.qrcodeSize
    this.highQuantity = high ? parseInt(high) : this.highQuantity
    this.widthQuantity = width ? parseInt(width) : this.widthQuantity
  },
  methods: {
    Init_websocket(){
      this.socket = new WebSocket(config.WebSocket_URL);

      // 监听WebSocket消息事件
      this.socket.onmessage = this.onMessage;
      this.socket.onclose = this.onclose;

    },

    changeTitle(){
      document.title = "生成器: " + this.authorizationCode;
    },

    getQueryParam(key) {
      const queryParams = new URLSearchParams(window.location.search);
      return queryParams.get(key);
    },
    /**
     * 开始轮巡播放二维码
     * @param codeNum
     * @returns
     */
    startLoopShowQRCode(codeNum){
      if(this.imgList.length === 0) return

      this.qrcodeExit = true;
      this.intervalList = [];

      const dataList = this.chunkArray(this.imgList, codeNum);

      for(let num = 0; num < dataList.length; num ++){
        const intervalId  = this.LoopShowQRCode(num, dataList[num]);
        this.intervalList.push(intervalId);
      }
    },

    endLoopShowQRCode(codeNum){
      this.qrcodeExit = false;
      if ( this.intervalList.length === 0) return

      for(const item of this.intervalList){
        clearInterval(item);
      }

      for(let num = 0; num < codeNum; num ++) {
        this.$refs.qrCodeDiv[num].src = "";
      }
    },

    /**
     * 定时换图
     * @param el_index
     * @param dataList
     * @returns {number}
     */
    LoopShowQRCode(el_index, dataList){
      let index = 0;

      return setInterval(() => {
        if (dataList.length === 0) return
        index = this.showQRCode(el_index, dataList, index)


        if (dataList[index - 1].disposable) {
          setTimeout(() => {
            if (dataList.length === 0) return

            if (dataList[index - 1].disposable) {
              const url = dataList[index - 1].url;
              this.imgList = this.imgList.filter(item => item.url !== url);
              dataList = dataList.filter(item => item.url !== url);
            }

            if(dataList.length === 0) {
              this.$refs.qrCodeDiv[el_index].src = ''
            }
          }, 100);
        }
        
      }, 100)
    },

    removeData(dataList, index){
      let dataListIndex = dataList.indexOf(dataList[index - 1]);
      if (dataListIndex !== -1) {
        dataList.splice(dataListIndex, 1);
      }
      return dataList
    },

    showQRCode(el_index, dataList, currentIndex) {
      if (dataList.length === currentIndex) currentIndex = 0
      // this.borderColor = this.borderColorList[currentIndex % this.borderColorList.length]
      this.$refs.qrCodeDiv[el_index].src = dataList[currentIndex].url
      currentIndex += 1
      return currentIndex
    },

    chunkArray(array, numberOfChunks) {
      const result = [];
      const length = array.length;
      const chunkSize = Math.ceil(length / numberOfChunks);
      let index = 0;

      while (index < length) {
        const chunk = array.slice(index, index + chunkSize);
        result.push(chunk);
        index += chunkSize;
      }

      return result;
    },

    addImg(JSONData){
      const NowGenerator = (parseInt(this.authorizationCode) + this.lastGenerator) % this.allGenerator;

      const NowImgList =  this.chunkArray(JSONData['list'], this.allGenerator)
      if (NowImgList.length <= NowGenerator) return;

      for(const item of NowImgList[NowGenerator]) {
        const pathString = `${config.QRCode_URL}${item}.jpg`
        const data = {
          url: pathString,
          disposable: JSONData['disposable']
        }
        this.imgList.push(data);
      }

      const codeNum = this.highQuantity * this.widthQuantity;
      this.endLoopShowQRCode(codeNum)
      this.startLoopShowQRCode(codeNum)
    },

    deleteImg(JSONData){
      for(const item of JSONData['list']) {
        const pathString = `${config.QRCode_URL}${item}.jpg`
        this.imgList = this.imgList.filter(item => item.url !== pathString);
      }

      const codeNum = this.highQuantity * this.widthQuantity;
      this.endLoopShowQRCode(codeNum)

      if(this.imgList.length !== 0){
        this.startLoopShowQRCode(codeNum)
      }

    },

    onclose(){
      setTimeout(() => {
        this.Init_websocket();

        const codeNum = this.highQuantity * this.widthQuantity;
        this.endLoopShowQRCode(codeNum)
        this.startLoopShowQRCode(codeNum)
        this.imgList = []
      }, 2000);
    },

    onMessage(data) {
      const JSONData = JSON.parse(data.data);

      if (JSONData['method'] === 'add'){
        this.lastGenerator = JSONData['lastGenerator'];
        this.allGenerator = JSONData['allGenerator'];
        this.addImg(JSONData);

        // eslint-disable-next-line no-dupe-else-if
      } else if(JSONData['method'] === 'delete'){
        this.deleteImg(JSONData);

        // eslint-disable-next-line no-dupe-else-if
      } else if(JSONData['method'] === 'authorization'){
        this.authorizationCode = JSONData['code'];
        this.changeTitle();
      }

    },
  }
}
</script>

<style>
.container {
  min-height: 90vh;
  display: flex;
  justify-content: center;
}

.section {
  display: flex;
  flex-direction: column;
  justify-content: space-between;
  margin-right: 70px;

}

.qrcodeBox {
  position: relative;
  border: 20px solid #ccc;
  background: green;
}

.vue-qrcode {
  object-fit: cover;
  width: 100%;
  height: 100%;
}

img[src=""],
img:not([src]) {
  visibility: hidden;
}
</style>
