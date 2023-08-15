# element-upload
element的一些封装好了的上传组件
**
*上传图片
**
子组件
<template>
  <div class="component-upload-image">
    <el-upload
      :action="uploadImgUrl"
      list-type="picture-card"
      :on-success="handleUploadSuccess"
      :before-upload="handleBeforeUpload"
      :limit="limit"
      :on-error="handleUploadError"
      :on-exceed="handleExceed"
      name="file"
      :on-remove="handleRemove"
      :show-file-list="true"
      :headers="headers"
      :file-list="fileList"
      :on-preview="handlePictureCardPreview"
      :class="{hide: this.fileList.length >= this.limit}"
    >
      <i class="el-icon-plus" />
    </el-upload>

    <!-- 上传提示 -->
    <div v-if="showTip" slot="tip" class="el-upload__tip">
      请上传
      <template v-if="fileSize"> 大小不超过 <b style="color: #f56c6c">{{ fileSize }}MB</b> </template>
      <template v-if="fileType"> 格式为 <b style="color: #f56c6c">{{ fileType.join("/") }}</b> </template>
      的文件
    </div>

    <el-dialog
      :visible.sync="dialogVisible"
      title="预览"
      width="800"
      append-to-body
    >
      <img
        :src="dialogImageUrl"
        style="display: block; max-width: 100%; margin: 0 auto"
      />
    </el-dialog>
  </div>
</template>

<script>
import { getToken } from '@/utils/auth'

export default {
  props: {
    value: [String, Object, Array],
    // 图片数量限制
    limit: {
      type: Number,
      default: 5
    },
    // 大小限制(MB)
    fileSize: {
      type: Number,
      default: 30
    },
    // 文件类型, 例如['png', 'jpg', 'jpeg']
    fileType: {
      type: Array,
      default: () => ['jpg', 'png', 'jpeg', 'bmp']
    },
    // 是否显示提示
    isShowTip: {
      type: Boolean,
      default: true
    }
  },
  data() {
    return {
      dialogImageUrl: '',
      dialogVisible: false,
      hideUpload: false,
      uploadImgUrl: process.env.VUE_APP_BASE_API + '/file/upload', // 上传的图片服务器地址
      headers: {
        Authorization: 'Bearer ' + getToken()
      },
      fileList: []
    }
  },
  computed: {
    // 是否显示提示
    showTip() {
      return this.isShowTip && (this.fileType || this.fileSize)
    }
  },
  watch: {
    value: {
      handler(val) {
        if (val) {
          console.log(val)
          // 首先将值转为数组
          const list = Array.isArray(val) ? val : this.value.split(',')
          // 然后将数组转为对象数组
          this.fileList = list.map(item => {
            if (typeof item === 'string') {
              item = { name: item, url: item }
            }
            return item
          })
          console.log(this.fileList)
        } else {
          this.fileList = []
          return []
        }
      },
      deep: true,
      immediate: true
    }
  },
  methods: {
    // 删除图片
    handleRemove(file, fileList) {
      const findex = this.fileList.map(f => f.name).indexOf(file.name)
      if (findex > -1) {
        this.fileList.splice(findex, 1)
        this.$emit('input', this.listToString(this.fileList))
      }
    },
    // 上传成功回调
    handleUploadSuccess(res) {
      this.fileList.push({ name: res.data.url, url: res.data.url })
      console.log(this.listToString(this.fileList))
      this.$emit('input', this.listToString(this.fileList))
      this.loading.close()
    },
    // 上传前loading加载
    handleBeforeUpload(file) {
      let isImg = false
      if (this.fileType.length) {
        let fileExtension = ''
        if (file.name.lastIndexOf('.') > -1) {
          fileExtension = file.name.slice(file.name.lastIndexOf('.') + 1)
        }
        isImg = this.fileType.some(type => {
          if (file.type.indexOf(type) > -1) return true
          if (fileExtension && fileExtension.indexOf(type) > -1) return true
          return false
        })
      } else {
        isImg = file.type.indexOf('image') > -1
      }

      if (!isImg) {
        this.$message.error(
          `文件格式不正确, 请上传${this.fileType.join('/')}图片格式文件!`
        )
        return false
      }
      if (this.fileSize) {
        const isLt = file.size / 1024 / 1024 < this.fileSize
        if (!isLt) {
          this.$message.error(`上传头像图片大小不能超过 ${this.fileSize} MB!`)
          return false
        }
      }
      this.loading = this.$loading({
        lock: true,
        text: '上传中',
        background: 'rgba(0, 0, 0, 0.7)'
      })
    },
    // 文件个数超出
    handleExceed() {
      this.$message.error(`上传文件数量不能超过 ${this.limit} 个!`)
    },
    // 上传失败
    handleUploadError() {
      this.$message({
        type: 'error',
        message: '上传失败'
      })
      this.loading.close()
    },
    // 预览
    handlePictureCardPreview(file) {
      this.dialogImageUrl = file.url
      this.dialogVisible = true
    },
    // 对象转成指定字符串分隔
    listToString(list, separator) {
      let strs = ''
      separator = separator || ','
      for (const i in list) {
        if (!list[i].url.includes('http')) {
          strs += process.env.VUE_APP_BASE_API_PROXY + 'file?fileName=' + list[i].url + separator
        } else {
          strs += list[i].url + separator
        }
      }
      return strs != '' ? strs.substr(0, strs.length - 1) : ''
    }
  }
}
</script>
<style scoped lang="scss">
// .el-upload--picture-card 控制加号部分
::v-deep.hide .el-upload--picture-card {
    display: none;
}
// 去掉动画效果
::v-deep .el-list-enter-active,
::v-deep .el-list-leave-active {
    transition: all 0s;
}

