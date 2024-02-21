<template>
  <div class="mod-menu-home app-container" v-if="refresh">
    <div class="question">
      <div class="title">{{ ruleForm.name }}</div>
      <div class="text">{{ ruleForm.remark }}</div>
    </div>
    <el-form :model="ruleForm" :rules="rules" ref="ruleForm" label-width="100px" class="demo-ruleForm">
      <!-- <el-form-item label="问卷标题：" prop="name">
        <el-input v-model="ruleForm.name" style="width: 60%" placeholder="请输入问卷标题名称"></el-input>
      </el-form-item> -->
      <!-- <el-form-item label="结束时间：" prop="dynamicformTypeId">
                <el-select v-model="ruleForm.dynamicformTypeId" placeholder="请选择模板类型" style="width:60%;">
                    <el-option :label="item.name" :value="item.id" v-for="item in typeList"></el-option>
                </el-select>
            </el-form-item> -->
      <!-- <el-form-item label="结束时间：" prop="time">
        <el-input v-model="ruleForm.time" style="width: 60%" placeholder="请输入问卷调研结束时间"></el-input>
      </el-form-item> -->
      <!-- <el-form-item label="欢迎语：" prop="remark">
        <el-input
          v-model="ruleForm.remark"
          style="width: 60%"
          placeholder="请输入欢迎语"
          type="textarea"
          :rows="4"
        ></el-input>
      </el-form-item> -->
      <el-form-item label="地区：" prop="area" v-if="ruleForm.area == 1">
        <!-- <el-input style="width: 60%" placeholder="请选择市区县"></el-input> -->
        <!-- <j-area-linkage type="cascader" style="width: 400px" v-model="city_s" placeholder="请选择省市区" /> -->
      </el-form-item>
      <el-form-item label="" class="timu">
        <ol class="box">
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
              <el-checkbox-group v-model="checkList" v-for="(item, index) in item.questionItem" :key="index">
                <el-checkbox disabled :label="item.content"></el-checkbox>
              </el-checkbox-group>
            </dl>
            <!-- <dl v-if="item.type == 'datetime'">
              <dt :class="item.require ? 'require' : ''">{{ item.title }}</dt>
              <el-date-picker v-model="value1" type="date" placeholder="选择日期" style="width: 60%"> </el-date-picker>
            </dl> -->
            <dl v-if="item.type == 'input'">
              <dt :class="item.require ? 'require' : ''">{{ item.title }}</dt>
              <el-input v-model="input" placeholder="请输入内容" style="width: 60%" disabled></el-input>
            </dl>
            <el-divider></el-divider>
          </li>
        </ol>
      </el-form-item>
    </el-form>
    <!-- 新增内容弹框 -->

    <!-- <button @click="click">cnm</button> -->
    <!-- <el-backtop></el-backtop> -->
  </div>
