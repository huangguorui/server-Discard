<template>
  <div>
    <div class="crumbs">
      <el-breadcrumb separator="/">
        <el-breadcrumb-item>
          <i class="el-icon-lx-cascades"></i> {{$route.meta.title}}
        </el-breadcrumb-item>
      </el-breadcrumb>
    </div>
    <div class="container">
      <el-input v-model="searchForm.shopName"
                @input="handleSearch"
                clearable
                size="mini"
                placeholder="店铺名称"
                style="width:150px"
                class="handle-input mr10"></el-input>
      <el-select v-model="searchForm.userName"
                 clearable
                 placeholder="接单人"
                 class="mr10"
                 @input="handleSearch"
                 style="width:150px"
                 size="mini">
        <el-option :label="item.userName"
                   v-for="item in userList"
                   :keys="item.id"
                   :value="item.userName"></el-option>
      </el-select>

      <el-date-picker v-model="searchForm.activityEndTime"
                      type="date"
                      @input="handleSearch"
                      size="mini"
                      clearable
                      @on-change="searchForm.activityEndTime=$event"
                      style="width:150px"
                      class="mr10"
                      format="yyyy-MM-dd"
                      placeholder="活动结束时间">
      </el-date-picker>
      <el-date-picker v-model="searchForm.couponEndTime"
                      type="date"
                      @input="handleSearch"
                      size="mini"
                      clearable
                      @on-change="searchForm.couponEndTime=$event"
                      style="width:150px"
                      class="mr10"
                      format="yyyy-MM-dd"
                      placeholder="券结束时间">
      </el-date-picker>

      <el-input v-model="searchForm.activityID"
                @input="handleSearch"
                clearable
                size="mini"
                placeholder="活动ID"
                style="width:150px"
                class="handle-input mr10"></el-input>

      <el-select v-model="searchForm.commissionMoney"
                 placeholder="佣金比例"
                 class="mr10"
                 clearable
                 @input="handleSearch"
                 style="width:150px"
                 size="mini">
        <el-option :label="item.commissionList"
                   v-for="item in commissionList"
                   :keys="item.id"
                   :value="item.text">{{item.text}}</el-option>
      </el-select>

      <el-button type="primary"
                 icon="el-icon-search"
                 size="mini"
                 @click="handleSearch">搜索</el-button>
      <div class="handle-box">
        <el-button type="primary"
                   icon="el-icon-delete"
                   class="handle-del mr10"
                   size="mini"
                   :disabled="isBtnDisable"
                   @click="delAllSelection">批量删除</el-button>

      </div>
      <el-table :data="tableData"
                v-loading="loading"
                border
                class="table"
                size="small"
                ref="multipleTable"
                header-cell-class-name="table-header"
                @selection-change="handleSelectionChange">
        <el-table-column type="selection"
                         width="55"
                         align="center"></el-table-column>
        <el-table-column prop="id"
                         fixed
                         label="ID"
                         width="55"
                         align="center"></el-table-column>

        <el-table-column prop="userName"
                         fixed
                         label="接单人"></el-table-column>
        <el-table-column prop="shopName"
                         fixed
                         label="店铺名称"></el-table-column>
        <el-table-column prop="shopkeeperWangWang"
                         label="掌柜旺旺"></el-table-column>
        </el-table-column>

        <el-table-column prop="shopLink"
                         label="商品链接">
          <template slot-scope="scope"
                    v-if="scope.row.shopLink">

            <a :href="scope.row.shopLink"
               target="_blank"
               style="color: blue;font-size:12px;text-align:center;">查看活动</a>
          </template>
        </el-table-column>

        <el-table-column prop="activityStartTime"
                         width="150"
                         label="活动开始时间"></el-table-column>
        <el-table-column prop="activityEndTime"
                         width="150"
                         label="活动结束时间"></el-table-column>

        <el-table-column prop="commission"
                         label="佣金+服务费%"></el-table-column>
        <el-table-column prop="documentaryTime"
                         label="添加时间"
                         width="150"></el-table-column>

        <el-table-column prop="activityID"
                         label="活动ID"></el-table-column>
        <el-table-column prop="activityLink"
                         label="活动链接"
                         width="300">
          <template slot-scope="scope"
                    prop="activityLink"
                    v-if="scope.row.activityLink">
            <div>
              <a href="##"
                 v-if="!scope.row.isShow"
                 @click="isHidden(scope.$index,scope.row)"
                 style="color: blue;font-size:12px;text-align:center;">查看链接</a>
              {{scope.row.isShow==true?scope.row.activityLink:''}}

            </div>
          </template>
        </el-table-column>
        <el-table-column prop="remarks"
                         label="备注"></el-table-column>
        <el-table-column prop="orderPhone"
                         label="店铺联系人"></el-table-column>
        <el-table-column prop="couponLink"
                         label="券链接">
          <template slot-scope="scope"
                    v-if="scope.row.couponLink">
            <a :href="scope.row.couponLink"
               target="_blank"
               style="color: blue;font-size:12px;text-align:center;">查看链接</a>
          </template>
        </el-table-column>

        <el-table-column prop="couponStartTime"
                         width="150"
                         label="优惠券开始时间"></el-table-column>

        <el-table-column prop="couponEndTime"
                         width="150"
                         label="优惠券结束时间"></el-table-column>

        <el-table-column label="操作"
                         width="150">
          <template slot-scope="scope">
            <el-button size="mini"
                       @click="edlt(scope.row)">编辑</el-button>
            <el-button size="mini"
                       type="danger"
                       @click="delAllSelection(scope.row.id,'single')">删除</el-button>
          </template>
        </el-table-column>

      </el-table>

      <!-- 分页 -->
      <!-- <pagination :page-info="query"></pagination> -->
      <div class="pagination">
        <el-pagination background
                       layout="total, prev, pager, next"
                       :current-page="query.page"
                       :page-size="query.page_size"
                       :total="query.count"
                       @current-change="handlePageChange"></el-pagination>
      </div>

    </div>
  </div>
