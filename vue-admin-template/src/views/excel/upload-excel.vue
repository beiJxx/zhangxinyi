<template>
  <div class="app-container">
    <upload-excel-component :before-upload="beforeUpload" :on-success="handleSuccess1"/>
    <upload-excel-component2 :before-upload="beforeUpload" :on-success="handleSuccess2"/>
    <div class="drop">
      <el-button :disabled="false" type="success" @click="handleCompare">
        开始比对
      </el-button>
    </div>

    <div style="margin-top: 20px">
      <el-row class="demo-autocomplete">
        <el-col :span="11" style="margin-left: 20px">
          <el-input v-model="r1Array.length" disabled style="margin: 20px 0 20px 0;width: 100%;text-align: right">
            <template slot="prepend">手工台账</template>
            <template slot="append">条记录</template>
          </el-input>
          <div style="position: relative">
            <el-tag effect="dark">手工台账多余记录 {{ table1_2 ? ' ( 共' + table1_2.length + '条 ) ' : '' }}</el-tag>
          </div>
          <el-table
            :cell-style="{padding:'0px'}"
            :data="table1_2"
            :row-style="{height:'35px'}"
            border
            stripe
            style="width: 100%;margin-top: 20px;font-family: 幼圆;font-size: 16px"
            tooltip-effect="dark">
            <el-table-column
              label="管护经理"
              prop="name"
              width="120">
            </el-table-column>
            <el-table-column
              label="户名"
              prop="company"
              width="500">
            </el-table-column>
            <el-table-column
              label="贷款余额"
              prop="balance"
            >
            </el-table-column>
          </el-table>
        </el-col>
        <el-col :span="11" style="margin-left: 5%">
          <el-input v-model="r2Array.length" disabled style="margin: 20px 0 20px 0;width: 100%;">
            <template slot="prepend">贷款导出</template>
            <template slot="append">条记录</template>
          </el-input>
          <div>
            <el-tag effect="dark">贷款导出多余记录 {{ table2_1 ? ' ( 共' + table2_1.length + '条 ) ' : '' }}</el-tag>
          </div>
          <el-table
            :cell-style="{padding:'0px'}"
            :data="table2_1"
            :row-style="{height:'35px'}"
            border
            stripe
            style="width: 100%;margin-top: 20px;font-family: 幼圆;font-size: 16px">
            <el-table-column
              label="责任人"
              prop="name"
              width="120">
            </el-table-column>
            <el-table-column
              label="客户名称"
              prop="company"
              width="500">
            </el-table-column>
            <el-table-column
              label="贷款余额"
              prop="balance">
            </el-table-column>
          </el-table>
        </el-col>
      </el-row>
    </div>
  </div>
</template>

<script>
import UploadExcelComponent from '@/components/UploadExcel/index.vue'
import UploadExcelComponent2 from '@/components/UploadExcel/index2.vue'

export default {
  name: 'UploadExcel',
  components: {UploadExcelComponent, UploadExcelComponent2},
  data() {
    return {
      r1Array: [],
      r2Array: [],
      file1Name: null,
      file2Name: null,
      // table1_2: [{'name': 11, 'company': 11, 'balance': 11}, {'name': 11, 'company': 11, 'balance': 11}, {'name': 11, 'company': 11, 'balance': 11}],
      table1_2: null,
      table2_1: null
    }
  },
  methods: {
    beforeUpload(file) {
      const isLt1M = file.size / 1024 / 1024 < 10

      if (isLt1M) {
        return true
      }
      this.$message.warning('文件大小不能超过10M')
      return false
    },
    handleSuccess1(excelData) {
      this.r1Array = excelData.resultsArray;
      if (excelData.resultsArray.length === 0) {
        this.$message.error('文件选择有误，请选择正确的 手工台账excel！')
        return
      }
      this.file1Name = excelData.fileName
      this.$message.success('手工台账excel 上传成功！')
    },
    handleSuccess2(excelData) {
      this.r2Array = excelData.resultsArray
      if (excelData.resultsArray.length === 0) {
        this.$message.error('文件选择有误，请选择正确的 贷款导出excel！')
        return
      }
      this.file2Name = excelData.fileName
      this.$message.success('贷款导出excel 上传成功！')
    },
    handleCompare() {
      if (!this.file1Name || !this.r1Array) {
        this.$message.error('请上传 手工台账excel！')
        return;
      }
      if (!this.file2Name || !this.r2Array) {
        this.$message.error('请上传 贷款导出excel！')
        return;
      }
      let r1 = this.r1Array
      let r2 = this.r2Array
      // console.dir(r1)
      // console.dir(r2)

      let t1_2 = [];
      r1.filter(function (v) {
        let split = v.split('-');
        if (r2.indexOf(v) === -1) {
          t1_2.push({'name': split[0], 'company': split[1], 'balance': split[2]})
        }
      })
      t1_2.sort((a, b) => a.name.localeCompare(b.name))
      this.table1_2 = t1_2

      let t2_1 = [];
      r2.filter(function (v) {
        let split = v.split('-');
        if (r1.indexOf(v) === -1) {
          t2_1.push({'name': split[0], 'company': split[1], 'balance': split[2]})
        }
      })
      t2_1.sort((a, b) => a.name.localeCompare(b.name))
      this.table2_1 = t2_1
    },
    balanceFormatter(row, column, cellValue) {
      cellValue += '';
      if (!cellValue.includes('.')) cellValue += '.';
      return cellValue.replace(/(\d)(?=(\d{3})+\.)/g, function ($0, $1) {
        return $1 + ',';
      }).replace(/\.$/, '');
    }
  }
}
</script>
<style>
.drop {
  border: 2px dashed #bbb;
  width: 450px;
  height: 100px;
  line-height: 100px;
  margin: -120px 0 0 70%;
  font-size: 24px;
  border-radius: 5px;
  text-align: center;
  color: #bbb;
  position: absolute;
}
</style>
