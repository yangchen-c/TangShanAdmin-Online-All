<template>
  <div id="app">
    <div class="btn">
      <el-select v-model="valueState" clearable placeholder="选择状态" @change="changeState($event)">
        <el-option
          v-for="item in optionsState"
          :key="item.value"
          :label="item.label"
          :value="item.value"
        />
      </el-select>
    </div>
    <div class="tablee">
      <el-table :data="tableData" border style="width: 100%">
        <el-table-column align="center" prop="id" label="ID" width="50" />
        <el-table-column align="center" prop="name" label="操作的对象" />
        <el-table-column align="center" prop="remark" label="操作的数据" />
        <el-table-column align="center" prop="createTime" label="操作时间" />
        <el-table-column align="center" prop="state" label="操作状态">
          <template slot-scope="scope">
            <span v-if="scope.row.state == '0'" style="color:#67C23A">增加</span>
            <span v-if="scope.row.state == '1'" style="color:#4B0082">删除</span>
            <span v-if="scope.row.state == '2'" style="color:#f78989">修改</span>
          </template>
        </el-table-column>
        <el-table-column align="center" prop="update" label="更新的数据" />
        <el-table-column align="center" prop="staff.name" label="操作人" />
      </el-table>
    </div>
  </div>
</template>

<script>
import { logList } from '@/api/api'
import { getToken } from '@/utils/auth'

export default {
  name: 'Log',
  data() {
    return {
      // 表格数据
      valueState: '',
      tableData: [],
      tableDataAll: [],
      value: '',
      optionsState: [
        {
          value: 'state1',
          label: '增加'
        },
        {
          value: 'state2',
          label: '删除'
        },
        {
          value: 'state3',
          label: '修改'
        }
      ]
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
  },

  methods: {
    // 状态框筛选
    changeState(val) {
    //   console.log(val)
      if (val === 'state1') {
        this.tableData = this.tableDataAll.filter((el) => el.state === 0)
      } else if (val === 'state2') {
        this.tableData = this.tableDataAll.filter((el) => el.state === 1)
      } else if (val === 'state3') {
        this.tableData = this.tableDataAll.filter((el) => el.state === 2)
      } else if (val === '') {
        this.tableData = this.tableDataAll
      }
    },
    // 获取数据
    getList() {
      logList({ name: this.name })
        .then((response) => {
          this.tableData = response.data.data
          this.tableDataAll = response.data.data
        })
        .catch(() => {
          this.tableData = []
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
}
</style>
