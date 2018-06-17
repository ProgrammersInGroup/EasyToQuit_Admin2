<template>
    <div>
        <i-layout>
            <i-breadcrumb :t3="breadcrumbTitle">
                <Button class="fr vue-back-btn" @click="$router.go(-1)" shape="circle">返回</Button>
            </i-breadcrumb>
            <div class="vue-panel-desc">
                <Row type="flex" justify="center">
                    <Col span="12" :md="14" :lg="12" :xs="24" :sm="24">
                    <Form ref="formValidate" :model="formValidate" :rules="ruleValidate" :label-width="100">
                        <FormItem label="編碼:" prop="menuId">
                            <label v-if="!$route.query.menuId" placeholder="請輸入菜單編碼">{{formValidate.menuId}}
                                <a style="margin-left:10px" @click="makeCode">換一個</a>
                            </label>
                            <label v-if="$route.query.menuId">{{formValidate.menuId}}</label>
                        </FormItem>
                        <FormItem label="個管師名:" prop="userCode">
              <Input v-if="!$route.query.userCode" v-model="formValidate.userCode" placeholder="請輸入個管師名稱"></Input>
              <label v-if="$route.query.userCode">{{formValidate.userCode}}</label>
            </FormItem>
            <FormItem label="電話:" prop="userName">
              <Input v-model="formValidate.userName" placeholder="請輸入電話"></Input>
            </FormItem>
            <FormItem label="生日:" prop="identifyNo">
              <Input v-model="formValidate.identifyNo" placeholder="請輸入生日"></Input>
            </FormItem>
            <FormItem label="性別:" prop="refUserRoleCode">
              <Select v-model="formValidate.refUserRoleCode" placeholder="請選擇性別">
                <Option value="USER">男</Option>
                <Option value="ADMIN">女</Option>
              </Select>
            </FormItem>
            <FormItem label="備註:" prop="desc">
              <Input v-model="formValidate.desc" type="textarea" :autosize="{minRows: 2,maxRows: 5}" placeholder="請輸入備註"></Input>
            </FormItem>
                        <FormItem>
                            <Button type="primary" @click="handleSubmit('formValidate')">保存</Button>
                            <Button type="ghost" @click="handleReset('formValidate')" style="margin-left: 8px">清空</Button>
                        </FormItem>
                    </Form>
                    </Col>
                </Row>
            </div>
        </i-layout>
    </div>
</template>

<script>
import Vue from "vue";
import iLayout from "../../components/layout.vue";
import iBreadcrumb from "../../components/breadcrumb.vue";
import randomWord from "../../service/randomWord";
import { getMenuList, updateMenu, createMenu } from "../../service/getData";
import md5 from "js-md5";
export default {
  name: "addMenu",
  components: {
    iLayout,
    iBreadcrumb
  },
  data() {
    return {
      formValidate: {
        platType: "200",
        menuLevel: "100"
      },
      ruleValidate: {
        menuName: [
          {
            required: true,
            message: "请输入菜单名称",
            trigger: "blur"
          },
          {
            pattern: /^[\u4E00-\u9FA5]{2,5}$/,
            message: "请输入2-5个中文菜单名称",
            trigger: "blur"
          }
        ],
        url: [
          {
            required: true,
            message: "请输入菜单地址",
            trigger: "blur"
          }
        ],
        platType: [
          {
            required: true,
            message: "请选择所属平台",
            trigger: "blur"
          }
        ],
        menuLevel: [
          {
            required: true,
            message: "请选择菜单级别",
            trigger: "blur"
          }
        ],
        icon: [
          {
            required: true,
            message: "请输入菜单图标编码",
            trigger: "blur"
          }
        ],
        remark: [
          {
            type: "string",
            max: 100,
            message: "菜单描述不能超过100个字",
            trigger: "blur"
          }
        ]
      }
    };
  },
  created() {
    if (this.$route.query && this.$route.query.menuId) {
      this.formValidate.menuId = this.$route.query.menuId;
      this.findMenuInfo();
      this.breadcrumbTitle = "修改菜單";
    } else {
      // 新增菜单时提示一下
      this.breadcrumbTitle = "新增菜單";
      this.message();
    }
  },
  methods: {
    async findMenuInfo() {
      const res = await getMenuList({
        params: {
          userCode: this.formValidate.userCode
        }
      });
      if (res.respCode === "000000") {
        this.formValidate = res.values[0];
      }
    },
    makeCode() {
      this.formValidate.menuId = Vue.set(
        this.formValidate,
        "menuId",
        +new Date() + randomWord({ num: false })
      );
    },
    message() {
      this.makeCode();
      this.$Notice.open({
        title: "温馨提示",
        duration: 10,
        render: h => {
          return h("span", {}, [
            h("span", {}, "新增菜单编码为:"),
            h(
              "span",
              {
                props: {
                  checkable: "true",
                  color: "green"
                },
                style: {
                  color: "#2d8cf0",
                  fontWeight: "800"
                }
              },
              "当前时间戳和随机字母组成"
            )
          ]);
        }
      });
    },
    async register() {
      this.tableLoading = true;
      let res;
      if (this.$route.query && this.$route.query.menuId) {
        // 更新菜单信息
        res = await updateMenu({
          icon: this.formValidate.icon,
          menuId: this.formValidate.menuId,
          menuLevel: this.formValidate.menuLevel,
          menuName: this.formValidate.menuName,
          platType: this.formValidate.platType,
          pmenuId: this.formValidate.pmenuId,
          remark: this.formValidate.remark,
          url: this.formValidate.url,
          userCode: this.formValidate.userCode,
          updateTime: +new Date()
        });
      } else {
        // 新增菜单
        res = await createMenu({
          icon: this.formValidate.icon,
          menuId: this.formValidate.menuId,
          menuLevel: this.formValidate.menuLevel,
          menuName: this.formValidate.menuName,
          platType: this.formValidate.platType,
          pmenuId: this.formValidate.pmenuId,
          remark: this.formValidate.remark,
          url: this.formValidate.url,
          userCode: this.formValidate.userCode,
          createTime: +new Date(),
          updateTime: +new Date()
        });
      }
      this.tableLoading = false;
      if (res && res.respCode === "000000") {
        this.$Message.success(res.respMsg);
        setTimeout(() => {
          this.$router.push("/menuManage/menuList");
        }, 2000);
      } else {
        this.$Message.error(res.respMsg);
      }
    },
    handleSubmit(name) {
      this.$refs[name].validate(valid => {
        if (valid) {
          this.register();
        } else {
          //   this.$Message.error("Fail!");
        }
      });
    },
    handleReset(name) {
      if (this.$route.query && this.$route.query.userCode) {
        this.formValidate.userCode = this.$route.query.userCode;
      } else {
        this.formValidate = {};
      }
      this.$refs[name].resetFields();
    }
  }
};
</script>

<style>

</style>