::v-deep .el-list-enter, .el-list-leave-active {
    opacity: 0;
    transform: translateY(0);
}
</style>



父组件
<ImageUpload v-model="noticeForm.enterpriseLogo" :limit="1" />

**
*上传视频
**
子组件
<template>
  <el-upload
    class="upload-wrapper"
    drag
    :action="uploadFileUrl"
    :headers="headers"
    :multiple="false"
    :limit="1"
    :before-upload="handleBeforeUpload"
    :on-success="handleUploadSuccess"
    :on-error="handleUploadError"
    :file-list="fileList"
    :on-remove="handleUploadRemove"
  >
    <i class="el-icon-upload" />
    <div class="el-upload__text">
      将视频拖到此处，或
      <em>点击上传</em>
    </div>
    <template slot="tip">
      <div class="red-color">
        {{ tip }}
      </div>
    </template>
  </el-upload>
</template>

<script>
import { getToken } from '@/utils/auth'

export default {
  props: {
    value: {
      type: Object,
      default: () => null
    }
  },
  data() {
    return {
      uploadFileUrl: process.env.VUE_APP_BASE_API + '/file/upload',
      headers: {
        Authorization: 'Bearer ' + getToken()
      },
      fileType: ['mp4'],
      fileSize: 100,
      fileList: []
    }
  },
  computed: {
    tip() {
      return `注：请上传${this.fileSize}MB以内 ${this.fileType.join('、')}格式文件`
    }
  },
  watch: {
    value: {
      handler(newVal) {
        this.fileList = newVal ? [newVal] : []
      },
      immediate: true
    }
  },
  methods: {
    // 交给父组件清空
    clearContent() {
      this.fileList = []
    },
    // 上传前校检格式和大小
    handleBeforeUpload(file) {
      // 校检文件类型
      if (this.fileType) {
        let fileExtension = ''
        if (file.name.lastIndexOf('.') > -1) {
          fileExtension = file.name.slice(file.name.lastIndexOf('.') + 1)
        }
        const isTypeOk = this.fileType.some((type) => {
          if (file.type.indexOf(type) > -1) return true
          if (fileExtension && fileExtension.indexOf(type) > -1) return true
          return false
        })
        if (!isTypeOk) {
          this.$message.error(`文件格式不正确, 请上传${this.fileType.join('/')}格式文件!`)
          return false
        }
      }
      // 校检文件大小
      if (this.fileSize) {
        const isLt = file.size / 1024 / 1024 < this.fileSize
        if (!isLt) {
          this.$message.error(`上传文件大小不能超过 ${this.fileSize} MB!`)
          return false
        }
      }
      return true
    },
    handleUploadError() {
      this.$message.error('上传失败, 请重试')
    },
    handleUploadSuccess(res, file) {
      this.$message.success('上传成功')
      this.$emit('input', { name: file.name, url: res.data.url })
    },
    handleUploadRemove() {
      this.fileList = []
      this.$emit('input', null)
    }
  }
}
</script>

