<template>
  <div
    v-loading="saveLoading"
    class="inner-container"
    :element-loading-text="saveText"
  >
    <!-- <el-scrollbar class="data-set-scrollbar"> -->
    <div class="header">
      <el-page-header class="db-el-page-header">
        <template slot="content">
          <div class="page-header">
            <div class="page-header-left">
              {{ !isEdit ? '原始数据集详情' : dataForm.id ? '编辑原始数据集' : '新增原始数据集' }}
            </div>
            <div class="page-header-right">
              <el-button
                class="db-el-button-default"
                @click="openNewWindow('https://www.yuque.com/chuinixiongkou/bigscreen/ry3ggnrts7q3ro1g')"
              >
                帮助
              </el-button>
              <el-button
                v-if="isEdit"
                type="primary"
                @click="save('form')"
              >
                保存
              </el-button>
              <el-button
                class="db-el-button-default"
                @click="goBack"
              >
                返回
              </el-button>
            </div>
          </div>
        </template>
      </el-page-header>
    </div>
    <el-row style="margin: 16px 16px 0;">
      <el-col :span="isEdit ? 16 : 24">
        <el-form
          ref="form"
          :model="dataForm"
          :rules="rules"
          label-width="120px"
          style="padding: 16px 16px 0;"
        >
          <el-row :gutter="20">
            <el-col :span="12">
              <el-form-item
                label="数据集名称"
                prop="name"
              >
                <el-input
                  v-model="dataForm.name"
                  class="db-el-input"
                  clearable
                  :disabled="!isEdit"
                />
              </el-form-item>
            </el-col>
            <el-col :span="12">
              <el-form-item
                label="分组"
                prop="typeId"
              >
                <el-select
                  ref="selectParentName"
                  v-model="dataForm.typeId"
                  class="db-el-select"
                  popper-class="db-el-select"
                  placeholder="请选择分组"
                  clearable
                  :disabled="!isEdit"
                  filterable
                  :filter-method="selectorFilter"
                  @clear="clearType"
                  @visible-change="setCurrentNode"
                >
                  <el-option
                    style="height: auto;padding: 0;"
                    :label="typeName"
                    :value="dataForm.typeId"
                  >
                    <div>
                      <el-tree
                        ref="categorySelectTree"
                        :data="categoryData"
                        node-key="id"
                        :indent="0"
                        :props="{ label: 'name', children: 'children' }"
                        :default-expand-all="true"
                        :highlight-current="true"
                        :expand-on-click-node="false"
                        class="db-el-tree"
                        :filter-node-method="treeFilter"
                        @node-click="selectParentCategory"
                      >
                        <span
                          slot-scope="{ data }"
                          class="custom-tree-node"
                        >
                          <span>
                            <i
                              :class="data.children && data.children.length ? 'el-icon el-icon-folder' : 'el-icon el-icon-document'"
                            />
                            {{ data.name }}
                          </span>
                        </span>
                      </el-tree>
                    </div>
                  </el-option>
                </el-select>
              </el-form-item>
            </el-col>
          </el-row>
          <el-row :gutter="20">
            <el-col :span="12">
              <el-form-item
                label="数据源"
                prop="sourceId"
              >
                <el-select
                  v-model="dataForm.sourceId"
                  class="db-el-select"
                  popper-class="db-el-select"
                  clearable
                  filterable
                  :disabled="!isEdit"
                  @change="setSource"
                >
                  <el-option
                    v-for="source in sourceList"
                    :key="source.id"
                    :label="source.sourceName"
                    :value="source.id"
                  />
                </el-select>
              </el-form-item>
            </el-col>
            <el-col :span="12">
              <el-form-item
                label="关联原始表"
                prop="tableName"
              >
                <el-select
                  v-model="dataForm.tableName"
                  v-loading="selectorLoading"
                  element-loading-spinner="el-icon-loading"
                  class="db-el-select"
                  popper-class="db-el-select"
                  clearable
                  filterable
                  :disabled="!isEdit"
                  @change="setTable"
                >
                  <el-option-group label="表">
                    <el-option
                      v-for="table in tableList"
                      :key="table.name"
                      :label="table.name"
                      :value="table.name"
                    />
                  </el-option-group>
                  <el-option-group label="视图">
                    <el-option
                      v-for="table in viewList"
                      :key="table.name"
                      :label="table.name"
                      :value="table.name"
                    />
                  </el-option-group>
                </el-select>
              </el-form-item>
            </el-col>
          </el-row>
          <el-row :gutter="20">
            <el-col :span="12">
              <el-form-item
                label="输出字段"
                prop="fieldInfo"
              >
                <el-select
                  v-model="dataForm.fieldInfo"
                  class="selectStyle"
                  popper-class="selectStyle"
                  placeholder="请选择字段（为空时默认选择全部字段）"
                  clearable
                  filterable
                  multiple
                  collapse-tags
                  :disabled="!isEdit"
                  @change="setFields"
                >
                  <el-option
                    v-if="fieldList.length"
                    label="全选"
                    value="全选"
                  >
                    <el-checkbox
                      v-model="isSelectAll"
                      @click.prevent.native
                    >
                      全选
                    </el-checkbox>
                  </el-option>
                  <el-option
                    v-for="field in fieldList"
                    :key="field.columnName"
                    :label="field.columnName"
                    :value="field.columnName"
                  >
                    <el-checkbox
                      v-model="field.isCheck"
                      @click.prevent.native
                    >
                      {{ field.columnName }}
                    </el-checkbox>
                  </el-option>
                </el-select>
              </el-form-item>
            </el-col>
            <el-col :span="12">
              <el-form-item
                label="是否去重"
                prop="repeatStatus"
              >
                <el-radio-group
                  v-model="dataForm.repeatStatus"
                  class="db-radio-wrap"
                  :disabled="!isEdit"
                  @input="initData"
                >
                  <el-radio :label="1">
                    是
                  </el-radio>
                  <el-radio :label="0">
                    否
                  </el-radio>
                </el-radio-group>
              </el-form-item>
            </el-col>
          </el-row>
          <el-row :gutter="20">
            <el-col :span="12">
              <el-form-item
                label="描述"
                prop="remark"
              >
                <el-input
                  v-model="dataForm.remark"
                  class="db-el-input"
                  :disabled="!isEdit"
                />
              </el-form-item>
            </el-col>
            <el-col :span="12">
              <el-form-item
                label="数据缓存"
                prop="cache"
              >
                <el-radio-group
                  v-model="dataForm.cache"
                  class="db-el-radio-group"
                >
                  <el-radio :label="1">
                    开启
                  </el-radio>
                  <el-radio :label="0">
                    关闭
                  </el-radio>
                </el-radio-group>
                <el-tooltip
                  class="item"
                  content="开启缓存:会在首次调用该数据集时，将结果缓存，在接下来的十分钟内，若再次被调用则直接返回缓存中的数据，注意：在当前数据集编辑页面缓存不生效"
                  placement="top"
                >
                  <i
                    class="el-icon-warning-outline"
                    style="color: #E3C98C;margin-left: 16px;font-size:14px"
                  />
                </el-tooltip>
              </el-form-item>
            </el-col>
          </el-row>
          <el-row>
            <el-col :span="12">
              <el-form-item
                label="标签"
                prop="labelIds"
              >
                <LabelSelect
                  :dataset-id="datasetId"
                  :id-list="dataForm.labelIds"
                  @commit="(ids) =>{dataForm.labelIds = ids}"
                />
              </el-form-item>
            </el-col>
          </el-row>
        </el-form>
      </el-col>
      <el-col
        v-if="isEdit"
        :span="8"
      >
        <div class="structure">
          <div class="title-style db-title-style">
            输出字段
            <el-button
              type="text"
              style="float: right;border: none;margin-top: -4px;"
              @click="fieldsetVisible = true"
            >
              配置
            </el-button>
          </div>
          <div class="field-wrap db-field-wrap db-scrollbar">
            <div
              v-for="field in structurePreviewList"
              :key="field.fieldName"
              class="field-item"
              @click="fieldsetVisible = true"
            >
              <span>{{ field.fieldName }}</span>&nbsp;<span
                v-show="field.fieldDesc"
                style="color: #909399;"
              >({{
                field.fieldDesc }})</span>
              <el-button
                class="edit_field"
                type="text"
                style="float: right;border: none;margin-top: 2px;"
                @click="fieldsetVisible = true"
              >
                配置
              </el-button>
            </div>
          </div>
        </div>
      </el-col>
    </el-row>
    <div
      v-if="isEdit"
      class="dataPreView"
      style="margin-top: 12px;"
    >
      <div class="result-view">
        数据预览
      </div>
      <div
        v-loading="tableLoading"
        class="db-table-box is-Edit db-scrollbar"
      >
        <el-table
          align="center"
          :data="dataPreviewList"
          max-height="420"
          class="db-el-table db-scrollbar"
        >
          <el-table-column
            v-for="(value, key) in dataPreviewList[0] ? dataPreviewList[0] : noDataTableDisplayFields"
            :key="key"
            :label="key"
            align="center"
            show-overflow-tooltip
            :render-header="renderHeader"
          >
            <template slot-scope="scope">
              <span>{{ scope.row[key] }}</span>
            </template>
          </el-table-column>
        </el-table>
      </div>
      <div class="db-pagination">
        <el-pagination
          class="db-el-pagination"
          popper-class="db-el-pagination"
          :current-page="current"
          :page-sizes="[10, 20, 50, 100]"
          :page-size="size"
          :total="totalCount"
          background
          prev-text="上一页"
          next-text="下一页"
          layout="total, prev, pager, next,sizes"
          @size-change="sizeChangeHandle"
          @current-change="currentChangeHandle"
        />
      </div>
    </div>
    <el-dialog
      title="提示"
      :visible.sync="fieldDescVisible"
      width="420px"
      append-to-body
      :close-on-click-modal="false"
      custom-class="fieldDescCheck"
      class="db-dialog-wrap db-el-dialog"
    >
      <p style="color:var(--db-el-text);line-height: 24px;padding-left: 10px;display: flex;">
        <i
          class="el-icon-warning"
          style="color: #E6A23C;font-size: 24px;margin-right: 5px;"
        />存在字段描述信息为空，请确认
      </p>
      <span
        slot="footer"
        class="dialog-footer"
      >
        <el-button
          class="db-el-button-default"
          @click="fieldDescFill"
        >使用字段名填充</el-button>
        <el-button
          class="db-el-button-default"
          @click="fieldDescEdit"
        >进入编辑</el-button>
        <el-button
          type="primary"
          @click="toSave"
        >继续保存</el-button>
      </span>
    </el-dialog>
    <!-- 字段填充 -->
    <el-dialog
      title="输出字段配置"
      :visible.sync="fieldsetVisible"
      width="1000px"
      append-to-body
      :close-on-click-modal="false"
      :before-close="cancelField"
      class="db-dialog-wrap db-el-dialog"
    >
      <div class="db-table-box">
        <el-table
          :data="structurePreviewListCopy"
          :border="true"
          align="center"
          class="db-el-table"
        >
          <el-empty slot="empty" />
          <el-table-column
            align="left"
            show-overflow-tooltip
            prop="fieldName"
            label="字段值"
          />
          <el-table-column
            align="center"
            show-overflow-tooltip
            prop="fieldType"
            label="字段类型"
          />
          <el-table-column
            align="center"
            prop="fieldDesc"
            label="字段描述"
          >
            <template slot-scope="scope">
              <el-input
                v-if="isEdit"
                v-model="scope.row.fieldDesc"
                size="small"
                class="labeldsc db-el-input"
              />
              <span v-else>{{ scope.row.fieldDesc }}</span>
            </template>
          </el-table-column>
          <!-- <el-table-column
            align="center"
            prop="orderNum"
            label="字段排序"
            sortable
          >
            <template slot-scope="scope">
              <el-input
                v-if="isEdit"
                v-model="scope.row.orderNum"
                size="small"
                class="labeldsc db-el-input"
              />
              <span v-else>{{ scope.row.orderNum }}</span>
            </template>
          </el-table-column> -->
          <el-table-column
            align="center"
            prop="sourceTable"
            label="字段来源"
          />
          <!-- 添加一个插槽，供其他人可扩展表格列，并把表格列的数据返回出去 -->
          <slot name="output-field-table-column" />
        </el-table>
      </div>
      <span
        slot="footer"
        class="dialog-footer"
      >
        <el-button
          class="db-el-button-default"
          @click="cancelField"
        >
          取消
        </el-button>
        <el-button
          type="primary"
          @click="setField"
        >
          确定
        </el-button>
      </span>
    </el-dialog>
    <!-- </el-scrollbar> -->
  </div>
