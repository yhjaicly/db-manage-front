<template>
  <a-card :bordered="false">
    <!-- 查询区域 -->
    <div class="table-page-search-wrapper">
      <a-form layout="inline" @keyup.enter.native="searchQuery">
        <a-row :gutter="24">
          <a-col :xl="6" :lg="7" :md="8" :sm="24">
            <a-form-item label="字段名称">
              <a-input placeholder="请输入字段名称" v-model="queryParam.fieldName"></a-input>
            </a-form-item>
          </a-col>
          <a-col :xl="6" :lg="7" :md="8" :sm="24">
            <a-form-item label="字段中文名">
              <a-input placeholder="请输入字段中文名" v-model="queryParam.fieldNameCn"></a-input>
            </a-form-item>
          </a-col>
          <a-col :xl="6" :lg="7" :md="8" :sm="24">
            <span style="float: left;overflow: hidden;" class="table-page-search-submitButtons">
              <a-button type="primary" @click="searchQuery" icon="search">查询</a-button>
              <a-button type="primary" @click="searchReset" icon="reload" style="margin-left: 8px">重置</a-button>
              <a @click="handleToggleSearch" style="margin-left: 8px">
                {{ toggleSearchStatus ? '收起' : '展开' }}
                <a-icon :type="toggleSearchStatus ? 'up' : 'down'"/>
              </a>
            </span>
          </a-col>
        </a-row>
      </a-form>
    </div>
    <!-- 查询区域-END -->

    <!-- 操作按钮区域 -->
    <div class="table-operator">
      <a-button @click="handleAdd" type="primary" icon="plus">新增</a-button>
      <a-button type="primary" icon="download" @click="handleExportXls('字段管理')">导出</a-button>
      <a-upload name="file" :showUploadList="false" :multiple="false" :headers="tokenHeader" :action="importExcelUrl"
                @change="handleImportExcel">
        <a-button type="primary" icon="import">导入</a-button>
      </a-upload>
      <!-- 高级查询区域 -->
      <j-super-query :fieldList="superFieldList" ref="superQueryModal"
                     @handleSuperQuery="handleSuperQuery"></j-super-query>
      <a-dropdown v-if="selectedRowKeys.length > 0">
        <a-menu slot="overlay">
          <a-menu-item key="1" @click="batchDel">
            <a-icon type="delete"/>
            删除
          </a-menu-item>
        </a-menu>
        <a-button style="margin-left: 8px"> 批量操作
          <a-icon type="down"/>
        </a-button>
      </a-dropdown>
        <a-button @click="downloadSysFile" type="primary" icon="download">下载</a-button>
        <a-progress :percent="100" :successPercent="parseInt(fake.progress*100)" />
    </div>

    <!-- table区域-begin -->
    <div>
      <div class="ant-alert ant-alert-info" style="margin-bottom: 16px;">
        <i class="anticon anticon-info-circle ant-alert-icon"></i> 已选择 <a
          style="font-weight: 600">{{ selectedRowKeys.length }}</a>项
        <a style="margin-left: 24px" @click="onClearSelected">清空</a>
      </div>

      <a-table
          ref="table"
          size="middle"
          :scroll="{x:true}"
          bordered
          rowKey="id"
          :columns="columns"
          :dataSource="dataSource"
          :pagination="ipagination"
          :loading="loading"
          :rowSelection="{selectedRowKeys: selectedRowKeys, onChange: onSelectChange}"
          class="j-table-force-nowrap"
          @change="handleTableChange"
      >
        <!--字典弹窗-->
        <a slot="fieldId" slot-scope="text" @click="info(text)">{{ text }}</a>

        <template slot="htmlSlot" slot-scope="text">
          <div v-html="text"></div>
        </template>
        <template slot="imgSlot" slot-scope="text">
          <span v-if="!text" style="font-size: 12px;font-style: italic;">无图片</span>
          <img v-else :src="getImgView(text)" height="25px" alt=""
               style="max-width:80px;font-size: 12px;font-style: italic;"/>
        </template>
        <template slot="fileSlot" slot-scope="text">
          <span v-if="!text" style="font-size: 12px;font-style: italic;">无文件</span>
          <a-button
              v-else
              :ghost="true"
              type="primary"
              icon="download"
              size="small"
              @click="downloadFile(text)">
            下载
          </a-button>
        </template>

        <span slot="action" slot-scope="text, record">
          <a @click="handleEdit(record)">编辑</a>
          <a-divider type="vertical"/>
          <a-dropdown>
            <a class="ant-dropdown-link">更多 <a-icon type="down"/></a>
            <a-menu slot="overlay">
              <a-menu-item>
                <a @click="handleDetail(record)">详情</a>
              </a-menu-item>
              <a-menu-item>
                <a-popconfirm title="确定删除吗?" @confirm="() => handleDelete(record.id)">
                  <a>删除</a>
                </a-popconfirm>
              </a-menu-item>
            </a-menu>
          </a-dropdown>
        </span>
      </a-table>
    </div>

    <tb-field-manager-modal ref="modalForm" @ok="modalFormOk"></tb-field-manager-modal>
  </a-card>
