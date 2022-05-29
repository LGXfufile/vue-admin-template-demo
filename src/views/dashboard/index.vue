<template>
  <el-container>
    <!--    <el-row class="headerSelf">-->
    <!--      <span>-->
    <!--        <el-menu-item index="1">文案搜索工具</el-menu-item>-->
    <!--      </span>-->
    <!--    </el-row>-->
    <el-header>
      <el-row>
        <el-col :span="8">
          <div class="grid-content " />
        </el-col>
        <el-col :span="8">
          <!--          <el-menu :default-active="activeIndex" mode="horizontal" @select="handleSelect">-->
          <!--            <el-menu-item index="1" class="menuStyle1">文案搜索工具</el-menu-item>-->
          <!--&lt;!&ndash;            <el-menu-item index="2" class="menuStyle1">赞助我</el-menu-item>&ndash;&gt;-->
          <!--            &lt;!&ndash;              <el-submenu index="2" class="menuStyle2" >&ndash;&gt;-->
          <!--            &lt;!&ndash;                <template slot="title">赞助我</template>&ndash;&gt;-->
          <!--            &lt;!&ndash;                <el-menu-item index="2-1">微信赞助</el-menu-item>&ndash;&gt;-->
          <!--            &lt;!&ndash;              </el-submenu>&ndash;&gt;-->
          <!--          </el-menu>-->
          <!--          <div class="xxx">-->
          <!--            影视剧文案万能搜索工具-->
          <!--          </div>-->
          <!--        <el-menu>-->
          <!--          <el-menu-item class="xxx"></el-menu-item>-->
          <!--        </el-menu>-->
          <div style="margin: 80px 0">

            <el-input v-model="result.title" class="inputStyle" placeholder="请输入要查询的影视剧名" />
            <el-button type="primary" class="btnStyle" @click="searchOne">搜索</el-button>
            <el-button type="success" icon="el-icon-download">
              数据导出
            </el-button>
          </div>

        </el-col>
        <el-col :span="8">
          <div class="grid-content " />
        </el-col>

      </el-row>
    </el-header>
    <el-main>
      <el-row>
        <el-col :span="2">
          <div class="grid-content " />
        </el-col>

        <el-col :span="16">

          <el-table
            id="out-table"
            :data="tableData.slice((cur_page-1)*pageSize,cur_page*pageSize)"
            border
            :pagination="pagination"
            style="width: 100%;text-align: center;"
          >
            <el-table-column
              prop="id"
              label="序号"
              align="center"
              width="180"
            />
            <el-table-column
              :show-overflow-tooltip="true"
              prop="title"
              label="标题"
              align="center"
              width="180"
            />
            <el-table-column
              :show-overflow-tooltip="true"
              prop="des"
              align="center"
              label="影视剧情文案"
            />
          </el-table>
          <el-pagination
            :current-page="cur_page"
            :page-sizes="pageArray"
            :page-size="pageSize"
            layout="total, sizes, prev, pager, next, jumper"
            :total="dataSizes"
            @size-change="handleSizeChange"
            @current-change="handleCurrentChange"
          />
        </el-col>
        <el-col :span="6">
          <div class="grid-content bg-purple">
            <el-button type="primary" class="zzStyle" @click="dialogVisible = true">扫码查看更多文案</el-button>
            <img src="./wechatImg.jpg" class="imgStyle">
          </div>
        </el-col>
        <el-dialog
          title="欢迎加入群聊"
          :visible.sync="dialogVisible"
          width="25%"
        >
          <img src="./wechatImg.jpg" style="width:100%;height:100%;">
          <span slot="footer" class="dialog-footer">
            <el-button @click="dialogVisible = false">取 消</el-button>
            <el-button type="primary" @click="dialogVisible = false">确 定</el-button>
          </span>
        </el-dialog>

      </el-row>
    </el-main>
  </el-container>
</template>

