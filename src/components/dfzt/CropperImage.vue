<template>
  <div class="cropperModel">
    <div class="demo" @click="show">
      <img :src="imgurl" v-if="imgurl" alt="avatar" />
      <a-icon type="plus" v-else />
    </div>
    <a-modal v-model="cropperModel" title="选择图片">
      <div class="cropper-content">
        <div class="cropper-box">
          <div class="cropper">
            <vue-cropper
              ref="cropper"
              :img="option.img"
              :outputSize="option.outputSize"
              :outputType="option.outputType"
              :info="option.info"
              :canScale="option.canScale"
              :autoCrop="option.autoCrop"
              :autoCropWidth="option.autoCropWidth"
              :autoCropHeight="option.autoCropHeight"
              :fixed="option.fixed"
              :fixedNumber="option.fixedNumber"
              :full="option.full"
              :fixedBox="option.fixedBox"
              :canMove="option.canMove"
              :canMoveBox="option.canMoveBox"
              :original="option.original"
              :centerBox="option.centerBox"
              :height="option.height"
              :infoTrue="option.infoTrue"
              :maxImgSize="option.maxImgSize"
              :enlarge="option.enlarge"
              :mode="option.mode"
              @realTime="realTime"
              @imgLoad="imgLoad"
            >
            </vue-cropper>
          </div>
          <!--底部操作工具按钮-->
        </div>
        <!-- <div class="show-preview">
      <div :style="previews.div" class="preview">
        <img :src="previews.url" :style="previews.img" />
      </div>
    </div> -->
      </div>
      <div class="footer-btn" slot="footer">
        <div class="scope-btn">
          <label class="btn" for="uploads">选择图片</label>
          <input
            type="file"
            id="uploads"
            style="position:absolute; clip:rect(0 0 0 0);"
            accept="image/png, image/jpeg, image/gif, image/jpg"
            @change="selectImg($event)"
          />
          <a-button shape="round" type="primary" @click="rotateLeft">↺ 左旋转</a-button>
          <a-button shape="round" type="primary" @click="rotateRight">↻ 右旋转</a-button>
          <a-button type="primary" shape="round" @click="uploadImg('blob')"
            >保存 <i class="el-icon-upload"></i
          ></a-button>
          <a-button shape="round" @click="close">关闭</a-button>
        </div>
      </div>
      <!--预览效果图-->
    </a-modal>
  </div>
</template>