</template>

<script>
import interList from '@/common/mixins/list'
import set from '@/common/mixins/set'

import { postDocumentaryDel, getDocumentarysList, UserList } from '../../../api/index';
export default {
  name: 'getDocumentarysList',
  data () {
    return {
      formData: {
        userPhone: '',//用户电话
        userName: '',//用户真实姓名
        userPwd: '',//用户密码
      },
      rules: {

        userPhone: [
          // { type: 'number', message: '手机号码必须为数字值' },
          { required: true, message: '手机号码不能为空', trigger: 'blur' },

          // { min: 1, max: 5, message: '长度在 5 到 11 个字符', trigger: 'blur' },
        ],
        userName: [
          { min: 2, max: 20, message: '长度在 2 到 20 个字符', trigger: 'blur' }, ,
          { required: true, message: '请输入用户真实姓名', trigger: 'blur' }
        ],
        userPwd: [
          { required: true, message: '请输入密码', trigger: 'blur' },
          { min: 8, max: 20, message: '长度在 8 到 20 个字符', trigger: 'blur' }, ,
        ],
      },
      drawerTitle: '新增用户',
      isShowDrawer: false,
      tableData: [
        {
          id: '',
          shopkeeperWangWang: "", //根据掌柜旺旺
          orderPhone: "",       //店铺联系人
          shopLink: "",         //商品链接
          couponLink: "",         //优惠券链接
          couponStartTime: "",    //优惠券开始时间
          couponEndTime: "",     //优惠券结束时间
          activityStartTime: "",  //活动开始时间
          activityEndTime: "", //活动结束时间
          commission: "",    //佣金+服务费
          userID: "",
          shopName: "",    //店铺名称
          activityID: "", //活动ID
          activityLink: "",//活动链接
          remarks: "",     //备注
          documentaryTime: "",  //订单生产时间，后台生成
        }
      ], //当前表数据
      userList: [],  //用户列表
      commissionList: [
        {
          id: 0,
          text: '>=5',
        },
        {
          id: 1,
          text: '>=10',
        },
        {
          id: 2,
          text: '>=20',
        },
        {
          id: 3,
          text: '>=30',
        },
      ],  //佣金比例列表
      editVisible: false, //弹框删除
    };
  },
  mixins: [interList],
  created () {

  },
  methods: {
    isHidden (i, row) {
      //点击显示链接
      row.isShow = true
      this.$set(this.tableData, i, row)
    },

    edlt (row) {
      console.log(row)
      this.$router.push({  //核心语句
        path: '/editSingle',   //跳转的路径
        query: {           //路由传参时push和query搭配使用 ，作用时传递参数
          id: row.id,
        }
      })
    },
    getData () {
      let _this = this
      this.loading = true
      getDocumentarysList(this.query).then(res => {
        this.tableData = res.list;
        this.tableData.forEach((item, index) => {
          this.tableData[index].isShow = false

        })
        //请求用户
        UserList({ page_size: 100 }).then(res => {
          this.userList = res.list;

        }).catch(function (error) {
          this.active.error()
        })
        this.query = res.page_info
        this.loading = false

      }).catch(function (error) {
        this.active.error()
        _this.loading = false
      })
    },
    //单选多选都可删除
    delAllSelection (id, flag) {
      //通过点击删除进来的 传入的参数必须为一个数组
      if (flag == 'single') {
        this.DelId = [id]
        console.log('id', id)
      }
      // 二次确认删除
      this.$confirm(`确定要删除ID序号为[${this.DelId}]`, '提示', {
        type: 'warning'
      })
        .then(() => {
          this.loading = true
          postDocumentaryDel({ id: this.DelId }).then(res => {
            this.active.success()
            this.getData()
          })
        })
        .catch(() => {
          this.loading = false
        });
      this.multipleSelection = [];
    },
  }
};
</script>

<style scoped>
@import '~@/assets/css/user_index.css';
</style>
