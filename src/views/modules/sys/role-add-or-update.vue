<template>
  <el-dialog
    :title="!dataForm.id ? '新增' : '修改'"
    :close-on-click-modal="false"
    :visible.sync="visible">
    <el-form :model="dataForm" :rules="dataRule" ref="dataForm" @keyup.enter.native="dataFormSubmit()" label-width="80px">
      <el-form-item label="角色名称" prop="roleName">
        <el-input v-model="dataForm.roleName" placeholder="角色名称"></el-input>
      </el-form-item>
      <el-form-item label="部门" prop="deptName">
        <el-popover
          ref="deptListPopover"
          placement="bottom-start"
          trigger="click">
          <el-tree
            :data="deptList"
            :props="deptListTreeProps"
            node-key="deptId"
            ref="selectDeptListTree"
            @current-change="deptListTreeCurrentChangeHandle"
            :default-expand-all="true"
            :highlight-current="true"
            :expand-on-click-node="false">
          </el-tree>
        </el-popover>
        <el-input v-model="dataForm.deptName" v-popover:deptListPopover :readonly="true" placeholder="点击选择上级部门" class="dept-list__input"></el-input>
      </el-form-item>
      <el-form-item label="备注" prop="remark">
        <el-input v-model="dataForm.remark" placeholder="备注"></el-input>
      </el-form-item>
      <el-row>
        <el-col :span="12">
          <div class="grid-content bg-purple">
            <el-tag>功能权限</el-tag>
            <el-tree
              :data="menuList"
              :props="menuListTreeProps"
              node-key="menuId"
              ref="menuListTree"
              :default-expand-all="true"
              show-checkbox>
            </el-tree>
          </div>
        </el-col>
        <el-col :span="12">
          <div class="grid-content bg-purple-light">
            <el-tag>数据权限</el-tag>
            <el-tree
              :data="deptList"
              :props="deptListTreeProps"
              node-key="deptId"
              ref="deptListTree"
              :default-expand-all="true"
              :check-strictly="true"
              show-checkbox>
            </el-tree>
          </div>
        </el-col>
      </el-row>
    </el-form>
    <span slot="footer" class="dialog-footer">
      <el-button @click="visible = false">取消</el-button>
      <el-button type="primary" @click="dataFormSubmit()">确定</el-button>
    </span>
  </el-dialog>
</template>

<script>
  import { treeDataTranslate } from '@/utils'
  export default {
    data () {
      return {
        visible: false,
        menuList: [],
        deptList: [],
        menuListTreeProps: {
          label: 'name',
          children: 'children'
        },
        deptListTreeProps: {
          label: 'name',
          children: 'children'
        },
        dataForm: {
          id: 0,
          roleName: '',
          remark: '',
          deptId: 0,
          deptName: ''
        },
        dataRule: {
          roleName: [
            { required: true, message: '角色名称不能为空', trigger: 'blur' }
          ],
          deptName: [
            { required: true, message: '部门不能为空', trigger: 'change' }
          ],
          remark: [
            { required: true, message: '备注不能为空', trigger: 'blur' }
          ]
        }
      }
    },
    methods: {
      init (id) {
        this.dataForm.id = id || 0
        this.$http({
          url: this.$http.adornUrl('/sys/menu/perms'),
          method: 'get',
          params: this.$http.adornParams()
        }).then(({data}) => {
          this.menuList = treeDataTranslate(data, 'menuId')
        }).then(() => {
          this.visible = true
          this.$nextTick(() => {
            this.$refs['dataForm'].resetFields()
            this.$refs.menuListTree.setCheckedKeys([])
          })
        }).then(() => {
          this.$http({
            url: this.$http.adornUrl('/sys/dept/list'),
            method: 'get',
            params: this.$http.adornParams()
          }).then(({data}) => {
            this.deptList = treeDataTranslate(data, 'deptId')
          }).then(() => {
            this.visible = true
            this.$nextTick(() => {
              this.$refs['dataForm'].resetFields()
              this.$refs.deptListTree.setCheckedKeys([])
            })
          }).then(() => {
            if (this.dataForm.id) {
              this.$http({
                url: this.$http.adornUrl(`/sys/role/info/${this.dataForm.id}`),
                method: 'get',
                params: this.$http.adornParams()
              }).then(({data}) => {
                if (data && data.code === 0) {
                  this.dataForm.roleName = data.role.roleName
                  this.dataForm.deptId = data.role.deptId
                  this.dataForm.remark = data.role.remark
                  this.deptListTreeSetCurrentNode()
                  this.$refs.menuListTree.setCheckedKeys(data.role.menuIdList)
                  this.$refs.deptListTree.setCheckedKeys(data.role.deptIdList)
                }
              })
            }
          })
        })
      },
      // 部门树选中
      deptListTreeCurrentChangeHandle (data, node) {
        this.dataForm.deptId = data.deptId
        this.dataForm.deptName = data.name
      },
      // 部门树设置当前选中节点
      deptListTreeSetCurrentNode () {
        this.$refs.selectDeptListTree.setCurrentKey(this.dataForm.deptId)
        this.dataForm.deptName = (this.$refs.selectDeptListTree.getCurrentNode() || {})['name']
      },
      // 表单提交
      dataFormSubmit () {
        this.$refs['dataForm'].validate((valid) => {
          if (valid) {
            this.$http({
              url: this.$http.adornUrl(`/sys/role/${!this.dataForm.id ? 'save' : 'update'}`),
              method: 'post',
              data: this.$http.adornData({
                'roleId': this.dataForm.id || undefined,
                'roleName': this.dataForm.roleName,
                'deptId': this.dataForm.deptId,
                'remark': this.dataForm.remark,
                'menuIdList': [].concat(this.$refs.menuListTree.getCheckedKeys(), this.$refs.menuListTree.getHalfCheckedKeys()),
                'deptIdList': [].concat(this.$refs.deptListTree.getCheckedKeys(), this.$refs.deptListTree.getHalfCheckedKeys())
              })
            }).then(({data}) => {
              if (data && data.code === 0) {
                this.$message({
                  message: '操作成功',
                  type: 'success',
                  duration: 1500,
                  onClose: () => {
                    this.visible = false
                    this.$emit('refreshDataList')
                  }
                })
              } else {
                this.$message.error(data.msg)
              }
            })
          }
        })
      }
    }
  }
</script>