<style lang="scss" scoped>
.upload-wrapper {
  margin-bottom: 20px;
  width: 380px;
}

.red-color {
  color: #f56c6c;
}
</style>

父组件
<UploadVideo v-model="noticeForm.promotionalVideo" />

**
*上传封面并剪切
**
子组件
<template>
  <div class="upload-container">
    <el-dialog title="图片上传" :visible.sync="dialogAvtFirst" width="30%" @close="closeUpload">
      <span class="avt-title">上传文件图片</span>
      <span class="avt-tip">图片尺寸需要大于{{ autoCropWidth }} * {{ autoCropHeight }}像素，</span>
      <span class="avt-tip">支持jpg、png、jpeg等格式，大小不能超过10MB</span>
      <span slot="footer" class="dialog-footer">
        <el-button type="cancel" size="small" @click="closeUpload">取 消</el-button>
        <!-- 选取文件 -->
        <el-upload
          ref="upload"
          action=""
          :file-list="fileList"
          :auto-upload="false"
          :show-file-list="false"
          :on-change="beforeUpload"
          :limit="uploadlimit"
          style="display: inline-block; margin-left: 20px"
        >
          <el-button type="confirm" size="small">上传</el-button>
        </el-upload>
      </span>
    </el-dialog>
    <el-dialog :title="selectTilte" :visible.sync="dialogSelectAvt" width="806px">
      <div class="select-avt-box">
        <div class="cropper-container">
          <vueCropper
            ref="cropper"
            :img="option.img"
            :output-size="option.outputSize"
            :output-type="option.outputType"
            :auto-crop="option.autoCrop"
            :auto-crop-width="option.autoCropWidth"
            :auto-crop-height="option.autoCropHeight"
            :fixed="option.fixed"
            :info-true="option.infoTrue"
            :enlarge="option.enlarge"
            :fixed-box="option.fixedBox"
            @realTime="realTime"
          />
        </div>
        <div class="avt-preview-container">
          <span class="avt-preview-title">{{ title }}预览</span>
          <div :style="{width: previews.w + 'px',height: previews.h + 'px',overflow: 'hidden',margin: '5px auto'}">
            <div :style="previews.div" style="border: 1px solid #e8e9eb; overflow: hidden">
              <img :src="option.img" :style="previews.img" />
            </div>
          </div>
          <p style="margin: 20px auto" />
          <div :style="previewStyle1">
            <div :style="previews.div" style="border: 1px solid #e8e9eb; overflow: hidden">
              <img :src="previews.url" :style="previews.img" />
            </div>
          </div>
          <p style="margin: 20px auto" />
          <!--          <div :style="previewStyle2">-->
          <!--            <div :style="previews.div" style="border: 1px solid #e8e9eb; overflow: hidden">-->
          <!--              <img :src="previews.url" :style="previews.img"/>-->
          <!--            </div>-->
          <!--          </div>-->
        </div>
      </div>

      <span slot="footer" class="dialog-footer">
        <el-upload
          ref="uploadRe"
          action=""
          :file-list="fileList"
          :auto-upload="false"
          :show-file-list="false"
          :on-change="beforeUpload"
          :limit="uploadlimit"
          style="display: inline-block; margin-right: 20px"
        >
          <el-button type="cancel" size="small" @click="closeUpload">重新上传</el-button>
        </el-upload>
        <el-button type="confirm" :loading="uploadLoading" size="small" @click="uploadAvt">确定</el-button>
      </span>
    </el-dialog>
  </div>
</template>