<script>
import { mapGetters } from 'vuex'
import axios from 'axios'
export default {
  name: 'Dashboard',
  data() {
    return {
      dataSizes: '',
      source: {
        src: 'E:\\javaLearn\\vue\\vue_demo01\\wechatImg.jpg'
      },
      // ruleForm: {
      //   pass: '',
      //   checkPass: '',
      //   age: ''
      // },
      dialogVisible: false,
      cur_page: 1,
      pageSize: 15,
      pageArray: [],
      pagination: {
        defaultPageSize: 12
      },
      result: {
        title: ''
      },
      baseUrl: 'http://localhost:8081',
      filesUploadUrl: 'http://localhost:9999/upload',
      tableData: []
    }
  },
  created() {
    this.initData()
  },
  methods: {
    handleSizeChange(val) {
      this.pageSize = val
      console.log(`每页 ${val} 条`)
    },
    handleCurrentChange(val) {
      this.cur_page = val
      console.log(`当前页: ${val}`)
    },
    exportData() {
      // this.$http.post(this.baseUrl+"/selectOne",this.result,{
      //   responseType: "blob"
      // })
      if (this.result.title.length === 0) {
        this.$message.error('搜索内容为空，无数据导出')
      } else {
        axios.post(this.baseUrl + '/selectOne', this.result)
          .then(res => {
            /* 方式1，文件名随机
            let blob = new Blob([res.data], { type: "application/vnd.ms-excel" });
            let url = window.URL.createObjectURL(blob);
            window.location.href = url;
            */
            /* 方式2，支持IE10；文件名自定义
             */
            const blob = new Blob([res.data.data], { type: 'application/vnd.ms-excel' }) // 将服务端返回的文件流（二进制）excel文件转化为blob
            const fileName = '学生列表.xls'

            if (window.navigator && window.navigator.msSaveOrOpenBlob) {
              // IE
              window.navigator.msSaveOrOpenBlob(blob, fileName)
            } else {
              const objectUrl = (window.URL || window.webkitURL).createObjectURL(
                blob
              )
              const downFile = document.createElement('a')
              downFile.style.display = 'none'
              downFile.href = objectUrl
              downFile.download = fileName // 下载后文件名
              document.body.appendChild(downFile)
              downFile.click()
              document.body.removeChild(downFile) // 下载完成移除元素
              // window.location.href = objectUrl
              window.URL.revokeObjectURL(objectUrl) // 只要映射存在，Blob就不能进行垃圾回收，因此一旦不再需要引用，就必须小心撤销URL，释放掉blob对象。
            }
          })
      }
    },
    initData() {
      this.pageArray = [this.pageSize, 50, 100],
      axios.get(this.baseUrl + '/selectAll')
        .then(res => {
          console.log(res.data)
          this.tableData = res.data.data
          console.log(this.tableData)
          this.dataSizes = res.data.data.length
        })
    },
    search() {
      axios.get().then(res => {
        console.log(res.data)
      })
    },
    searchOne() {
      axios.post(this.baseUrl + '/selectOne', this.result)
        .then(res => {
          console.log(res.data)
          this.tableData = res.data.data
          this.dataSizes = res.data.data.length
          console.log(this.tableData)
        })
    },

    sucessFiles(response) {
      console.log(response)
      this.result = response
    },
    downloadFiles() {
      console.log(this.result)
      axios.get(this.result).then(res => {
        console.log(res.data)
        // if(!res.data){
        //   return
        // }
        // var name = this.date
        // var blob = new Blob([res.data]);
        // var url = window.URL.createObjectURL(blob);
        // var aLink = document.createElement("a");
        // aLink.style.display = "none";
        // aLink.href = url;
        // aLink.setAttribute("download", name);
        // document.body.appendChild(aLink);
        // aLink.click();
        // document.body.removeChild(aLink); //下载完成移除元素
        // window.URL.revokeObjectURL(url); //释放掉blob对象
      })
    },
    videoToMusic() {
      axios.get(this.result + '/videoToMusic').then(res => {
        console.log(res.data)
      })
    }
  },
  computed: {
    ...mapGetters([
      'name'
    ])
  }
}
</script>

<style lang="scss" scoped>
.dashboard {
  &-container {
    margin: 30px;
  }
  &-text {
    font-size: 30px;
    line-height: 46px;
  }
}

.imgStyle{
  margin: 10px 20px;
  width: 80%;
  width: 80%;
}
.xxx {
  text-align: center;
  font-weight: bold;
  font-size: 30px;
  margin: 10px 0;
  color: black;
  position: fixed;
}

/*.bg-purple {*/
/*  background: #d3dce6;*/
/*}*/

.grid-content {
  border-radius: 4px;
  min-height: 36px;

}

/*.aaa {*/
/*  background: darkcyan;*/
/*}*/

.el-col-16 {
  width: 66.66667%;
  background: cadetblue;
}

.inputStyle {
  width: 350px;
  margin: 0 50px;
}

.btnStyle {
  width: 100px;
  font-size: 15px;
  margin: 0 10px;
}

.yyy {
  margin: 0 100px;
}

.has-gutter {
  text-align: center;
}

.menuStyle1 {
  margin: 0 300px;
  color: aquamarine;
  font-size: 30px;
}

.menuStyle2 {

  color: aquamarine;
}

.el-menu--horizontal el-menu {
  text-align: center;
}

/*.el-row {*/
/*  background-color: #1E90FF			;*/

/*}*/
//.el-header {
//
//  box-sizing: border-box;
//  width: 100%;
//  flex-shrink: 0;
//  background-color: cornflowerblue;
//  position: fixed;
//}

.el-menu {
  border-right: solid 1px #e6e6e6;
  list-style: none;
  position: relative;

  padding-left: 0;
  background-color: cornflowerblue;
}

.el-main {
  position: absolute;
  left: 0;
  right: 0;
  top: 150px;
  bottom: 0;
  overflow-y: scroll;
  height: 100%;
}

/*.el-aside {*/
/*  !*display: block;*!*/
/*  !*position: absolute;*!*/
/*  left: 0;*/
/*  !*top: 60px;*!*/
/*  bottom: 0;*/
/*}*/
.grid-content[data-v-61dd7a3d] {
  border-radius: 4px;
  min-height: 36px;
  background: unset;
}

element.style {
  background: unset;
}

.zzStyle {
  width: 80%;
  margin: 20px;
}
.inSty{
  width: 80%;
}
/*.headerSelf{*/
/*  !*background-color: cornflowerblue ;*!*/
/*  height: 30px;*/
/*}*/
.headerSelf[data-v-61dd7a3d] {
  background-color: cornflowerblue;
  height: 0;
}
</style>
