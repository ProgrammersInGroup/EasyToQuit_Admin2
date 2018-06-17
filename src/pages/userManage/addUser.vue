<template>
  <div>
    <i-layout>
      <i-breadcrumb :t3="breadcrumbTitle">
        <Button class="fr vue-back-btn" @click="$router.go(-1)" shape="circle">返回</Button>
      </i-breadcrumb>
      <div class="vue-panel-desc">
        <Row type="flex" justify="center">
          <Col span="12" :md="14" :lg="12" :xs="24" :sm="24">
          <Form ref="formValidate" :model="formValidate" :rules="ruleValidate" :label-width="80">
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
            <FormItem label="狀態:" prop="status">
              <RadioGroup v-model="formValidate.status">
                <Radio label="100">在職</Radio>
                <Radio label="500">離職</Radio>
              </RadioGroup>
            </FormItem>
            <FormItem label="備註:" prop="desc">
              <Input v-model="formValidate.desc" type="textarea" :autosize="{minRows: 2,maxRows: 5}" placeholder="請輸入備註"></Input>
            </FormItem>
            <FormItem>
              <Button type="primary" @click="handleSubmit('formValidate')">儲存</Button>
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
import iLayout from "../../components/layout.vue";
import iBreadcrumb from "../../components/breadcrumb.vue";
import { getUserList, updateUser, createUser } from "../../service/getData";
import md5 from "js-md5";
export default {
  name: "addUser",
  components: {
    iLayout,
    iBreadcrumb
  },
  data() {
    return {
      formValidate: {},
      ruleValidate: {
        userCode: [
          {
            required: true,
            message: "請輸入個管師姓名",
            trigger: "blur"
          }
        ],
        userName: [
          {
            required: true,
            message: "請輸入電話",
            trigger: "blur"
          }
        ],
        identifyNo: [
          { required: true,
            message: "請輸入生日",
            trigger: "blur"
          }
        ],
        refUserRoleCode: [
          {
            required: true,
            message: "请选择用户类型",
            trigger: "change"
          }
        ],
        desc: [
          {
            type: "string",
            max: 100,
            message: "备注不能超过100个字",
            trigger: "blur"
          }
        ],
        status: [
          {
            required: true,
            message: "请选择用户状态",
            trigger: "change"
          }
        ]
      }
    };
  },
  created() {
    if (this.$route.query && this.$route.query.userCode) {
      this.formValidate.userCode = this.$route.query.userCode;
      this.findUserInfo();
      this.breadcrumbTitle = "修改用户";
    } else {
      // 新增用户时提示一下
      this.breadcrumbTitle = "新增用户";
      this.message();
    }
  },
  methods: {
    async findUserInfo() {
      const res = await getUserList({
        params: {
          userCode: this.formValidate.userCode
        }
      });
      if (res.respCode === "000000") {
        this.formValidate = res.values[0];
      }
    },
    message() {
      this.$Notice.open({
        title: "温馨提示",
        duration: 10,
        render: h => {
          return h("span", {}, [
            h("span", {}, "新增用户密码为:"),
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
              "123456"
            )
          ]);
        }
      });
    },
    async register() {
      this.tableLoading = true;
      let res;
      if (this.$route.query && this.$route.query.userCode) {
        // 更新用户信息
        res = await updateUser({
          userCode: this.formValidate.userCode,
          status: this.formValidate.status,
          userName: this.formValidate.userName,
          identifyNo: this.formValidate.identifyNo,
          desc: this.formValidate.desc,
          refUserRoleCode: this.formValidate.refUserRoleCode,
          updateTime: +new Date()
        });
      } else {
        // 新增用户
        res = await createUser({
          userCode: this.formValidate.userCode,
          status: this.formValidate.status,
          userName: this.formValidate.userName,
          identifyNo: this.formValidate.identifyNo,
          desc: this.formValidate.desc,
          refUserRoleCode: this.formValidate.refUserRoleCode,
          createTime: +new Date(),
          updateTime: +new Date(),
          password: md5("123456")
        });
      }
      this.tableLoading = false;
      if (res && res.respCode === "000000") {
        this.$Message.success(res.respMsg);
        setTimeout(() => {
          this.$router.push("/userManage/userList");
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
