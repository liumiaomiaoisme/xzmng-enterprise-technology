<template>
  <div class="team-menagement-container">
    <!--  表单查询  -->
    <el-form :model="searchForm" class="search-container" size="mini" ref="searchForm">
      <el-form-item prop="tecGroupName">
        <el-input v-model="searchForm.tecGroupName" prefix-icon="el-icon-search" placeholder="请输入组名"></el-input>
      </el-form-item>
      <el-form-item prop="tecGroupType">
        <el-select placeholder="请选择组类型" v-model="searchForm.tecGroupType" clearable>
          <el-option :label="item.groupTypeName" :value="item.groupTypeId" v-for="item in groupType" :key="item.groupTypeId"></el-option>
        </el-select>
      </el-form-item>
      <el-form-item prop="tecGroupLevel">
        <el-select v-model="searchForm.tecGroupLevel" placeholder="请选择组等级" clearable>
          <el-option label="非正式组" value="0"></el-option>
          <el-option label="正式组" value="1"></el-option>
        </el-select>
      </el-form-item>
      <el-form-item prop="tecGroupLeaderName">
        <el-input  v-model="searchForm.tecGroupLeaderName" prefix-icon="el-icon-search" placeholder="请输入组长"></el-input>
      </el-form-item>
      <el-form-item prop="tecGroupDeputyName">
        <el-input  v-model="searchForm.tecGroupDeputyName" prefix-icon="el-icon-search" placeholder="请输入副组长"></el-input>
      </el-form-item>
      <el-form-item prop="member">
        <el-input  v-model="searchForm.member" prefix-icon="el-icon-search" placeholder="请输入成员"></el-input>
      </el-form-item>
      <el-form-item prop="createDate" class="date-form">
        <el-date-picker
          v-model="searchForm.createDate"
          type="daterange"
          range-separator="至"
          start-placeholder="创建日期开始日期"
          end-placeholder="创建日期结束日期">
        </el-date-picker>
      </el-form-item>
      <el-form-item prop="expiredDate" class="date-form">
        <el-date-picker
          v-model="searchForm.expiredDate"
          type="daterange"
          range-separator="至"
          start-placeholder="过期日期开始日期"
          end-placeholder="过期日期结束日期">
        </el-date-picker>
      </el-form-item>
      <el-button type="primary" size="mini" icon="el-icon-search" @click="searchTeam" plain>查询</el-button>
      <el-button type="info" size="mini" icon="el-icon-refresh" plain @click="resetSearch('searchFormA')">重置</el-button>
    </el-form>
    <!--  表单  -->
    <el-button @click="openAddDialog" type="primary" size="mini" icon="el-icon-circle-plus">添加</el-button>
    <el-table :data="tableData" stripe border fit v-loading="loading">
      <el-table-column fixed prop="tecGroupName" label="组名称"></el-table-column>
      <el-table-column prop="groupTypeName" label="类型"></el-table-column>
      <el-table-column prop="tecGroupImg" label="组头像" width="100">
        <template slot-scope="scope">
           <img :src="scope.row.tecGroupImg" class="team-avatar"/>
        </template>
      </el-table-column>
      <el-table-column prop="leaderName" label="组长"></el-table-column>
      <el-table-column prop="deputyName" label="副组长"></el-table-column>
      <el-table-column prop="member" label="成员" width="180"></el-table-column>
      <el-table-column prop="tecGroupCount" label="人数" width="80"></el-table-column>
      <el-table-column prop="tecGroupCreateDate" label="创建时间" >
        <template slot-scope="scope">
           <span v-html="scope.row.tecGroupCreateDate"></span>
        </template>
      </el-table-column>
      <el-table-column prop="tecGroupExpiredDate" label="过期时间">
        <template slot-scope="scope">
           <span v-html="scope.row.tecGroupExpiredDate"></span>
        </template>
      </el-table-column>
      <el-table-column fixed="right" label="操作" width="88">
        <template slot-scope="scope">
          <el-button type="primary" icon="el-icon-edit" circle size="mini" @click="openEditDialog(scope.row.tecGroupId)"></el-button>
          <el-button type="danger" icon="el-icon-delete" circle size="mini" @click="deleteTeam(scope.row.tecGroupId)"></el-button>
        </template>
      </el-table-column>
    </el-table>
    <el-pagination
      background :page-size="pageSize"
      layout="total, prev, pager, next, jumper"
      :total="totalItem"
      @next-click="getNextPage" @prev-click="getPrevPage" @current-change="getCurrentPage">
    </el-pagination>
    <add-dialog :handler="addDialogFormVisible" @closeDialog="closeAddDialog"></add-dialog>
    <edit-dialog :handler="editDialogFormVisible" :form="form" @closeDialog="closeEditDialog"></edit-dialog>
  </div>
