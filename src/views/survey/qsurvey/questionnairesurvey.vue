<template>
  <div class="content">
    <el-card class="box-card">
      <div class="header">
        <div class="wjinput">
          <div class="title">问卷标题：</div>
          <el-input v-model="questionnaireTitle" placeholder="请输入内容"></el-input>
        </div>
        <div class="wjinput">
          <span>问卷状态：</span>
          <el-select v-model="questionnaireStatus" placeholder="请选择" @change="statusChange">
            <el-option v-for="item in questionnaire_status" :key="item.value" :label="item.label" :value="item.value">
            </el-option>
          </el-select>
        </div>
        <div class="wjinput">
          <span>调研员：</span>
          <el-select v-model="researcher" placeholder="请选择" @change="change_researchers">
            <el-option v-for="item in researchers" :key="item.value" :label="item.value" :value="item.label">
            </el-option>
          </el-select>
        </div>
        <div class="btn">
          <div class="query"><el-button type="primary" @click="queryList">查询</el-button></div>
          <div class="query"><el-button type="primary" @click="resettingList">重置</el-button></div>
          <div class="query"><el-button type="primary" @click="open_delete_s">批量删除</el-button></div>
          <div><el-button type="primary" @click="questionnaire_creation">问卷创建</el-button></div>
        </div>
      </div>
      <div class="tb">
        <el-table
          ref="multipleTable"
          :row-key="
            (row) => {
              return row.id
            }
          "
          :data="bkbQuestionnaireInquiryList"
          border
          tooltip-effect="dark"
          style="width: 100%"
          @selection-change="handleSelectionChange"
        >
          <el-table-column type="selection" :reserve-selection="true" width="55" align="center"> </el-table-column>
          <el-table-column type="index" label="序号" width="55" align="center" :index="indexAdd"> </el-table-column>
          <el-table-column prop="title" label="问卷标题" align="center"> </el-table-column>
          <el-table-column align="center" prop="titleUrl" label="封面图" width="">
            <template slot-scope="scope">
              <div class="fengmiantu">
                <!-- {{ scope.row }} -->
                <img :src="scope.row.titleUrl" alt="" srcset="" />
              </div>
            </template>
          </el-table-column>
          <el-table-column prop="createTime" label="创建时间" align="center" width="200"> </el-table-column>
          <el-table-column prop="endDate" label="问卷结束时间" align="center" width="200"> </el-table-column>
          <!-- <el-table-column label="创建时间" align="center" width="200">
            <template slot-scope="scope">{{ scope.row.createTime }}</template>
          </el-table-column> -->
          <!-- <el-table-column label="问卷结束时间" width="200" align="center">
            <template slot-scope="scope">{{ scope.row.date }}</template>
          </el-table-column> -->
          <el-table-column prop="investigatorId_dictText" label="调研员" width="120" align="center"> </el-table-column>
          <el-table-column prop="status" label="问卷状态" width="80" align="center"> </el-table-column>
          <el-table-column prop="numberAnswers" label="答卷数量" width="80" align="center"> </el-table-column>
          <el-table-column prop="name" label="操作" width="340" align="center">
            <template slot-scope="scope">
              <el-button type="primary" v-if="scope.row.status != '草稿'" @click="check_ck(scope.row.id)"
                >查看</el-button
              >
              <el-button type="primary" v-if="scope.row.status != '草稿'" @click="questionnaire_data(scope.row.id)"
                >数据查看</el-button
              >
              <el-button type="primary" v-if="scope.row.status === '草稿'" @click="release(scope.row.id)"
                >发布</el-button
              >
              <el-button type="primary" v-if="scope.row.status === '草稿'" @click="edit(scope.row.id)">编辑</el-button>
              <el-button type="primary" @click="open_delete(scope.row.id)">删除</el-button>
            </template>
          </el-table-column>
        </el-table>
      </div>
      <div class="block">
        <el-pagination
          background
          layout="prev, pager, next"
          :total="total"
          :page-size="pageSize"
          @current-change="handleCurrentChange"
        >
        </el-pagination>
      </div>
    </el-card>
    <!-- 发布蒙城 -->
    <div class="researcher" v-if="researcher_show">
      <div class="msk">
        <div class="iocn" @click="researcher_qx">x</div>
        <div class="name">选择调研员</div>
        <div class="user">
          <span>调研员：</span>
          <el-select v-model="fb_researcher" multiple placeholder="请选择">
            <el-option v-for="item in researchers" :key="item.value" :label="item.value" :value="item.label">
            </el-option>
          </el-select>
          <!-- <el-select v-model="fb_researcher" placeholder="请选择">
            <el-option v-for="item in researchers" :key="item.value" :label="item.value" :value="item.label">
            </el-option>
          </el-select> -->
        </div>
        <div class="btn">
          <el-button type="primary" @click="release_confirm">确认</el-button>
          <el-button type="info" @click="researcher_qx">取消</el-button>
        </div>
      </div>
    </div>
    <!-- 问卷模板创建蒙层 -->

    <!-- <div class="template-msk" v-if="template_show">
      <div class="msk">
        <div class="iocn" @click="researcher_qx">x</div>
        <div class="name">问卷创建</div>
        <div class="mian-btn">
          <div style="width: 60px"></div>
          <div class="btn" @click="addByBlank">空白模板</div>
          <div class="btn" @click="addByTem">采用创建</div>
        </div>
        <div class="btns">
          <el-button type="primary">确认</el-button>
          <el-button type="info" @click="researcher_qx">取消</el-button>
        </div>
      </div>
    </div> -->

    <el-dialog title="问卷创建" :visible.sync="template_show" width="25%">
      <div class="mian-btn">
        <el-button class="btn" @click="addByBlank">空白模板</el-button>
        <el-button class="btn" @click="addByTem">采用创建</el-button>
      </div>
      <div class="btns">
        <el-button type="primary">确认</el-button>
        <el-button type="info" @click="researcher_qx">取消</el-button>
      </div>
    </el-dialog>

    <!-- 问卷数据查看 -->
    <el-dialog :visible.sync="dialogVisible" width="70%" :before-close="handleClose">
      <div class="wjsjck">
        <el-tabs v-model="activeName" @tab-click="handleClick">
          <el-tab-pane label="数据统计" name="first">
            <!-- <div class="image"><img :src="queryById.titleUrl" /></div> -->

            <div ref="bar">
              <div class="title">
                {{ queryById.title }}
              </div>
              <div class="tip">
                {{ queryById.welcomeSpeech }}
              </div>
              <div class="select daochu shujutongji">
                <div class="city" v-if="showRegion">
                  <span>地区：</span>

                  <!-- <j-area-linkage type="cascader" v-model="indexStyle" placeholder="请选择省市区" @change="city_change" /> -->
                  <VDistpicker v-if="v_selected" @selected="onSelected2"></VDistpicker>
                  <!-- <div>你当前选的的地址:{{ fullAddress }}</div> -->
                  <!-- <el-select v-model="value" placeholder="请选择">
                  <el-option v-for="item in options" :key="item.value" :label="item.label" :value="item.value">
                  </el-option>
                </el-select> -->
                  <el-button type="primary" style="margin-left: 15px" @click="regionalQuery">查询</el-button>
                  <el-button type="primary" @click="resetRegion">重置</el-button>
                </div>

                <el-button type="primary" @click="printOut">导出</el-button>
              </div>
              <div class="switch" style="margin-top: 20px">
                <!-- <button @click="printOut">xiazia </button> -->
                <el-radio-group v-model="radio2" size="medium" @input="switch_qh">
                  <el-radio-button label="柱状图统计"></el-radio-button>
                  <el-radio-button label="饼图统计"></el-radio-button>
                </el-radio-group>
              </div>
              <div v-if="pie_show">
                <div v-for="item in echartsList" :key="item.id" id="mycanvas" ref="mycanvas">
                  <div v-if="item.type != 'input'">
                    <div v-if="barRe">
                      <div class="pie-name">{{ item.title }}</div>
                      <!-- <bar :get-data="objectData" id="barId" style="height: 250px"></bar> -->
                      <echarts-bar
                        ref="echart"
                        :legend-data="item.legendData"
                        :x-axis-data="item.legendData"
                        :series-data="item.list"
                      ></echarts-bar>
                    </div>
                  </div>
                  <div v-else class="inputXianShi">
                    <div class="pie-name">{{ item.title }}</div>
                    <div class="inputNum">答题数为：{{ item.list[0] }}人</div>
                  </div>
                  <el-divider></el-divider>
                </div>
              </div>
              <div v-if="bt_show" ref="pie">
                <div v-for="(item, index) in pieChartData" :key="index">
                  <div v-if="item.type != 'input'">
                    <div class="pie-name">{{ item.content }}</div>
                    <PieChart :data="item.bean" :legend-data="item.legendData" />
                  </div>
                  <div v-else class="inputXianShi">
                    <div class="pie-name">{{ item.content }}</div>
                    <div class="inputNum">答题数为：{{ item.bean[0].value }}人</div>
                  </div>
                </div>
              </div>
            </div>
            <!-- <div class="projectCost">
              <div class="container">
                <div class="wrapper" v-for="(item, index) in list" :key="index">
                  <div class="roseChart"></div>
                </div>
              </div>
            </div> -->
          </el-tab-pane>
          <el-tab-pane label="数据报表" name="second">
            <div class="select daochu">
              <div class="city" v-if="showRegion">
                <span>地区：</span>
                <!-- <el-select v-model="value" placeholder="请选择">
                  <el-option v-for="item in options" :key="item.value" :label="item.label" :value="item.value">
                  </el-option>
                </el-select> -->
                <!-- <j-area-linkage type="cascader" v-model="indexStyleTable" placeholder="请选择省市区" /> -->
                <VDistpicker v-if="v_selected" @selected="onSelected"></VDistpicker>
                <el-button type="primary" @click="regionalQueryTable" style="margin-left: 15px">查询</el-button>
                <el-button type="primary" @click="resetRegion">重置</el-button>
              </div>
              <el-button type="primary" @click="downLoadTable">导出</el-button>
            </div>
            <div ref="downloadTable">
              <el-table :data="tableData" border style="width: 100%" :span-method="objectSpanMethod">
                <!-- <el-table-column align="center" type="index" label="序号" width="100"> </el-table-column> -->
                <el-table-column align="center" prop="content" label="问卷内容" />
                <el-table-column align="center" prop="son" label="问卷选项">
                  <template slot-scope="scope">
                    <div v-if="scope.row.type == 'input'">填空题</div>
                    <div v-else>
                      <span>{{ scope.row.son }}</span>
                    </div>
                  </template>
                </el-table-column>
                <el-table-column align="center" prop="amount" label="答题人数量" />

                <el-table-column align="center" prop="proportion" label="占比率">
                  <template slot-scope="scope">
                    <div v-if="scope.row.type == 'input'">
                      <el-button @click="checkTable(scope.row)" type="primary">查看</el-button>
                    </div>
                    <div v-else>
                      <span>{{ parseFloat(scope.row.proportion).toFixed(2) }}%</span>
                    </div>
                  </template>
                </el-table-column>
              </el-table>
            </div>
          </el-tab-pane>
        </el-tabs>
      </div>
    </el-dialog>

    <el-dialog :title="checkTableTitle" :visible.sync="tableVisible" width="50%" :append-to-body="true">
      <div class="dialogTable">
        <!-- <tem @closeDiaLog="closeDiaLog"></tem> -->
        <el-table :data="checkTableData" border style="width: 100%" :span-method="objectSpanMethod">
          <el-table-column align="center" type="index" label="序号" width="100"> </el-table-column>
          <el-table-column align="center" prop="content" label="答案内容" />
        </el-table>
      </div>
      <span slot="footer" class="dialog-footer">
        <el-button @click="tableVisible = false">取 消</el-button>
        <el-button type="primary" @click="tableVisible = false">确 定</el-button>
      </span>
    </el-dialog>

    <el-dialog title="问卷查看" :visible.sync="lookVisible" width="60%">
      <div class="dialogInput">
        <temLookBySurvey :temData="temData" :questionGroups="questionGroups"></temLookBySurvey>
      </div>
      <span slot="footer" class="dialog-footer">
        <el-button @click="lookVisible = false">取 消</el-button>
        <el-button type="primary" @click="lookVisible = false">确 定</el-button>
      </span>
    </el-dialog>

    <el-dialog title="编辑" :visible.sync="editVisible" width="60%">
      <div class="dialogInput">
        <temBySurvey
          @resettingList="resettingList"
          @closeDiaLog="closeDiaLog"
          :temData="temData"
          :questionGroups="questionGroups"
        ></temBySurvey>
      </div>
      <!-- <span slot="footer" class="dialog-footer">
        <el-button @click="editVisible = false">取 消</el-button>
        <el-button type="primary" @click="editClick">确 定</el-button>
      </span> -->
    </el-dialog>

    <el-dialog
      title="空白模板"
      :visible.sync="addByBlankVisible"
      width="60%"
      :close-on-click-modal="false"
      :destroy-on-close="true"
    >
      <div class="dialogInput">
        <temBySurvey @closeDiaLog="closeDiaLog" @resettingList="resettingList"></temBySurvey>
      </div>
    </el-dialog>

    <el-dialog title="采用模板" :visible.sync="addByTemVisible" width="60%">
      <div class="table">
        <el-table
          :data="tableDataTem"
          border
          style="width: 60%"
          @selection-change="handleSelectionChange"
          :row-key="
            (row) => {
              return row.id
            }
          "
        >
          <el-table-column align="center" type="index" label="序号" width="50" :index="indexAdd"> </el-table-column>
          <el-table-column align="center" prop="title" label="问卷标题" width=""> </el-table-column>
          <el-table-column align="center" prop="createTime" label="创建时间" width=""> </el-table-column>
          <el-table-column align="center" label="操作" width="150">
            <template slot-scope="scope">
              <el-button type="primary" @click="lookTem(scope.row)">查看</el-button>
            </template>
          </el-table-column>
        </el-table>
        <div class="pagination">
          <el-pagination
            background
            layout="prev, pager, next"
            :total="totalTem"
            :page-size="pageSize"
            @current-change="handleCurrentChangeTem"
          >
          </el-pagination>
        </div>
      </div>
    </el-dialog>

    <el-dialog title="问卷模板查看" :visible.sync="lookTemVisible" width="60%">
      <div class="dialogInput">
        <temcaiyong @closeDiaLog="closeDiaLog" :temData="temData" :questionGroups="questionGroups"></temcaiyong>
      </div>
      <!-- <span slot="footer" class="dialog-footer">
        <el-button @click="lookTemVisible = false">取 消</el-button>
        <el-button type="primary" @click="createByTem">采用此模板创建</el-button>
      </span> -->
    </el-dialog>
  </div>
