<template>
  <div class="app-container">
    <!--工具栏-->
    <div class="head-container">
      <div v-if="crud.props.searchToggle">
        <!-- 搜索 -->
        <el-input v-model="query.blurry" clearable placeholder="模糊搜索" style="width: 200px" class="filter-item" @keyup.enter.native="crud.toQuery" />
        <date-range-picker v-model="query.createTime" class="date-item" />
        <rrOperation />
      </div>
      <crudOperation :permission="permission">
        <el-button
          slot="right"
          v-permission="['admin','database:add']"
          :disabled="!selectIndex"
          class="filter-item"
          size="mini"
          type="warning"
          icon="el-icon-upload"
          @click="execute"
        >执行脚本
        </el-button>
      </crudOperation>
    </div>
    <!--表单组件-->
    <eForm ref="execute" :database-info="currentRow" />
    <el-dialog append-to-body :close-on-click-modal="false" :before-close="crud.cancelCU" :visible.sync="crud.status.cu > 0" :title="crud.status.title" width="530px">
      <el-form ref="form" :model="form" :rules="rules" size="small" label-width="100px">
        <el-form-item label="连接名称" prop="name">
          <el-input v-model="form.name" style="width: 370px" />
        </el-form-item>
        <el-form-item label="JDBC地址" prop="jdbcUrl">
          <el-input v-model="form.jdbcUrl" style="width: 300px" />
          <el-button :loading="loading" type="success" @click="testConnectDatabase">测试</el-button>
        </el-form-item>
        <el-form-item label="用户" prop="userName">
          <el-input v-model="form.userName" style="width: 370px" />
        </el-form-item>
        <el-form-item label="密码" prop="pwd">
          <el-input v-model="form.pwd" type="password" style="width: 370px" />
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button type="text" @click="crud.cancelCU">取消</el-button>
        <el-button :loading="crud.status.cu === 2" type="primary" @click="crud.submitCU">确认</el-button>
      </div>
    </el-dialog>
    <!--表格渲染-->
    <el-table ref="table" v-loading="crud.loading" :data="crud.data" highlight-current-row stripe style="width: 100%" @selection-change="handleCurrentChange">
      <el-table-column type="selection" width="55" />
      <el-table-column prop="name" width="130px" label="数据库名称" />
      <el-table-column prop="jdbcUrl" label="连接地址" />
      <el-table-column prop="userName" width="200px" label="用户名" />
      <el-table-column prop="createTime" width="200px" label="创建日期" />
      <el-table-column v-if="checkPer(['admin','database:edit','database:del'])" label="操作" width="150px" align="center">
        <template slot-scope="scope">
          <udOperation
            :data="scope.row"
            :permission="permission"
          />
        </template>
      </el-table-column>
    </el-table>
    <!--分页组件-->
    <pagination />
  </div>
</template>

<script>
import crudDatabase from '@/api/maint/database'
import { testDbConnect } from '@/api/maint/connect'
import eForm from './execute'
import CRUD, { presenter, header, form, crud } from '@crud/crud'
import rrOperation from '@crud/RR.operation'
import crudOperation from '@crud/CRUD.operation'
import udOperation from '@crud/UD.operation'
import pagination from '@crud/Pagination'
import DateRangePicker from '@/components/DateRangePicker'

const defaultForm = { id: null, name: null, jdbcUrl: 'jdbc:mysql://', userName: null, pwd: null }
export default {
  name: 'DataBase',
  components: { eForm, pagination, crudOperation, rrOperation, udOperation, DateRangePicker },
  cruds() {
    return CRUD({ title: '数据库', url: 'api/database', crudMethod: { ...crudDatabase }})
  },
  mixins: [presenter(), header(), form(defaultForm), crud()],
  data() {
    return {
      currentRow: {},
      selectIndex: '',
      databaseInfo: '',
      loading: false,
      permission: {
        add: ['admin', 'database:add'],
        edit: ['admin', 'database:edit'],
        del: ['admin', 'database:del']
      },
      rules: {
        name: [
          { required: true, message: '请输入数据库名称', trigger: 'blur' }
        ],
        jdbcUrl: [
          { required: true, message: '请输入数据库连接地址', trigger: 'blur' }
        ],
        userName: [
          { required: true, message: '请输入用户名', trigger: 'blur' }
        ],
        pwd: [
          { required: true, message: '请输入数据库密码', trigger: 'blur' }
        ]
      }
    }
  },
  methods: {
    testConnectDatabase() {
      this.$refs['form'].validate((valid) => {
        if (valid) {
          this.loading = true
          testDbConnect(this.form).then((res) => {
            this.loading = false
            this.crud.notify(res ? '连接成功' : '连接失败', res ? 'success' : 'error')
          }).catch(() => {
            this.loading = false
          })
        }
      })
    },
    execute() {
      this.$refs.execute.dialog = true
    },
    handleCurrentChange(selection) {
      this.crud.selections = selection
      if (selection.length === 1) {
        const row = selection[0]
        this.selectIndex = row.id
        this.currentRow = row
      } else {
        this.currentRow = {}
        this.selectIndex = ''
      }
    }
  }
}
</script>

<style scoped>
</style>