<script>
import { VueCropper } from 'vue-cropper'
import { faceUpload } from '@/api/basisData/population'
import { FileUpload } from '../../api/partyManage.js'
import { dataURLtoFile } from '../../utils/common.js'
import defaultAvt from '@/assets/images/people/icon-default.png'

export default {
  name: '',
  components: {
    VueCropper
  },
  props: {
    isList: {
      type: Boolean,
      default: false
    },
    dialogAvt: {
      type: Boolean,
      default: false
    },
    title: {
      // 标题
      type: String,
      default: '头像'
    },
    defaultAvt: {
      // 默认头像
      type: String,
      default: defaultAvt
    },
    autoCropWidth: {
      // 默认生成截图框宽度
      type: Number,
      default: 100
    },
    autoCropHeight: {
      // 默认生成截图框高度
      type: Number,
      default: 100
    },
    fixedBox: {
      // 是否固定截图框大小
      type: Boolean,
      default: true
    },
    avtData: {
      type: Object,
      default: {}
    }
  },
  data() {
    return {
      fileName: '',
      uploadLoading: false,
      uploadlimit: 1,
      fileList: [],
      dialogSelectAvt: false,
      option: {
        img: this.defaultAvt, // 裁剪图片的地址
        autoCropWidth: this.autoCropWidth, // 默认生成截图框宽度
        autoCropHeight: this.autoCropHeight, // 默认生成截图框高度
        fixed: false, // 是否开启截图框宽高固定比例
        enlarge: 2,
        fixedBox: this.fixedBox, // 固定截图框大小 不允许改变
        outputSize: 1, // 裁剪生成图片的质量
        autoCrop: true, // 是否默认生成截图框
        outputType: 'png', // 裁剪生成图片的格式
        infoTrue: true // true 为展示真实输出图片宽高 false 展示看到的截图框宽高
      },
      previews: {},
      previewStyle1: {},
      previewStyle2: {},
      fileOri: null
    }
  },
  computed: {
    selectTilte() {
      return '选择' + this.title
    },
    dialogAvtFirst: {
      get() {
        return this.dialogAvt
      },
      set() {
      }
    }
  },
  watch: {},
  created() {
  },
  mounted() {
  },
  methods: {
    /** 人脸变化 */
    changeAvtInfo() {
      if (this.avtData.image && this.avtData.image != null) {
        this.option.img = this.avtData.image
      }
    },
    beforeUpload(fileObj) {
      const file = fileObj.raw
      this.fileOri = file
      const isJPG =
          file.type === 'image/jpeg' ||
          file.type === 'image/png' ||
          file.type === 'image/jpg'
      const isLt2M = file.size / 1024 / 1024 < 10
      this.fileName = file.name
      if (!isJPG) {
        this.$message.error('上传图片只能是 jpg、png、jpeg 等格式!')
        this.fileList = []
        return false
      } else {
        if (!isLt2M) {
          this.$message.error('上传图片大小不能超过 10MB!')
          this.fileList = []
          return false
        } else {
          const _this = this
          // 上传头像图片尺寸需要大于100 * 100像素
          // 获取上传的图片的宽高
          var reader = new FileReader()
          reader.readAsDataURL(file)
          reader.onload = function (evt) {
            var replaceSrc = evt.target.result
            var imageObj = new Image()
            imageObj.src = replaceSrc
            imageObj.onload = function () {
              if (imageObj.width > 100 && imageObj.height > 100) {
                _this.option.img = URL.createObjectURL(file)
                _this.closeUpload()
                _this.dialogSelectAvt = true
                return true
              } else {
                _this.$message.error('上传图片尺寸需要大于100 * 100像素!')
                return false
              }
            }
          }
        }
      }
    },
    /** 人脸更新 */
    uploadAvt() {
      this.uploadLoading = true
      this.$refs.cropper.getCropData(data => {
        const fileName = this.fileName ? this.fileName : new Date().getTime() + '.png'
        const fileData = dataURLtoFile(data, fileName)
        this.fileUpload(fileData)
      })
    },
    /** 上传文件 */
    async fileUpload(file) {
      const formData = new FormData()
      formData.append('file', file)
      const res = await FileUpload(formData)
      /**
         * 1、如果是新增则将图片的url传回新增页面
         * 2、如果是编辑，则更新人脸图片
         **/
      if (!this.isList) {
        // 新增或者编辑页面
        const formDataOri = new FormData()
        formDataOri.append('file', this.fileOri)
        const resOri = await FileUpload(formDataOri)
        this.$listeners.setAddFace(res.data.url, resOri.data.url)
        this.uploadLoading = false
        this.dialogSelectAvt = false
        return
      }
      // 在列表页人脸更新提交
      const param = {
        faceUrl: res.data.url,
        residentId: this.avtData.residentId
      }
      faceUpload(param).then(res => {
        if (res.code === 200) {
          this.uploadLoading = false
          this.dialogSelectAvt = false
          this.$listeners.fetchData()
          this.$notify({ title: '人脸照片更新成功！', type: 'success' })
        } else {
          this.uploadLoading = false
        }
      })
        .catch(() => {
          this.uploadLoading = false
        })
    },
    /** 头像预览 */
    realTime(data) {
      var previews = data
      var h = 0.6
      var w = 0.3

      this.previewStyle1 = {
        width: previews.w + 'px',
        height: previews.h + 'px',
        overflow: 'hidden',
        margin: '0 auto',
        zoom: h
      }

      this.previewStyle2 = {
        width: previews.w + 'px',
        height: previews.h + 'px',
        overflow: 'hidden',
        margin: '0 auto',
        zoom: w
      }
      this.previews = data
    },
    /** 关闭弹框 */
    closeUpload() {
      this.fileList = []
      this.$emit('close')
    },
    selectAVt() {
      this.closeUpload()
    }
  }
}
</script>
<style lang="scss" scoped>
  .upload-container {
    .avt-title {
      display: block;
      margin-bottom: 22px;
      text-align: center;
      font-size: 18px;
      font-family: Source Han Sans CN;
      font-weight: bold;
      line-height: 25px;
      color: #1f2c4f;
    }

    .avt-tip {
      display: block;
      text-align: center;
      font-size: 14px;
      font-family: Source Han Sans CN;
      font-weight: 400;
      line-height: 19px;
      color: #8f99a4;
    }

    .select-avt-box {
      display: flex;
      //头像裁剪框
      .cropper-container {
        width: 400px;
        height: 400px;
        margin-right: 20px;
      }

      //头像预览
      .avt-preview-container {
        width: 200px;
        text-align: center;

        .preivew-item-box {
          margin-top: 12px;
        }

        .avt-preview-title {
          display: inline-block;
          margin-bottom: 10px;
          font-size: 16px;
          font-family: Source Han Sans CN;
          font-weight: 400;
          line-height: 21px;
          color: #1f2c4f;
        }

        .preview-title {
          display: inline-block;
          margin-bottom: 8px;
        }
      }

      .el-upload {
        display: inline-block;
      }
    }
  }
</style>

父组件
<el-form-item
  label="景区导览"
  prop="guideUrl"
  :class="noticeForm.guideUrl ? 'form-cover-upload' : ''"
>
  <div class="img-box" @click="changeAvtHandle">
    <el-image
      v-if="noticeForm.guideUrl"
      :src="headPortrait"
      class="avatar"
      alt="封面"
    />
    <i v-else class="el-icon-plus" />
  </div>
</el-form-item>
<UploadCover
      ref="uploadAvt"
      :dialog-avt="dialogAvt"
      :avt-data="avtData"
      :auto-crop-height="180"
      :auto-crop-width="180"
      :title="'景区导览'"
      @close="dialogAvt = false"
      @fetchData="fetchData"
      @setAddFace="setAddFace"
    />

methods: {
  // 更新封面后数据刷新
    fetchData() {
      this.$listeners.update()
  },
  // 新增封面
  setAddFace(image, fileOri) {
    this.noticeForm.guideUrl = image
    this.noticeForm.guideUrlOri = fileOri
    handleImgURL(image).then(res => {
      this.headPortrait = res
      this.$forceUpdate()
    })
  },
}  
