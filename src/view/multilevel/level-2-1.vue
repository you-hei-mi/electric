<template>
    <div class="couplet-card">
        <div class="portlet-title" v-show='!showLog'>
            <Icon type="person-stalker"></Icon>
            <span class='title_text'>{{titleText}}</span>
        </div>

        <!--员工信息列表-->
        <div v-show='showList' class='query-wrap'>

            <div class="operate-wrap">
                <operate class='operate' :deleteStatus='isSelect && deleteBtn' :edit='isSelect && editBtn' :addStatus='addBtn'
                    :data='data' @refreshHandler='refreshHandler' @deleteHandler='deleteHandler' @addHandler='addHandler'
                    @editHandler='editHandler'></operate>
            </div>
            <div class="search-wrap">
                <search class='search-component' @query='query' :removeInputFlag='removeInputFlag' :placeholderValue='placeholderValue'>
                </search>
            </div>
        </div>
        <div class="list" v-show='showList'>
            <div class='table-wrap'>
                <i-table stripe highlight-row :columns="columns1" :data="listData" @on-row-click='selectItem'>
                </i-table>
                <Spin :fix='true' v-show='isLoading'>
                    <Icon type="load-c" size=18 class="demo-spin-icon-load"></Icon>
                    <div>Loading</div>
                </Spin>
            </div>
            <Page class='page-wrap' show-elevator show-total :total="total" :current="queryParam.pageNumber+1"
                @on-change='pageChange'>
            </Page>
        </div>
        <Modal v-model="isAddcostomer" :closable="false" width="800">
            <p slot="header" style="color:#2db7f5;text-align:left">
                <span>{{modelTitle}}</span>
            </p>
            <div>
                <Form ref="formValidate" :model="formValidate" :label-width="100">
                    <Row :gutter="24" class='row-wrap'>
                        <Col span="10" class='col-wrap'>
                        <FormItem label="客户名称" prop="customerName">
                            <Input v-model="formValidate.customerName" placeholder="请输客户名称"></Input>
                        </FormItem>
                        </Col>
                        <Col span="10" class='col-wrap'>
                        <FormItem label="客户编号" prop="customerSerialNumber">
                            <Input v-model="formValidate.customerSerialNumber" placeholder="请输入客户编号"></Input>
                        </FormItem>
                        </Col>
                    </Row>
                    <Row :gutter="24" class='row-wrap'>
                        <Col span="10" class='col-wrap'>
                        <FormItem label="客户地址" prop="customerAddress">
                            <Input v-model="formValidate.customerAddress" placeholder="请输入客户地址"></Input>
                        </FormItem>
                        </Col>
                        <Col span="10" class='col-wrap'>
                        <FormItem label="户号" prop="customerAccount">
                            <Input v-model="formValidate.customerAccount" placeholder="请输入户号"></Input>
                        </FormItem>
                        </Col>
                    </Row>
                    <Row :gutter="24" class='row-wrap'>
                        <Col span="10" class='col-wrap'>
                        <FormItem label="备注" prop="remark">
                            <Input type="textarea" :rows="3" v-model="formValidate.remark" placeholder=""></Input>
                        </FormItem>
                        </Col>
                    </Row>
                </Form>
            </div>
            <div slot="footer">
                <Button type="info" size="large" @click="handleCancle">
                    取消
                </Button>
                <Button type="info" size="large" :loading="saving" @click="handelSave">
                    保存
                </Button>
            </div>
        </Modal>
    </div>
