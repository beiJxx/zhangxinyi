<template>
  <div>
    <input ref="excel-upload-input" accept=".xlsx, .xls" class="excel-upload-input" type="file" @change="handleClick">
    <div class="drop" @dragenter="handleDragover" @dragover="handleDragover" @drop="handleDrop">
      <label v-show="excelData.fileName">{{ excelData.fileName ? '( ' + excelData.fileName + ' ) ' + ' 已上传' : (loading ? '正在上传。。。' : '贷款导出(未上传)') }}</label>
      <el-button v-show="!excelData.fileName"
        v-loading.fullscreen.lock="loading"
        element-loading-background="rgba(0, 0, 0, 0.8)"
        element-loading-spinner="el-icon-loading"
        element-loading-text="正在疯狂加载，请等一会"
        type="primary" @click="handleUpload">
        选择文件 (贷款导出)
      </el-button>
    </div>
  </div>
</template>

<script>
import XLSX from 'xlsx'

export default {
  props: {
    beforeUpload: Function, // eslint-disable-line
    onSuccess: Function// eslint-disable-line
  },
  data() {
    return {
      loading: false,
      excelData: {
        resultsArray: [],
        fileName: null
      }
    }
  },
  methods: {
    generateData(resultsArray) {
      this.excelData.resultsArray = resultsArray
      this.onSuccess && this.onSuccess(this.excelData)
    },
    handleDrop(e) {
      e.stopPropagation()
      e.preventDefault()
      if (this.loading) return
      const files = e.dataTransfer.files
      if (files.length !== 1) {
        this.$message.error('Only support uploading one file!')
        return
      }
      const rawFile = files[0] // only use files[0]

      if (!this.isExcel(rawFile)) {
        this.$message.error('Only supports upload .xlsx, .xls, .csv suffix files')
        return false
      }
      this.upload(rawFile)
      e.stopPropagation()
      e.preventDefault()
    },
    handleDragover(e) {
      e.stopPropagation()
      e.preventDefault()
      e.dataTransfer.dropEffect = 'copy'
    },
    handleUpload() {
      this.$refs['excel-upload-input'].click()
    },
    handleClick(e) {
      const files = e.target.files
      const rawFile = files[0] // only use files[0]
      if (!rawFile) return
      this.upload(rawFile)
    },
    upload(rawFile) {
      this.$refs['excel-upload-input'].value = null // fix can't select the same excel
      this.excelData.fileName = null
      if (!this.beforeUpload) {
        this.readerData(rawFile)
        return
      }
      const before = this.beforeUpload(rawFile)
      if (before) {
        this.readerData(rawFile)
      }
    },
    readerData(rawFile) {
      this.loading = true
      return new Promise((resolve, reject) => {
        const reader = new FileReader()
        reader.onload = e => {
          var binary = ''
          const data = e.target.result
          var bytes = new Uint8Array(data)
          var length = bytes.byteLength
          for (var i = 0; i < length; i++) {
            binary += String.fromCharCode(bytes[i])
          }
          // 接下来就是xlsx了，具体可看api
          const wb = XLSX.read(binary, {
            type: 'binary',
            codepage: 936
          })
          const firstSheetName = wb.SheetNames[0]
          const worksheet = wb.Sheets[firstSheetName]
          const header = this.getHeaderRow(worksheet)
          const results = XLSX.utils.sheet_to_json(worksheet)
          // 自定义方法向父组件传递数据
          // console.dir(results)
          let count = 0
          let resultsArray = []
          if (header.indexOf('责任人') === -1 || header.indexOf('客户名称') === -1 || header.indexOf('贷款余额') === -1) {
            this.generateData(resultsArray)
            this.loading = false
          } else {


            for (let i = 0; i < results.length; i++) {
              // console.log(results[i])
              var name = results[i].责任人.replace(/\s+/g, "");
              var company = results[i].客户名称.replace(/\s+/g, "");
              var balance = results[i].贷款余额;
              var filter = results[i].业务登记人条线.replace(/\s+/g, "");
              var filterAll = ['大贷业务条线', '小企业业务条线']
              if (name && company && balance && filterAll.indexOf(filter) !== -1) {
              // if (name && company && balance ) {
                count++;
                // console.log(name + '-' + company + '-' + balance)
                resultsArray.push(name + '-' + company + '-' + balance);
              }
            }
            console.log('count:' + count);
            // console.log(resultsArray)
            this.excelData.fileName = rawFile.name;
            this.generateData(resultsArray)
            this.loading = false
            resolve()
          }
        }
        reader.readAsArrayBuffer(rawFile)
      })
    },
    getHeaderRow(sheet) {
      const headers = []
      const range = XLSX.utils.decode_range(sheet['!ref'])
      let C
      const R = range.s.r
      /* start in the first row */
      for (C = range.s.c; C <= range.e.c; ++C) { /* walk every column in the range */
        const cell = sheet[XLSX.utils.encode_cell({c: C, r: R})]
        /* find the cell in the first row */
        let hdr = 'UNKNOWN ' + C // <-- replace with your desired default
        if (cell && cell.t) hdr = XLSX.utils.format_cell(cell)
        headers.push(hdr)
      }
      return headers
    },
    isExcel(file) {
      return /\.(xlsx|xls|csv)$/.test(file.name)
    }
  }
}
</script>

<style scoped>
.excel-upload-input {
  display: none;
  z-index: -9999;
}

.drop {
  border: 2px dashed #bbb;
  width: 25%;
  height: 100px;
  line-height: 100px;
  margin: 20px auto;
  font-size: 16px;
  border-radius: 5px;
  text-align: center;
  color: red;
  position: relative;
}
</style>
