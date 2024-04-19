<template>
  <div id="app">
    <!-- <DDR></DDR> -->
    <div class="left" id="dom">
      <img
        :src="clothImgList[bgcIndex]"
        alt=""
        style="width: 100%"
        draggable="false"
      />
      <DDR :url="chooseLogoUrl" :obj="logoPositionObjList[doingIndex]">
        <!-- <img :src="chooseLogoUrl" alt=""> -->
      </DDR>
    </div>
    <div class="right">
      <div class="inputPlace">
        <div>
          <input id="input" type="file" @change="handleClothFile" multiple />
          <img
            v-for="(item, index) in clothImgList"
            :key="index"
            :src="item"
            alt=""
            style="width: 60px"
          />
        </div>
        <div style="margin-bottom: 10px">
          <input id="input" type="file" @change="handleImgFile" multiple />
          <img
            v-for="(item, index) in logoImgList"
            :key="index"
            :src="item"
            alt=""
            style="width: 60px; height: 60px; object-fit: contain"
            class="logoImg"
            @click="chooseImg(index, item)"
          />
        </div>

        <div>
          <el-form>
            <el-form-item label="导出图片格式">
              <el-radio v-model="way" label="toPng">png</el-radio>
              <el-radio v-model="way" label="toJpeg">jpg</el-radio>
              <el-radio v-model="way" label="toSvg">svg</el-radio>
            </el-form-item>
          </el-form>
        </div>
        <el-button @click="preview(false)">隐藏边框</el-button>
        <el-button @click="preview(true)">展示边框</el-button>

        <el-button @click="outPutOne()">导出本张</el-button>
        <el-button @click="outPutAll()">批量导出</el-button>
      </div>
    </div>
  </div>
</template>

