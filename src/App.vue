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
      bgcIndex: 0,
      way: "toPng",
      clothImgList: [],
      logoImgList: [],
      chooseLogoUrl: "", //选中的logo
      doingIndex: "", //当前操作项
      logoPositionObjList: [],
      clothNames: [],
      logoNames: [],
    };
  },
  mounted() {},
  methods: {
    async handleClothFile(e) {
      const files = e.target.files;
  //     const logoData = Array.from(files, (item, index) => ({
  //   name: item.name.split(".")[0],
  //   index,
  // }));

  //     this.clothNames.push(...logoData.map((entry) => entry.name))
      // 遍历选择的文件列表
      for (let i = 0; i < files.length; i++) {
        const file = files[i];

        // 使用 FileReader API 读取文件内容为 Data URL
        const reader = new FileReader();
        reader.readAsDataURL(file);

        // 当文件读取完成时，将 Data URL 添加到 clothImgList
        reader.onload = () => {
          this.clothImgList.push(reader.result);
          this.clothNames.push(file.name.split('.')[0])
        };
      }
    },
    async handleImgFile(e) {
      const files = e.target.files;
  //     const logoData = Array.from(files, (item, index) => ({
  //   name: item.name.split(".")[0],
  //   index,
  // }));
  // this.logoNames.push(...logoData.map((entry) => entry.name))
  //     console.log(this.logoNames);
      // 遍历选择的文件列表
      for (let i = 0; i < files.length; i++) {
        const file = files[i];

        // 使用 FileReader API 读取文件内容为 Data URL
        const reader = new FileReader();
        reader.readAsDataURL(file);

        // 当文件读取完成时，将 Data URL 添加到 logoImgList
        reader.onload = () => {
          this.logoImgList.push(reader.result);
          this.logoNames.push(file.name.split('.')[0])
          this.logoPositionObjList.push({
            transform: { x: 463, y: 376, width: 304, height: 304, rotation: 0 },
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
    async outPutOne() {
      this.preview(false);
      let dom = document.getElementById("dom");

      await domtoimage[this.way](dom)
        .then(async (dataUrl) => {
          const blob = await this.adjustImageSize(dataUrl, 1350, 1800);
          const a = document.createElement("a");
          const event = new MouseEvent("click");
          a.download = `${this.clothNames[this.bgcIndex]}.png`;
          a.href = URL.createObjectURL(blob);
          a.dispatchEvent(event);
        })
        .catch(function (error) {
          console.error("oops, something went wrong!", error);
        });
    },
    async outPutAll() {
      this.preview(false);
      let dom = document.getElementById("dom");
      let urlList = [];

      for (let j = 0; j < this.logoImgList.length; j++) {
        this.doingIndex = j;
        this.chooseLogoUrl = this.logoImgList[j];
        for (let i = 0; i < this.clothImgList.length; i++) {
          this.bgcIndex = i;

          // 生成原始Data URL
          const dataUrl = await domtoimage[this.way](dom).then((dataUrl) => {
            return dataUrl;
          });

          // 调整图片尺寸并获取调整后的Blob对象
          const adjustedBlob = await this.adjustImageSize(
            dataUrl,
            1350,
            1800
          );

          // 将调整后的Blob对象添加到urlList数组
          urlList.push(adjustedBlob);
        }
      }

      this.handleBatchDownload(urlList);
    },
    async handleBatchDownload(data) {
  const zip = new JSZip();

  for (let j = 0; j < this.logoImgList.length; j++) {
    const folderName = this.logoNames[j]; // 使用导入的 logo 图片名称作为文件夹名称
    const folder = zip.folder(folderName);

    for (let i = 0; i < this.clothImgList.length; i++) {
      const blob = data[j * this.clothImgList.length + i];
      const fileName = `${this.clothNames[i]}.${this.way === "toJpeg" ? "jpg" : "png"}`;
      // 将文件添加到文件夹中
      folder.file(fileName, blob, { binary: true });
    }
  }

  // 生成压缩包并下载
  zip.generateAsync({ type: "blob" }).then((content) => {
    FileSaver.saveAs(content, "batch_output.zip");
  });
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
    async adjustImageSize(dataUrl, targetWidth = 1350, targetHeight = 1800) {
  const response = await fetch(dataUrl);
  const blob = await response.blob();
  const img = await createImageBitmap(blob);

  const canvas = document.createElement("canvas");
  canvas.width = targetWidth;
  canvas.height = targetHeight;

  const ctx = canvas.getContext("2d");

  // Compute the scaled dimensions while maintaining aspect ratio
  const scaledWidth = img.width * (targetHeight / img.height);
  const scaledHeight = img.height * (targetWidth / img.width);

  // Center the image within the canvas
  const x = (canvas.width - scaledWidth) / 2;
  const y = (canvas.height - scaledHeight) / 2;

  ctx.drawImage(img, x, y, scaledWidth, scaledHeight);

  return new Promise((resolve, reject) => {
    canvas.toBlob((blob) => {
      if (blob) {
        resolve(blob);
      } else {
        reject(new Error("Failed to convert canvas to Blob"));
      }
    }, this.way === "toJpeg" ? "image/jpeg" : "image/png");
  });
}
  }
};
</script>

         

<style scoped lang="scss">
#app {
  display: flex;

  .left {
    width: 1300px;
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
