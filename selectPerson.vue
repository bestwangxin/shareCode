<template>
  <DialogModal :key="refreshKey" :slot-footer="true" :width="'1155px'" @cancel="cancel()" @ok="ok()" :title="title"
               :dialogVisible="dialogVisible">
    <div slot="content" class="popup clearfix" :class="!hasFooterFilter?'noBorderBottom':''">
      <div class="treeWrap">
        <el-input
            style="line-height: 40px"
            placeholder="输入搜索内容"
            @keyup.enter.native="searchGroup()"
            v-model="groupParams.searchWord">
          <i style="cursor: pointer" slot="suffix" @click="searchGroup" class="el-icon-search"></i>
        </el-input>
        <el-tree
            :default-expanded-keys="defaultExpand"
            node-key="name"
            @node-click="nodeClick"
            class="filter-tree"
            :props="props"
            :data="treeData"
            ref="tree">
        </el-tree>
      </div>
      <div class="tableWrap">
        <p>姓名</p>
        <el-input v-model="personParams.q_name" class="searchInput" placeholder="请输入"></el-input>
        <el-button @click="getTableData()" type="primary">查询</el-button>
        <el-button @click="reset()">重置</el-button>
        <el-table
            ref="multipleTable"
            :data="tableData"
            tooltip-effect="dark"
            :row-key="getRowKeys"
            style="width: 100%;margin-top: 16px;margin-bottom: 10px"
            @selection-change="handleSelectionChange"
            v-loading="loading"
        >
          <el-table-column
              type="selection"
              :reserve-selection="true"
              width="55">
          </el-table-column>
          <el-table-column
              label="姓名"
              width="120">
            <template slot-scope="scope">{{ scope.row.userName }}</template>
          </el-table-column>
          <el-table-column
              prop="position"
              label="职务"
              width="120">
          </el-table-column>
          <el-table-column
              prop="deptName"
              label="部门"
              show-overflow-tooltip>
          </el-table-column>
          <el-table-column
              prop="telephone"
              label="联系方式"
              show-overflow-tooltip>
          </el-table-column>
        </el-table>
        <el-pagination
            @current-change="getTableData"
            :current-page.sync="personParams.pageInfo.current"
            :page-size.sync="personParams.pageInfo.size"
            layout="->,prev, pager, next"
            :total="personParams.pageInfo.total">
        </el-pagination>
      </div>
      <div class="selectPersonList">
        <div class="title">
          已选人员
          <span @click="removePerson()" class="delete">全部移除</span>
        </div>
        <div class="personList">
          <p v-for="(item,index) in multipleSelection" :key="index">
            {{ item.userName }}
            <i @click="removePerson(item)" class="el-icon-delete"></i>
          </p>
        </div>
      </div>
    </div>
    <div slot="footer" class="dialog-footer">
      <div v-if="hasFooterFilter" class="filterWrap">
        <span>人员类型</span>
        <el-select style="width: 200px" v-model="roleType" placeholder="请选择">
          <el-option
              v-for="item in roleTypeList"
              :key="item.codeValue"
              :label="item.codeName"
              :value="item.codeValue">
          </el-option>
        </el-select>
        <span>值班频次</span>
        <el-select style="width: 200px" value="每日" placeholder="请选择">
          <el-option
              v-for="(item, key) in ['每日']"
              :key="key"
              :label="item"
              :value="item">
          </el-option>
        </el-select>
        <span>参加值班类型</span>
        <el-checkbox-group style="display: inline-block" v-model="dutyType">
          <el-checkbox label="weekday">工作日</el-checkbox>
          <el-checkbox label="weekend">周末</el-checkbox>
          <el-checkbox label="holiday">节假日</el-checkbox>
        </el-checkbox-group>
      </div>
      <el-button @click="cancel()">{{ btntitle[0] }}</el-button>
      <el-button type="primary" @click="ok()">{{ btntitle[1] }}</el-button>
    </div>
  </DialogModal>
</template>

<script>
import DialogModal from '@/components/dialog'
import {findPersonByGroup, getDeptmentList} from '@/api/system'