</template>

<script>
import LabelSelect from 'dashPackages/DataSetLabelManagement/src/LabelSelect.vue'
import {
  getCategoryTree,
  nameCheckRepeat,
  datasetExecuteTest,
  getDataset, datasetUpdate, datasetAdd
} from 'dashPackages/js/utils/datasetConfigService'
import { datasourceList, getSourceTable, getSourceView, getTableFieldList } from 'dashPackages/js/utils/dataSourceService'
import _ from 'lodash'
export default {
  name: 'OriginalEditForm',
  components: {
    LabelSelect
  },
  props: {
    isEdit: {
      type: Boolean,
      default: false
    },
    datasetId: {
      type: String,
      default: null
    },
    datasetName: {
      type: String,
      default: ''
    },
    typeId: {
      type: String,
      default: ''
    },
    appCode: {
      type: String,
      default: ''
    }
  },
  data () {
    const validateName = (rule, value, callback) => {
      nameCheckRepeat({
        id: this.datasetId,
        name: value,
        moduleCode: this.appCode
      }).then((res) => {
        if (res) {
          callback(new Error('数据集名称已存在'))
        } else {
          callback()
        }
      })
    }
    return {
      dataForm: {
        id: '',
        name: '',
        typeId: '',
        datasetType: 'original',
        remark: '',
        cache: 0,
        // 关联的标签id
        labelIds: [],
        // 以下为config信息
        sourceId: '',
        repeatStatus: 0,
        tableName: '',
        fieldInfo: [],
        fieldDesc: {},
        fieldList: []
      },
      rules: {
        name: [
          { required: true, message: '数据集名称不能为空', trigger: 'blur' },
          { validator: validateName, trigger: 'blur' }
        ],
        sourceId: [
          { required: true, message: '请选择数据源', trigger: 'blur' }
        ],
        tableName: [
          { required: true, message: '请选择原始表', trigger: 'change' }
        ],
        repeatStatus: [
          { required: true, message: '请选择是否去重', trigger: 'blur' }
        ],
        typeId: [
          { required: true, message: '请选择分组', trigger: 'blur' }
        ]
      },
      typeName: '',
      // 分组分类树
      categoryData: [],
      // 数据源列表
      sourceList: [],
      // 表列表
      tableList: [],
      // 视图列表
      viewList: [],
      // 字段列表
      fieldList: [],
      isSelectAll: false,
      activeName: 'data',
      // 预览数据
      dataPreviewList: [],
      // 字段结构
      structurePreviewList: [],
      // 字段结构副本
      structurePreviewListCopy: [],
      tableLoading: false,
      fieldDescVisible: false,
      fieldsetVisible: false,
      saveLoading: false,
      saveText: '',
      totalCount: 0,
      currentCount: 0,
      current: 1,
      size: 10,
      selectorLoading: false,
      oldStructurePreviewList: [],
      isInit: false
    }
  },
  computed: {
    noDataTableDisplayFields () {
      // 表格列对象
      const tableColumnObject = {}
      this.structurePreviewList.forEach(item => {
        tableColumnObject[item.fieldName] = ''
      })
      return tableColumnObject
    }
  },
  watch: {
    'dataForm.fieldInfo': {
      handler (value) {
        try {
          this.setCheck()
          const fieldDescMap = {}
          this.fieldList.forEach((item) => {
            if (value.length !== 0 && !value.includes(item.columnName)) {
              return
            }
            fieldDescMap[item.columnName] = item.columnComment
          })
          // 与this.dataForm.fieldDesc合并，columnName相同的，取this.dataForm.fieldDesc中的值
          const fieldDescMapNew = { ...fieldDescMap, ...this.dataForm.fieldDesc }
          this.getPreViewData(fieldDescMapNew)
        } catch (error) {
          console.error(error)
        }
      },
      deep: true,
      immediate: true
    },
    fieldList: {
      handler (val) {
        if (val && val.length) {
          try {
            this.setCheck()
            const fieldDescMap = {}
            this.fieldList.forEach((item) => {
              if (this.dataForm.fieldInfo.length !== 0 && !this.dataForm.fieldInfo.includes(item.columnName)) {
                return
              }
              fieldDescMap[item.columnName] = item.columnComment
            })
            // 与this.dataForm.fieldDesc合并，columnName相同的，取this.dataForm.fieldDesc中的值
            const fieldDescMapNew = { ...fieldDescMap, ...this.dataForm.fieldDesc }
            this.getPreViewData(fieldDescMapNew)
          } catch (error) {
            console.error(error)
          }
        }
      },
      deep: true,
      immediate: true
    }
  },
  mounted () {
    this.init()
  },
  methods: {
    initData () {
      this.getData()
    },
    /**
     * 初始化
     * 1.获取分类树
     * 2.获取数据源列表
     * 3.获取数据集详情
     * 4.获取原始表列表
     * 5.获取字段列表
     */
    async init () {
      // 获取分类树
      this.categoryData = await getCategoryTree({ type: 'dataset', moduleCode: this.appCode })
      // 如果传入了分类id，则设置分类id和分类名称
      if (this.typeId) {
        this.dataForm.typeId = this.typeId
        this.$nextTick(() => {
          try {
            this.typeName = this.$refs.categorySelectTree.getNode(this.dataForm.typeId).data.name
          } catch (error) {
            console.error(error)
          }
        })
      }
      this.queryAllSource()
      if (!this.datasetId) {
        return
      }
      // 获取详情
      getDataset(this.datasetId).then(res => {
        this.dataForm.id = res.id
        this.dataForm.name = res.name
        this.dataForm.typeId = res.typeId
        this.dataForm.remark = res.remark
        this.dataForm.cache = res.cache
        this.dataForm.datasetType = res.datasetType
        this.dataForm.moduleCode = res.moduleCode
        this.dataForm.editable = res.editable
        this.dataForm.sourceId = res.sourceId
        // config 配置
        this.dataForm.tableName = res.config.tableName
        this.dataForm.repeatStatus = res.config.repeatStatus
        this.dataForm.fieldList = res.config.fieldList
        this.dataForm.fieldDesc = res.config.fieldDesc
        this.oldStructurePreviewList = _.cloneDeep(res.config.fieldList)
        // 字段信息，转为数组
        this.dataForm.fieldInfo = res.config.fieldInfo ? res.config.fieldInfo.split(',') : []
        if (this.dataForm.typeId) {
          this.$nextTick(() => {
            try {
              this.typeName = this.$refs.categorySelectTree.getNode(this.dataForm.typeId).data.name
            } catch (error) {
              console.error(error)
            }
          })
        }
        this.isInit = true
        this.queryAllTable()
        this.queryAllField()
      })
    },
    /**
     * 获取预览数据
     */
    getData () {
      let allField = []
      if (this.dataForm.fieldInfo.length === 0) {
        // 从字段列表中取出所有字段
        allField = this.fieldList.map(field => field.columnName)
      }
      const executeParams = {
        dataSourceId: this.dataForm.sourceId,
        script: JSON.stringify({
          fieldInfo: this.dataForm.fieldInfo.length === 0 ? allField : this.dataForm.fieldInfo,
          tableName: this.dataForm.tableName,
          repeatStatus: this.dataForm.repeatStatus
        }),
        // 原始表数据集没有数据集参数
        params: [],
        dataSetType: 'original',
        size: this.size,
        current: this.current
      }
      this.tableLoading = true
      datasetExecuteTest(executeParams).then((data) => {
        if (this.dataForm.fieldList == null) {
          this.dataForm.fieldList = _.cloneDeep(data.structure)
        }
        this.dataPreviewList = data.data.list
        this.totalCount = data.data.totalCount
        this.tableLoading = false
      }).catch(() => {
        this.dataPreviewList = []
        this.totalCount = 0
        this.tableLoading = false
      })
    },
    // /**
    //  * 组装sql
    //  * @returns {string}
    //  */
    // getSql () {
    //   let sql = 'SELECT '
    //   if (this.dataForm.repeatStatus === 1) {
    //     sql += ' DISTINCT '
    //   }
    //   if (this.dataForm.fieldInfo.length > 0) {
    //     sql += this.dataForm.fieldInfo.join(',')
    //   } else {
    //     sql += '*'
    //   }
    //   sql += ` FROM ${this.dataForm.tableName}`
    //   return sql
    // },
    /**
     * 取消字段描述编辑
     */
    cancelField () {
      this.structurePreviewListCopy = _.cloneDeep(this.structurePreviewList)
      this.fieldsetVisible = false
    },
    /**
     * 保存字段描述编辑
     */
    setField () {
      this.structurePreviewList = _.cloneDeep(this.structurePreviewListCopy)
      this.oldStructurePreviewList = _.cloneDeep(this.structurePreviewListCopy)
      this.fieldsetVisible = false
    },
    /**
     * 使用字段名作为字段描述
     */
    fieldDescFill () {
      this.structurePreviewList.forEach(item => {
        if (item.fieldDesc === '') {
          item.fieldDesc = item.fieldName
        }
      })
      this.save('form')
      this.fieldDescVisible = false
    },
    /**
     * 进入字段描述编辑弹窗
     */
    fieldDescEdit () {
      this.fieldDescVisible = false
      this.fieldsetVisible = true
    },
    /**
     * 跳过字段描述编辑直接保存
     */
    toSave () {
      this.save('form', true)
      this.fieldDescVisible = false
    },
    /**
     * 保存数据集
     * @param formName 表单名称
     * @param noCheckToSave 是否不检查直接保存
     */
    save (formName, noCheckToSave = false) {
      if (!this.structurePreviewList.length) {
        this.$message.warning('该原始数据集未生成输出字段，请重新检查')
        return
      }
      if (!noCheckToSave) {
        const temp = this.structurePreviewList.some(item => {
          return item.fieldDesc === ''
        }) // true-存在为空
        if (temp) {
          this.fieldDescVisible = true
          return
        }
      }
      this.$refs[formName].validate((valid) => {
        if (!valid) {
          this.saveLoading = false
          this.saveText = ''
          return false
        }
        // 组装字段描述
        const columnMap = {}
        this.structurePreviewList.forEach(item => {
          columnMap[item.fieldName] = item.fieldDesc
        })
        // 组装保存参数
        const datasetParams = {
          id: this.dataForm.id,
          name: this.dataForm.name,
          typeId: this.dataForm.typeId,
          remark: this.dataForm.remark,
          cache: this.dataForm.cache,
          datasetType: 'original',
          sourceId: this.dataForm.sourceId,
          labelIds: this.dataForm.labelIds,
          config: {
            className: 'com.gccloud.dataset.entity.config.OriginalDataSetConfig',
            sourceId: this.dataForm.sourceId,
            tableName: this.dataForm.tableName,
            fieldInfo: this.dataForm.fieldInfo.length ? this.dataForm.fieldInfo.join(',') : '',
            fieldDesc: columnMap,
            fieldList: this.structurePreviewList,
            repeatStatus: this.dataForm.repeatStatus
          },
          moduleCode: this.appCode,
          editable: this.appCode ? 1 : 0
        }
        this.saveLoading = true
        this.saveText = '正在保存...'
        const saveOriginal = this.dataForm.id ? datasetUpdate : datasetAdd
        saveOriginal(datasetParams).then(res => {
          this.$message.success('保存成功')
          this.$parent.init(false)
          this.$parent.setType = null
          this.saveLoading = false
          this.saveText = ''
          this.goBack()
        }).catch(() => {
          this.$message.error('保存失败')
          this.saveLoading = false
          this.saveText = ''
        })
      })
    },
    /**
     * 获取数据源列表
     */
    queryAllSource () {
      const params = {
        sourceName: '',
        sourceType: '',
        moduleCode: this.appCode
      }
      datasourceList(params).then(res => {
        this.sourceList = res
      })
    },
    /**
     * 选中数据源
     * @param value
     */
    setSource (value) {
      this.dataForm.tableName = ''
      this.dataForm.fieldInfo = []
      this.tableList = []
      this.fieldList = []
      if (!this.dataForm.sourceId) return
      this.queryAllTable()
    },
    /**
     * 获取原始表列表
     * 1.获取表列表
     * 2.获取视图列表
     */
    queryAllTable () {
      getSourceTable(this.dataForm.sourceId).then(res => {
        this.tableList = res
      }).catch(() => {
        this.tableList = []
      })
      this.selectorLoading = true
      getSourceView(this.dataForm.sourceId).then(res => {
        this.selectorLoading = false
        this.viewList = res
      }).catch(() => {
        this.selectorLoading = false
        this.viewList = []
      })
    },
    /**
     * 选中原始表
     * @param value
     */
    async setTable(value) {
      this.fieldList = []
      if (!this.dataForm.tableName) {
        this.dataForm.fieldInfo = []
        return
      }
      await this.queryAllField()
      // dataForm.fieldInfo的清空会触发监听事件，如果在queryAllField之前清空，会导致监听事件中的getPreViewData方法获取不到fieldList
      this.dataForm.fieldInfo = []
    },
    queryAllField() {
      return new Promise((resolve, reject) => {
        getTableFieldList(this.dataForm.sourceId, this.dataForm.tableName)
          .then((data) => {
            const fieldDescMap = {}
            this.fieldList = data.map(field => {
              // field.columnName = '`' + field.columnName + '`'
              fieldDescMap[field.columnName] = field.columnComment
              field.isCheck = false
              if (this.dataForm.fieldInfo.includes(field.columnName)) {
                field.isCheck = true
              }
              return field
            })
            resolve()
          })
          .catch(() => {
            this.fieldList = []
            reject()
          })
      })
    },
    /**
     * 选中字段
     * @param values 选中的字段列表
     */
    setFields (values) {
      if (values.includes('全选')) {
        if (values.length > this.fieldList.length) {
          this.dataForm.fieldInfo = []
        } else {
          this.dataForm.fieldInfo = this.fieldList.map(field => field.columnName)
        }
      }
    },
    /**
     * 设置字段选中状态
     */
    setCheck () {
      this.fieldList.forEach(field => {
        if (this.dataForm.fieldInfo.includes(field.columnName)) {
          field.isCheck = true
        } else {
          field.isCheck = false
        }
      })
      if (this.fieldList.length && this.dataForm.fieldInfo.length === this.fieldList.length) {
        this.isSelectAll = true
      } else {
        this.isSelectAll = false
      }
    },
    /**
     * 获取数据预览
     * @param fieldDescMap 字段描述
     */
    getPreViewData (fieldDescMap) {
      this.dataPreviewList = []
      this.structurePreviewList = []
      this.structurePreviewListCopy = []
      if (!this.dataForm.sourceId || !this.dataForm.tableName) return
      let allField = []
      if (this.dataForm.fieldInfo.length === 0) {
        // 从字段列表中取出所有字段
        allField = this.fieldList.map(field => field.columnName)
      }
      const executeParams = {
        dataSourceId: this.dataForm.sourceId,
        script: JSON.stringify({
          fieldInfo: this.dataForm.fieldInfo.length ? this.dataForm.fieldInfo : allField,
          tableName: this.dataForm.tableName,
          repeatStatus: this.dataForm.repeatStatus
        }),
        // 原始表数据集没有数据集参数
        params: [],
        dataSetType: 'original',
        size: this.size,
        current: this.current
      }
      this.tableLoading = true
      datasetExecuteTest(executeParams).then((data) => {
        this.dataPreviewList = data.data.list
        this.structurePreviewList = data.structure
        if (this.dataPreviewList.length != 0) {
          this.structurePreviewList = []
          Object.keys(this.dataPreviewList[0]).forEach(item => {
            this.structurePreviewList.push(data.structure.filter(x => x.fieldName == item)[0])
          })
        }
        // 如果是初始化
        if (this.isInit) {
          this.structurePreviewList = this.dataForm.fieldList
          this.isInit = false
        } else {
          this.structurePreviewList.forEach(item => {
            if (!item.hasOwnProperty('orderNum')) {
              this.$set(item, 'orderNum', 0)
            }
            if (!item.hasOwnProperty('sourceTable')) {
              this.$set(item, 'sourceTable', this.dataForm.tableName)
            }
            if (!item.hasOwnProperty('fieldDesc')) {
              let fieldDesc = ''
              if (fieldDescMap && fieldDescMap[item.fieldName]) {
                fieldDesc = fieldDescMap[item.fieldName]
              }
              this.$set(item, 'fieldDesc', fieldDesc)
              // this.structurePreviewList 和 this.oldStructurePreviewList 比较，如果旧的数据里fieldDesc有值则重新赋值给新的数据
              this.structurePreviewList.forEach(item => {
                const oldItem = this.oldStructurePreviewList.find(oldItem => oldItem.fieldName === item.fieldName)
                if (oldItem && oldItem.fieldDesc) {
                  item.fieldDesc = oldItem.fieldDesc
                }
              })
            }
          })
        }
        this.structurePreviewListCopy = _.cloneDeep(this.structurePreviewList)
        this.totalCount = data.data.totalCount
        this.currentCount = data.data.currentCount
        this.tableLoading = false
      }).catch((e) => {
        this.dataPreviewList = []
        this.structurePreviewList = []
        this.structurePreviewListCopy = []
        this.totalCount = 0
        this.currentCount = 0
        this.tableLoading = false
      })
    },
    /**
     * 表头添加提示
     */
    renderHeader (h, { column, index }) {
      const labelLong = column.label.length // 表头label长度
      const size = 14 // 根据需要定义标尺，直接使用字体大小确定就行，也可以根据需要定义
      column.minWidth = labelLong * size < 120 ? 120 : labelLong * size // 根据label长度计算该表头最终宽度
      return h('span', { class: 'cell-content', style: { width: '100%' } }, [column.label])
    },
    /**
     * 回到数据集列表页面
     */
    goBack () {
      this.$emit('back')
    },
    /**
     * 清空分类
     */
    clearType () {
      this.typeName = ''
      this.dataForm.typeId = ''
    },
    /**
     * 分类展开高亮
     */
    setCurrentNode ($event) {
      if ($event) {
        const key = this.dataForm.typeId || null
        this.$refs.categorySelectTree.setCurrentKey(key)
      }
    },
    /**
     * 分类选择
     */
    selectParentCategory (value) {
      this.dataForm.typeId = value.id
      this.typeName = value.name
      this.$refs.selectParentName.blur()
    },
    // 每页大小改变触发
    sizeChangeHandle (value) {
      this.size = value
      this.current = 1
      this.getData()
    },
    // 当前页数改变
    currentChangeHandle (value) {
      this.current = value
      this.getData()
    },
    openNewWindow (url) {
      window.open(url, '_blank')
    },
    selectorFilter (value) {
      this.$refs.categorySelectTree.filter(value)
    },
    treeFilter (value, data) {
      if (!value) return true
      return data.name.indexOf(value) !== -1
    }
  }
}
</script>