</template>
<script>
// import * as api from './api'
import { nanoid } from 'nanoid'
export default {
  name: 'temLook',
  props: ['temData', 'questionGroups'],
  data() {
    return {
      city_s: '',
      refresh: true,
      myQuestionGroups: [],
      myTemData: {},
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
        isDisabled: true,
      },
      contentForm: {
        questionItem: [],
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
        questionItem: [{ required: true, message: '请输入选项', trigger: 'change' }],
      },
      questionList: [],
    }
  },
  watch: {
    temData: {
      handler(newValue) {
        this.myTemData = newValue
        this.ruleForm.area = this.myTemData.isArea
        this.ruleForm.name = this.myTemData.title
        this.ruleForm.remark = this.myTemData.welcomeSpeech
        // this.city_s = this.myTemData.region
        this.imgSrc = this.myTemData.titleUrl
        this.fresh()
        console.log('2222', this.ruleForm, this.city_s)
      },
      deep: true,
      immediate: true,
    },
    questionGroups: {
      handler(newValue) {
        console.log('questionGroups', newValue)
        this.questionList = []
        this.myQuestionGroups = newValue
        this.myQuestionGroups.forEach((item) => {
          let temp = {}
          temp.title = item.bkbQuestionnaireQuestionsTemplate.questionTitle
          temp.type = item.bkbQuestionnaireQuestionsTemplate.type
          temp.id = item.bkbQuestionnaireQuestionsTemplate.id
          temp.require = item.bkbQuestionnaireQuestionsTemplate.require ? false : true
          if (temp.type != 'input') {
            temp.questionItem = []
            item.list.forEach((item) => {
              temp.questionItem.push({ content: item.selectedName })
            })
          }
          this.questionList.push(temp)
        })
        console.log('1111', this.questionList)
      },
      deep: true,
      immediate: true,
    },
  },
  mounted() {
    // this.getRouterValue()
    // this.getTypePage()
    // this.remark
  },
  methods: {
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
      let obj = {
        content: '',
        isShow: false,
        isRepulsion: false,
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

      console.log('@@@')
    },
    dataFormSubmit() {
      this.$refs.ruleForm.validate((valid) => {
        if (valid) {
          this.ruleForm.fields = JSON.stringify(this.questionList)
          console.log(this.ruleForm.fields)
          // const request = this.$route.query.id ? api.update(this.ruleForm) : api.save(this.ruleForm)
          // request.then(data => {
          //     this.$message({
          //         message: this.$t('table.actionSuccess'),
          //         type: 'success',
          //         duration: 1500,
          //         onClose: () => {
          //             this.$refs.ruleForm.resetFields()
          //             this.$router.go(-1)
          //         }
          //     })
          // })
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
    click() {
      this.questionList = [
        {
          questionItem: [
            {
              content: '萨达',
              isShow: false,
              isRepulsion: false,
            },
          ],
          title: '大声道',
          type: 'radio',
          require: true,
          id: 1683538949236,
        },
        {
          questionItem: [
            {
              content: '萨达',
              isShow: false,
              isRepulsion: false,
            },
            {
              content: 'sadasd ',
              isShow: false,
              isRepulsion: false,
            },
          ],
          title: 'sdasdasdwww',
          type: 'radio',
          require: true,
          id: 16835,
        },
        {
          questionItem: [
            {
              content: '萨达',
              isShow: false,
              isRepulsion: false,
            },
            {
              content: '萨达',
              isShow: false,
              isRepulsion: false,
            },
            {
              content: '萨达',
              isShow: false,
              isRepulsion: false,
            },
          ],
          title: '大声sdadw道',
          type: 'checkbox',
          require: true,
          id: 16835389492,
        },
      ]
    },
    //添加题目
    submitContentForm() {
      this.$refs.contentRuleForm.validate((valid) => {
        if (valid) {
          console.log('valid', valid)

          this.contentForm.require = this.contentForm.require == '是' ? true : false

          let Obj = { ...this.contentForm }
          console.log('this.contentForm', Obj, this.contentForm)
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

          console.log(this.questionList, '当前模板内容')
          this.dialogVisible = false
          this.$refs.contentRuleForm.resetFields()
          this.contentForm = {
            questionItem: [],
          }
        } else {
          return false
        }
      })
    },
    //清空大表单
    cancelPage() {
      this.$refs.ruleForm.resetFields()
      this.$router.go(-1)
    },
    //清空题目表单
    cancelDialog() {
      this.$refs.contentRuleForm.resetFields()
      this.dialogVisible = false
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
    // height: 200px;
    text-align: center;
    img {
      // width: 800px;
      height: 250px;
    }
    // display: flex;
    // // justify-content: ;
    // align-items: center;
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

.question {
  .title {
    margin: 0 auto;
    text-align: center;
    font-size: 24px;
  }

  .text {
    padding: 20px 30px;
  }
}

/deep/.el-radio__label,
/deep/.el-checkbox__label {
  color: #000 !important;
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
</style>