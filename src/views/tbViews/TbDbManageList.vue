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
            <a-form-item label="表名称">
              <a-input placeholder="请输入表名称" v-model="queryParam.tbName"></a-input>
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
      <a-button type="primary" icon="download" @click="handleExportXls('数据库字段管理')">导出</a-button>
      <a-upload name="file" :showUploadList="false" :multiple="false" :headers="tokenHeader" :action="importExcelUrl" @change="handleImportExcel">
        <a-button type="primary" icon="import">导入</a-button>
      </a-upload>
      <!-- 高级查询区域 -->
      <j-super-query :fieldList="superFieldList" ref="superQueryModal" @handleSuperQuery="handleSuperQuery"></j-super-query>
      <a-dropdown v-if="selectedRowKeys.length > 0">
        <a-menu slot="overlay">
          <a-menu-item key="1" @click="batchDel"><a-icon type="delete"/>删除</a-menu-item>
        </a-menu>
        <a-button style="margin-left: 8px"> 批量操作 <a-icon type="down" /></a-button>
      </a-dropdown>
    </div>

    <!-- table区域-begin -->
    <div>
      <div class="ant-alert ant-alert-info" style="margin-bottom: 16px;">
        <i class="anticon anticon-info-circle ant-alert-icon"></i> 已选择 <a style="font-weight: 600">{{ selectedRowKeys.length }}</a>项
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
        @change="handleTableChange">

        <template slot="htmlSlot" slot-scope="text">
          <div v-html="text"></div>
        </template>
        <template slot="imgSlot" slot-scope="text">
          <span v-if="!text" style="font-size: 12px;font-style: italic;">无图片</span>
          <img v-else :src="getImgView(text)" height="25px" alt="" style="max-width:80px;font-size: 12px;font-style: italic;"/>
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

          <a-divider type="vertical" />
          <a-dropdown>
            <a class="ant-dropdown-link">更多 <a-icon type="down" /></a>
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

    <tb-db-manage-modal ref="modalForm" @ok="modalFormOk"></tb-db-manage-modal>
  </a-card>
</template>

<script>

import '@/assets/less/TableExpand.less'
import {mixinDevice} from '@/utils/mixin'
import {JeecgListMixin} from '@/mixins/JeecgListMixin'
import TbDbManageModal from './modules/TbDbManageModal'


export default {
    name: 'TbDbManageList',
    mixins:[JeecgListMixin, mixinDevice],
    components: {
      TbDbManageModal
    },
    data () {
      return {
        description: '数据库字段管理管理页面',
        // 表头
        columns: [
          {
            title: '序号',
            dataIndex: '',
            key:'rowIndex',
            width:60,
            align:"center",
            customRender:function (t,r,index) {
              return parseInt(index)+1;
            }
          },
          {
            title:'字段名称',
            align:"center",
            dataIndex: 'fieldName'
          },
          {
            title:'字段类型',
            align:"center",
            dataIndex: 'fieldType'
          },
          {
            title:'字段长度',
            align:"center",
            dataIndex: 'fieldLength'
          },
          {
            title:'主键',
            align:"center",
            dataIndex: 'primaryKey'
          },
          {
            title:'外键',
            align:"center",
            dataIndex: 'foreignKey'
          },
          {
            title:'索引',
            align:"center",
            dataIndex: 'tbIndex'
          },
          {
            title:'表名称',
            align:"center",
            dataIndex: 'tbName'
          },
          {
            title:'备注',
            align:"center",
            dataIndex: 'remark',
            scopedSlots: {customRender: 'htmlSlot'}
          },
          {
            title:'字段是否必填',
            align:"center",
            dataIndex: 'fieldIsNull'
          },
          {
            title:'字典ID',
            align:"center",
            dataIndex: 'dictId'
          },
          {
            title: '操作',
            dataIndex: 'action',
            align:"center",
            fixed:"right",
            width:147,
            scopedSlots: { customRender: 'action' }
          }
        ],
        url: {
          list: "/tbDbManage/tbDbManage/list",
          delete: "/tbDbManage/tbDbManage/delete",
          deleteBatch: "/tbDbManage/tbDbManage/deleteBatch",
          exportXlsUrl: "/tbDbManage/tbDbManage/exportXls",
          importExcelUrl: "tbDbManage/tbDbManage/importExcel",

        },
        dictOptions:{},
        superFieldList:[],
      }
    },
    created() {
    this.getSuperFieldList();
    },
    computed: {
      importExcelUrl: function(){
        return `${window._CONFIG['domianURL']}/${this.url.importExcelUrl}`;
      },
    },
    methods: {
      initDictConfig(){
      },
      getSuperFieldList(){
        let fieldList=[];
        fieldList.push({type:'string',value:'fieldName',text:'字段名称',dictCode:''})
        fieldList.push({type:'string',value:'fieldType',text:'字段类型',dictCode:''})
        fieldList.push({type:'string',value:'fieldLength',text:'字段长度',dictCode:''})
        fieldList.push({type:'string',value:'primaryKey',text:'主键',dictCode:''})
        fieldList.push({type:'string',value:'foreignKey',text:'外键',dictCode:''})
        fieldList.push({type:'string',value:'tbIndex',text:'索引',dictCode:''})
        fieldList.push({type:'string',value:'tbName',text:'表名称',dictCode:''})
        fieldList.push({type:'Text',value:'remark',text:'备注',dictCode:''})
        fieldList.push({type:'string',value:'fieldIsNull',text:'字段是否必填',dictCode:''})
        fieldList.push({type:'string',value:'dictId',text:'字典ID',dictCode:''})
        this.superFieldList = fieldList
      }
    }
  }
</script>
<style scoped>
  @import '~@assets/less/common.less';
</style>
