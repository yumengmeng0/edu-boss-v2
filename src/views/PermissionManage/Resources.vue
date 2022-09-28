<template>
  <div class="app-container">
    <el-card class="filter-container" shadow="never">
      <div>
        <el-button
          style="float:right"
          type="primary"
          @click="handleSearchList()"
          size="small"
          >查询搜索</el-button
        >
        <el-button
          style="float:right;margin-right: 15px"
          @click="handleResetSearch()"
          size="small"
          >重置</el-button
        >
      </div>
      <div style="margin-top: 15px">
        <el-form
          :inline="true"
          :model="listQuery"
          size="small"
          label-width="140px"
        >
          <el-form-item label="资源名称：">
            <el-input
              v-model="listQuery.name"
              class="input-width"
              placeholder="资源名称"
              clearable
            ></el-input>
          </el-form-item>
          <el-form-item label="资源路径：">
            <el-input
              v-model="listQuery.url"
              class="input-width"
              placeholder="资源路径"
              clearable
            ></el-input>
          </el-form-item>
          <el-form-item label="资源分类：">
            <el-select
              v-model="listQuery.categoryId"
              placeholder="全部"
              clearable
              class="input-width"
            >
              <el-option
                v-for="item in categoryOptions"
                :key="item.id"
                :label="item.name"
                :value="item.id"
              ></el-option>
            </el-select>
          </el-form-item>
        </el-form>
      </div>
    </el-card>
    <el-card class="operate-container" shadow="never">
      <el-button
        size="mini"
        class="btn-add"
        @click="handleAdd()"
        style="margin-left: 20px"
        >添加</el-button
      >
      <el-button size="mini" class="btn-add" @click="handleShowCategory()"
        >资源分类</el-button
      >
    </el-card>
    <div class="table-container">
      <el-table
        ref="resourceTable"
        :data="list"
        style="width: 100%;"
        v-loading="listLoading"
        border
      >
        <el-table-column label="编号" width="100" align="center">
          <template slot-scope="scope">{{ scope.row.id }}</template>
        </el-table-column>
        <el-table-column label="资源名称" align="center">
          <template slot-scope="scope">{{ scope.row.name }}</template>
        </el-table-column>
        <el-table-column label="资源路径" align="center">
          <template slot-scope="scope">{{ scope.row.url }}</template>
        </el-table-column>
        <el-table-column label="描述" align="center">
          <template slot-scope="scope">{{ scope.row.description }}</template>
        </el-table-column>
        <el-table-column label="添加时间" width="180" align="center">
          <template slot-scope="scope">{{
            scope.row.createdTime | formatDateTime
          }}</template>
        </el-table-column>
        <el-table-column label="操作" width="140" align="center">
          <template slot-scope="scope">
            <el-button size="mini" type="text" @click="handleUpdate(scope.row)"
              >编辑</el-button
            >
            <el-button size="mini" type="text" @click="handleDelete(scope.row)"
              >删除</el-button
            >
          </template>
        </el-table-column>
      </el-table>
    </div>
    <div class="pagination-container">
      <el-pagination
        background
        @size-change="handlePageSizeChange"
        @current-change="handleCurrentPageChange"
        layout="total, sizes, prev, pager, next, jumper"
        :current-page="listQuery.currentPage"
        :page-sizes="[5, 10, 20]"
        :page-size="listQuery.pageSize"
        :total="total"
      ></el-pagination>
    </div>

    <!-- 添加&修改表单 -->
    <el-dialog
      :title="isEdit ? '编辑资源' : '添加资源'"
      :visible.sync="dialogVisible"
      width="40%"
    >
      <el-form
        :model="resource"
        ref="resourceForm"
        label-width="150px"
        size="small"
      >
        <el-form-item label="资源名称：">
          <el-input v-model="resource.name" style="width: 250px"></el-input>
        </el-form-item>
        <el-form-item label="资源路径：">
          <el-input v-model="resource.url" style="width: 250px"></el-input>
        </el-form-item>
        <el-form-item label="资源分类：">
          <el-select
            v-model="resource.categoryId"
            placeholder="全部"
            clearable
            style="width: 250px"
          >
            <el-option
              v-for="item in categoryOptions"
              :key="item.id"
              :label="item.name"
              :value="item.value"
            ></el-option>
          </el-select>
        </el-form-item>
        <el-form-item label="描述：">
          <el-input
            v-model="resource.description"
            type="textarea"
            :rows="5"
            style="width: 250px"
          ></el-input>
        </el-form-item>
      </el-form>
      <span slot="footer" class="dialog-footer">
        <el-button @click="dialogVisible = false" size="small">取 消</el-button>
        <el-button type="primary" @click="handleSave()" size="small"
          >确 定</el-button
        >
      </span>
    </el-dialog>
  </div>