</template>
<script>
import htmlToPdf from '@/utils/downloadPDF.js'
import VDistpicker from 'v-distpicker'
import * as echarts from 'echarts'
import html2Canvas from 'html2canvas'
import JsPDF from 'jspdf'
import { httpAction, getAction } from '@/api/manage'
import EchartsBar from '../components/PieChart.vue'
import PieChart from '../components/btehart.vue'
import Bar from '../components/zzechart.vue'
import temLook from '../components/temLook.vue'
import temLookBySurvey from '../components/temLookBySurvey.vue'
import temBySurvey from '../components/temBySurvey.vue'
import temcaiyong from '../components/temcaiyong.vue'
export default {
  install(Vue, options) {
    Vue.prototype.getPdf = function (domId, title) {
      html2Canvas(document.getElementById(domId), {
        allowTaint: true,
        useCORS: true,
      }).then(function (canvas) {
        let contentWidth = canvas.width
        let contentHeight = canvas.height
        let pageHeight = (contentWidth / 592.28) * 841.89
        let leftHeight = contentHeight
        let position = 0
        let imgWidth = 595.28
        let imgHeight = (592.28 / contentWidth) * contentHeight
        let pageData = canvas.toDataURL('image/jpeg', 1.0)
        let PDF = new JsPDF('', 'pt', 'a4')
        if (leftHeight < pageHeight) {
          PDF.addImage(pageData, 'JPEG', 0, 0, imgWidth, imgHeight)
        } else {
          while (leftHeight > 0) {
            PDF.addImage(pageData, 'JPEG', 0, position, imgWidth, imgHeight)
            leftHeight -= pageHeight
            position -= 841.89
            if (leftHeight > 0) {
              PDF.addPage()
            }
          }
        }
        PDF.save(title + '.pdf')
      })
    }
  },
  components: { VDistpicker, EchartsBar, PieChart, Bar, temcaiyong, temLook, temBySurvey, temLookBySurvey },
  name: 'questionnairesurvey',
  data() {
    return {
      showRegion: true,
      barRe: true,
      v_selected: true,
      selected: {
        province: '',
        city: '',
        area: '',
      },
      selected2: {
        province: '',
        city: '',
        area: '',
      },
      myId: '',
      imgSrc: '',
      // questionGroups
      // 模板创建
      totalTem: 0,
      tableDataTem: [],
      pageNoTem: 1,
      tableTemVisible: false,
      temData: {},
      questionGroups: [],
      lookVisible: false,
      editVisible: false,
      addByBlankVisible: false,
      addByTemVisible: false,
      lookTemVisible: false,
      indexStyle: '',
      indexStyleTable: '',
      radio2: '柱状图统计',
      textTitle: '',
      nameTitle: '',
      nameArray: [],
      dataArray: [],
      colorArray: [],
      pageNo: 1,
      pageSize: 10,
      total: 0,
      pie_show: true,
      bt_show: false,
      echartsList: [
        // {
        //   title: '性别是男是女？',
        //   legendData: ['男', '女', '未知', '保密'],
        //   list: [5, 20, 36, 10],
        //   // xAxisData: ['男', '女', '未知', '保密'],
        // },
        // {
        //   title: '名称2',
        //   legendData: ['A', 'B', 'C', 'D', 'E', 'F', 'G'],
        //   list: [15, 28, 36, 19, 24, 78, 19],
        //   // xAxisData: ['A', 'B', 'C', 'D', 'E', 'F', 'G'],
        // },
      ],

      pieChartData: [
        {
          title: '饼图1',
          legendData: ['Apple', 'Orange', 'Banana'],
          list: [
            { name: 'Apple', value: 10 },
            { name: 'Orange', value: 90 },
            { name: 'Banana', value: 50 },
          ],
        },
        {
          title: '性别是男是女',
          legendData: ['男', '女', 'Ban未知ana'],
          list: [
            { name: '男', value: 30 },
            { name: '女', value: 20 },
            { name: '未知', value: 50 },
            { name: '保密', value: 10 },
          ],
        },
        {
          title: '饼图3',
          legendData: ['Apple', 'Orange', 'Banana'],
          list: [
            { name: 'Apple', value: 30 },
            { name: 'Orange', value: 20 },
            { name: 'Banana', value: 50 },
          ],
        },
      ],
      checkTableTitle: '',
      checkTableData: [
        {
          content: '没有什么明确的建议',
        },
        {
          content: '怎么方便怎么来',
        },
        {
          content: '任何建议',
        },
      ],
      tableVisible: false,
      dialogVisible: false,
      spanArr: [],
      colFields: ['content', 'options', 'amount', 'proportion'],
      tableData: [],
      activeName: 'first',
      input: '',
      flg: 2,
      researcher_show: false,
      template_show: false,
      researchers: [], //问卷人员下拉数组
      questionnaire_status: [
        {
          value: 0,
          label: '草稿',
        },
        {
          value: 1,
          label: '发布',
        },
        {
          value: 2,
          label: '已结束',
        },
      ], //问卷状态数组
      researcher: '', //问卷人员下拉绑定v-model
      fb_researcher: [], //发布调研人员下拉绑定v-model
      questionnaireStatus: '', //问卷状态下拉绑定v-model
      questionnaireTitle: '', //问卷标题
      bkbQuestionnaireInquiryList: [], //列表
      fb_researcher_id: '', //选中列表id
      rowKey: [], //批量选中
      detaleList: [],
      ids: [], //批量删除
      queryById: {},
      options: [],
      value: '',
      multipleSelection: [],
      currentPage4: 4,
      elm: {},
    }
  },
  created() {
    this.getAListOfInvestigators() //调研人员下拉
    this.bkbQuestionnaireInquiry_list() //列表
  },
  computed: {
    fullAddress: function () {
      return this.selected.province + this.selected.city + this.selected.area
    },
    fullAddress2: function () {
      return this.selected2.province + this.selected2.city + this.selected2.area
    },
  },
  methods: {
    onSelected(data) {
      const { province, city, area } = data
      if (!province.code && !city.code && !city.code) return
      this.selected.province = province.value
      this.selected.city = city.value
      this.selected.area = area.value
    },
    onSelected2(data) {
      const { province, city, area } = data
      if (!province.code && !city.code && !city.code) return
      this.selected2.province = province.value
      this.selected2.city = city.value
      this.selected2.area = area.value
      console.log(data, this.fullAddress2)
    },
    downLoadTable() {
      htmlToPdf.downloadPDF(this.$refs.downloadTable, '数据报表统计')
    },
    // 导出PDF
    printOut() {
      console.log(this.radio2)
      // if (this.radio2 == '饼图统计') {
      // htmlToPdf.downloadPDF(this.$refs.pie, '饼图统计')
      // } else if (this.radio2 == '柱状图统计') {
      htmlToPdf.downloadPDF(this.$refs.bar, '数据统计')
      // }
    },
    //地区下拉
    city_change(e) {
      console.log(this.indexStyle, e, 'xxx')
      console.log(this.elm, 'this.elm')
    },
    //调研人员下拉
    getAListOfInvestigators() {
      httpAction('/mission/bkbInvestigator/getAListOfInvestigators', {}, 'get').then((res) => {
        if (res.code == 200) {
          this.researchers = res.result
        }
      })
    },
    //问卷调研列表
    bkbQuestionnaireInquiry_list() {
      httpAction(
        `/mission/bkbQuestionnaireInquiry/list?title=${this.questionnaireTitle}&status=${this.questionnaireStatus}&investigatorId=${this.researcher}&pageNo=${this.pageNo}&pageSize=${this.pageSize}`,
        {},
        'get'
      ).then((res) => {
        if (res.code == 200) {
          console.log(res.result.total, '列表')
          res.result.records.forEach((v) => {
            if (v.status === 0) {
              v.status = '草稿'
            } else if (v.status === 1) {
              v.status = '发布'
            } else if (v.status === 2) {
              v.status = '已结束'
            }
          })
          this.bkbQuestionnaireInquiryList = res.result.records
          this.total = res.result.total
        }
      })
    },
    //确认发布
    release_confirm() {
      if (this.fb_researcher.length <= 0) {
        console.log('111')
        this.$message({
          message: '请选择调研员',
          type: 'warning',
        })
      } else {
        console.log(this.fb_researcher, '222')
        httpAction(
          `/mission/bkbQuestionnaireInquiry/release?id=${this.fb_researcher_id}&investigatorId=${this.fb_researcher}`,
          {},
          'get'
        ).then((res) => {
          if (res.code === 200) {
            this.$message({
              message: '发布成功',
              type: 'success',
            })
            this.bkbQuestionnaireInquiry_list()
            this.researcher_show = false
            // this.researchers=res.result
            // console.log(res.result,'调研人员下拉')
          }
        })
      }
    },
    //通过id删除数据
    open_delete(e) {
      this.$confirm('确认是否删除此问卷信息?', '删除', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning',
      })
        .then(() => {
          httpAction(`/mission/bkbQuestionnaireInquiry/delete?id=${e}`, {}, 'DELETE').then((res) => {
            if (res.code == 200) {
              this.bkbQuestionnaireInquiry_list()
              this.$message({
                type: 'success',
                message: '删除成功!',
              })
            }
          })
        })

        .catch(() => {
          this.$message({
            type: 'info',
            message: '已取消删除',
          })
        })
    },
    //批量删除
    open_delete_s() {
      if (this.ids.length != 0) {
        this.$confirm('确认是否删除此问卷信息?', '删除', {
          confirmButtonText: '确定',
          cancelButtonText: '取消',
          type: 'warning',
        })
          .then(() => {
            httpAction(`/mission/bkbQuestionnaireInquiry/deleteBatch?ids=${this.ids}`, {}, 'DELETE').then((res) => {
              if (res.code == 200) {
                this.bkbQuestionnaireInquiry_list()
                this.$message({
                  type: 'success',
                  message: '删除成功!',
                })
              }
            })
          })

          .catch(() => {
            this.$message({
              type: 'info',
              message: '已取消删除',
            })
          })
      } else {
        this.$message({
          type: 'warning',
          message: '请先勾选数据在删除!',
        })
      }
    },
    //查看
    check_ck(e) {
      httpAction(`/mission/bkbQuestionnaireInquiry/queryById?id=${e}`, {}, 'get').then((res) => {
        if (res.code == 200) {
          console.log(res, 'id查询得到')
          this.temData = res.result
          this.queryQuestionGroups(res.result.id)
          this.lookVisible = true
        }
      })
    },
    //编辑
    edit(e) {
      httpAction(`/mission/bkbQuestionnaireInquiry/queryById?id=${e}`, {}, 'GET').then((res) => {
        console.log('asdasdas', e, res)
        if (res.code == 200) {
          console.log(res, 'id查询得到')
          this.temData = res.result
          this.queryQuestionGroups(res.result.id)
          this.editVisible = true
        }
      })
    },
    //空白模板 创建
    addByBlank() {
      this.addByBlankVisible = true
    },
    // 采用模板 创建
    addByTem() {
      this.addByTemVisible = true
      this.searchTem()
    },
    // 模板列表
    searchTem() {
      httpAction(`/mission/bkbQuestionnaireInquiryTemplate/list?pageNo=${this.pageNoTem}`, {}, 'get').then((res) => {
        console.log(res)
        if (res.success) {
          this.totalTem = res.result.total
          this.tableDataTem = res.result.records
        } else {
          this.$message.warning(res.message)
        }
      })
    },
    // 采用模板 查看
    lookTem(e) {
      console.log('e', e)
      // this.queryQuestionGroups(e.id)

      httpAction(`/mission/bkbQuestionnaireQuestionsTemplate/queryQuestionGroups?id=${e.id}`, {}, 'get').then((res) => {
        if (res.success) {
          this.questionGroups = res.result.tm
          console.log('this.questionGroups', this.questionGroups, res)
        } else {
          this.$message.warning(res.message)
        }
      })

      this.temData = e
      this.lookTemVisible = true
    },

    // 使用模板创建
    createByTem() {
      console.log(this.questionGroups, this.temData, 'tem')
      this.temData.id = ''
      let questionList = []
      this.questionGroups.forEach((item) => {
        let questionItem = []
        item.list.forEach((item) => {
          questionItem.push({
            content: item.selectedName,
            id: item.id,
          })
        })
        questionList.push({
          questionItem,
          type: item.bkbQuestionnaireQuestionsTemplate.type,
          title: item.bkbQuestionnaireQuestionsTemplate.questionTitle,
          require: item.bkbQuestionnaireQuestionsTemplate.required,
          id: item.bkbQuestionnaireQuestionsTemplate.questionnaireInquiryId,
        })
      })
      console.log('object', questionList)
      // debugger
      httpAction(
        `/mission/bkbQuestionnaireInquiry/add?questionnaireVo=${JSON.stringify(questionList)}`,
        this.temData,
        'post'
      ).then((res) => {
        console.log(res)
        if (res.success) {
          this.$message.success(res.message)
        } else {
          this.$message.warning(res.message)
        }
        this.resettingList()
      })
    },

    //弹窗关闭
    closeDiaLog(value) {
      this.editVisible = value
      this.addByBlankVisible = value
      this.addByTemVisible = value
      this.resettingList()
    },
    //数据查看
    questionnaire_data(e) {
      this.dialogVisible = true

      this.selected = {
        province: '',
        city: '',
        area: '',
      }
      this.selected2 = {
        province: '',
        city: '',
        area: '',
      }
      this.v_selected = false
      setTimeout(() => {
        this.v_selected = true
      }, 0)
      // alert(1)
      this.myId = e
      let data = {
        id: e,
      }
      httpAction(`/mission/bkbQuestionnaireInquiry/dateExcel`, data, 'post').then((res) => {
        if (res.code == 200) {
          this.tableData = res.result

          console.log(res.result, 'id查询得到====')
        }
      })
      //查询数据
      httpAction(`/mission/bkbQuestionnaireInquiry/queryById?id=${e}`, {}, 'get').then((res) => {
        if (res.code == 200) {
          // this.tableData=res.result
          console.log(res.result, 'id5646565查询得到')
          this.queryById = res.result
          this.showRegion = res.result.isArea == 0 ? false : true
        }
      })
      //柱状图
      httpAction(`/mission/bkbQuestionnaireInquiry/histogram`, data, 'post').then((res) => {
        this.echartsList = {}
        if (res.code == 200) {
          // this.tableData=res.result
          console.log(res.result, '柱状图')
          res.result.forEach((v) => {
            // console.log(v.legendData, 'aaa')
            v.list = v.list.map(Number)
          })
          let temp = res.result

          temp.forEach((item, index) => {
            if (item.type != 'input') {
              let count = 0
              let tempList = []
              item.list.forEach((item2) => {
                count += item2
              })
              item.list.forEach((item3) => {
                tempList.push(((item3 / count) * 100).toFixed(2))
              })
              temp[index].list = JSON.parse(JSON.stringify(tempList))
              console.log(count, tempList, 'kkk')
            }
          })

          // this.echartsList = res.result
          this.echartsList = JSON.parse(JSON.stringify(temp))
          console.log(this.echartsList, '柱状图123')
          // this.$refs['echart'].getChart()

          // this.queryById=res.result
          this.barRe = false
          setTimeout(() => {
            this.barRe = true
          }, 0)
        }
      })
      //饼图数据
      httpAction(`/mission/bkbQuestionnaireInquiry/pieChart`, data, 'post').then((res) => {
        if (res.code == 200) {
          // this.tableData=res.result
          console.log(res.result, '饼图')
          res.result.forEach((v) => {
            console.log(v.bean, 'bing')
            v.bean.map((item) => {
              let obj = {
                name: item.selected_name,
                value: item.count_column,
              }
              v.bean.push(obj)
            })
          })
          this.pieChartData = res.result
        }
      })
    },
    //查询列表
    queryList() {
      this.bkbQuestionnaireInquiry_list()
    },
    //重置列表
    resettingList() {
      this.questionnaireTitle = ''
      this.questionnaireStatus = ''
      this.researcher = ''
      this.template_show = false
      this.addByTemVisible = false
      this.lookTemVisible = false
      this.bkbQuestionnaireInquiry_list()
      // alert(1)
    },
    //调研人员下拉change事件
    change_researchers(e) {
      console.log(e, this.researcher, 'xx')
    },
    //问卷状态下拉change事件
    statusChange(e) {
      console.log(e, this.questionnaireStatus, 'xx')
      // this.status=e
    },
    switch_qh(e) {
      console.log(e)
      if (e === '柱状图统计') {
        this.pie_show = true
        this.bt_show = false
      } else {
        this.pie_show = false
        this.bt_show = true
      }
    },
    checkTable(e) {
      this.tableVisible = true
      console.log(e)
      // this.checkTableTitle = e.content

      httpAction(`/mission/bkbQuestionnaireInquiry/lookDate?id=${e.id}&region=${this.fullAddress}`, {}, 'post').then(
        (res) => {
          if (res.code == 200) {
            this.checkTableData = res.result
          }
        }
      )
    },
    // 合并单元格
    getSummaries(param) {
      const { columns, data } = param
      const sums = []
      columns.forEach((column, index) => {
        if (index === 0) {
          sums[index] = '合计'
        } else if (index === 2 || index === 7) {
          const values = data.map((item) => Number(item[column.property]))
          if (!values.every((value) => isNaN(value))) {
            sums[index] = values.reduce((prev, curr) => {
              const value = Number(curr)
              if (!isNaN(value)) {
                return prev + curr
              } else {
                return prev
              }
            }, 0)
          } else {
            sums[index] = ''
          }
        } else {
          sums[index] = ''
        }
      })
      return sums
    },
    // table合并列
    objectSpanMethod({ row, column, rowIndex, columnIndex }) {
      if (columnIndex == 0) {
        let data = this.tableData //拿到当前table中数据
        let cellValue = row[column.property] //当前位置的值
        let noSortArr = ['num1', 'num2', 'num3', 'num4', 'num5', 'num6'] //不需要合并的字段（不进行合并行的prop）
        if (cellValue && !noSortArr.includes(column.property)) {
          let prevRow = data[rowIndex - 1] //获取到上一条数据
          let nextRow = data[rowIndex + 1] //下一条数据
          if (prevRow && prevRow[column.property] === cellValue) {
            //当有上一条数据，并且和当前值相等时
            return { rowspan: 0, colspan: 0 }
          } else {
            let countRowspan = 1
            while (nextRow && nextRow[column.property] === cellValue) {
              //当有下一条数据并且和当前值相等时,获取新的下一条
              nextRow = data[++countRowspan + rowIndex]
            }
            if (countRowspan > 1) {
              return { rowspan: countRowspan, colspan: 1 }
            }
          }
        }
      }
    },
    //合并方法
    // objectSpanMethod({ row, column, rowIndex, columnIndex }) {
    //   console.log("!!!!!",row,column,rowIndex,columnIndex);
    //   if (columnIndex == 1) {
    //     //合并相同的名字
    //     let nameSpan = this.getSpanNumber(this.tableData, 'name')
    //     return {
    //       rowspan: nameSpan[rowIndex],
    //       colspan: 1,
    //     }
    //   }
    // },
    //获取要合并的行数
    getSpanNumber(data, prop) {
      //data要处理的数组，prop要合并的属性，比如name

      //数组的长度，有时候后台可能返回个null而不是[]
      let length = Array.isArray(data) ? data.length : 0
      if (length > 0) {
        //用于标识位置
        let position = 0
        //用于对比的数据
        let temp = data[0][prop]
        //要返回的结果
        let result = [1]
        //假设数据是AABCC，我们的目标就是返回20120
        for (let i = 1; i < length; i++) {
          if (data[i][prop] == temp) {
            //标识位置的数据加一
            result[position] += 1
            //当前位置添0
            result[i] = 0
          } else {
            //不相同时，修改标识位置，该位置设为1，修改对比值
            position = i
            result[i] = 1
            temp = data[i][prop]
          }
        }
        //返回结果
        return result
      } else {
        return [0]
      }
    },

    handleClose(done) {
      done()
    },

    handleSelectionChange(val) {
      this.ids = []
      val.map((item) => {
        this.ids.push(item.id)
      })
      console.log(this.ids, 'listresultTmp')
    },
    getRowKey(row) {
      return row[this.rowKey]
    },
    handleClick(tab, event) {
      console.log(tab, event)
    },

    researcher_qx() {
      this.researcher_show = false
      this.template_show = false
    },
    questionnaire_creation() {
      this.template_show = true
    },
    release(value) {
      console.log(value)
      this.fb_researcher = ''
      this.fb_researcher_id = value
      this.researcher_show = true
    },

    indexAdd(index) {
      // console.log(index)
      const page = this.pageNo // 当前页码
      const pagesize = 10 // 每页条数
      return index + 1 + (page - 1) * pagesize
    },
    handleCurrentChange(val) {
      this.pageNo = val
      this.bkbQuestionnaireInquiry_list()
    },
    handleCurrentChangeTem(val) {
      this.pageNoTem = val
      this.searchTem()
    },
    // 查询题目
    queryQuestionGroups(id) {
      // console.log(id,'id');
      // id = '1656222258321121281'
      httpAction(`/mission/bkbQuestionnaireQuestions/queryQuestionGroups?id=${id}`, {}, 'get').then((res) => {
        if (res.success) {
          this.questionGroups = res.result
          console.log('this.questionGroups', this.questionGroups, res)
        } else {
          this.$message.warning(res.message)
        }
      })
    },

    regionalQueryTable() {
      httpAction(
        `/mission/bkbQuestionnaireInquiry/dateExcel?region=${this.fullAddress}`,
        { id: this.myId },
        'POST'
      ).then((res) => {
        if (res.code == 200) {
          if (res.message == '该地区没有问卷信息') {
            this.tableData = []
            this.$message.warning('该地区没有问卷信息')
          } else {
            this.tableData = res.result
          }

          // console.log(res.result, 'id查询得到')
        }
      })
    },
    regionalQuery() {
      //柱状图
      httpAction(
        `/mission/bkbQuestionnaireInquiry/histogram?region=${this.fullAddress2}`,
        { id: this.myId },
        'post'
      ).then((res) => {
        if (res.code == 200) {
          // this.tableData=res.result
          console.log(res.result, '柱状图')
          res.result.forEach((v) => {
            // console.log(v.legendData, 'aaa')
            v.list = v.list.map(Number)
          })
          let temp = res.result

          temp.forEach((item, index) => {
            let count = 0
            let tempList = []
            item.list.forEach((item2) => {
              count += item2
            })
            item.list.forEach((item3) => {
              tempList.push(((item3 / count) * 100).toFixed(2))
            })
            temp[index].list = JSON.parse(JSON.stringify(tempList))
            console.log(count, tempList, 'kkk')
          })

          // this.echartsList = res.result
          this.echartsList = JSON.parse(JSON.stringify(temp))
          console.log(this.echartsList, '柱状图123')
          // this.$refs['echart'].getChart()
          this.barRe = false
          setTimeout(() => {
            this.barRe = true
          }, 0)
          // // this.queryById=res.result
        }
      })
      //饼图数据
      httpAction(
        `/mission/bkbQuestionnaireInquiry/pieChart?region=${this.fullAddress2}`,
        { id: this.myId },
        'post'
      ).then((res) => {
        if (res.code == 200) {
          // this.tableData=res.result
          console.log(res.result, '饼图')
          if (res.message == '该地区没有问卷信息') {
            this.pieChartData = []
            this.$message.warning('该地区没有问卷信息')
          } else {
            res.result.forEach((v) => {
              console.log(v.bean, 'bing')
              v.bean.map((item) => {
                let obj = {
                  name: item.selected_name,
                  value: item.count_column,
                }
                v.bean.push(obj)
              })
            })
          }
          this.pieChartData = res.result
        }
      })
    },
    resetRegion() {
      this.selected = {
        province: '',
        city: '',
        area: '',
      }
      this.selected2 = {
        province: '',
        city: '',
        area: '',
      }
      this.v_selected = false
      setTimeout(() => {
        this.v_selected = true
      }, 0)
      // this.$router.go(0)
      // this.indexStyle = ''
      httpAction(`/mission/bkbQuestionnaireInquiry/dateExcel`, { id: this.myId }, 'POST').then((res) => {
        if (res.code == 200) {
          this.tableData = res.result
          // console.log(res.result, 'id查询得到')
        }
      })

      //柱状图
      httpAction(`/mission/bkbQuestionnaireInquiry/histogram`, { id: this.myId }, 'post').then((res) => {
        if (res.code == 200) {
          // this.tableData=res.result
          console.log(res.result, '柱状图')
          res.result.forEach((v) => {
            // console.log(v.legendData, 'aaa')
            v.list = v.list.map(Number)
          })
          let temp = res.result

          temp.forEach((item, index) => {
            let count = 0
            let tempList = []
            item.list.forEach((item2) => {
              count += item2
            })
            item.list.forEach((item3) => {
              tempList.push(((item3 / count) * 100).toFixed(2))
            })
            temp[index].list = JSON.parse(JSON.stringify(tempList))
            console.log(count, tempList, 'kkk')
          })

          // this.echartsList = res.result
          // this.$nextTick(() => {})
          this.echartsList = JSON.parse(JSON.stringify(temp))
          console.log(this.echartsList, '柱状图123')
          // this.$refs['echart'].getChart()
          this.barRe = false
          setTimeout(() => {
            this.barRe = true
          }, 0)
        }
      })
      //饼图数据
      httpAction(`/mission/bkbQuestionnaireInquiry/pieChart`, { id: this.myId }, 'post').then((res) => {
        if (res.code == 200) {
          // this.tableData=res.result
          console.log(res.result, '饼图')
          res.result.forEach((v) => {
            console.log(v.bean, 'bing')
            v.bean.map((item) => {
              let obj = {
                name: item.selected_name,
                value: item.count_column,
              }
              v.bean.push(obj)
            })
          })
          this.pieChartData = res.result
        }
      })
    },
  },
}
</script>
<style lang="less" scoped>
.content {
  background: #fff;

  .projectCost {
    z-index: 999;
    margin-left: 40px;

    .container {
      display: flex;
      width: 680px;
      height: 240px;
      background-size: 100% 100%;

      // background-image: url('../../../assets/images/projectTest/costDetail.png');
      .wrapper {
        margin-top: 20px;
        width: 340px;
        height: 180px;
        border-right: 1px solid #0b61b3;

        .roseChart {
          width: 260px;
          height: 180px;
        }
      }
    }
  }

  .wjsjck {
    .image {
      display: flex;
      justify-content: center;
      // width: 300px;
      height: 200px;
      margin: 0 auto;
      margin-top: 20px;

      img {
        // width: 70%;
        height: 200px;
        object-fit: scale-down;
      }
    }

    .title {
      width: 600px;
      margin: 0 auto;
      font-size: 24px;
      font-family: 700;
      text-align: center;
      margin-top: 25px;
    }

    .tip {
      width: 80%;
      margin: 0 auto;
      margin-top: 20px;
      line-height: 30px;
    }

    .select {
      width: 80%;
      margin: 30px auto;
      // padding-left: 50px;
      display: flex;
      align-items: center;
      // margin-top: 20px;

      .el-select {
        width: 700px;
      }
      .city {
        display: flex;
        align-items: center;
        // width: 740px;
        margin-right: 10px;
        height: 52px !important;
        line-height: 38px;
        ::v-deep.j-area-linkage {
          // height: 52px !important;
          width: 680px !important;
          ::v-deep.area-cascader-wrap {
            // height: 52px !important;
          }
          .area-selected-trigger {
            height: 38px !important;
            line-height: 22px;
          }
        }
      }
    }
    .switch {
      margin: 0 auto;
      width: 940px;
      display: flex;
      justify-content: flex-end;
    }
    .pie-name {
      text-align: center;
      font-size: 22px;
      font-weight: 550;
    }
    #pie {
      margin: 0 auto;
      margin-top: 30px;
    }
  }

  .template-msk {
    position: fixed;
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
    background: rgba(0, 0, 0, 0.5);
    z-index: 999;

    .box {
      width: 70%;
      height: 2000px;
      background: #fff;
      padding: 30px;
      position: absolute;
      left: 50%;
      margin-left: -35%;
      top: 50%;
      margin-top: -500px;
      overflow: scroll;

      .image {
        width: 70%;
        height: 200px;
        margin: 0 auto;
        margin-top: 20px;

        img {
          width: 70%;
          height: 200px;
          object-fit: scale-down;
        }
      }

      .title {
        width: 600px;
        margin: 0 auto;
        font-size: 24px;
        font-family: 700;
        text-align: center;
        margin-top: 25px;
      }

      .tip {
        width: 80%;
        margin: 0 auto;
        margin-top: 20px;
        line-height: 30px;
      }

      .select {
        width: 80%;
        margin: 0 auto;
        display: flex;
        align-items: center;
        margin-top: 20px;

        .el-select {
          width: 700px;
        }
      }

      #pie {
        margin: 0 auto;
        margin-top: 30px;
      }
    }

    .msk {
      width: 400px;
      height: 240px;
      background: #fff;
      position: absolute;
      left: 50%;
      margin-left: -200px;
      top: 50%;
      margin-top: -120px;

      .mian-btn {
        width: 400px;
        display: flex;
        align-items: center;

        .btn {
          cursor: pointer;
          width: 100px;
          height: 35px;
          line-height: 35px;
          text-align: center;
          border: 1px solid #000;

          margin-left: 20px;
          margin-top: 20px;
        }
      }

      .btns {
        display: flex;
        align-items: center;
        justify-content: flex-end;
        margin-right: 30px;
        margin-top: 40px;
      }

      .iocn {
        display: flex;
        justify-content: flex-end;
        font-size: 20px;
        margin-right: 20px;
      }

      .name {
        font-size: 18px;
        font-weight: 600;
        text-align: center;
        margin-top: 20px;
      }
    }
  }

  .researcher {
    position: fixed;
    top: 0;
    left: 0;
    right: 0;
    bottom: 0;
    background: rgba(0, 0, 0, 0.5);
    z-index: 999;

    .msk {
      width: 400px;
      height: 240px;
      background: #fff;
      position: absolute;
      left: 50%;
      margin-left: -200px;
      top: 50%;
      margin-top: -120px;

      .btn {
        display: flex;
        align-items: center;
        justify-content: flex-end;
        margin-right: 30px;
      }

      .user {
        width: 300px;
        margin: 0 auto;
        margin: 30px;
      }

      .name {
        text-align: center;
        margin-top: 20px;
      }

      .iocn {
        cursor: pointer;
        display: flex;
        justify-content: flex-end;
        font-size: 20px;
        margin-right: 20px;
      }
    }
  }

  .header {
    display: flex;
    align-items: center;

    .btn {
      display: flex;
      align-items: center;
      justify-content: flex-end;
      margin-left: 100px;

      .query {
        margin-right: 15px;
      }
    }

    // justify-content: space-between;
    .wjinput {
      // width: 400px;
      display: flex;
      align-items: center;
      margin-left: 30px;

      .title {
        width: 100px;
      }
    }
  }

  .tb {
    margin-top: 30px;
  }

  .block {
    display: flex;
    justify-content: flex-end;
    margin-top: 40px;
  }
}

.box-card {
  width: 100%;
}
.table {
  .pagination {
    margin: 20px 320px 0 0;
    display: flex;
    justify-content: right;
    // margin-right: 300px;
  }
}
.mian-btn {
  // width: 400px;
  display: flex;
  align-items: center;
  justify-content: center;
  height: 100px;
}
.btns {
  margin-left: 250px;
}

.daochu {
  display: flex;
  justify-content: left;
  margin-left: 800px;
}
.fengmiantu {
  height: 80px;
  img {
    height: 80px;
  }
}

.inputXianShi {
  .inputNum {
    margin: 20px auto;
    width: 120px;
    font-size: 16px;
  }
}
</style>