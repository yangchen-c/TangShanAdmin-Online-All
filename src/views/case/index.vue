<template>
  <div id="app">
    <div class="btn">
      <el-button type="primary" @click="addShop">新建</el-button>
      <el-select v-model="value" clearable placeholder="所属门店" @change="changeShop($event)">
        <el-option v-for="item in options" :key="item.id" :label="item.name" :value="item.id" />
      </el-select>
      <!-- <el-date-picker v-model="value1" type="date" placeholder="选择日期" /> -->
      <!-- <el-select v-model="value2" clearable placeholder="选择状态">
        <el-option
          v-for="item in options2"
          :key="item.value"
          :label="item.label"
          :value="item.value"
        />
      </el-select>-->
      <!-- <el-button type="primary">查询</el-button>
      <el-input
        v-model="name"
        placeholder="请输入门店名称"
        clearable
        style="width:180px;margin-left:50px"
      />
      <el-button type="primary" @click="getList()">搜索</el-button>-->
    </div>
    <div class="tablee">
      <el-table :data="tableData" border style="width: 100%">
        <el-table-column align="center" prop="id" label="ID" width="50" />
        <el-table-column align="center" prop="name" label="案例名称" width="100" />
        <el-table-column align="center" prop="createTime" label="时间" width="70" />
        <el-table-column align="center" prop="combo.store.name" label="所属门店" />
        <el-table-column align="center" prop="combo.classify.name" label="所属分类" />
        <el-table-column align="center" prop="combo.name" label="所属套餐" />
        <el-table-column align="center" prop="cover" label="封面图" width="200">
          <template slot-scope="scope">
            <img :src="scope.row.cover" alt style="width:50px;height:50px">
          </template>
        </el-table-column>
        <el-table-column align="center" prop="infoPhoto" label="套餐轮播图" width="400">
          <template slot-scope="scope">
            <img
              v-for="item in scope.row.infoPhoto.split(',')"
              :key="item.id"
              :src="item"
              alt
              style="width:50px;height:50px"
            >
          </template>
        </el-table-column>
        <el-table-column align="center" label="操作">
          <template slot-scope="scope">
            <el-button
              size="mini"
              icon="el-icon-document-copy"
              type="primary"
              @click="getEditData(scope.row)"
            >编辑</el-button>
            <el-button size="mini" icon="el-icon-user" type="danger" @click="delData(scope.row)">删除</el-button>
          </template>
        </el-table-column>
      </el-table>
    </div>
    <!-- dialog弹出框 -->
    <el-dialog :title="title1" :visible.sync="dialogFormVisible">
      <el-form :model="form">
        <el-form-item label="案例名称" :label-width="formLabelWidth">
          <el-input v-model="form.name" placeholder="请输入套餐名称" style="width:400px" />
        </el-form-item>
        <el-form-item label="所属门店" :label-width="formLabelWidth">
          <el-select v-model="form.combo.store.id" clearable placeholder="选择门店">
            <el-option v-for="item in options" :key="item.id" :label="item.name" :value="item.id" />
          </el-select>
        </el-form-item>
        <el-form-item label="所属分类" :label-width="formLabelWidth">
          <el-select v-model="form.combo.classify.id" clearable placeholder="选择分类">
            <el-option
              v-for="item in optionsCombo"
              :key="item.id"
              :label="item.name"
              :value="item.id"
            />
          </el-select>
        </el-form-item>
        <el-form-item label="所属套餐" :label-width="formLabelWidth">
          <el-select v-model="form.combo.id" clearable placeholder="选择分类">
            <el-option
              v-for="item in options3"
              :key="item.id"
              :label="item.name"
              :value="item.id"
            />
          </el-select>
        </el-form-item>
        <el-form-item label="封面图片" :label-width="formLabelWidth">
          <el-upload
            :headers="headers"
            :action="uploadPath"
            :show-file-list="false"
            :on-success="uploadUrl"
            :before-upload="checkFileSize"
            class="avatar-uploader"
            accept=".jpg, .jpeg, .png"
          >
            <img v-if="form.cover" :src="form.cover" class="avatar">
            <i v-else class="el-icon-plus avatar-uploader-icon" />
            <div slot="tip" class="el-upload__tip">只能上传jpg/png文件，且不超过1024kb</div>
          </el-upload>
        </el-form-item>
        <!-- <el-form-item label="轮播图" :label-width="formLabelWidth">
          <el-upload
            :headers="headers"
            :action="uploadPath"
            :show-file-list="false"
            :on-success="uploadUrl"
            :before-upload="checkFileSize"
            class="avatar-uploader"
            accept=".jpg, .jpeg, .png"
          >
            <img v-if="form.banner" :src="form.banner" class="avatar">
            <i v-else class="el-icon-plus avatar-uploader-icon" />
            <div slot="tip" class="el-upload__tip">只能上传jpg/png文件，且不超过1024kb</div>
          </el-upload>
        </el-form-item>-->
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogFormVisible = false">取 消</el-button>
        <el-button type="primary" :loading="btnLoading" @click="addSubmit">确 定</el-button>
      </div>
    </el-dialog>
  </div>
</template>

