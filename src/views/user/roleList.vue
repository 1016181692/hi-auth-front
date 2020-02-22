<template>
  <div class="app-container">

    <el-table
      v-loading="roleTable.listLoading"
      :key="roleTable.tableKey"
      :data="roleTable.list"
      border
      fit
      highlight-current-row
      style="width: 100%;">
      <el-table-column label="序号" width="50" align="center">
        <template slot-scope="scope">
          <span>{{ scope.$index + (roleTable.queryParams.pageNum - 1) * roleTable.queryParams.pageSize + 1 }}</span>
        </template>
      </el-table-column>
      <el-table-column label="角色" align="center">
        <template slot-scope="scope">
          <span>{{ scope.row.code }}</span>
        </template>
      </el-table-column>
      <el-table-column label="角色名">
        <template slot-scope="scope">
          <span>{{ scope.row.name }}</span>
        </template>
      </el-table-column>
      <!--<el-table-column label="创建时间" align="center" width="150">
        <template slot-scope="scope">
          <span>{{ scope.row.createTime | parseTime('{y}-{m}-{d} {h}:{i}') }}</span>
        </template>
      </el-table-column>-->
      <el-table-column v-if="checkPermission(['ROLE_ADMIN'])" label="操作" align="center" class-name="small-padding fixed-width">
        <template slot-scope="scope">
          <el-button type="primary" size="mini" @click="handleUpdate(scope.row)">{{ $t('table.edit') }}</el-button>
          <el-button v-if="scope.row.status!='deleted'" size="mini" type="danger" @click="handleModifyStatus(scope.row,'deleted')">{{ $t('table.delete') }}
          </el-button>
        </template>
      </el-table-column>
    </el-table>
  </div>
</template>

<script>
import checkPermission from '@/utils/permission' // 权限判断函数
import { fetchPv } from '@/api/article'
import { queryRole } from '@/api/role'
import waves from '@/directive/waves'
import { parseTime } from '@/utils'
import Pagination from '@/components/Pagination'

export default {
  name: 'RoleTable',
  components: { Pagination },
  directives: { waves },
  data() {
    return {
      roleTable: {
        tableKey: 0,
        list: null,
        total: 0,
        listLoading: true,
        queryParams: {
          pageNum: 1,
          pageSize: 10,
          name: undefined,
          code: undefined,
          sort: '+id'
        }
      },
      role: {
        id: undefined,
        name: '',
        code: '',
        // createTime: undefined
      },
      dialogFormVisible: false,
      dialogStatus: '',
      textMap: {
        update: 'Edit',
        create: 'Create'
      },
      dialogPvVisible: false,
      pvData: [],
      downloadLoading: false
    }
  },
  created() {
    this.getRolePage()
  },
  methods: {
    checkPermission,
    getRolePage() {
      this.roleTable.listLoading = true
      queryRole(this.roleTable.queryParams.pageNum, this.roleTable.queryParams.pageSize).then(response => {
        this.roleTable.list = response.data.data.list
        this.roleTable.total = response.data.data.total

        // Just to simulate the time of the request
        setTimeout(() => {
          this.roleTable.listLoading = false
        }, 1.5 * 1000)
      })
    },
    handleFilter() {
      this.roleTable.queryParams.pageNum = 1
      // this.getList()
    },
    resetUser() {
      this.role = {
        id: undefined,
        code: '',
        name: ''
        // createTime: undefined
      }
    },
    handleCreate() {
      this.resetUser()
      this.dialogStatus = 'create'
      this.dialogFormVisible = true
      this.$nextTick(() => {
        this.$refs['dataForm'].clearValidate()
      })
    },
    // createData() {
    //   this.$refs['dataForm'].validate((valid) => {
    //     if (valid) {
    //       const userData = Object.assign({}, this.user)
    //       saveuser(undefined, userData).then(response => {
    //         this.user.id = response.data.data
    //         this.user.createTime = new Date()
    //         this.userTable.list.unshift(this.user)
    //         this.dialogFormVisible = false
    //         this.$notify({
    //           title: '成功',
    //           message: '创建成功',
    //           type: 'success',
    //           duration: 2000
    //         })
    //       })
    //     }
    //   })
    // },
    // handleUpdate(row) {
    //   // this.user = Object.assign({}, row)
    //   this.user.id = row.id
    //   this.user.name = row.name
    //   this.user.username = row.username
    //   this.user.tel = row.tel
    //   this.user.password = row.password
    //   this.user.gender = row.gender
    //   this.dialogStatus = 'update'
    //   this.dialogFormVisible = true
    //   this.$nextTick(() => {
    //     this.$refs['dataForm'].clearValidate()
    //     getRoleByUserId(this.user.id).then(response => {
    //       var roleIds = []
    //       var roles = response.data.data
    //       if (roles && roles.length > 0) {
    //         roles.forEach(function(role, i) {
    //           roleIds.push(role.id)
    //         })
    //       }
    //       this.user.roleIds = roleIds
    //     })
    //   })
    // },
    // updateData() {
    //   this.$refs['dataForm'].validate((valid) => {
    //     if (valid) {
    //       const userData = Object.assign({}, this.user)
    //       savaUser(this.user.id, userData).then(response => {
    //         for (const v of this.userTable.list) {
    //           if (v.id === this.user.id) {
    //             const index = this.userTable.list.indexOf(v)
    //             this.userTable.list.splice(index, 1, this.user)
    //             break
    //           }
    //         }
    //         this.dialogFormVisible = false
    //         this.$notify({
    //           title: '成功',
    //           message: '更新成功',
    //           type: 'success',
    //           duration: 2000
    //         })
    //       })
    //     }
    //   })
    // },
    handleDelete(row) {
      this.$notify({
        title: '成功',
        message: '删除成功',
        type: 'success',
        duration: 2000
      })
      const index = this.userTable.list.indexOf(row)
      this.userTable.list.splice(index, 1)
    },
    handleFetchPv(pv) {
      fetchPv(pv).then(response => {
        this.pvData = response.data.pvData
        this.dialogPvVisible = true
      })
    },
    formatJson(filterVal, jsonData) {
      return jsonData.map(v => filterVal.map(j => {
        if (j === 'timestamp') {
          return parseTime(v[j])
        } else {
          return v[j]
        }
      }))
    }
  }
}
</script>