</template>
<script>
    import search from '@/view/components/search/search.vue';
    import operate from '@/view/components/button-group/index.vue';
    import { getDate } from '@/libs/tools'
    import { getCustomerList,addCustomer,delCustomer,updataCustomer } from '@/api/customer'
    export default {
        name: 'level_2_1',
        data() {
            return {
                columns1: [{
						title: '序号',
						key: 'index',
                        width: 60,
                        align: 'center'
                    },
                    {
                        title: '客户名称',
                        key: 'customerName',
                        align: 'center',

                    },
                    {
                        title: '客户编号',
                        key: 'customerSerialNumber',
                        align: 'center',
                    },
                    {
                        title: '客户地址',
                        key: 'customerAddress',
                        align: 'center',
                    },
                    {
                        title: '户号',
                        key: 'customerAccount',
                        align: 'center',
                    },
                    {
                        title: '创建时间',
                        key: 'createTime',
                        align: 'center',
                    },
                    {
                        title: '备注',
                        key: 'remark',
                        align: 'center',
                    }
                ],
                listData: [{
                    customerName: '鹿城创业服务中心',
                    customerNumber: '0213554',
                    customerAddress: '温州鹿城区蛟凤路108号',
                    customerNb: '',
                    creatTime: '2018-08-08 16:18:58',
                    remarks: ''
                }],
                employeeGroupList: [],
                data: {},
                total: 1,
                page: 1,
                showList: true, //显示列表
                showLog: false,
                isSelect: false,
                add: false,
                titleText: '客户信息列表',
                modelTitle: '',
                iconType: 'person',
                placeholderValue: '请输入客户名称或编号',
                isLoading: false,
                isGroup: false,
                isEdit: false, // 点击编辑切换
                queryParam: {
                    "pageSize": 10,
                    "keyWords": '',
                    "pageNumber": 0,
                    "skipCount": 0
                },
                addGroupBtn: {
                    status: true,
                    text: '组管理'
                },
                numberParam: '',
                removeInputFlag: 0,
                addBtn: true, //新增按钮权限
                deleteBtn: false, //删除按钮权限
                editBtn: false,
                isAddcostomer: false,
                formValidate: {},
                saving: false,
            }
        },
        methods: {
            init() {
                this.getListData();
            },
            getListData(){
                return new Promise((resolve, reject) => {
                    getCustomerList(this.queryParam.pageSize,this.queryParam.skipCount).then(res => {
                        const data = res.data.result;
                        this.isLoading = false;
                        this.listData = data.items;
                        let size=this.queryParam.skipCount+1;
                        this.listData.forEach(element=>{
                            element.index=size++;
                            element.createTime=getDate(element.createTime,"year");
                        })
                        this.total = data.totalCount;
                        resolve()
                    }).catch(err => {
                        reject(err)
                    })
                })     
            },
            query(data) {
                this.queryParam.pageNumber = 0;
                this.queryParam.skipCount = 0;
                this.queryParam.keyWords = data.keyWords;
                this.getListData();
            },

            selectItem(data, index) {
                this.deleteBtn = true;
                this.editBtn = true;
                this.data = data;
                this.selectIndex = index;
                this.isSelect = true;
            },
            pageChange(data) {
                this.queryParam.pageNumber = data - 1;
                this.queryParam.skipCount = (data - 1) * this.queryParam.pageSize;
                this.getListData();
            },
            // 刷新页面
            refreshHandler() {
                this.queryParam.keyWords = '';
                this.isSelect = false;
                this.getListData();
            },
            deleteHandler(data) {
                return new Promise((resolve, reject) => {
                    delCustomer(data.id).then(res => {
                        this.$Message.success('删除客户成功');
                        this.isSelect = false;
                        this.getListData();
                        resolve()
                    }).catch(err => {
                        reject(err)
                    })
                })     
                
            },
            addHandler() {
                this.isAddcostomer = true;
                this.modelTitle = '新增人员组';
            },
            //编辑
            editHandler(index) {
                this.isEdit = true;
                this.isAddcostomer = true;
                this.modelTitle = '编辑人员组';
                this.formValidate = this.data;
            },
            handleCancle() {
                this.isAddcostomer = false;
                this.formValidate = {};
            },
            handelSave() {
                if(this.isEdit){
                    return new Promise((resolve, reject) => {
                    updataCustomer(this.formValidate.id,this.formValidate).then(res => {
                        this.$Message.success('修改客户成功');
                        this.isAddcostomer = false;
                        this.formValidate = {};
                        this.getListData();
                        resolve()
                    }).catch(err => {
                        reject(err)
                    })
                })    
                }
                return new Promise((resolve, reject) => {
                    addCustomer(this.formValidate).then(res => {
                        this.$Message.success('新增客户成功');
                        this.isAddcostomer = false;
                        this.formValidate = {};
                        this.getListData();
                        resolve()
                    }).catch(err => {
                        reject(err)
                    })
                })     
                
            }
        },
        components: {
            search,
            operate,
        },
        mounted() {
            this.init();
        }
    };

</script>
<style lang='less'>
    @import "../components/search/searchAndOperate.less";

    .control-group {
        height: 55px;

        .error_text {
            color: red;
            padding-left: 10px;
        }
    }

    .dec {
        display: inline-block;
        vertical-align: middle;
        width: 90px;
        text-align: right;
        height: 25px;
        line-height: 25px;
        font-size: 14px;
        margin-right: 10px;
    }

</style>