<script>
import {
  shopList, // 门店
  comboClassifyList, // 分类
  comboList, // 套餐
  caseList,
  caseAdd,
  caseUpdate,
  caseDelete,
  uploadPath
} from '@/api/api'
import { getToken } from '@/utils/auth'

export default {
  name: 'Case',
  data() {
    return {
      options: [], // 门店下拉框
      optionsCombo: [], // 分类下拉框
      options3: [], // 套餐下拉框
      uploadPath,
      value: '',
      value1: '',
      value2: '',
      value3: '',
      // uploadPath,
      btnLoading: false,
      name: '', // 门店名称
      // 表格数据
      tableData: [],
      dialogFormVisible: false,
      form: {
        id: '',
        name: '',
        combo: {
          id: '',
          name: '',
          classify: {
            id: '',
            name: '',
            paper: ''
          },
          store: {
            id: '',
            name: ''
          }
        },
        price: '',
        cover: '',
        banner: ''
      },
      formLabelWidth: '120px',
      title1: ''
    }
  },
  computed: {
    headers() {
      return {
        token: getToken()
      }
    }
  },
  created() {
    this.getList()
    this.getShopList()
    this.getComboList()
    this.getCombo3List()
  },
  methods: {
    // 文件上传
    uploadUrl: function(response) {
      console.log(response)
      this.form.cover = response.data
    },
    // uploadUrls: function(response) {
    //   console.log(response)
    //   this.form.banner = response.data
    // },
    checkFileSize: function(file) {
      if (file.size > 1048576) {
        this.$message.error(
          `${file.name}文件大于1024KB，请选择小于1024KB大小的图片`
        )
        return false
      }
      return true
    },
    // 获取门店数据
    getShopList() {
      shopList()
        .then((response) => {
          this.options = response.data.data
        })
        .catch(() => {
          this.options = []
        })
    },
    // 商店下拉框筛选
    changeShop(val) {
      if (val === '') {
        this.tableData = this.tableDataAll
      } else {
        this.tableData = this.tableDataAll.filter(
          (el) => el.combo.store.id === val
        )
      }
    },
    // 获取分类数据
    getComboList() {
      comboClassifyList()
        .then((response) => {
          this.optionsCombo = response.data.data
        })
        .catch(() => {
          this.optionsCombo = []
        })
    },
    // 获取套餐数据
    getCombo3List() {
      comboList()
        .then((response) => {
          this.options3 = response.data.data
        })
        .catch(() => {
          this.options3 = []
        })
    },
    // 获取数据
    getList() {
      caseList({ name: this.name })
        // caseList({ state: 1 })
        .then((response) => {
          // console.log(response.data.data)
          this.tableData = response.data.data
          this.tableDataAll = response.data.data
          // console.log(this.tableDataAll[2].combo)
        })
        .catch(() => {
          this.tableData = []
        })
    },
    // 新建
    addShop() {
      this.dialogFormVisible = true
      this.title1 = '新建案例'
      this.form.id = ''
      this.form.name = ''
      this.form.combo.classify.id = ''
      this.form.combo.store.id = ''
    },
    // 编辑
    getEditData(data) {
      console.log(data)
      this.dialogFormVisible = true
      this.title1 = '编辑案例'
      this.form.id = data.id
      this.form.name = data.name
      this.form.combo.classify.id = data.combo.classify.id
      this.form.combo.store.id = data.combo.store.id
    },
    // 编辑新增确定事件
    addSubmit() {
      // this.btnLoading = true
      if (this.form.id) {
        caseUpdate(this.form)
          // comboUpdate(JSON.stringify(this.form))
          .then(() => {
            this.$notify.success({
              title: '成功',
              message: '案例修改成功'
            })
            this.dialogFormVisible = false
            this.getList()
          })
          .catch((response) => {
            this.$notify.error({
              title: '失败',
              message: response.data.message
            })
          })
      } else {
        caseAdd(this.form)
          .then(() => {
            this.$notify.success({
              title: '成功',
              message: '案例添加成功'
            })
            this.dialogFormVisible = false
            this.getList()
          })
          .catch((response) => {
            this.$notify.error({
              title: '失败',
              message: response.data.message
            })
          })
      }
    },
    // 删除
    delData(row) {
      this.$confirm('此操作将永久删除该案例, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      })
        .then(() => {
          const params = {
            id: row.id
          }
          caseDelete(params)
            .then((response) => {
              this.$notify.success({
                title: '成功',
                message: '案例删除成功'
              })
              this.getList()
            })
            .catch((response) => {
              this.$notify.error({
                title: '失败',
                message: response.data.message
              })
            })
        })
        .catch(() => {
          this.$notify.error({
            title: '取消',
            message: '已取消删除'
          })
        })
    }
  }
}
</script>

<style lang="less" scoped>
@rem: 1920/100rem;
#app {
  box-sizing: border-box;
  padding-left: 30 / @rem;
  padding-top: 30 / @rem;
  .tablee {
    margin-top: 20 / @rem;
  }
  .avatar-uploader-icon {
    font-size: 28px;
    color: #8c939d;
    width: 120px;
    height: 120px;
    line-height: 120px;
    text-align: center;
  }
  .avatar {
    width: 145px;
    height: 145px;
    display: block;
  }
}
</style>