<script>
import { VueCropper } from 'vue-cropper'
import { blobToDataURL, dataURLtoBlob } from '@/utils/util'
export default {
  name: 'CropperImage',
  components: {
    VueCropper
  },
  props: {
    value: {
      type: String,
      required: false
    }
  },
  model: {
    prop: 'value',
    event: 'change'
  },
  data() {
    return {
      cropperModel: false,
      name: this.Name,
      previews: {},
      option: {
        img: '', //裁剪图片的地址
        outputSize: 1, //裁剪生成图片的质量(可选0.1 - 1)
        outputType: 'png', //裁剪生成图片的格式（jpeg || png || webp）
        info: true, //图片大小信息
        canScale: false, //图片是否允许滚轮缩放
        autoCrop: true, //是否默认生成截图框
        autoCropWidth: 1200, //默认生成截图框宽度
        autoCropHeight: 1600, //默认生成截图框高度
        fixed: true, //是否开启截图框宽高固定比例
        fixedNumber: [3, 4], //截图框的宽高比例
        full: true, //false按原比例裁切图片，不失真
        fixedBox: true, //固定截图框大小，不允许改变
        canMove: false, //上传图片是否可以移动
        canMoveBox: false, //截图框能否拖动
        original: false, //上传图片按照原始比例渲染
        centerBox: true, //截图框是否被限制在图片里面
        height: true, //是否按照设备的dpr 输出等比例图片
        infoTrue: false, //true为展示真实输出图片宽高，false展示看到的截图框宽高
        maxImgSize: 1000, //限制图片最大宽度和高度
        enlarge: 1, //图片根据截图框输出比例倍数
        mode: '200px 200px' //图片默认渲染方式
      },
      imgurl: ''
    }
  },
  watch: {
    value: {
      immediate: true,
      handler: function() {
        console.log("init img")
        this.initVal()
      }
    }
  },
  methods: {
    initVal() {
      if (!this.value) {
        this.imgurl = ''
      } else {
        let blob = dataURLtoBlob(this.value)
        this.imgurl = URL.createObjectURL(blob)
        this.option.img = this.imgurl
      }
    },
    close() {
      this.cropperModel = false
    },
    show() {
      this.cropperModel = true
    },
    //初始化函数
    imgLoad(msg) {
      console.log('工具初始化函数=====' + msg)
    },
    //图片缩放
    changeScale(num) {
      num = num || 1
      this.$refs.cropper.changeScale(num)
    },
    //向左旋转
    rotateLeft() {
      this.$refs.cropper.rotateLeft()
    },
    //向右旋转
    rotateRight() {
      this.$refs.cropper.rotateRight()
    },
    //实时预览函数
    realTime(data) {
      this.previews = data
    },
    //选择图片
    selectImg(e) {
      let file = e.target.files[0]
      if (!/\.(jpg|jpeg|png|JPG|PNG)$/.test(e.target.value)) {
        this.$message.warning('图片类型要求：jpeg、jpg、png')
        return false
      }
      //转化为blob
      let reader = new FileReader()
      reader.onload = e => {
        let data
        if (typeof e.target.result === 'object') {
          data = window.URL.createObjectURL(new Blob([e.target.result]))
        } else {
          data = e.target.result
        }
        this.option.img = data
      }
      //转化为base64
      reader.readAsDataURL(file)
    },
    //上传图片
    async uploadImg(type) {
      let _this = this
      if (type === 'blob') {
        //获取截图的blob数据
        this.$refs.cropper.getCropBlob(async data => {
          let base64 = await blobToDataURL(data)
          this.$emit('change', base64)
          this.close()
        })
      }
    }
  }
}
</script>

<style scoped lang="less">
/deep/ .ant-modal-body {
  padding: 0;
}
.cropper-content {
  // display: flex;
  // display: -webkit-flex;
  // justify-content: flex-end;
  margin: 32px;
  .cropper-box {
    display: flex;
    justify-content: center;
    .cropper {
      width: auto;
      height: 300px;
      width: 300px;
    }
  }

  .show-preview {
    flex: 1;
    -webkit-flex: 1;
    display: flex;
    display: -webkit-flex;
    justify-content: center;
    .preview {
      overflow: hidden;
      border: 1px solid #67c23a;
      background: #cccccc;
    }
  }
}
.footer-btn {
  display: flex;
  display: -webkit-flex;
  justify-content: center;
  .ant-btn {
    margin-right: 8px;
  }
  .scope-btn {
    display: flex;
    display: -webkit-flex;
    justify-content: center;
    padding-right: 10px;
  }
  .upload-btn {
    flex: 1;
    -webkit-flex: 1;
    display: flex;
    display: -webkit-flex;
    justify-content: center;
  }
  .btn {
    outline: none;
    display: inline-block;
    line-height: 1;
    white-space: nowrap;
    cursor: pointer;
    -webkit-appearance: none;
    text-align: center;
    -webkit-box-sizing: border-box;
    box-sizing: border-box;
    outline: 0;
    -webkit-transition: 0.1s;
    transition: 0.1s;
    font-weight: 500;
    padding: 8px 15px;
    font-size: 12px;
    border-radius: 3px;
    color: #fff;
    background-color: #409eff;
    border-color: #409eff;
    margin-right: 10px;
    border-radius: 16px;
  }
}
.demo {
  border-radius: 2px;
  width: 104px;
  height: 104px;
  border: 1px dashed #e2e2e2;
  border-radius: 2px;
  padding: 8px;
  display: flex;
  justify-content: center;
  align-items: center;
  font-size: 60px;
  color: #f0f2f5;
  img {
    height: 100%;
  }
}
</style>