</template>

<script scope>
import { formatDate } from "@/utils/date";
import r from "highlight.js/lib/languages/r";
import { axios } from "../../utils";

//查询条件
let defaultListQuery = {
  currentPage: 1,
  pageSize: 5,
  name: null,
  url: null,
  categoryId: null
};

//资源对象
const defaultResource = {
  id: null,
  name: null,
  url: null,
  categoryId: null,
  description: ""
};

export default {
  name: "resourceList",
  title: "资源管理",
  data() {
    return {
      listQuery: Object.assign({}, defaultListQuery), //查询条件
      total: 0,
      list: [], //资源数据
      cateList: [], //资源分类数据
      listLoading: false,
      dialogVisible: false,
      resource: Object.assign({}, defaultResource),
      isEdit: false,
      categoryOptions: [], // 搜索框资源分类下拉选项
      defaultCategoryId: null
    };
  },
  //钩子函数
  created() {
    //获取资源数据
    this.getResourceList();

    //获取资源分类数据
    this.getResourceCateList();
  },
  methods: {
    //方法1: 获取资源数据
    getResourceList() {
      axios
        .post("/resource/findAllResource", this.listQuery)
        .then(response => {
          // console.log(response.data.content);
          this.list = response.data.content.list;
          this.total = response.data.content.total;
        })
        .catch(error => {
          this.$message.error("获取资源数据失败！");
        });
    },

    //方法2: 获取资源分类数据
    getResourceCateList() {
      axios
        .post("/resourceCategory/findAllResourceCategory")
        .then(response => {
          // console.log(response.data.content);
          this.cateList = response.data.content;
          // this.categoryOptions = response.data.content;

          for (let i = 0; i < this.cateList.length; i++) {
            const cate = this.cateList[i];
            this.categoryOptions.push({ value: cate.id, name: cate.name });
          }
        })
        .catch(error => {
          this.$message.error("获取资源分类数据失败！");
        });
    },

    //页码变化触发的函数
    handleCurrentPageChange(page) {
      this.listQuery.currentPage = page;
      this.getResourceList();
    },

    //每页显示条数变化触发的函数
    handlePageSizeChange(size) {
      this.listQuery.pageSize = size;
      this.getResourceList();
    },

    //查询按钮
    handleSearchList() {
      console.log(this.listQuery);
      this.getResourceList();
    },

    //添加资源回显
    handleAdd() {
      this.resource = Object.assign({}, defaultListQuery);
      this.dialogVisible = true;
      this.isEdit = false;
    },

    //添加&修改资源
    handleSave() {
      return axios
        .post("/resource/saveOrUpdateResource", this.resource)
        .then(response => {
          // console.log(response.data.content);
          this.dialogVisible = false;
          this.getResourceList();
        })
        .catch(error => {
          this.$message.error("更新资源数据失败！");
        });
    },

    //编辑资源 回显
    handleUpdate(row) {
      this.dialogVisible = true;
      this.isEdit = true;
      // console.log(row)
      this.resource = Object.assign({}, row);
      // return axios
      //   .get("/resource/findResourceById?id=" + row.id)
      //   .then(response => {
      //     this.resource = response.data.content;
      //   })
      //   .catch(error => {
      //     this.$message.error("获取资源数据失败！");
      //   });
    },

    //删除资源
    handleDelete(row) {
      return axios
        .get("/resource/deleteResource?id=" + row.id)
        .then(response => {
          this.dialogVisible = false;
          this.getResourceList();
        })
        .catch(error => {
          this.$message.error("删除资源数据失败！");
        });
    },

    //资源分类管理
    handleShowCategory() {
      this.$router.push({ path: "/resourceCategory" });
    },

    handleResetSearch() {
      // console.log(defaultListQuery)
      this.listQuery = Object.assign({}, defaultListQuery);
    }
  },

  filters: {
    formatDateTime(time) {
      if (time == null || time === "") {
        return "N/A";
      }
      const date = new Date(time);
      return formatDate(date, "yyyy-MM-dd hh:mm:ss");
    }
  }
};
</script>

<style></style>
