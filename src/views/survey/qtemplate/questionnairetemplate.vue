<template>
  <div class="content">
    <el-card class="box-card">
      <div class="hear">
        <div class="inputDiv">
          <span>模板名称：</span><el-input v-model="templateName" placeholder="请输入内容"></el-input>
        </div>
        <div>
          <el-button type="primary" @click="inquiry">查询</el-button>
          <el-button type="primary" @click="reset">重置</el-button>
        </div>
      </div>
      <div class="add">
        <el-button type="primary" @click="add">模板创建</el-button>
        <el-button type="primary" @click="batchDel">批量删除</el-button>
      </div>
      <div class="table">
        <el-table
          :data="tableData"
          border
          style="width: 80%"
          @selection-change="handleSelectionChange"
          :row-key="
            (row) => {
              return row.id
            }
          "
        >
          <el-table-column align="center" type="selection" :reserve-selection="true" width="55"> </el-table-column>
          <el-table-column align="center" type="index" label="序号" width="50" :index="indexAdd"> </el-table-column>
          <el-table-column align="center" prop="title" label="问卷标题" width=""> </el-table-column>
          <el-table-column align="center" prop="titleUrl" label="封面图" width="">
            <template slot-scope="scope">
              <div class="fengmiantu">
                <!-- {{ scope.row }} -->
                <img :src="scope.row.titleUrl" alt="" srcset="" />
              </div>
            </template>
          </el-table-column>
          <el-table-column align="center" prop="createTime" label="创建时间" width=""> </el-table-column>
          <el-table-column align="center" label="操作" width="350">
            <template slot-scope="scope">
              <el-button type="primary" @click="look(scope.row)">查看</el-button>
              <el-button type="primary" @click="edit(scope.row)">编辑</el-button>
              <el-button type="primary" @click="del(scope.row)">删除</el-button>
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

    <el-dialog title="编辑" :visible.sync="editVisible" width="60%" :destroy-on-close="true">
      <div class="dialogInput">
        <tem @closeDiaLog="closeDiaLog" @refreshTable="reset" :temData="temData" :questionGroups="questionGroups"></tem>
      </div>
      <!-- <span slot="footer" class="dialog-footer">
        <el-button @click="editVisible = false">取 消</el-button>
        <el-button type="primary" @click="editClick">确 定</el-button>
      </span> -->
    </el-dialog>
    <el-dialog title="新增" :visible.sync="addVisible" width="60%" :destroy-on-close="true">
      <div class="dialogInput">
        <tem @closeDiaLog="closeDiaLog" @refreshTable="reset"></tem>
      </div>
      <!-- <span slot="footer" class="dialog-footer">
        <el-button @click="addVisible = false">取 消</el-button>
        <el-button type="primary" @click="addClick">确 定</el-button>
      </span> -->
    </el-dialog>
    <el-dialog title="问卷模板查看" :visible.sync="lookVisible" width="60%">
      <div class="dialogInput">
        <temLook :temData="temData" :questionGroups="questionGroups"></temLook>
      </div>
      <span slot="footer" class="dialog-footer">
        <el-button @click="lookVisible = false">取 消</el-button>
        <el-button type="primary" @click="lookVisible = false">确 定</el-button>
      </span>
    </el-dialog>
  </div>