</template>

<script>
import addDialog from '@/views/AppMain/MainModules/TeamManage/component/addTeam.vue'
import editDialog from '@/views/AppMain/MainModules/TeamManage/component/editTeam.vue'
export default {
  data () {
    return {
      currentPage: 1,
      pageSize: 10,
      loading: false,
      searchForm: {
        tecGroupName: '',
        tecGroupType: '',
        tecGroupLeaderName: '',
        tecGroupDeputyName: '',
        tecGroupLevel: '',
        member: '',
        createDate: '',
        expiredDate: '',
        createStartDate: '',
        createEndDate: '',
        expiredStartDate: '',
        expiredEndDate: ''
      },
      tableData: [],
      totalItem: 0,
      addDialogFormVisible: false,
      editDialogFormVisible: false,
      form: {
        tecGroupName: '',
        tecGroupImg: '',
        tecGroupLevel: null,
        tecGroupLeader: null,
        tecGroupDeputy: null,
        memeberIds: '',
        tecGroupCount: null,
        tecGroupType: null,
        tecGroupExpiredDate: '',
        tecGroupId: null
      },
      relativePath: '',
      formLabelWidth: '100px',
      staffList: [],
      groupType: [],
      rules: {
        tecGroupName: [
          { required: true, message: '组名不能为空', trigger: 'blur' }
        ],
        tecGroupImg: [
          { required: true, message: '请上传组头像', trigger: 'change' }
        ],
        tecGroupLevel: [
          { required: true, message: '请选择等级', trigger: 'blur' }
        ],
        tecGroupLeader: [
          { required: true, message: '请选择组长', trigger: 'blur' }
        ],
        tecGroupDeputy: [
          { required: true, message: '请选择副组长', trigger: 'blur' }
        ],
        memeberIds: [
          { required: true, message: '请选择成员', trigger: 'blur' }
        ],
        tecGroupType: [
          { required: true, message: '请选择组类型', trigger: 'blur' }
        ],
        tecGroupExpiredDate: [
          { type: 'date', required: true, message: '请选择日期', trigger: 'change' }
        ]
      }
    }
  },
  created () {
    this.getTeamList()
    this.getAllMenber()
  },
  methods: {
    searchTeam () {
      let searchFormSend = { ...this.searchForm }
      if (searchFormSend.tecGroupLevel) {
        searchFormSend.tecGroupLevel = parseInt(searchFormSend.tecGroupLevel)
      }
      if (searchFormSend.createDate) {
        searchFormSend.createStartDate = this.getFormatTime(searchFormSend.createDate[0])
        searchFormSend.createEndDate = this.getFormatTime(searchFormSend.createDate[1])
      }
      if (searchFormSend.expiredDate) {
        searchFormSend.expiredStartDate = this.getFormatTime(searchFormSend.expiredDate[0])
        searchFormSend.expiredEndDate = this.getFormatTime(searchFormSend.expiredDate[1])
      }
      this.loading = true
      this.$axios.fetchGet('/api/group/lists', searchFormSend)
        .then(res => {
          if (res.data.statuscode === 200) {
            let resList = res.data.content.list
            for (let i of resList) {
              i.tecGroupCreateDate = i.tecGroupCreateDate.replace(/\s/g, '<br>')
              i.tecGroupExpiredDate = i.tecGroupExpiredDate.replace(/\s/g, '<br>')
            }
            this.tableData = resList
            this.totalItem = res.data.content.totalCount
          } else if (res.data.statuscode === 400) {
            this.tableData = []
          }
          this.loading = false
        })
    },
    // 重置表单方法
    resetSearch () {
      this.getTeamList()
      this.$refs['searchForm'].resetFields()
    },
    openAddDialog () {
      this.addDialogFormVisible = true
    },
    closeAddDialog () {
      this.addDialogFormVisible = false
    },
    getAllMenber () {
      this.$axios.fetchGet('api/emp/tecEmp')
        .then(res => {
          if (res.data.statuscode === 200) {
            this.staffList = res.data.content
          }
        })
      this.$axios.fetchGet('api/group/type')
        .then(res => {
          if (res.data.statuscode === 200) {
            this.groupType = res.data.content
          }
        })
    },
    openEditDialog (id) {
      this.$axios.fetchGet('api/group/list/' + id)
        .then(res => {
          if (res.data.statuscode === 200) {
            let editForm = res.data.content[0]
            let memberArr = editForm.member.split(',')
            let indexArr
            indexArr = this.staffList.filter((item) => {
              for (let i of memberArr) {
                if (item.empName.includes(i)) {
                  return item
                }
              }
            })
            let members = []
            for (let i of indexArr) {
              members.push(i.empId)
            }
            let deputy = null
            let leader = null
            for (let i of this.staffList) {
              if (i.empName === editForm.leaderName) {
                leader = i.empId
              }
              if (i.empName === editForm.deputyName) {
                deputy = i.empId
              }
            }
            let typeId = null
            for (let i of this.groupType) {
              if (i.groupTypeName === editForm.groupTypeName) {
                typeId = i.groupTypeId
              }
            }
            this.form = {
              tecGroupName: editForm.tecGroupName,
              tecGroupImg: editForm.tecGroupImg,
              tecGroupLevel: editForm.tecGroupLevel === 1 ? '正式组' : '非正式组',
              tecGroupLeader: leader,
              tecGroupDeputy: deputy,
              memeberIds: members,
              tecGroupType: parseInt(typeId),
              tecGroupExpiredDate: new Date(editForm.tecGroupExpiredDate),
              tecGroupId: editForm.tecGroupId
            }
          }
        })
      this.editDialogFormVisible = true
    },
    closeEditDialog () {
      this.editDialogFormVisible = false
    },
    deleteTeam (id) {
      this.$confirm('此操作将永久删除该小组, 是否继续?', '提示', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        this.$axios.fetchPost('api/group/delete/' + id)
          .then(res => {
            if (res.data.statuscode === 200) {
              this.getTeamList()
              this.$message({
                type: 'success',
                message: '删除成功'
              })
            }
          })
      }).catch(() => {
        this.$message({
          type: 'info',
          message: '已取消删除'
        })
      })
    },
    getTeamList (currentPage) {
      this.loading = true
      this.$axios.fetchGet('api/group/lists', {
        currentPage: currentPage,
        pageSize: this.pageSize
      })
        .then(res => {
          if (res.data.statuscode === 200) {
            let resList = res.data.content.list
            for (let i of resList) {
              i.tecGroupCreateDate = i.tecGroupCreateDate.replace(/\s/g, '<br>')
              i.tecGroupExpiredDate = i.tecGroupExpiredDate.replace(/\s/g, '<br>')
            }
            this.tableData = resList
            this.totalItem = res.data.content.totalCount
          }
          this.loading = false
        })
    },
    getNextPage () {
      this.currentPage++
      this.getTeamList(this.currentPage)
    },
    getPrevPage () {
      this.currentPage--
      this.getTeamList(this.currentPage)
    },
    getCurrentPage (val) {
      this.currentPage = val
      this.getTeamList(this.currentPage)
    },
    getFormatTime (timestr) {
      let newTime = new Date(timestr)
      let y = newTime.getFullYear()
      let mo = (newTime.getMonth() + 1).toString().padStart(2, '0')
      let d = newTime.getDate().toString().padStart(2, '0')
      let h = newTime.getHours().toString().padStart(2, '0')
      let mi = newTime.getMinutes().toString().padStart(2, '0')
      let s = newTime.getSeconds().toString().padStart(2, '0')
      return `${y}-${mo}-${d} ${h}:${mi}:${s}`
    }
  },
  components: {
    'add-dialog': addDialog,
    'edit-dialog': editDialog
  }
}