<style lang="scss"></style>

<style lang="scss" scoped>
@import '../../assets/style/bsTheme.scss';

.data-set-scrollbar {
  height: calc(100vh - 190px);
  overflow-y: auto;
  overflow-x: none;
}

// .tree-box {
//   padding: 0;
//   max-height: 270px;
// }

::v-deep  .el-input__inner {
  width: 100% !important;
}

.page-header {
  display: flex;
  position: relative;

  .page-header-right {
    position: absolute;
    right: 16px;
  }
}

::v-deep  .fieldDescCheck {
  .el-dialog__body {
    height: fit-content !important;
    min-height: unset !important;
  }
}

.title-style {
  padding: 8px 12px;
  background-color: #f6f7fb;
  border-left: 5px solid var(--db-el-color-primary);
  margin: 16px 16px 0 0;
}

.field-wrap {
  max-height: 156px;
  overflow: auto;
  margin-right: 16px;

  .field-item {
    line-height: 32px;
    padding: 0 12px 0 16px;
    cursor: pointer;

    .edit_field {
      display: none;
    }

    &:hover {
      background-color: #f2f7fe;

      .edit_field {
        display: block;
      }
    }
  }
}

::v-deep  .db-table-box.is-Edit .el-table {
  max-height: calc(100vh - 600px) !important;
  // overflow-y: auto !important;

  .el-table__body-wrapper {
    max-height: calc(100vh - 636px) !important;
    overflow-y: auto !important;
  }
}

.result-view {
  font-size: 14px;
  font-weight: 600;
  color: var(--db-el-text);
  position: relative;
  padding: 16px 0;
  padding-left: 12px;
  border-bottom: 1px solid var(--db-background-1);

  &::before {
    content: "";
    height: 14px;
    position: absolute;
    left: 0;
    top: 50%;
    transform: translateY(-50%);
    border-left: 4px solid var(--db-el-color-primary);
  }
}

.db-table-box {
  height: 100% !important;
  margin-bottom: 0 !important;
}

.db-pagination {
  position: relative !important;
  bottom: 0 !important;
  padding: 16px 12px 0 16px !important;
  ::v-deep  .el-input__inner {
    width: 110px !important;
    border: none;
    background: var(--db-el-background-1);
  }
}

// 修改el-select样式 loading 位置
::v-deep  .el-loading-spinner{
  top: 75%;
}

</style>
