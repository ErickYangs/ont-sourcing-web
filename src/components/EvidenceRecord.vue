<template>
  <div class="czDetail">
    <TopBar></TopBar>
    <div class="czDetailCon">
      <div class="buttons">
        <el-button type="primary" round @click="back">返回</el-button>
        <el-button type="primary" id="exporto" icon="el-icon-download" round @click="exporto">导出</el-button>
      </div>
      <div class="listDetail" v-loading="fullscreenLoading">
        <p class="review">存证记录</p>
        <el-table
          class="my-table"
          id="rebateSetTable"
          :data="listDetail"
          :border="false"
          style="width: 100%"
        >
          <el-table-column prop="txhash" label="存证编号" width="300" align="center"></el-table-column>
          <el-table-column prop="createTime" label="创建时间" width="300" align="center"></el-table-column>
          <el-table-column prop="height" label="区块高度" width="100" align="center"></el-table-column>
          <el-table-column prop="timestamp" label="时间戳" width="300" align="center"></el-table-column>
          <el-table-column prop="ontid" label="存证者ONTID" width="400" align="center"></el-table-column>
          <el-table-column
            prop="companyOntid"
            v-if="isShow"
            label="被存证者ONTID"
            width="400"
            align="center"
          ></el-table-column>
          <el-table-column prop="type" label="类型" width="100" align="center"></el-table-column>
          <el-table-column type="text" label="存证详情" width="200" align="center">
            <template slot-scope="scope">
              <el-button @click="lookDetail(scope.$index)" type="text" size="small">点击查看详情</el-button>
            </template>
          </el-table-column>
        </el-table>
        <div class="block">
          <el-pagination
            @size-change="handleSizeChange"
            @current-change="handleCurrentChange"
            :page-sizes="[5, 10]"
            :page-size="pageSize"
            :page-count="5"
            :current-page="nowPage"
            background
            layout="->,sizes, prev, pager, next"
            :total="totalNum"
          ></el-pagination>
        </div>
      </div>
    </div>
  </div>
</template>
<script>
import TopBar from './TopBar'
import FileSaver from 'file-saver'
import XLSX from 'xlsx'
import dateFormat from '../util/dateFormat'
export default {
  components: {
    TopBar
  },
  data() {
    return {
      fullscreenLoading: false,//加载
      totalNum: 0,//总数
      listDetail: [],
      pageSize: 10,//每页显示条目个数
      tableList: [],
      nowPage: 1,
      activeBar: "first",
      access_token: '',
      url: '',
      totalUrl: '',
      isShow: true
    }
  },
  mounted() {
    this.listDetail = this.listDetail.slice(0, this.pageSize);
    this.access_token = sessionStorage.getItem('access_token');
    let type = sessionStorage.getItem('TYPE')
    if (type === '2c') {
      this.isShow = false
      this.url = process.env.TOC_API_ROOT + 'api/v1/c/attestation/history'
      this.totalUrl = process.env.TOC_API_ROOT + 'api/v1/c/attestation/count'
    } else {
      this.url = process.env.API_ROOT + 'api/v1/contract/history'
      this.totalUrl = process.env.API_ROOT + 'api/v1/contract/count'
    }
    this.getTotalNum(this.access_token);//获取总数
    this.handleCurrentChange(1);//默认获取第一页

  },
  methods: {
    getTotalNum(access_token) {
      this.$http.post(this.totalUrl, {
        "access_token": this.access_token      })
        .then((response) => {
          this.totalNum = response.data.result;//存证记录总数量
        })
        .catch((error) => {
          this.$message({ type: 'error', message: error });
        });
    },
    back() {
      this.$router.push({ name: 'newEvidence' });
    },
    exporto() {//导出
      /* generate workbook object from table */
      let xlsxParam = { raw: true };//转换成excel时，使用原始的格式
      let wb = XLSX.utils.table_to_book(document.querySelector('#rebateSetTable'), xlsxParam);  //表格的id名
      /* get binary string as output */
      let wbout = XLSX.write(wb, { bookType: 'xlsx', bookSST: true, type: 'array' });
      try {
        FileSaver.saveAs(new Blob([wbout], { type: 'application/octet-stream' }), '用户提交返利表.xlsx');
      } catch (e) {
        if (typeof console !== 'undefined')
          console.log(e, wbout)
      }
      return wbout
    },
    handleSizeChange(val) {//点击条数
      this.pageSize = val;
      this.handleCurrentChange(this.nowPage);
    },
    handleCurrentChange(val) {
      this.fullscreenLoading = true;
      this.nowPage = val;
      this.$http.post(this.url, {
        "access_token": this.access_token,
        "pageNum": this.nowPage,//页数
        "pageSize": this.pageSize,//每页记录数 10以内 
      })
        .then((response) => {
          this.fullscreenLoading = false;
          this.listDetail = response.data.result;
          this.listDetail.forEach(item => {
            item.createTime = dateFormat.format('yyyy-MM-dd hh:mm:ss', new Date(item.createTime));
            item.timestamp = dateFormat.format('yyyy-MM-dd hh:mm:ss', new Date(item.timestamp));
          });
        })
        .catch((error) => {
          this.fullscreenLoading = false;
          this.$message({ type: 'error', message: error });
        });
    },
    lookDetail(index) {
      this.$router.push({ path: '/detailEvidence/' + this.listDetail[index].txhash });
    }
  }
}
</script>
<style scoped>
.listDetail .my-table .el-table__body-wrapper::-webkit-scrollbar {
  width: 5px;
  height: 5px;
}
.listDetail.my-table .el-table__body-wrapper::-webkit-scrollbar-thumb {
  background-color: #409eff;
  border-radius: 2px;
}
.czDetailCon {
  width: 80%;
  margin: 0 auto;
  margin: 2rem auto;
}
#exporto {
  cursor: pointer;
}
.listDetail {
  margin: 0 auto;
  margin-top: 2rem;
  background: #fff;
  padding-bottom: 2rem;
  /* height: 550px;; */
}
.review {
  font-size: 1.4rem;
  font-weight: bold;
  text-align: center;
  border-bottom: 1px solid #eae7e7;
  line-height: 4rem;
}
.el-table.el-table--fit.el-table--enable-row-hover {
  height: 100%;
}
.buttons {
  display: flex;
  justify-content: space-between;
  margin-bottom: 2rem;
}
.el-table th > .cell {
  color: #000;
}
.block {
  margin-top: 2rem;
}
</style>