</script>

<style lang="scss">
  .team-menagement-container{
    width: 100%;
    position: relative;
    .search-container{
      .el-form-item{
        display: inline-block;
        width:172px;
        margin-right: 6px;
        margin-bottom: 4px;
        &.date-form{
          width:350px;
        }
      }
      margin-bottom: 10px;
    }
    .el-table{
      margin-top:20px;
      width: 99.9%;
    }
    .team-avatar {
      width:50px;
      height: 50px;
     }
    .el-table td, .el-table th {
      padding: 4px 0!important;
    }
    .el-pagination{
      position: absolute;
      left:50%;
      transform: translateX(-50%);
      bottom: -60px;
    }
    .avatar-uploader .el-upload {
    border: 1px dashed #d9d9d9;
    border-radius: 6px;
    cursor: pointer;
    position: relative;
    overflow: hidden;
  }
  .avatar-uploader .el-upload:hover {
    border-color: #409EFF;
  }
  .avatar-uploader-icon {
    font-size: 28px;
    color: #8c939d;
    width: 50px;
    height: 50px;
    line-height: 50px;
    text-align: center;
  }
  .avatar {
    width: 50px;
    height: 50px;
    display: block;
  }
    .add-team-container{
      .el-dialog{
        box-sizing: border-box;
        min-width: 600px;
        width: 40%;
      }
      .el-input,.el-date-picker{
        width: 400px;
      }
    }
  }

</style>