export default {
  name: "selectPerson",
  components: {DialogModal},
  props: {
    title: {
      type: String,
      default: '选择成员'
    },
    hasFooterFilter: {
      type: Boolean,
      default: true
    },
    relateInfo: {
      type: Number,
      default: 0
    },
    selectPersonList: {
      type: Array,
      default: () => {
        return []
      }
    },
    btntitle: {
      type: Array,
      default: () => {
        return ['取消', '引入']
      }
    },
    roleTypeList: {
      type: Array,
      default: () => {
        return []
      }
    },
    filterCityOrNot: {
      type: String,
      default: "0"
    },
  },

  data() {
    return {
      loading: false,
      refreshKey: new Date().getTime(),
      dialogVisible: false,
      filterText: '',
      roleType: '', // 人员类型
      treeData: [],
      tableData: [],
      props: {
        label: 'name',
        children: 'children',
      },
      multipleSelection: [],//选择的人员
      dutyType: [], //参加值班类型
      groupParams: {//获取组织机构参数
        relateInfo: 2,//组织关联位置(2-预案组织体系;3-预案专家队伍
        searchWord: '',
        filterCityOrNot: '',
      },
      personParams: {//获取人员参数
        pageInfo: {
          size: 10,
          current: 1,
          total: 0,
        },
        selectGroupInfo: {
          groupId: null,
          deptId: null
        },//选中的组织机构id
        q_name: ''
      },
      node: null,
      defaultExpand: [],

    }
  },
  watch: {
    dialogVisible: function (newv) {
      if (newv) {
        console.log(this.selectPersonList)
        this.refreshKey = new Date().getTime()
        this.$nextTick(() => {
          this.multipleSelection = []
          this.dutyType = []
          this.roleType = ""
          this.groupParams.relateInfo = this.relateInfo
          this.groupParams.filterCityOrNot = this.filterCityOrNot
          this.groupParams.searchWord = ''
          this.getDeptmentList(this.groupParams)
          this.multipleSelection = [...this.selectPersonList]
        })
      }
    }, immediate: true,
  },
  created() {
    // this.getDeptmentList()
  },
  mounted() {

  },
  methods: {
    getRowKeys(row) {
      return row.userId; // id为列表数据的唯一标识
    },
    getDeptmentList() {
      getDeptmentList(this.groupParams).then(res => {
        res.data.forEach(val => {
          this.recursion(val)
        })
        this.treeData = res.data
        this.defaultExpand = [this.treeData[0].name]
        this.$nextTick(() => {
          this.$refs.tree.setCurrentKey(this.treeData[0].name);
        })
        this.personParams.selectGroupInfo.groupId = this.treeData[0].groupId
        this.personParams.selectGroupInfo.deptId = this.treeData[0].deptId
        this.getTableData()
      })
    },
    recursion(val) {
      if (val.children) {
        val.children.forEach(child => {
          child.groupId = val.groupId
          this.recursion(child)
        })
      }
    },
    removePerson(item) {
      if (!item) {
        this.multipleSelection = []
        this.refreshSelect()
        return
      }
      let index
      this.multipleSelection.forEach((val, _index) => {
        if (val.userId == item.userId) {
          index = _index
        }
      })
      this.multipleSelection.splice(index, 1)
      this.refreshSelect()
    },
    searchGroup() {
      this.getDeptmentList()
    },
    getTableData(page) {
      this.loading = true
      if (page) {
        this.personParams.pageInfo.current = page
      }
      let params = {
        current: 1,
        size: 10,
        groupId: '',
        deptId: '',
        userName: ''
      }
      if (this.personParams.pageInfo) {
        params.current = this.personParams.pageInfo.current
        params.size = this.personParams.pageInfo.size
      }
      params.groupId = this.personParams.selectGroupInfo.groupId
      if (this.personParams.selectGroupInfo.deptId) {
        params.deptId = this.personParams.selectGroupInfo.deptId
      } else {
        delete params.deptId
      }
      params.userName = this.personParams.q_name
      findPersonByGroup(params).then(res => {
        this.tableData = res.data
        this.personParams.pageInfo.total = res.totalCount
        this.$nextTick(() => {
          /*this.tableData.forEach((item)=>{
            console.log(this.multipleSelection)
            this.multipleSelection.forEach((multItem)=>{
              if(item.userId == multItem.userId){
                console.log(item.userId)
                this.$refs.multipleTable.toggleRowSelection(item,true);//这步操作是让  页面显示选中的数据
              }
            })
          })*/
          this.refreshSelect()
          this.loading = false
        })


      })
    },
    nodeClick(data, node, component) {
      this.personParams.selectGroupInfo.groupId = node.data.groupId
      this.personParams.selectGroupInfo.deptId = node.data.deptId
      this.getTableData()
      console.log(component)
    },

    refreshSelect() {
      let _select = []
      let _noSelect = []
      this.$refs.multipleTable.tableData.forEach(val => {
        if (this.multipleSelection.find(select => {
          return val.userId == select.userId
        })) {
          _select.push(val)
        } else {
          _noSelect.push(val)
        }
      })
      _select.forEach(row => {
        this.$refs.multipleTable.toggleRowSelection(row, true);
      });
      _noSelect.forEach(row => {
        this.$refs.multipleTable.toggleRowSelection(row, false);
      });
    },
    cancel() {
      this.dialogVisible = false
    },
    ok() {
      if (this.hasFooterFilter) {
        if (!this.roleType || this.dutyType.length == 0) {
          this.$message.warning('请填写人员类型和值班类型')
          return
        }
        const data = {
          multipleSelection: this.multipleSelection,
          dutyType: this.dutyType,
          roleType: this.roleType
        }
        this.$emit('ok', data)
      } else {
        this.$emit('ok', this.multipleSelection)
      }
      this.dialogVisible = false
    },
    handleNodeClick(data) {
      console.log(data);
    },
    toggleSelection(rows) {
      if (rows) {
        rows.forEach(row => {
          this.$refs.multipleTable.toggleRowSelection(row);
        });
      } else {
        this.$refs.multipleTable.clearSelection();
      }
    },
    handleSelectionChange(val) {
      //此处由于编辑的时候有已选择的人员传进来，
      // element Table :reserve-selection="true"和:row-key="getRowKeys"的组合虽然在你手动勾选时能记住分页（即数据刷新后的）勾选的数据，
      // 但无法记住你编辑传进来的默认值，所以此处需要手动合并数组并去重
      //如果没有传进来的默认数据，那么只需要一行代码this.multipleSelection = val即可，无需下面一大堆
      let arr = this.multipleSelection.concat(val)
      let hash = {};
      //去重
      arr = arr.reduce(function (item, next) {
        hash[next.userId] ? '' : hash[next.userId] = true && item.push(next);
        return item
      }, [])
      //获取当前页未被选择的项
      let _select = []
      let _noSelect = []
      this.$refs.multipleTable.tableData.forEach(data => {
        if (val.find(select => {
          return data.userId == select.userId
        })) {
          _select.push(data)
        } else {
          _noSelect.push(data)
        }
      })
      //如果当前页未被选择的项存在于总回显数据中，将其删除
      _noSelect.forEach(val => {
        if (arr.find(select => {
          return val.userId == select.userId
        })) {
          arr.splice(arr.findIndex(item => item.userId === val.userId), 1)
        }
      })
      this.multipleSelection = arr

    },
    reset() {
      this.personParams.q_name = ''
      this.getTableData()
    }
  }
  ,
}
</script>

