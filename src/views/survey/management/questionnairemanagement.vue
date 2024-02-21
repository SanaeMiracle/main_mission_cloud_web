<template>
  <div class="content">
    <el-card class="box-card">
      <div class="hear">
        <div class="inputDiv">
          <span>调研员名称：</span><el-input v-model="inquiryName" placeholder="请输入内容"></el-input>
        </div>
        <div>
          <el-button type="primary" @click="inquiry">查询</el-button>
          <el-button type="primary" @click="reset">重置</el-button>
        </div>
      </div>
      <div class="add">
        <el-button type="primary" @click="add">新增</el-button>
      </div>
      <div class="table">
        <el-table :data="tableData" border style="width: 80%">
          <el-table-column align="center" type="index" label="序号" width="50" :index="indexAdd"> </el-table-column>
          <el-table-column align="center" prop="investigatorName" label="调研员名称" width=""> </el-table-column>
          <el-table-column align="center" prop="investigatorPhon" label="调研员手机号" width=""> </el-table-column>
          <el-table-column align="center" label="操作" width="120">
            <template slot-scope="scope">
              <el-button type="primary" @click="edit(scope.row)">编辑</el-button>
            </template>
          </el-table-column>
        </el-table>
        <div class="pagination">
          <el-pagination
            background
            layout="prev, pager, next"
            :total="total"
            :page-size="pageSize"
            @current-change="handleCurrentChange"
          >
          </el-pagination>
        </div>
      </div>
    </el-card>

    <el-dialog title="编辑" :visible.sync="editVisible" width="30%">
      <!-- <div class="dialogInput">
        <div class="inputDiv">
          <span>调研员名称：</span><el-input v-model="personName" placeholder="请输入督导员名称"></el-input>
        </div>
        <div class="inputDiv">
          <span>调研员电话：</span><el-input v-model="personPhone" placeholder="请输入督导员电话"></el-input>
        </div>
      </div> -->
      <el-form :model="form" ref="formData" :rules="rules">
        <el-form-item label="调研员名称：" prop="personName">
          <el-input v-model="form.personName" autocomplete="off" required="required"></el-input>
        </el-form-item>
        <el-form-item label="调研员电话：" prop="personPhone">
          <el-input v-model="form.personPhone" autocomplete="off" required="required"></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="editVisible = false">取 消</el-button>
        <el-button type="primary" @click="editClick">确 定</el-button>
      </span>
    </el-dialog>
    <el-dialog title="新增" :visible.sync="addVisible" width="30%">
      <el-form :model="form" ref="formData" :rules="rules">
        <el-form-item label="调研员名称：" prop="personName">
          <el-input v-model="form.personName" autocomplete="off" required="required"></el-input>
        </el-form-item>
        <el-form-item label="调研员电话：" prop="personPhone">
          <el-input v-model="form.personPhone" autocomplete="off" required="required"></el-input>
        </el-form-item>
      </el-form>
      <!-- <div class="dialogInput">
        <div class="inputDiv">
          <span>调研员名称：</span><el-input v-model="personName" placeholder="请输入督导员名称"></el-input>
        </div>
        <div class="inputDiv">
          <span>调研员电话：</span><el-input v-model="personPhone" placeholder="请输入督导员电话"></el-input>
        </div>
      </div> -->
      <span slot="footer" class="dialog-footer">
        <el-button @click="addVisible = false">取 消</el-button>
        <el-button type="primary" @click="addClick">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>
<script>
import { httpAction, getAction } from '@/api/manage'
import { Result } from 'ant-design-vue'
export default {
  name: 'questionnairemanagement',
  //   mixins: [JeecgListMixin],
  data() {
    return {
      form: {
        personPhone: '',
        personName: '',
      },
      rules: {
        personPhone: [{ required: true, message: '手机号码不能为空' }],
        personName: [{ required: true, message: '调研员名称不能为空' }],
      },
      pageNo: 1,
      pageSize: 10,
      total: 10,
      editVisible: false,
      addVisible: false,
      inquiryName: '',
      personName: '',
      personId: '',
      personPhone: '',

      tableData: [],
    }
  },
  mounted() {
    this.reset()
  },
  methods: {
    inquiry() {
      this.searchPerson(this.inquiryName)
    },
    reset() {
      this.searchPerson()
      this.pageNo = 1
      this.inquiryName = ''
      this.personName = ''
      this.personPhone = ''
      this.form = {
        personPhone: '',
        personName: '',
      }
    },
    edit(e) {
      this.editVisible = true
      console.log(e)
      this.personId = e.id
      this.form.personName = e.investigatorName
      this.form.personPhone = e.investigatorPhon
    },
    editClick(e) {
      this.$refs.formData.validate((valid) => {
        if (valid) {
          this.editVisible = false
          console.log(e)
          const bkbInvestigator = {
            id: this.personId,
            investigatorName: this.form.personName,
            investigatorPhon: this.form.personPhone,
          }
          console.log('bkbInvestigator', bkbInvestigator)
          httpAction('/mission/bkbInvestigator/edit', bkbInvestigator, 'post')
            .then((res) => {
              console.log(res)
              if (res.success) {
                // this.$message.success(res.message)
                // this.$emit('ok')
                this.$message.success('编辑成功')
                this.reset()
              } else {
                this.$message.warning(res.message)
              }
            })
            .finally(() => {
              // that.confirmLoading = false;
            })
        }
      })
    },
    add() {
      this.addVisible = true
      this.form.personName = ''
      this.form.personPhone = ''
    },
    addClick() {
      this.$refs.formData.validate((valid) => {
        if (valid) {
          this.addVisible = false
          const bkbInvestigator = {
            investigatorName: this.form.personName,
            investigatorPhon: this.form.personPhone,
          }
          httpAction('/mission/bkbInvestigator/add', bkbInvestigator, 'post')
            .then((res) => {
              console.log(res)
              if (res.success) {
                this.$message.success('新增成功')
                this.reset()
              } else {
                this.$message.warning(res.message)
              }
            })
            .finally(() => {
              // that.confirmLoading = false;
            })
        }
      })
    },

    indexAdd(index) {
      // console.log(index)
      const page = this.pageNo // 当前页码
      const pagesize = 10 // 每页条数
      return index + 1 + (page - 1) * pagesize
    },
    handleCurrentChange(val) {
      this.pageNo = val
      this.searchPerson(this.inquiryName)
    },
    searchPerson(value) {
      let url
      if (value) {
        url = `/mission/bkbInvestigator/list?investigatorName=${value}&pageNo=${this.pageNo}`
      } else {
        url = `/mission/bkbInvestigator/list?pageNo=${this.pageNo}`
      }
      httpAction(url, {}, 'get')
        .then((res) => {
          console.log(res)
          if (res.success) {
            // this.$message.success(res.message)
            // this.$emit('ok')
            this.total = res.result.total
            this.tableData = res.result.records
          } else {
            this.$message.warning(res.message)
          }
        })
        .finally(() => {
          // that.confirmLoading = false;
        })
    },
  },
}
</script>
<style lang="less" scoped>
.hear {
  display: flex;
  justify-content: left;
  width: 1400px;
}

.dialogInput {
  .inputDiv {
    margin-top: 20px;
  }
}
.inputDiv {
  display: flex;
  justify-content: left;
  align-items: center;
  margin-right: 50px;
  span {
    width: 150px;
  }
}

.add {
  margin: 20px 0 20px 1230px;
}

.table {
  .pagination {
    margin: 20px 320px 0 0;
    display: flex;
    justify-content: right;
    // margin-right: 300px;
  }
}
</style>