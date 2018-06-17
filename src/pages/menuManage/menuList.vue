<template>
    <i-layout>
        <i-breadcrumb>
        </i-breadcrumb>

        <div class="vue-panel">
            <Form :model="formItem" :label-width="80">
                <Row>
                    <Col :xs="24" :sm="24" :md="8" :lg="8">
                    <FormItem label="創建日期:">
                        <DatePicker style="width:100%" v-model="formItem.createTime" format="yyyy-MM-dd" type="daterange" placement="bottom-start" placeholder="請選擇日期"></DatePicker>
                    </FormItem>
                    </Col>
                    <Col :xs="24" :sm="24" :md="8" :lg="8">
                    <FormItem label="用戶姓名:">
                        <Input v-model="formItem.menuName" placeholder="請輸入用戶姓名" clearable></Input>
                    </FormItem>
                    </Col>
                    <Col :xs="24" :sm="24" :md="8" :lg="8">
                    <FormItem label="電話:">
                        <Input v-model="formItem.menuId" placeholder="請輸入電話" clearable></Input>
                    </FormItem>
                    </Col>
                    <Col :xs="24" :sm="24" :md="8" :lg="8">
                    <FormItem label="性別:">
                        <Select v-model="formItem.platType" placeholder="請選擇" clearable>
                            <Option value="100">男</Option>
                            <Option value="200">女</Option>
                        </Select>
                    </FormItem>
                    </Col>
                    <Col :xs="24" :sm="24" :md="8" :lg="8">
                    <FormItem label="用戶狀態:">
                        <Select v-model="formItem.menuLevel" placeholder="請選擇" clearable>
                            <Option value="100">禁用</Option>
                            <Option value="200">啟用</Option>
                        </Select>
                    </FormItem>
                    </Col>
                    <Col :xs="24" :sm="24" :md="8" :lg="8">
                    <FormItem label="用戶名:">
                        <Input v-model="formItem.userCode" placeholder="請輸入用戶姓名" clearable></Input>
                    </FormItem>
                    </Col>
                </Row>
                <FormItem>
                    <Button type="primary" @click="getList" style="width:80px" long shape="circle">查詢</Button>
                    <Button type="ghost" style="width:80px;margin-left: 8px" @click="clearForm" shape="circle">清除</Button>
                </FormItem>
            </Form>
        </div>
        <div class="vue-panel-table">
            <nav-content>
                <router-link to="/menuManage/addMenu">
                    <Button class="fr vue-back-btn" shape="circle">新增菜单</Button>
                </router-link>
            </nav-content>
            <Table :loading="tableLoading" :data="tableData1" :columns="tableColumns1" stripe></Table>
            <div class="vue-panel-page" v-if="tableData1.length>10">
                <div style="float: right;">
                    <Page :total="total" show-total show-elevator show-sizer :page-size="pageSize" :current="pageNo" @on-page-size-change="changeSize" @on-change="changePage"></Page>
                </div>
            </div>
        </div>
        <!-- 对话框 -->

        <Modal v-model="modal6" :title="modalTitle" :loading="modalLoading" @on-ok="asyncOK">
            <p>{{modalContent}}</p>
        </Modal>
    </i-layout>