<style scoped lang='scss'>
.popup {
  //去掉叶子节点前的三角符号
  ::v-deep .is-leaf {
    &::before {
      color: transparent !important;
    }
  }

  padding: 24px;

  .treeWrap {
    width: 282px;
    height: 436px;
    border: 1px solid #e6e6e6;
    border-radius: 2px;
    padding: 12px;
    overflow-y: auto;
    float: left;
  }

  .filter-tree {
    margin-top: 10px;
  }

  .treeWrap::-webkit-scrollbar {
    /*滚动条整体样式*/
    width: 10px; /*高宽分别对应横竖滚动条的尺寸*/
    height: 1px;
  }

  .treeWrap::-webkit-scrollbar-thumb {
    /*滚动条里面小方块*/
    border-radius: 10px;
    box-shadow: inset 0 0 5px rgba(0, 0, 0, 0.2);
    background: #8b8787;
  }

  .treeWrap::-webkit-scrollbar-track {
    /*滚动条里面轨道*/
    box-shadow: inset 0 0 5px rgba(0, 0, 0, 0.2);
    border-radius: 10px;
    background: #ededed;
  }

  ::v-deep.el-tree-node__content {
    height: 28px !important;
  }

  ::v-deep.el-tree-node.is-focusable.is-current > .el-tree-node__content::after {
    display: none;
  }

  ::v-deep {
    .el-tree-node.is-focusable.is-current > .el-tree-node__content {
      color: #3786FD;
      background: rgba(55, 134, 253, 0.12);

      .el-icon-caret-right {
        &:before {
          color: #3786FD;
        }
      }
    }
  }

  .tableWrap {
    width: 540px;
    height: 436px;
    float: left;
    padding: 0 24px;
    overflow: auto;

    > p {
      &:nth-of-type(1) {
        margin-bottom: 10px;
      }
    }

    .searchInput {
      width: 255px;
      margin-right: 16px;
    }
  }

  .selectPersonList {
    float: left;
    width: 282px;
    height: 436px;
    border: 1px solid #e6e6e6;
    border-radius: 2px;
    overflow-y: auto;

    .title {
      height: 43px;
      line-height: 43px;
      font-size: 14px;
      color: rgba(0, 0, 0, 0.85);
      padding: 0 16px;
      border-bottom: 1px solid #e6e6e6;

      .delete {
        float: right;
        color: #F06060;
        cursor: pointer;
      }

    }

    .personList {
      overflow: auto;

      p {
        line-height: 28px;
        width: 250px;
        margin: 0 auto;
        padding: 0 16px;

        &:hover {
          background: rgba(55, 134, 253, 0.12);

          i {
            display: block;
          }
        }

        &:nth-of-type(1) {
          margin-top: 10px;
        }

        i {
          float: right;
          line-height: 28px;
          cursor: pointer;
          display: none;
        }
      }
    }
  }

  border-bottom: 1px solid #e6e6e6;

  &.noBorderBottom {
    border-bottom: none;
  }

  margin-bottom: 20px;


}

.filterWrap {
  padding-left: 25px;

  > span {
    margin: 0 15px;
  }
}
</style>
