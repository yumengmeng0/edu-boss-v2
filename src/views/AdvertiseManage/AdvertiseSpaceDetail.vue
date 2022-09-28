<template>
  <el-card class="form-container" shadow="never">
    <el-form
      :model="homeAdvertise"
      :rules="rules"
      ref="form"
      label-width="150px"
      size="small"
    >
      <el-form-item label="广告位名称：" prop="name">
        <el-input v-model="homeAdvertise.name" class="input-width"></el-input>
      </el-form-item>
      <el-form-item>
        <el-button type="primary" @click="handleSave()">提交</el-button>
      </el-form-item>
    </el-form>
  </el-card>
</template>
<script>
import { axios } from "../../utils";

export default {
  name: "HomeAdvertiseDetail",
  //组件传参
  props: {
    isEdit: {
      type: Boolean,
      default: false
    }
  },
  data() {
    return {
      homeAdvertise: { name: null, id: null },
      rules: {
        name: [
          { required: true, message: "请输入广告名称", trigger: "blur" },
          {
            min: 2,
            max: 140,
            message: "长度在 2 到 140 个字符",
            trigger: "blur"
          }
        ]
      }
    };
  },

  //钩子函数
  created() {
    console.log(this.$route.query.id);
    this.loadPromotionSpace(this.$route.query.id); // 根据路由传递的参数查询广告位信息
  },

  methods: {
    //方法1: 保存广告位信息
    handleSave() {
      // console.log(this.homeAdvertise);

      axios
        .post("/promotionSpace/saveOrUpdatePromotionSpace", {
          name: this.homeAdvertise.name,
          id: this.homeAdvertise.id
        })
        .then(resp => {})
        .catch(error => {
          this.$message.error("添加广告位失败！");
        });
    },

    //方法2: 回显广告位信息（有可能是添加或更新）
    loadPromotionSpace(id) {
      //
      if (id === undefined) {  // 路由没有传id就是新建广告位
        // alert("添加广告位");
        this.homeAdvertise = {
          name: ""
        };
        console.log(this.homeAdvertise);
      } else {
        // alert("修改广告位");
        axios
          .get("/promotionSpace/findPromotionSpaceById", {
            params: {
              id: id
            }
          })
          .then(resp => {
            this.homeAdvertise.name = resp.data.content.name;
            this.homeAdvertise.id = resp.data.content.id;
            console.log(this.homeAdvertise);
          })
          .catch(error => {
            this.$message.error("查询广告位失败！");
          });
      }
    }
  }
};
</script>
<style scoped>
.input-width {
  width: 70%;
}
</style>