</template>
<script>
import { httpAction, getAction } from '@/api/manage'
import tem from '@/views/survey/components/tem'
import temLook from '@/views/survey/components/temLook'
export default {
  name: 'questionnairetemplate',
  //   mixins: [JeecgListMixin],
  components: {
    tem,
    temLook,
  },
  data() {
    return {
      temData: {},
      pageNo: 1,
      pageSize: 10,
      total: 10,
      editVisible: false,
      addVisible: false,
      lookVisible: false,
      templateName: '',
      personName: '',
      personPhone: '',
      tableData: [],
      ids: [],
      questionGroups: [],
    }
  },
  mounted() {
    this.reset()
  },
  methods: {
    //批量删除
    handleSelectionChange(val) {
      this.ids = []
      val.map((item) => {
        this.ids.push(item.id)
      })
      console.log(this.ids, 'listresultTmp')
    },
    closeDiaLog(value) {
      this.editVisible = value
      this.addVisible = value
    },
    //     refreshTable(){
    // this.reset
    //     },
    inquiry() {
      this.searchTem(this.templateName)
    },
    reset() {
      this.searchTem()
      this.templateName = ''
      this.personName = ''
      this.personPhone = ''
    },
    edit(e) {
      this.editVisible = true
      this.queryQuestionGroups(e.id)
      console.log(e, '3333333')
      this.temData = e
    },
    batchDel() {
      if (this.ids.length == 0) {
        // console.log(this.ids, '111')
        this.$message({
          type: 'warning',
          message: '请先勾选数据在删除!',
        })
      } else {
        // console.log(this.ids, '222')
        this.$confirm('确认是否要删除此问卷模板?', '删除', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning',
        })
          .then(() => {
            httpAction(`/mission/bkbQuestionnaireInquiryTemplate/deleteBatch?ids=${this.ids}`, {}, 'DELETE').then(
              (res) => {
                // httpAction(`/mission/bkbQuestionnaireInquiry/deleteBatch?ids=${this.ids}`, {}, 'DELETE').then((res) => {
                if (res.code == 200) {
                  this.reset()
                  this.$message({
                    type: 'success',
                    message: '删除成功!',
                  })
                }
              }
            )
          })

          .catch(() => {
            this.$message({
              type: 'info',
              message: '已取消删除',
            })
          })
      }

      // this.$confirm('确认是否要删除这些问卷模板?', '提示', {
      //   confirmButtonText: '确定',
      //   cancelButtonText: '取消',
      //   type: 'warning',
      // })
      //   .then(() => {
      //     this.$message({
      //       type: 'success',
      //       message: '删除成功!',
      //     })
      //   })
      //   .catch(() => {
      //     this.$message({
      //       type: 'info',
      //       message: '已取消删除',
      //     })
      //   })
    },
    editClick(e) {
      this.editVisible = false
    },
    del(e) {
      this.$confirm('确认是否要删除此问卷模板?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning',
      }).then(() => {
        this.temDel(e.id)
      })
    },
    look(e) {
      this.queryQuestionGroups(e.id)
      console.log(e, '3333333')
      this.temData = e
      this.lookVisible = true
      console.log('!!!!!!!', this.temData, this.questionGroups)
    },
    add() {
      this.addVisible = true
    },
    addClick() {
      this.addVisible = false
    },

    indexAdd(index) {
      // console.log(index)
      const page = this.pageNo // 当前页码
      const pagesize = 10 // 每页条数
      return index + 1 + (page - 1) * pagesize
    },
    handleCurrentChange(val) {
      this.pageNo = val
      this.searchTem(this.inquiryName)
    },
    searchTem(title) {
      let url
      if (title) {
        url = `/mission/bkbQuestionnaireInquiryTemplate/list?title=${title}&pageNo=${this.pageNo}`
        // url = `/mission/bkbQuestionnaireInquiry/list?title=${title}&pageNo=${this.pageNo}`
      } else {
        url = `/mission/bkbQuestionnaireInquiryTemplate/list?pageNo=${this.pageNo}`
        // url = `/mission/bkbQuestionnaireInquiry/list?pageNo=${this.pageNo}`
      }
      httpAction(url, {}, 'get').then((res) => {
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
    },
    searchTemById(id) {
      httpAction(`/mission/bkbQuestionnaireInquiry/queryById?id=${id}`, {}, 'get').then((res) => {
        if (res.success) {
        } else {
          this.$message.warning(res.message)
        }
      })
    },
    temDel(id) {
      httpAction(`/mission/bkbQuestionnaireInquiryTemplate/delete?id=${id}`, {}, 'delete').then((res) => {
        // httpAction(`/mission/bkbQuestionnaireInquiry/delete?id=${id}`, {}, 'delete').then((res) => {
        if (res.success) {
          this.$message.success(res.message)
          this.reset()
        } else {
          this.$message.warning(res.message)
        }
      })
    },
    temBatchDel(id) {
      // httpAction(`/mission/bkbQuestionnaireInquiry/deleteBatch?id=${id}`, {}, 'delete').then((res) => {
      //   if (res.success) {
      //     this.$message.success(res.message)
      //     this.reset()
      //   } else {
      //     this.$message.warning(res.message)
      //   }
      // })
    },
    // 查询题目
    queryQuestionGroups(id) {
      // id = "1656222258321121281"
      httpAction(`/mission/bkbQuestionnaireQuestionsTemplate/newqueryQuestionGroups?id=${id}`, {}, 'get').then(
        (res) => {
          // httpAction(`/mission/bkbQuestionnaireQuestions/queryQuestionGroups?id=${id}`, {}, 'get').then((res) => {
          if (res.success) {
            console.log(res.result, 'res.result')
            this.questionGroups = res.result
          } else {
            this.$message.warning(res.message)
          }
        }
      )
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
    width: 100px;
  }
}

.add {
  margin: 20px 0 20px 1095px;
}

.table {
  .pagination {
    margin: 20px 320px 0 0;
    display: flex;
    justify-content: right;
    // margin-right: 300px;
  }
}

.fengmiantu {
  height: 80px;
  img {
    height: 80px;
  }
}
</style>