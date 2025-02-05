<template>
  <div class="app-container">

    <!-- 搜索工作栏 -->
    <el-form :model="queryParams" ref="queryForm" :inline="true" v-show="showSearch" label-width="68px">
      <el-form-item label="流程名" prop="name">
        <el-input v-model="queryParams.name" placeholder="请输入流程名" clearable size="small" @keyup.enter.native="handleQuery"/>
      </el-form-item>
      <el-form-item label="创建时间">
        <el-date-picker v-model="dateRangeCreateTime" size="small" style="width: 240px" value-format="yyyy-MM-dd"
                        type="daterange" range-separator="-" start-placeholder="开始日期" end-placeholder="结束日期" />
      </el-form-item>
      <el-form-item>
        <el-button type="primary" icon="el-icon-search" size="mini" @click="handleQuery">搜索</el-button>
        <el-button icon="el-icon-refresh" size="mini" @click="resetQuery">重置</el-button>
      </el-form-item>
    </el-form>

    <!-- 列表 -->
    <el-table v-loading="loading" :data="list">
      <el-table-column label="任务编号" align="center" prop="id" width="320" />
      <el-table-column label="任务名称" align="center" prop="name" />
      <el-table-column label="所属流程" align="center" prop="processInstance.name" />
      <el-table-column label="流程发起人" align="center" prop="processInstance.startUserNickname" />
      <el-table-column label="创建时间" align="center" prop="createTime" width="180">
        <template slot-scope="scope">
          <span>{{ parseTime(scope.row.createTime) }}</span>
        </template>
      </el-table-column>
      <el-table-column label="状态" align="center" prop="version" width="80">
        <template slot-scope="scope">
          <el-tag type="success" v-if="scope.row.suspensionState === 1">激活</el-tag>
          <el-tag type="warning" v-if="scope.row.suspensionState === 2">挂起</el-tag>
        </template>
      </el-table-column>
      <el-table-column label="操作" align="center" class-name="small-padding fixed-width">
        <template slot-scope="scope">
          <!-- TODO 权限、颜色 -->
          <el-button size="mini" type="text" icon="el-icon-edit">审批</el-button>
          <el-button size="mini" type="text" icon="el-icon-edit" @click="audit(scope.row, true)">通过</el-button>
          <el-button size="mini" type="text" icon="el-icon-edit"  @click="audit(scope.row, false)">不通过</el-button>
          <el-button size="mini" type="text" icon="el-icon-edit" v-if="scope.row.suspensionState === 2">激活</el-button>
          <el-button size="mini" type="text" icon="el-icon-edit" v-if="scope.row.suspensionState === 1">挂起</el-button>
        </template>
      </el-table-column>
    </el-table>
    <!-- 分页组件 -->
    <pagination v-show="total > 0" :total="total" :page.sync="queryParams.pageNo" :limit.sync="queryParams.pageSize"
                @pagination="getList"/>

  </div>
</template>

<script>
import {approveTask, getTodoTaskPage, rejectTask} from '@/api/bpm/task'

export default {
  name: "Todo",
  components: {
  },
  data() {
    return {
      // 遮罩层
      loading: true,
      // 显示搜索条件
      showSearch: true,
      // 总条数
      total: 0,
      // 待办任务列表
      list: [],
      // 查询参数
      dateRangeCreateTime: [],
      queryParams: {
        pageNo: 1,
        pageSize: 10,
        name: null,
      },
    };
  },
  created() {
    this.getList();
  },
  methods: {
    /** 查询列表 */
    getList() {
      this.loading = true;
      // 处理查询参数
      let params = {...this.queryParams};
      this.addBeginAndEndTime(params, this.dateRangeCreateTime, 'createTime');
      getTodoTaskPage(params).then(response => {
        this.list = response.data.list;
        this.total = response.data.total;
        this.loading = false;
      });
    },
    /** 搜索按钮操作 */
    handleQuery() {
      this.queryParams.pageNo = 1;
      this.getList();
    },
    /** 重置按钮操作 */
    resetQuery() {
      this.dateRangeCreateTime = [];
      this.resetForm("queryForm");
      this.handleQuery();
    },
    audit(row, pass) {
      if (pass) {
        approveTask({
          id: row.id,
          comment: '通过'
        })
      } else {
        rejectTask({
          id: row.id,
          comment: '不通过'
        })
      }
    }
  }
};
</script>