</template>
<script>
import Vue from "vue";
import iLayout from "../../components/layout.vue";
import iBreadcrumb from "../../components/breadcrumb.vue";
import navContent from "../../components/navcontent.vue";
import filters from "../../filters";
import { getMenuList, deleteMenuList } from "../../service/getData";
export default {
  name: "menuList",
  components: {
    iLayout,
    iBreadcrumb,
    navContent
  },
  filters: {
    formatDate(time) {
      let date = new Date(time);
      return filters.formatDate(date, "yyyy-MM-dd hh:mm:ss");
    }
  },
  data() {
    return {
      pageSize: 10,
      total: 0,
      pageNo: 1,
      value2: "",
      value14: "",
      tableLoading: false,
      modal6: false,
      // 对话框
      modalLoading: true,
      modalContent: "",
      modalTitle: "",
      modalType: "",
      formItem: {
        createTime: []
      },
      tableData1: [],
      tableColumns1: [
        {
          title: "序號",
          type: "index",
          width: 60,
          align: "center"
        },
        {
          title: "創建日期",
          width: 150,
          sortable: true,
          key: "createTime",
          render: (h, params) => {
            const row = params.row;
            const time = row.createTime
              ? filters.formatDate(row.createTime, "yyyy-MM-dd hh:mm:ss")
              : "";
            return h("span", time);
          }
        },
        {
          title: "病患姓名",
          key: "menuId",
          sortable: true
        },
        {
          title: "電話",
          key: "menuName"
        },
        {
          title: "生日",
          key: "userCode"
        },
        {
          title: "性別",
          key: "platType",
          render: (h, params) => {
            const row = params.row;
            const platType = row.platType
              ? filters.platType(row.platType)
              : row.platType;
            return h("span", platType);
          }
        },
        {
          title: "備註",
          key: "menuLevel",
          render: (h, params) => {
            const row = params.row;
            const menuLevel = row.menuLevel
              ? filters.menuLevel(row.menuLevel)
              : row.menuLevel;
            return h("span", menuLevel);
          }
        },
         {
           title: "操作",
          width: 180,
           key: "edit",
          render: (h, params) => {
           return h("div", [
              h(
                 "Button",
                {
                  props: {
                 type: "primary",
                   size: "small"
                 },
                  style: {
                    marginRight: "5px"
                 },
                   on: {
                   click: () => {
                      this.editMenu(params.row.menuId);
                    }
                 }
                },
                "修改"
              ),
              h(
                "Button",
                {
                  props: {
                    type: "error",
                    size: "small"
                  },
                  style: {
                    marginRight: "5px"
                  },
                  on: {
                    click: () => {
                      this.openModal({
                        id: params.row._id,
                        modalType: "delete",
                        modalTitle: "提示",
                        modalContent: "此操作将永久删除该项, 是否继续?"
                      });
                    }
                  }
                },
                "删除"
              ),
              h(
                "Button",
                {
                  props: {
                    type: "default",
                    size: "small"
                  },
                  on: {
                    click: () => {
                      this.goDesc(params.row.menuId);
                    }
                  }
                },
                "详情"
              )
            ]);
          }
        }
      ]
    };
  },
  methods: {
    changePage(pageNo) {
      this.pageNo = pageNo;
      this.getList();
    },
    changeSize(pageSize) {
      this.pageSize = pageSize;
      this.getList();
    },
    // 获取表格数据
    async getList() {
      this.tableLoading = true;
      const res = await getMenuList({
        params: this.formItem,
        pagenation: {
          pageNo: this.pageNo,
          pageSize: this.pageSize
        }
      });
      this.tableLoading = false;
      if (res && res.respCode === "000000") {
        if (res.values) {
          this.tableData1 = res.values;
          this.total = res.pagenation.itemCount;
          this.pageNo = res.pagenation.pageNo;
        } else {
          this.tableData1 = [];
          this.total = 0;
          this.pageNo = 1;
        }
      }
    },
    clearForm() {
      this.formItem = {};
    },
    // 删除菜单
    async deleteMenu(id) {
      const res = await deleteMenuList(id);
      this.modal6 = false;
      this.modalLoading = false;
      if (res && res.respCode === "000000") {
        this.$Message.success(res.respMsg);
        this.getList();
      } else {
        this.$Message.error(res.respMsg);
      }
    },
    // 打开对话框
    openModal(obj) {
      this.modal6 = true;
      this.modalTitle = obj.modalTitle;
      this.modalContent = obj.modalContent;
      this.modalType = obj.modalType;
      this.modalId = obj.id;
    },
    editMenu(menuId) {
      this.$router.push({
        name: "addMenu",
        path: "/menuManage/addMenu",
        query: { menuId }
      });
    },
    goDesc(menuId) {
      this.$router.push({
        name: "menuDesc",
        path: "/menuManage/menuList/menuDesc",
        query: { menuId }
      });
    },
    //对话框 is-ok
    asyncOK() {
      this.modalLoading = true;
      if (this.modalType === "delete") {
        this.deleteMenu(this.modalId);
      }
    }
  },
  created() {
    this.getList();
  }
};
</script>
