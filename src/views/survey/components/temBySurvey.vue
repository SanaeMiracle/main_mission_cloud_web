<template>
  <div class="mod-menu-home app-container" v-if="refresh">
    <!-- <div class="header">
      <div class="image" v-if="imgSrc && !changeImg">
        <div @click="changeImg = true">
          <img :src="imgSrc" alt="" />
        </div>
      </div>
      <div class="image" v-else>
        <el-upload
          action="http://lz.api.ts.cestech.com.cn/mission_b/mission/file/imgUpload"
          list-type="picture-card"
          :limit="1"
          :on-preview="handlePictureCardPreview"
          :on-remove="handleRemove"
          :on-success="handlePictureSuccess"
        >
          <i class="el-icon-plus"></i>
        </el-upload>
        <el-dialog :visible.sync="dialogVisible">
        </el-dialog>
        <div class="text">添加主题封面</div>
      </div>
    </div> -->
    <el-form :model="ruleForm" :rules="rules" ref="ruleForm" label-width="100px" class="demo-ruleForm">
      <el-form-item label="问卷标题：" prop="name">
        <el-input
          @input="change1"
          v-model="ruleForm.name"
          style="width: 60%"
          placeholder="请输入问卷标题名称"
        ></el-input>
      </el-form-item>
      <el-form-item label="封面图：" prop="fengmiantu">
        <el-upload
          v-loading="fullscreenLoading"
          action="http://lz.api.ts.cestech.com.cn/mission_b/mission/file/imgUpload"
          :show-file-list="false"
          style="text-align: center; border: 1px dashed #d9d9d9; width: 120px; height: 120px"
          :on-preview="handlePictureCardPreview"
          :on-remove="handleRemove"
          :on-success="handlePictureSuccess"
          :before-upload="beforeAvatarUpload"
        >
          <img v-if="dialogImageUrl" :src="dialogImageUrl" style="width: 120px; height: 120px" />
          <i v-else style="margin-top: 50px" class="el-icon-plus"></i>
        </el-upload>
      </el-form-item>
      <!-- <el-form-item label="结束时间：" prop="dynamicformTypeId">
                <el-select v-model="ruleForm.dynamicformTypeId" placeholder="请选择模板类型" style="width:60%;">
                    <el-option :label="item.name" :value="item.id" v-for="item in typeList"></el-option>
                </el-select>
            </el-form-item> -->
      <el-form-item label="结束时间：" prop="time">
        <!-- <el-input v-model="ruleForm.time" style="width: 60%" placeholder="请输入问卷调研结束时间"></el-input> -->
        <el-date-picker
          v-model="ruleForm.time"
          type="datetime"
          placeholder="选择日期时间"
          value-format="yyyy-MM-dd HH:mm:ss"
          @change="change1"
        >
        </el-date-picker>
      </el-form-item>
      <el-form-item label="欢迎语：" prop="remark">
        <el-input
          @input="change1"
          v-model="ruleForm.remark"
          style="width: 60%"
          placeholder="请输入欢迎语"
          type="textarea"
          :rows="5"
        ></el-input>
      </el-form-item>
      <el-form-item label="地区：">
        <div class="areaBox">
          <!-- <j-area-linkage
            v-if="ruleForm.isArea"
            type="cascader"
            style="width: 400px"
            v-model="city_s"
            placeholder="请选择省市区"
            @change="city_change"
          /> -->
          <!-- <el-input style="width: 40%" placeholder="请选择市区县" v-model="city"></el-input> -->
          <el-radio-group v-model="ruleForm.isArea" style="margin-left: 40px">
            <el-radio :label="1">显示</el-radio>
            <el-radio :label="0">不显示</el-radio>
          </el-radio-group>
        </div>
      </el-form-item>
      <el-divider></el-divider>
      <el-form-item label="" class="timu">
        <el-button plain @click="dialogVisible = true" class="tianjiatimu">添加题目</el-button>
        <ol>
          <li v-for="(item, index) in questionList" :key="item.id">
            <dl v-if="item.type == 'radio'">
              <dt :class="item.require ? 'require' : ''">{{ item.title }}</dt>
              <el-radio
                disabled
                v-model="radio"
                :label="index"
                v-for="(item, index) in item.questionItem"
                :key="index"
                >{{ item.content }}</el-radio
              >
            </dl>
            <dl v-if="item.type == 'checkbox'">
              <dt :class="item.require ? 'require' : ''">{{ item.title }}</dt>
              <el-checkbox-group disabled v-model="checkList" v-for="(item, index) in item.questionItem" :key="index">
                <el-checkbox :label="item.content"></el-checkbox>
              </el-checkbox-group>
            </dl>
            <!-- <dl v-if="item.type == 'datetime'">
              <dt :class="item.require ? 'require' : ''">{{ item.title }}</dt>
              <el-date-picker v-model="value1" type="date" placeholder="选择日期" style="width: 60%"> </el-date-picker>
            </dl> -->
            <dl v-if="item.type == 'input'">
              <dt :class="item.require ? 'require' : ''">{{ item.title }}</dt>
              <el-input v-model="input" disabled placeholder="请输入内容" style="width: 60%"></el-input>
            </dl>
            <div class="operation">
              <el-button size="mini" @click="editContent(item)">编辑</el-button>
              <el-popconfirm
                title="您确定要删除吗"
                @confirm="deleteItem(index)"
                style="margin-left: 10px; margin-right: 10px"
              >
                <span slot="reference" style="cursor: pointer; color: #1890ff">
                  <el-button size="mini" slot="reference">删除</el-button>
                </span>
              </el-popconfirm>
              <el-button size="mini" @click="copyContent(item)">复制</el-button>
              <el-button size="mini" :disabled="index === 0" @click="moveUp(index)">上移</el-button>
              <el-button size="mini" :disabled="index === questionList.length - 1" @click="moveDown(index)">
                下移
              </el-button>
            </div>
            <el-divider></el-divider>
          </li>
        </ol>
      </el-form-item>
      <el-form-item>
        <!-- <el-button type="primary" @click="dataFormSubmit">{{ updateTitle }}</el-button> -->
        <div class="btnList">
          <el-button @click="cancelPage">取消</el-button>
          <el-button type="primary" @click="dataFormSubmit">确定</el-button>
        </div>
      </el-form-item>
    </el-form>
    <!-- 新增内容弹框 -->
    <el-dialog
      title="新增题目"
      :close-on-click-modal="false"
      :append-to-body="true"
      :visible.sync="dialogVisible"
      width="40%"
      @close="cancelDialog"
    >
      <div>
        <el-form
          :model="contentForm"
          :rules="contentrules"
          ref="contentRuleForm"
          label-width="100px"
          class="demo-ruleForm"
        >
          <el-form-item label="标题：" prop="title">
            <el-input v-model="contentForm.title" @change="change1"></el-input>
          </el-form-item>
          <el-form-item label="类型：" prop="type">
            <el-select v-model="contentForm.type" placeholder="请选择类型" style="width: 100%" @change="getMarkChange">
              <el-option label="单选" value="radio"></el-option>
              <el-option label="多选" value="checkbox"></el-option>
              <el-option label="文本" value="input"></el-option>
              <el-option label="批量添加" value="batchAdd"></el-option>
              <!-- <el-option label="日期" value="datetime"></el-option> -->
            </el-select>
          </el-form-item>
          <el-form-item label="是否必填：" prop="require">
            <el-radio-group v-model="contentForm.require" @change="change1">
              <el-radio label="是" value="true"></el-radio>
              <el-radio label="否" value="false"></el-radio>
            </el-radio-group>
          </el-form-item>
          <el-form-item
            label="选项："
            prop="questionItem"
            v-if="contentForm.type == 'checkbox' || contentForm.type == 'radio'"
          >
            <el-button plain size="mini" @click="addLine()">新增选项</el-button>
            <el-table :data="contentForm.questionItem" style="width: 100%" border class="tableBB">
              <el-table-column label="序号" type="index" width="55" align="center"> </el-table-column>
              <el-table-column label="选项内容" prop="content" align="left">
                <template slot-scope="scope">
                  <span v-show="scope.row.isShow">{{ scope.row.content }}</span>
                  <el-input
                    type="text"
                    :placeholder="scope.row.content"
                    v-model="scope.row.content"
                    v-show="!scope.row.isShow"
                    palceholder="请输入"
                    @change="change1"
                  />
                </template>
              </el-table-column>
              <!-- <el-table-column
                label="互斥"
                prop="check"
                align="center"
                width="55"
                v-if="contentForm.type == 'checkbox'"
              >
                <template slot-scope="scope">
                  <el-checkbox v-show="!scope.row.isShow" v-model="scope.row.isRepulsion" size="medium "></el-checkbox>
                  <el-checkbox
                    v-show="scope.row.isShow"
                    v-model="scope.row.isRepulsion"
                    size="medium "
                    disabled
                  ></el-checkbox>
                </template>
              </el-table-column> -->
              <el-table-column align="center" label="操作" width="200">
                <template slot-scope="scope">
                  <!-- <el-button
                    size="mini"
                    type="primary"
                    plain
                    @click="handleEdit(scope.$index, scope.row)"
                    v-show="scope.row.isShow"
                    >编辑</el-button
                  >
                  <el-button
                    @click="hold(scope.$index, scope.row)"
                    size="mini"
                    type="success"
                    plain
                    v-show="!scope.row.isShow"
                    >保存</el-button
                  > -->
                  <el-button size="mini" type="danger" plain @click="handleDelete(scope.$index, scope.row)">
                    删除
                  </el-button>
                </template>
              </el-table-column>
            </el-table>
          </el-form-item>

          <el-form-item label="批量添加：" v-if="contentForm.type == 'batchAdd'">
            <el-input type="textarea" rows="10" v-model="batchAddValue" :placeholder="placeholder"></el-input>
          </el-form-item>
        </el-form>
      </div>
      <span slot="footer" class="dialog-footer">
        <el-button @click="cancelDialog">取 消</el-button>
        <el-button type="primary" @click="submitContentForm">确 定</el-button>
      </span>
    </el-dialog>
    <!-- <el-backtop></el-backtop> -->
  </div>
