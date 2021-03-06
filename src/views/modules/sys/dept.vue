<template>
  <div class="mod-dept">
    <el-form :inline="true" :model="dataForm">
      <el-form-item>
        <el-button @click="getDataList()">查询</el-button>
        <el-button v-if="isAuth('sys:dept:save')" type="primary" @click="addOrUpdateHandle()">新增</el-button>
      </el-form-item>
    </el-form>

    <el-table
      :data="dataList"
      v-loading.body="dataListLoading"
      element-loading-text="拼命加载中"
      row-key="deptId"
      border
      ref="table"
      highlight-current-row
      @cell-click="handleCellClick"
      :cell-style="tableCellStyle"
      style="width: 100%;">
      <el-table-column
        prop="name"
        header-align="center"
        min-width="150"
        label="部门名称" >
      </el-table-column>
      <el-table-column
        prop="parentName"
        header-align="center"
        align="center"
        width="120"
        label="上级部门">
      </el-table-column>
      <el-table-column
        prop="orderNum"
        header-align="center"
        align="center"
        label="排序号">
      </el-table-column>
      <el-table-column
        fixed="right"
        header-align="center"
        align="center"
        width="150"
        label="操作">
        <template slot-scope="scope">
          <el-button v-if="isAuth('sys:dept:update')" type="text" size="small" @click="addOrUpdateHandle(scope.row.deptId)">修改</el-button>
          <el-button v-if="isAuth('sys:dept:delete')" type="text" size="small" @click="deleteHandle(scope.row.deptId)">删除</el-button>
        </template>
      </el-table-column>
    </el-table>
    <!-- 弹窗, 新增 / 修改 -->
    <add-or-update v-if="addOrUpdateVisible" ref="addOrUpdate" @refreshDataList="getDataList"></add-or-update>
  </div>
</template>

<script>
  import AddOrUpdate from './dept-add-or-update'
  import { treeDataTranslate } from '@/utils'
  export default {
    data () {
      return {
        dataForm: {},
        dataList: [],
        dataListLoading: false,
        addOrUpdateVisible: false
      }
    },
    components: {
      AddOrUpdate
    },
    activated () {
      this.getDataList()
    },
    methods: {
      // 获取数据列表
      getDataList () {
        this.dataListLoading = true
        this.$http({
          url: this.$http.adornUrl('/sys/dept/list'),
          method: 'get',
          params: this.$http.adornParams()
        }).then(({data}) => {
          this.dataList = treeDataTranslate(data, 'deptId')
          this.dataListLoading = false
        })
      },
      // 新增 / 修改
      addOrUpdateHandle (id) {
        this.addOrUpdateVisible = true
        this.$nextTick(() => {
          this.$refs.addOrUpdate.init(id)
        })
      },
      // 删除
      deleteHandle (id) {
        this.$confirm(`确定对[id=${id}]进行[删除]操作?`, '提示', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning'
        }).then(() => {
          this.$http({
            url: this.$http.adornUrl(`/sys/dept/delete/${id}`),
            method: 'post',
            data: this.$http.adornData()
          }).then(({data}) => {
            if (data && data.code === 0) {
              this.$message({
                message: '操作成功',
                type: 'success',
                duration: 1500,
                onClose: () => {
                  this.getDataList()
                }
              })
            } else {
              this.$message.error(data.msg)
            }
          })
        }).catch(() => {})
      },
      handleCellClick (row, column) {
        if (column.property === 'name') {
          // 临时解决方案：https://github.com/ElemeFE/element/issues/15863
          this.$nextTick(() => {
            const expandIcon = this.$el.querySelector('.current-row .el-table__expand-icon')
            if (expandIcon) {
              expandIcon.click()
            }
          })
          // this.$refs.table.toggleRowExpansion(row.children, true)
        }
      },
      tableCellStyle (row) {
        if (row.column.property === 'name') {
          return 'cursor:pointer'
        }
      }
    }
  }
</script>