<script>
import DDR from "./components/ddr.vue";
import domtoimage from "dom-to-image";
import axios from "axios";
import JSZip from "jszip";
import FileSaver from "file-saver";
export default {
  name: "App",
  components: {
    DDR,
  },
  data() {
    return {
      bgcIndex:0,
      way: "toPng",
      clothImgList: [],
      logoImgList: [],
      chooseLogoUrl: "", //选中的logo
      doingIndex: "", //当前操作项
      logoPositionObjList: [],
      nameList:[],
      logosNameList:[]
    };
  },
  mounted() {},
  methods: {
    async handleClothFile(e) {
      const files = e.target.files;
      this.nameList = Array.from(files).map(item=>{
       return item.name.split('.')[0]
      })
      // 遍历选择的文件列表
      for (let i = 0; i < files.length; i++) {
        const file = files[i];
        // 使用 FileReader API 读取文件内容为 Data URL
        const reader = new FileReader();
        reader.readAsDataURL(file);

        // 当文件读取完成时，将 Data URL 添加到 clothImgList
        reader.onload = () => {
          this.clothImgList.push(reader.result);
        };
      }
    },
    async handleImgFile(e) {
      const files = e.target.files;
      this.logosNameList = Array.from(files).map(item=>{
       return item.name.split('.')[0]
      })
      // 遍历选择的文件列表
      for (let i = 0; i < files.length; i++) {
        const file = files[i];

        // 使用 FileReader API 读取文件内容为 Data URL
        const reader = new FileReader();
        reader.readAsDataURL(file);

        // 当文件读取完成时，将 Data URL 添加到 logoImgList
        reader.onload = () => {
          this.logoImgList.push(reader.result);
          this.logoPositionObjList.push({
            transform: { x: 172, y: 116, width: 114, height: 114, rotation: 0 },
            active: true,
            rotatable: true,
            draggalbe: true,
            resizable: true,
            parent: true,
            resizeHandler: ["tl", "tr", "br", "bl"],
            minWidth: 10,
            minHeight: 10,
            acceptRatio: true,
          });
        };
      }
    },
    preview(val) {
      this.logoPositionObjList.forEach((item) => {
        item.active = val;
      });
    },
    chooseImg(index, url) {
      this.chooseLogoUrl = url;
      this.doingIndex = index;
    },
    outPutOne() {
      this.preview(false);
      let dom = document.getElementById("dom");
      domtoimage[this.way](dom)
        .then((dataUrl) => {
          //输出图片的Base64,dataUrl
          // 下载到PC
          const a = document.createElement("a"); // 生成一个a元素
          const event = new MouseEvent("click"); // 创建一个单击事件
          a.download = "one"; // 设置图片名称没有设置则为默认
          a.href = dataUrl; // 将生成的URL设置为a.href属性
          a.dispatchEvent(event); // 触发a的单击事件
        })
        .catch(function (error) {
          console.error("oops, something went wrong!", error);
        });
    },
    getFile(url) {
      return new Promise((resolve, reject) => {
        axios({
          method: "get",
          url,
          responseType: "arraybuffer",
        })
          .then((data) => {
            resolve(data.data);
          })
          .catch((error) => {
            reject(error.toString());
          });
      });
    },
    async handleBatchDownload(data) {
  const zip = new JSZip();
  let folders = []
  const promises = [];
  for(let j = 0;j<this.logoImgList.length;j++){
    folders[j] = zip.folder(`${this.logosNameList[j]}`)
    for (let i = 0; i < this.clothImgList.length; i++) {
        const dataUrl = data[j*this.clothImgList.length+i];
        const promise = (async () => {
          const blob = await this.dataUrlToBlob(dataUrl);
          const fileName = `${this.nameList[i]}.${this.way === "toJpeg" ? "jpg" : "png"}`;
          // 将文件添加到文件夹中
          folders[j].file(fileName, blob, { binary: true });
        })();
        promises.push(promise);
      }
    }
    await Promise.all(promises);

  // 生成压缩包并下载
  zip.generateAsync({ type: "blob" }).then((content) => {
    FileSaver.saveAs(content, "batch_output.zip");
  });

  // 创建一个名为 images 的文件夹
  // const folder = zip.folder("images");

  // const promises = [];

  // for (let i = 0; i < data.length; i++) {
  //   const dataUrl = data[i];
  //   const promise = (async () => {
  //     const blob = await this.dataUrlToBlob(dataUrl);
  //     const fileName = `image_${i}.${this.way === "toJpeg" ? "jpg" : "png"}`;
  //     // 将文件添加到文件夹中
  //     folder.file(fileName, blob, { binary: true });
  //   })();
  //   promises.push(promise);
  // }

  // await Promise.all(promises);

  // // 生成压缩包并下载
  // zip.generateAsync({ type: "blob" }).then((content) => {
  //   FileSaver.saveAs(content, "batch_output.zip");
  // });
},
    dataUrlToBlob(dataUrl) {
      const arr = dataUrl.split(",");
      const mime = arr[0].match(/:(.*?);/)[1];
      const bstr = atob(arr[1]);
      let n = bstr.length;
      const u8arr = new Uint8Array(n);

      while (n--) {
        u8arr[n] = bstr.charCodeAt(n);
      }

      return new Blob([u8arr], { type: mime });
    },
    async outPutAll() {
      this.preview(false);
      let dom = document.getElementById("dom");
      let urlList = []
        for(let j = 0;j<this.logoImgList.length;j++){
          this.doingIndex = j
          this.chooseLogoUrl = this.logoImgList[j]
          for(let i = 0;i<this.clothImgList.length;i++){
            this.bgcIndex = i
            await domtoimage[this.way](dom).then((dataUrl) => {
              urlList.push(dataUrl)
            });
          }
        }
      this.handleBatchDownload(urlList);
    },
  },
};
</script>

<style scoped lang="scss">
#app {
  display: flex;

  .left {
    width: 470px;
    height: fit-content;
  }
  .right {
    flex: 1;
    .inputPlace {
      .logoImg {
        cursor: pointer;
        margin-right: 10px;
        border: 1px solid rgb(160, 160, 160);
      }
    }
  }
}
</style>