</template>
<script>
// import * as api from './api'
import { nanoid } from 'nanoid'
import { httpAction, getAction } from '@/api/manage'
export default {
  name: 'temBySurvey',
  props: ['temData', 'questionGroups'],
  data() {
    return {
      fullscreenLoading: false,
      dialogImageUrl: '',
      changeImg: false,
      refresh: true,
      myQuestionGroups: [],
      myTemData: {},
      placeholder: `添加规则：
1、第一行为题目，中间不要换行，在题目后面用括号写上单选、多选、文本，为此题目信息代表单选、多选、文本。
2、下一行为题的选项，选项与选项之间不要空行。
3、选项结束后，在后面增加"下一题"表示问题的结束，然后在下行直接填写信息为新的题目信息。
例：
题目一（单选）
A
B
下一题
题目二（文本）
下一题
题目三（多选）
C
D
      `,
      city: '',
      batchAddValue: '',
      batchAdd: '',
      imageUrl: '',
      imgSrc: '',
      typeList: [],
      pageQuery: {
        pageSize: 10,
        pageNum: 1,
      },
      // 查询参数
      searchParam: {},
      // 返回参数
      pageVO: {
        list: [],
        total: 0,
        pages: 0,
      },
      markData: '',
      DelVisible: false,
      title: '新增', //表单新增/编辑
      updateTitle: '立即创建',
      dialogVisible: false,
      input: '', //体重
      value1: '', //时间选择器
      radio: '1',
      checkList: ['选中且禁用', '复选框 A'],
      ruleForm: {
        isArea: 1,
        remark: `尊敬的女士/先生

您好，我们是眉山市垃圾分类调查工作人员。感谢您抽出宝贵的时间来完成这份关于垃圾分类的问卷调查，您的回答非常重要。您的评价意见只作为总体测评之用，个人资料绝对保密，敬请放心。我们的调查采用匿名方式，故请放心答题，非常感谢您的合作!填写要求:

1.请您在所选择答案的字母上打“√”;2.对只许选择一个答案的问题只能打一个“√”;对可选多个答案的问题，请在你认为合适的答案打“√”;

3.在必要处写上相关信息。`,
      },
      contentForm: {
        questionItem: [],
        require: '是',
      },
      rules: {
        name: [{ required: true, message: '请输入模板名称', trigger: 'blur' }],
        time: [{ required: true, message: '请输入问卷调研结束时间', trigger: 'change' }],
        remark: [{ max: 300, message: '请输入欢迎语', trigger: 'blur' }],
      },
      contentrules: {
        title: [
          { required: true, message: '请输入模板名称', trigger: 'blur' },
          { min: 2, message: '长度至少2个字符', trigger: 'blur' },
        ],
        type: [{ required: true, message: '请选择模板类型', trigger: 'change' }],
        require: [{ required: true, message: '请选择是否必填', trigger: 'change' }],
        // questionItem: [{ required: true, message: '请输入选项', trigger: 'change' }],
      },
      questionList: [],
      temId: '',
      city_s: '',
    }
  },
  watch: {
    temData: {
      handler(newValue) {
        console.log('object', newValue)
        if (newValue) {
          this.myTemData = newValue
          this.ruleForm.isArea = this.myTemData.isArea
          this.ruleForm.name = this.myTemData.title
          this.ruleForm.remark = this.myTemData.welcomeSpeech
          // this.ruleForm.time = this.myTemData.endDate
          // this.city_s = this.myTemData.region
          this.dialogImageUrl = this.myTemData.titleUrl
          this.$set(this.ruleForm, 'time', this.myTemData.endDate)
          this.temId = this.myTemData.id
        }

        // this.fresh()
      },
      deep: true,
      immediate: true,
    },
    questionGroups: {
      handler(newValue) {
        console.log('questionGroups11', newValue)
        if (newValue) {
          this.questionList = []
          this.myQuestionGroups = newValue
          this.myQuestionGroups.forEach((item) => {
            let temp = {}
            console.log('item', item)
            temp.title = item.bkbQuestionnaireQuestions.questionTitle
            temp.type = item.bkbQuestionnaireQuestions.type
            temp.id = item.bkbQuestionnaireQuestions.id
            temp.require = item.bkbQuestionnaireQuestions.require ? false : true
            if (temp.type != 'input') {
              temp.questionItem = []
              item.list.forEach((item) => {
                temp.questionItem.push({ content: item.selectedName, id: item.id })
              })
            }
            this.questionList.push(temp)
          })
        }
      },
      deep: true,
      immediate: true,
    },
  },
  mounted() {
    // this.getRouterValue()
    // this.getTypePage()
  },
  methods: {
    reset() {
      this.city_s = ''
      this.imgSrc = ''
      this.imageUrl = ''
      this.dialogImageUrl = ''
      this.questionList = []
      // this.contentForm = []
      this.contentForm = {
        questionItem: [],
        require: '是',
      }
    },
    //地区下拉
    city_change(e) {
      console.log(e, 'xxx')
      this.city_s = e
    },
    change1() {
      this.$forceUpdate()
      console.log('truleForm', this.ruleForm)
    },
    fresh() {
      this.refresh = false
      this.$nextTick(() => {
        this.refresh = true
      })
    },
    //删除内容
    deleteItem(index) {
      this.questionList.splice(index, 1)
    },
    /* 编辑题目的选项 */
    handleEdit(index, row) {
      row.isShow = false
    },
    /* 保存题目的选项 */
    hold(index, row) {
      row.isShow = true
    },
    /* 删除 */
    handleDelete(index, row) {
      this.contentForm.questionItem.splice(index, 1)
    },
    /* 添加新的一行 (默认是可编辑状态)*/
    addLine() {
      // let obj = {
      //   content: '',
      //   isShow: false,
      //   isRepulsion: false,
      // }
      console.log('this.contentForm', this.contentForm)
      let obj = {
        content: '',
        id: nanoid(),
      }
      if (!this.contentForm.questionItem) {
        this.contentForm.questionItem = []
        this.$forceUpdate()
      }
      this.contentForm.questionItem.push(Object.assign({}, obj))
    },
    getTypePage() {
      // api.typePage({ ...this.pageQuery, ...this.searchParam }).then(res => {
      //     this.pageVO = res
      //     this.typeList = res.list
      //     console.log(this.typeList, '模板类型列表')
      // })
    },
    //监听选择框的值
    getMarkChange(value) {
      this.markData = value
      this.$forceUpdate()
    },
    // 上移
    moveUp(index) {
      let questionList = this.questionList
      questionList.splice(index - 1, 1, ...questionList.splice(index, 1, questionList[index - 1]))
    },
    // 下移
    moveDown(index) {
      let questionList = this.questionList
      questionList.splice(index, 1, ...questionList.splice(index + 1, 1, questionList[index]))
    },
    //编辑题目
    editContent(item) {
      this.contentForm = { ...item }
      if (this.contentForm.require == true) {
        this.contentForm.require = '是'
      } else {
        this.contentForm.require = '否'
      }
      console.log(this.contentForm, '点击编辑查看当前表单')
      this.dialogVisible = true
    },
    copyContent(value) {
      let temp = JSON.parse(JSON.stringify(value))
      temp.id = nanoid()

      this.questionList.forEach((item, index) => {
        if (item.id == value.id) {
          console.log('foreach', item)
          this.questionList.splice(index + 1, 0, temp)
          return
        }
      })
    },
    dataFormSubmit() {
      this.$refs.ruleForm.validate((valid) => {
        if (valid) {
          let data = {}
          data.questionList = JSON.parse(JSON.stringify(this.questionList))

          this.ruleForm.fields = JSON.parse(JSON.stringify(this.questionList))
          console.log(this.ruleForm)

          if (this.temId) {
            const bkbQuestionnaireInquiry = {
              title: this.ruleForm.name,
              endDate: this.ruleForm.time,
              welcomeSpeech: this.ruleForm.remark,
              isArea: this.ruleForm.isArea,
              id: this.temId,
              // region: this.city_s,
              titleUrl: this.dialogImageUrl,
            }
            httpAction(
              `/mission/bkbQuestionnaireInquiry/edit?questionnaireVo=${JSON.stringify(this.questionList)}`,
              bkbQuestionnaireInquiry,
              'post'
            ).then((res) => {
              console.log(res)
              if (res.success) {
                this.$message.success('编辑成功')
              } else {
                this.$message.warning(res.message)
              }
              this.$emit('resettingList')
            })

            this.$refs.ruleForm.resetFields()
            this.temId = ''
            this.reset()
            this.$emit('closeDiaLog', false)
          } else {
            const bkbQuestionnaireInquiry = {
              title: this.ruleForm.name,
              endDate: this.ruleForm.time,
              welcomeSpeech: this.ruleForm.remark,
              isArea: this.ruleForm.isArea,
              // region: this.city_s,
              titleUrl: this.dialogImageUrl,
            }
            httpAction(
              `/mission/bkbQuestionnaireInquiry/add?questionnaireVo=${JSON.stringify(this.questionList)}`,
              bkbQuestionnaireInquiry,
              'post'
            ).then((res) => {
              console.log(res)
              if (res.success) {
                this.$message.success(res.message)
              } else {
                this.$message.warning(res.message)
              }
              this.$emit('resettingList')
            })

            this.$refs.ruleForm.resetFields()
            this.reset()
            this.$emit('closeDiaLog', false)
          }
        } else {
          this.$nextTick(() => {
            let isError = document.getElementsByClassName('is-error')
            isError[0].scrollIntoView({
              block: 'center',
              behavior: 'smooth',
            })
          })
        }
      })
    },

    //添加题目
    submitContentForm() {
      if (this.contentForm.type == 'batchAdd') {
        console.log(this.batchAddValue)
        // const str = ``

        const quest = this.batchAddValue.split('下一题')
        console.log(quest)
        quest.forEach((item, index) => {
          const temp = item.trim().split('\n')
          console.log('!!', temp)
          // temp.forEach((item) => {
          //   item.replace(/\s*/g, '')
          // })
          let problem = {}
          console.log(temp[0].indexOf('单选'))
          if (temp[0].indexOf('单选') > 0) {
            const title = temp[0].slice(0, temp[0].indexOf('单选') - 1)
            let questionItem = []
            // console.log(temp.slice().shift());
            temp.slice().forEach((item) => {
              questionItem.push({ content: item })
            })
            questionItem.shift()
            problem = {
              id: nanoid(),
              questionItem,
              require: this.contentForm.require,
              title,
              type: 'radio',
            }
            console.log(title)
          } else if (temp[0].indexOf('多选') > 0) {
            const title = temp[0].slice(0, temp[0].indexOf('多选') - 1)
            let questionItem = []
            temp.slice().forEach((item) => {
              questionItem.push({ content: item })
            })
            questionItem.shift()
            problem = {
              id: nanoid(),
              questionItem,
              require: this.contentForm.require,
              title,
              type: 'checkbox',
            }
            console.log(title)
          } else if (temp[0].indexOf('文本') > 0) {
            const title = temp[0].slice(0, temp[0].indexOf('文本') - 1)
            problem = {
              id: nanoid(),
              questionItem: [],
              require: this.contentForm.require,
              title,
              type: 'input',
            }
          }
          this.questionList.push(problem)
          this.batchAddValue = ''
          this.dialogVisible = false
        })
      } else {
        this.$refs.contentRuleForm.validate((valid) => {
          if (valid) {
            console.log('valid', valid)
            this.contentForm.require = this.contentForm.require == '是' ? true : false
            let Obj = { ...this.contentForm }
            if (Obj.id) {
              this.questionList = this.questionList.map((ele) => {
                console.log('ele', ele)
                return ele.id == Obj.id ? Obj : ele
              })
            } else {
              // Obj.id = Date.now()
              Obj.id = nanoid()
              this.questionList = [...this.questionList, Object.assign({}, Obj)]
            }
            this.dialogVisible = false
            this.$refs.contentRuleForm.resetFields()
            this.contentForm = {
              questionItem: [],
              require: '是',
            }
          } else {
            return false
          }
        })
      }
    },
    //清空大表单
    cancelPage() {
      this.$refs.ruleForm.resetFields()
      // this.$router.go(-1)
      this.reset()
      this.$emit('closeDiaLog', false)
    },
    //清空题目表单
    cancelDialog() {
      this.dialogVisible = false
      this.$refs.contentRuleForm.resetFields()
    },
    getRouterValue() {
      if (this.$route.query.id) {
        this.title = '编辑'
        this.updateTitle = '保存'
        // api.getTemplateById(this.$route.query.id).then(res => {
        //     this.ruleForm = { ...res }
        //     this.questionList = JSON.parse(res.fields)
        //     console.log(this.updateTitle, '修改？')
        // })
      }
    },
    addImg() {},
    handleRemove(file, fileList) {
      console.log(file, fileList)
    },
    handlePictureCardPreview(file) {
      this.dialogImageUrl = file.url
      this.dialogVisible = true
    },
    handlePictureSuccess(file) {
      console.log(file, 'file')
      this.dialogImageUrl = file.result
      this.fullscreenLoading = false
    },
    // handleAvatarSuccess(res, file) {
    //   this.imageUrl = URL.createObjectURL(file.raw)
    // },
    // beforeAvatarUpload(file) {
    //   const isJPG = file.type === 'image/jpeg'
    //   const isLt2M = file.size / 1024 / 1024 < 2

    //   if (!isJPG) {
    //     this.$message.error('上传头像图片只能是 JPG 格式!')
    //   }
    //   if (!isLt2M) {
    //     this.$message.error('上传头像图片大小不能超过 2MB!')
    //   }
    //   return isJPG && isLt2M
    // },
    beforeAvatarUpload() {
      // alert(1)
      this.fullscreenLoading = true
    },
  },
}
</script>
 
