<template>
 <div class="v-simple-cropper" style="margin-top:200px;">
   <div v-for="(item, index) in log" :key="index">{{item}}</div>
 <slot>
  <button @click="upload">上传图片</button>
 </slot>
 <input class="file" ref="file" type="file" accept="image/*" @change="uploadChange">
 <div class="v-cropper-layer" ref="layer">
  <div class="layer-header">
  <button class="cancel" @click="cancelHandle">取消</button>
  <button class="confirm" @click="confirmHandle">完成</button>
  </div>
  <img ref="cropperImg">
 </div>
 </div>
</template>
<script>
/* eslint-disable */ 
import Cropper from 'cropperjs'
import axios from 'axios'
// toBlob plloyfile
if (!HTMLCanvasElement.prototype.toBlob) {
  Object.defineProperty(HTMLCanvasElement.prototype, 'toBlob', {
    value (callback, type, quality) {
      var binStr = atob(this.toDataURL(type, quality).split(',')[1])
      var len = binStr.length
      var arr = new Uint8Array(len)
      for (var i = 0; i < len; i++) {
        arr[i] = binStr.charCodeAt(i)
      }
      callback(new Blob([arr], {type: type || 'image/png'}))
    }
  })
}
export default {
  name: 'v-simple-cropper',
  props: {
    initParam: Object,
    successCallback: {
      type: Function,
      default: () => {}
    }
  },
  data () {
    return {
      cropper: {},
      filename: '',
      log: [],
      imgFile:{}
    }
  },
  mounted () {
    this.init()
  },
  methods: {
    // 初始化裁剪插件
    init () {
      let cropperImg = this.$refs['cropperImg']
      this.cropper = new Cropper(cropperImg, {
        aspectRatio: 1 / 1,
        dragMode: 'move'
      })
    },
    // 点击上传按钮
    upload () {
      this.$refs['file'].click()
    },
    // 选择上传文件
    uploadChange (e) {
      let file = e.target.files[0]
      this.filename = file['name']
      let URL = window.URL || window.webkitURL
      this.$refs['layer'].style.display = 'block'
      this.cropper.replace(URL.createObjectURL(file))
    },
    // 取消上传
    cancelHandle () {
      this.cropper.reset()
      this.$refs['layer'].style.display = 'none'
      this.$refs['file'].value = ''
    },
    // 将base64转换为文件
    dataURLtoBlob (base64Data) {
      // let arr = dataurl.split(',')
      // let mime = arr[0].match(/:(.*?);/)[1]
      // let bstr = atob(arr[1])
      // let n = bstr.length
      // let u8arr = new Uint8Array(n)
      // while (n--) {
      //   this.n = n
      //   u8arr[n] = bstr.charCodeAt(n)
      // }
      // return new File([u8arr], filename, {type: mime})
      let byteString
      if (base64Data.split(',')[0].indexOf('base64') >= 0) {
        byteString = atob(base64Data.split(',')[1])
      } else {
        byteString = unescape(base64Data.split(',')[1])
      }
      let mimeString = base64Data.split(',')[0].split(':')[1].split(';')[0]
      let ia = new Uint8Array(byteString.length)
      for (let i = 0; i < byteString.length; i++) {
        ia[i] = byteString.charCodeAt(i)
      }
      let blob =  new Blob([ia], {type: mimeString})
      return new File([blob], parseInt(Math.random() * 10000) + '.jpeg', {type: mimeString})
    },
    confirmHandle () {
      this.$refs['layer'].style.display = 'none'
      this.$emit('progress', 0)
      let cropBox = this.cropper.getCropBoxData()
      // let scale = this.initParam['scale'] || 1
      let cropCanvas = this.cropper.getCroppedCanvas({
        width: cropBox.width * 1,
        height: cropBox.height * 1
      })
      let imgData = cropCanvas.toDataURL('image/jpeg')
      let formData = new FormData()
      let file = this.dataURLtoBlob(imgData)
      // let file = new File([blob], parseInt(Math.random() * 10000) + '.jpeg')
      this.handleInputChange(file)
    },
    // 确定上传
    handleInputChange (file) {
      this.log.push('handleInputChange')
      // const file = event.target.files[0]
      const imgMasSize = 1024 * 1024 * 10
      // 检查文件类型
      if (['jpeg', 'png', 'gif', 'jpg'].indexOf(file.type.split('/')[1]) < 0) {
        // 自定义报错方式
        return
      }
      // 文件大小限制
      if (file.size > imgMasSize) {
        return
      }
      // 判断是否是ios
      // eslint-disable-next-line
      if (!!window.navigator.userAgent.match(/\(i[^]+( U)? CPU.+Mac OS X/)) {
        this.transformFileToFormData(file)
        return
      }
      // 图片压缩
      this.transformFileToDataUrl(file)
    },
    // 将File append进 FormData
    transformFileToFormData (file) {
      this.log.push('transformFileToFormData')
      const formData = new FormData()
      // append 文件
      // 上传图片
      this.uploadImg(formData)
    },
    // 将file转成dataUrl
    transformFileToDataUrl (file) {
      this.log.push('transformFileToDataUrl')
      const imgCompassMaxSize = 200 * 1024 // 超过 200k 就压缩
      const processData = this.processData
      // 存储文件相关信息
      this.imgFile.type = file.type || 'image/jpeg' // 部分安卓出现获取不到type的情况
      this.imgFile.size = file.size
      this.imgFile.name = file.name
      this.imgFile.lastModifiedDate = file.lastModifiedDate
      // 封装好的函数
      const reader = new FileReader()
      // file转dataUrl是个异步函数，要将代码写在回调里
      reader.onload = (e) => {
        const result = e.target.result
        if (result.length < imgCompassMaxSize) {
          this.compress(result, processData, false) // 图片不压缩
        } else {
          this.compress(result, processData) // 图片压缩
        }
      }
      reader.readAsDataURL(file)
    },
    // 使用canvas绘制图片并压缩
    compress (dataURL, callback, shouldCompress = true) {
      this.log.push('compress')
      const img = new window.Image()
      img.src = dataURL
      img.onload = () => {
        const canvas = document.createElement('canvas')
        const ctx = canvas.getContext('2d')
        canvas.width = img.width
        canvas.height = img.height
        ctx.drawImage(img, 0, 0, canvas.width, canvas.height)
        let compressedDataUrl
        if (shouldCompress) {
          compressedDataUrl = canvas.toDataURL(this.imgFile.type, 0.2)
        } else {
          compressedDataUrl = canvas.toDataURL(this.imgFile.type, 1)
        }
        callback(compressedDataUrl)
      }
    },
    processData (dataUrl) {
      this.log.push('processData')
      // 这里使用二进制方式处理dataUrl
      const binaryString = window.atob(dataUrl.split(',')[1])
      const arrayBuffer = new ArrayBuffer(binaryString.length)
      const intArray = new Uint8Array(arrayBuffer)
      for (let i = 0, j = binaryString.length; i < j; i++) {
        intArray[i] = binaryString.charCodeAt(i)
      }
      const data = [intArray]
      let blob
      try {
        blob = new Blob(data, {
          type: this.imgFile.type
        })
      } catch (error) {
        window.BlobBuilder = window.BlobBuilder ||
          window.WebKitBlobBuilder ||
          window.MozBlobBuilder ||
          window.MSBlobBuilder
        let BlobBuilder = window.BlobBuilder
        if (error.name === 'TypeError' && window.BlobBuilder) {
          const builder = new BlobBuilder()
          builder.append(arrayBuffer)
          blob = builder.getBlob(this.imgFile.type)
        } else {
          throw new Error('版本过低，不支持上传图片')
        }
      }
      // blob 转file
      const fileOfBlob = new File([blob], this.imgFile.name)
      const formData = new FormData()
      // append 文件
      formData.append('uploadFile', fileOfBlob)
      this.log.push('fileOfBlob' + fileOfBlob.size)
      this.uploadImg(formData)
    },
    // 上传图片
    uploadImg (formData) {
      // console.log('this.initParam.uploadURL',this.initParam.uploadURL)
      for (var i in formData) {
        this.log.push('formData i' + i + ',' + formData[i])
      }

      // let formData = new FormData();        
      // formData.append('uploadFile',this.signFile);
      $.ajax({
          url: this.apiPath+this.URL['uploadProgress'],
          dataType: 'json',
          type: 'POST',
          // headers: {'token': sessionStorage.getItem("token")},
          processData: false,
          contentType: false,
          cache: false,
          data: formData,
      }).done((json) => {
          if(json.meta.success)
          this.uploadFileState = false;
          this.docId =  json.data.docId;
          this.setSData('docId',this.docId)
      }).fail((err) => {
          console.log(err)    
      });
      
      // axios.post(this.initParam.uploadURL, formData, {
      //     headers: { 'Content-Type': 'multipart/form-data' },
      //     onUploadProgress: e => {
      //       // 对原生进度事件的处理
      //       let rate = parseInt(e.loaded / e.total * 100)
      //       this.$emit('progress', rate)
      //     },
      //   })
      //   .then(res => {
      //     // this.successCallback(res.data)
      //     // this.cancelHandle()
      //     let {data} = res
      //     console.log(data)
      //     if (data.statusCode === 200) {
      //       // 上传成功
      //       this.$emit('success', data)
      //     } else {
      //       // 上传失败
      //       this.$emit('fail', data)
      //     }
      //     this.log.push(JSON.stringify(data))
      //   })
    }
    // confirmHandle () {
    //   this.$refs['layer'].style.display = 'none'
    //   this.$emit('progress', 0)
    //   let cropBox = this.cropper.getCropBoxData()
    //   let scale = this.initParam['scale'] || 1
    //   let cropCanvas = this.cropper.getCroppedCanvas({
    //     width: cropBox.width * scale,
    //     height: cropBox.height * scale
    //   })
    //   let imgData = cropCanvas.toDataURL('image/jpeg')
    //   let formData = new FormData()
    //   try{
    //     let blob = this.dataURLtoBlob(imgData)  
    //     let file = new File([blob], parseInt(Math.random() * 10000) + '.jpeg')
    //     formData.append('upload_file', file)
    //     this.log.push(blob.isClosed)  
    //     this.log.push(file.size)      
    //   } catch(e) {
    //     this.log.push('124',e)   
    //   }
    //   const xhr = new XMLHttpRequest()
    //   // 进度监听
    //   xhr.upload.addEventListener('progress', (e) => {
    //     let rate = parseInt(e.loaded / e.total * 100)
    //     this.$emit('progress', rate)
    //   }, false)
    //   // 加载监听
    //   xhr.addEventListener('load', () => console.log('加载中'), false)
    //   // 错误监听
    //   xhr.addEventListener('error', () => {}, false)
    //   xhr.onreadystatechange = () => {
    //     if (xhr.readyState === 4) {
    //       const result = JSON.parse(xhr.responseText)
    //       if (xhr.status === 200) {
    //         // 上传成功
    //         this.$emit('success', result)
    //       } else {
    //         // 上传失败
    //         this.$emit('fail', result)
    //       }
    //     }
    //   }
    //   xhr.open('POST', this.initParam.uploadURL, true)
    //   xhr.send(formData)
    // }
  }
}
</script>
