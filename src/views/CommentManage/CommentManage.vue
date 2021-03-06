<template>
	<el-container class="sub-page comment-manage">
		<el-header>
			<SubHeader :pageTitle="pageTitle"></SubHeader>
			<ConditionFilter showDatePicker>
				<div class="select">
					<span class="select-label">游戏名称：</span>
					<el-select size="small" v-model="searchParams.gameId" placeholder="请选择" @change="handleGameSelect" clearable filterable>
						<el-option v-for="(item,index) in gameList" :key="index" :label="item.game_name" :value="item.id">
						</el-option>
					</el-select>
				</div>
			</ConditionFilter>
		</el-header>
		<el-scrollbar style="height:100%;">
			<el-main>
				<el-table v-loading="loading" ref="multipleTable" :data="commentList" @selection-change="handleSelectionChange" border stripe tooltip-effect="light" style="width: 100%">
					<el-table-column type="selection" width="55">
					</el-table-column>
					<el-table-column prop="username" show-overflow-tooltip label="用户姓名" width="160">
					</el-table-column>
					<el-table-column label="用户头像" width="100">
						<template slot-scope="scope">
							<img :src="scope.row.user_avatar == '/public/system/default_avatar.jpg' ? `${baseUrl}${scope.row.user_avatar}`:scope.row.user_avatar" alt="" width="30" height="30">
						</template>
					</el-table-column>
					<el-table-column prop="game_name" show-overflow-tooltip label="游戏名称" width="200">
					</el-table-column>
					<el-table-column prop="comment_content" label="评论内容">
					</el-table-column>
					<el-table-column label="评论时间" show-overflow-tooltip width="200">
						<template slot-scope="scope">{{ scope.row.create_date | formatDate }}</template>
					</el-table-column>
					<el-table-column label="操作" width="80">
						<template slot-scope="scope">
							<el-button size="mini" type="danger" @click="handleDelete(scope.row)" v-permission>删除</el-button>
						</template>
					</el-table-column>
				</el-table>
			</el-main>
		</el-scrollbar>
		<el-footer>
			<SubFooter ref="subFooter" showDelete :showPagination="Boolean(commentTotal)" :total="commentTotal" @refreshData="refreshData" @handleDelete="handleDelete"></SubFooter>
		</el-footer>
	</el-container>
</template>

<script type='es6'>
import { baseUrl } from "utils/env";
import userService from "http/userService";
import SubHeader from "components/SubHeader";
import SubFooter from "components/SubFooter";
import ConditionFilter from "components/ConditionFilter";
export default {
  name: "",
  data() {
    return {
      baseUrl,
      loading: false,
      pageTitle: "评论管理",
      searchParams: {
        gameId: "",
        pageSize: 16,
        currentPage: 1
      },
      gameList: [],
      commentList: [],
      commentTotal: 0,
      multipleSelection: []
    };
  },
  created() {
    this.getGameCommentList(this.searchParams);
    this.getGameList();
  },
  methods: {
    getGameList() {
      //获取所有游戏，isFilter用于根据游戏名进行筛选
      userService
        .getRequest("getGameList", { isFilter: 1 })
        .then(response => {
          this.gameList = response.data.list;
        })
        .catch(error => {});
    },
    getGameCommentList(searchParams) {
      this.loading = true;
      userService
        .getRequest("getGameCommentList", searchParams)
        .then(response => {
          this.commentList = response.data.list;
          this.commentTotal = response.data.total;
          this.loading = false;
        })
        .catch(error => {});
    },
    handleGameSelect() {
      this.searchParams.currentPage = 1;
      this.getGameCommentList(this.searchParams);
      this.$refs.subFooter.ininPageConfig();
    },
    refreshData(searchParams, isConditionSearch) {
      if (searchParams.selectValue) {
        searchParams.gameId = searchParams.selectValue;
        delete searchParams.selectValue;
      }
      //如果是根据条件查询触发的此方法 应调用subFoot组件中的ininPageConfig方法来初始化分页参数
      Object.assign(this.searchParams, searchParams);
      this.getGameCommentList(this.searchParams);

      isConditionSearch && this.$refs.subFooter.ininPageConfig();
    },
    handleDelete(log) {
      let ids = [];
      if (!log) {
        this.multipleSelection.length &&
          this.multipleSelection.map(item => {
            ids.push(item.id);
          });
      }
      let idsLength = ids.length;
      ids = ids.length ? ids.join(",") : log.id;

      this.$confirm("您确定是否要删除此记录?", "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning"
      })
        .then(() => {
          userService
            .deleteRequest("deleteGameComments", ids)
            .then(response => {
              this.$message({
                type: "success",
                message: "删除成功!"
              });
              if (
                idsLength === this.commentList.length &&
                this.searchParams.currentPage > 1
              ) {
                this.searchParams.currentPage =
                  this.searchParams.currentPage - 1;
              }
              this.getGameCommentList(this.searchParams);
            })
            .catch(error => {});
        })
        .catch(() => {});
    },
    handleSelectionChange(val) {
      this.multipleSelection = val;
    }
  },
  components: {
    SubHeader,
    SubFooter,
    ConditionFilter
  }
};
</script>

<style lang='stylus' scoped>
</style>