<style scoped lang="less">
.require:before {
  content: '* ';
  color: red;
}

::v-deep .tableBB .el-input__inner {
  border: none !important;
}

.operation {
  width: 90%;
  text-align: right;
}

.header {
  padding: 24px;
  font-family: Helvetica Neue, Helvetica, PingFang SC, Hiragino Sans GB, Microsoft YaHei, SimSun, sans-serif;
  font-weight: 400;
  -webkit-font-smoothing: antialiased;
  -webkit-tap-highlight-color: transparent;
  font-size: 18px;
  font-weight: bolder;
  height: 312px;
  // line-height: 312px;
  margin-bottom: 30px;
  .image {
    margin: 0 auto;
    // width: 1200px;
    text-align: center;
    // display: flex;
    // // justify-content: ;
    // align-items: center;
    img {
      // width: 800px;
      height: 200px;
    }
    .addImg {
      margin: 60px auto 30px;
      width: 80px;
      height: 80px;
      line-height: 80px;
      border: 2px #bfbfbf dotted;
      font-size: 48px;
      text-align: center;
      color: #bfbfbf;
    }
  }
}

.btnList {
  margin-left: 800px;
}

/deep/.el-radio__label,
/deep/.el-checkbox__label {
  color: #000 !important;
}

.avatar-uploader .el-upload {
  border: 1px dashed #d9d9d9;
  border-radius: 6px;
  cursor: pointer;
  position: relative;
  overflow: hidden;
}
.avatar-uploader .el-upload:hover {
  border-color: #409eff;
}
.avatar-uploader-icon {
  font-size: 28px;
  color: #8c939d;
  width: 178px;
  height: 178px;
  line-height: 178px;
  text-align: center;
}
.avatar {
  width: 178px;
  height: 178px;
  display: block;
}

.areaBox {
  display: flex;
  align-items: center;
  justify-content: left;
  // margin-top: 200px;
  height: 45px;
}
.timu {
  ::v-deep.el-form-item__content {
    margin-left: 0 !important;
  }
  ::v-deep.el-radio {
    display: block;
    margin-top: 20px;
  }
}
.tianjiatimu {
  margin-left: 900px;
}
</style>