<template>
    <div class="page-wrap">
        <div class="wrap lg-show">
          <Header></Header>
        </div>
        <div class="sm-show page-sm-header-wrap">
          <smHeader></smHeader>
        </div>
        <div class="sm-show sm-search-input-wrap">
          <searchInput></searchInput>
        </div>
        <div class="main-wrap">
          <div class="wrap">
            <div class="describe-title-wrap">
              <span class="title">{{$t('navs.transaction')}}</span>
              <span v-show="isShow" class="title-content">
              {{$t('forBlock')}} {{content}}
              </span>
              <ul class="link-wrap">
                <li><a href="/">{{$t("navs.home")}}</a></li>
                <li><i class="el-icon-arrow-right"></i></li>
                <li class="current">{{$t('navs.transaction')}}</li>
              </ul>
            </div>
            <ShardSelect></ShardSelect>
            <el-table
              class="list-wrap"
              :data="transactionList"
              :empty-text="$t('message.noData')"
              style="width: 100%">
              <el-table-column
                prop="txHash"
                width="340"
                :label="$t('listHeader.txHash')">
                <template slot-scope="scope">
                  <router-link :to="{path: '/transaction/detail', query: { txhash: scope.row.txHash }}">
                    <span class="table-link-color list-content">{{scope.row.txHash}}</span>
                  </router-link>
                </template>
              </el-table-column>
              <!-- <el-table-column
                prop="age"
                :label="$t('listHeader.age')"
                width="130">
              </el-table-column> -->
              <el-table-column
                prop="block"
                :label="$t('listHeader.block')"
                width="110">
                <template slot-scope="scope">
                  <router-link :to="{path: '/block/detail', query: { height: scope.row.block, s: shardValue }}">
                    <span class="table-link-color list-content">{{scope.row.block}}</span>
                  </router-link>
                </template>
              </el-table-column>
              <el-table-column
                prop="from"
                width="310"
                :label="$t('listHeader.from')">
                <template slot-scope="scope">
                  <span :class="{'table-link-color': isLink(scope.row.from)}" class="list-content" @click="toFrom(scope.row.from)">{{[scope.row.from, 'from'] | setFormatAd}}</span>
                </template>
              </el-table-column>
              <el-table-column
                prop="to"
                width="310"
                :label="$t('listHeader.to')">
                <template slot-scope="scope">
                  <span :class="{'table-link-color': isLink(scope.row.to)}" class="list-content" @click="toFrom(scope.row.to)">{{[scope.row.to, 'to'] | setFormatAd}}</span>
                </template>
              </el-table-column>
              <el-table-column
                prop="value"
                width="130"
                :label="$t('listHeader.value')">
              </el-table-column>
            </el-table>
            <el-pagination
              class="el-pagination-wrap fr"
              @size-change="handleSizeChange"
              @current-change="handleCurrentChange"
              :current-page="page"
              :page-size="pageSize"
              layout="prev, pager, next"
              :total="total">
            </el-pagination>
          </div>
        </div>
        <Footer></Footer>
    </div>
</template>
<script>
import { mapActions } from 'vuex'
import router from '../../router'
import Header from '../header'
import smHeader from '../sm-header'
import searchInput from '../search-input'
import TransactionDescribe from '../describe'
import Footer from '../footer'
import ShardSelect from '../shard-select'
import { filtersAd, formatAd } from '../../untils/format'

export default {
  data () {
    return {
      content: this.$route.query.block,
      linkColor: false,
      pageSize: 25
    }
  },
  mounted () {
    if (this.$route.query.block) {
      this.getHeightShow(true)
      this.getBlockList(1, this.$route.query)
    } else {
      this.getList(1)
      this.getHeightShow(false)
    }
  },
  components: {
    Header,
    smHeader,
    searchInput,
    TransactionDescribe,
    Footer,
    ShardSelect
  },
  computed: {
    transactionList: {
      get () {
        return this.$store.state.transaction.transactionList
      }
    },
    page: {
      get () {
        return this.$store.state.transaction.page
      }
    },
    total: {
      get () {
        return this.$store.state.transaction.total
      }
    },
    isShow: {
      get () {
        return this.$store.state.block.heightShow
      }
    },
    shardValue: {
      get () {
        return this.$store.state.shard.shardValue
      }
    }
  },
  filters: {
    setFormatAd (params) {
      return formatAd(params)
    }
  },
  methods: {
    ...mapActions(['getTransactionList']),
    ...mapActions(['getTransactionBlock']),
    ...mapActions(['getHeightShow']),

    handleSizeChange (val) {
      this.getList(val, this.shardValue)
    },
    handleCurrentChange (val) {
      if (this.isShow) {
        this.getBlockList(val, this.$route.query, this.shardValue)
      } else {
        this.getList(val, this.shardValue)
      }
    },
    getBlockList (page, params) {
      this.getTransactionBlock([page, params, this.shardValue])
    },
    getList (page) {
      this.getTransactionList([page, this.shardValue])
    },
    toFrom (address) {
      return filtersAd(address) ? router.push({path: '/account/detail', query: {address: address}}) : ''
    },
    isLink (address) {
      return filtersAd(address)
    }
  },
  watch: {
    '$route' (to, from) {
      this.getList(1)
      this.getHeightShow(false)
    },
    shardValue: {
      handler: function (val, oldval) {
        if (this.isShow) {
          this.getBlockList(1, this.$route.query, this.shardValue)
        } else {
          this.getList(1, this.shardValue)
        }
      }
    }
  }
}
</script>
<style lang="less">
  @import "../../assets/css/page.less";
  @import "../../assets/css/describe.less";
  @import "../../assets/css/list.less";
</style>