</template>

<script>

import '@/assets/less/TableExpand.less'
import {mixinDevice} from '@/utils/mixin'
import {JeecgListMixin} from '@/mixins/JeecgListMixin'
import TbFieldManagerModal from './modules/TbFieldManagerModal'
import {getDictDtl} from "@api/getDictForSearch";
import FakeProgress from "fake-progress";

export default {
  name: 'TbFieldManagerList',
  mixins: [JeecgListMixin, mixinDevice],
  components: {
    TbFieldManagerModal
  },
  data() {
    return {
        fake: new FakeProgress({
            timeConstant: 6000,
            autoStart: false
        }),
      description: '字段管理管理页面',
      // 表头
      columns: [
        {
          title: '#',
          dataIndex: '',
          key: 'rowIndex',
          width: 60,
          align: "center",
          customRender: function (t, r, index) {
            return parseInt(index) + 1;
          }
        },
        {
          title: '字段名称',
          align: "center",
          dataIndex: 'fieldName',
          scopedSlots: {customRender: 'fieldName'},
        },
        {
          title: '字段是否是字典',
          align: "center",
          dataIndex: 'fieldIsDictFlag_dictText'
        },
        {
          title: '字段中文名',
          align: "center",
          dataIndex: 'fieldNameCn'
        },
        {
          title: '字段长度',
          align: "center",
          dataIndex: 'fieldLength'
        },
        {
          title: '字段关联字典',
          align: "center",
          dataIndex: 'fieldId',
          scopedSlots: {customRender: 'fieldId'},
        },
        {
          title: '操作',
          dataIndex: 'action',
          align: "center",
          fixed: "right",
          width: 147,
          scopedSlots: {customRender: 'action'}
        }
      ],
      url: {
        list: "/tbFieldManager/tbFieldManager/list",
        delete: "/tbFieldManager/tbFieldManager/delete",
        deleteBatch: "/tbFieldManager/tbFieldManager/deleteBatch",
        exportXlsUrl: "/tbFieldManager/tbFieldManager/exportXls",
        importExcelUrl: "tbFieldManager/tbFieldManager/importExcel",
      },
      superFieldList: [],
    }
  },
  created() {
    this.getSuperFieldList();
  },
  computed: {
    importExcelUrl: function () {
      return `${window._CONFIG['domianURL']}/${this.url.importExcelUrl}`;
    },
  },
  methods: {

    initDictConfig() {
    },
    getSuperFieldList() {
      let fieldList = [];
      fieldList.push({type: 'string', value: 'fieldName', text: '字段名称', dictCode: ''})
      fieldList.push({type: 'string', value: 'fieldIsDictFlag', text: '字段是否是字典', dictCode: 'is_dict'})
      fieldList.push({type: 'string', value: 'fieldNameCn', text: '字段中文名', dictCode: ''})
      fieldList.push({type: 'string', value: 'fieldLength', text: '字段长度', dictCode: ''})
      fieldList.push({type: 'string', value: 'fieldId', text: '字段关联字典', dictCode: ''})
      this.superFieldList = fieldList
    },
    //显示模态框
    async info(record) {
      let dictDtls = []
      const h = this.$createElement
      let dictDtl = await getDictDtl(record)
      for (let dtlKey in dictDtl) {
        dictDtls.push(h('p', '字典子项为：' + dictDtl[dtlKey].value + '子项名称为：' + dictDtl[dtlKey].text))
      }
      if (dictDtls.length ===0){
        dictDtls.push(h('p', '无字典子项！！！'))
      }
      this.$info({
        title: '字典明细',
        content: h('div', {}, dictDtls),
        destroyOnClose: true,
        onOk() {
        },
      })
    },
      // 下载进度条
      downloadSysFile(){
        this.fake.start();
        // 从后端获取进度，获取值为100时，结束下载



        this.fake.end();
      }
  }
}
</script>
<style scoped>
@import '~@assets/less/common.less';
</style>