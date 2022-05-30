<template>
  <div>
    <el-row type="flex" justify="space-between" align="middle">
      <el-col>
        <el-switch
          v-model="draggable"
          active-text="开启拖拽"
          inactive-text="关闭拖拽"
        >
        </el-switch>
      </el-col>
      <el-col align="right">
        <el-button type="primary" v-show="draggable" @click="saveDropNode"
          >保存修改</el-button
        >
        <el-button type="danger" @click="batchRemove">删除已选</el-button>
      </el-col>
    </el-row>
    <el-tree
      :data="data"
      :props="defaultProps"
      show-checkbox
      :expand-on-click-node="false"
      :default-expanded-keys="defaultExpandedKeys"
      node-key="catId"
      :draggable="draggable"
      :allow-drop="allowDrop"
      @node-drop="nodeDrop"
      ref="categoryTree"
    >
      <span class="custom-tree-node" slot-scope="{ node, data }">
        <span>{{ node.label }}</span>
        <span>
          <el-button
            v-if="node.level < 3"
            type="text"
            size="mini"
            @click="() => buildAppend(data)"
          >
            Append
          </el-button>
          <el-button
            v-if="node.childNodes.length <= 0"
            type="text"
            size="mini"
            @click="() => remove(node, data)"
          >
            Delete
          </el-button>
          <el-button type="text" size="mini" @click="() => buildEdit(data)">
            Edit
          </el-button>
        </span>
      </span>
    </el-tree>

    <!-- 添加分类 - 对话框 -->
    <el-dialog
      width="30%"
      :title="dislogType == 0 ? '添加' : '修改'"
      :visible.sync="dialogFormVisible"
      :close-on-click-modal="false"
    >
      <el-form :model="category">
        <el-form-item label="分类名">
          <el-input v-model="category.name" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="图标">
          <el-input v-model="category.icon" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="产品单位">
          <el-input
            v-model="category.productUnit"
            autocomplete="off"
          ></el-input>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogFormVisible = false">取 消</el-button>
        <el-button type="primary" @click="submitDialogInfo">确 定</el-button>
      </div>
    </el-dialog>
  </div>
</template>

<script>
//这里可以导入其他文件（比如：组件，工具js，第三方插件js，json文件，图片文件等等）
//例如：import 《组件名称》 from '《组件路径》';

export default {
  //import引入的组件需要注入到对象中才能使用
  components: {},
  data() {
    //这里存放数据
    return {
      // 是否开启拖拽
      draggable: false,
      // 拖拽节点数据
      dropNodeData: new Map(),
      data: [],
      defaultProps: {
        children: "children",
        label: "name",
      },
      defaultExpandedKeys: [],
      category: {
        icon: "",
        catId: "",
        name: "",
        parentCid: 0,
        catLevel: 1,
        showStatus: 1,
        sort: 0,
        productUnit: "",
      },
      dialogFormVisible: false,
      dislogType: 0, // 对话框类型，0 - add，1 - edit
    };
  },
  //监听属性 类似于data概念
  computed: {},
  //监控data中的数据变化
  watch: {},
  //方法集合
  methods: {
    // 获取数据
    async fetchData() {
      const { data } = await this.$http({
        url: this.$http.adornUrl("/product/category/tree-list"),
        method: "get",
        params: this.$http.adornParams({}),
      });
      this.data = data.data;
    },
    // 构建添加数据时需要的配置
    buildAppend(data) {
      Object.assign(this.category, this.$options.data().category);
      this.dislogType = 0;
      this.dialogFormVisible = true;
      this.category.parentCid = data.catId;
      this.category.catLevel = data.catLevel * 1 + 1;
      console.log(data);
    },
    // 添加数据
    async append() {
      await this.$http({
        url: this.$http.adornUrl("/product/category/save"),
        method: "post",
        data: this.$http.adornData(this.category, false),
      });
    },
    // 删除数据
    remove(node, data) {
      console.log(node, data);
      this.$confirm(`此操作将删除[${data.name}]分类, 是否继续?`, "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning",
      }).then(async () => {
        await this.$http({
          url: this.$http.adornUrl("/product/category/delete"),
          method: "delete",
          data: this.$http.adornData([data.catId], false),
        });
        this.defaultExpandedKeys = [node.parent.data.catId];
        this.$message({
          message: "删除成功",
          type: "success",
        });
        this.fetchData();
      });
    },
    // 批量删除数据
    batchRemove() {
      const checkedNodeNames = [];
      const checkNodeIds = this.$refs.categoryTree
        .getCheckedNodes()
        .map((node) => {
          checkedNodeNames.push(node.name);
          return node.catId;
        });
      this.$confirm(
        `此操作将删除选中的[${checkedNodeNames}]分类, 是否继续?`,
        "提示",
        {
          confirmButtonText: "确定",
          cancelButtonText: "取消",
          type: "warning",
        }
      ).then(async () => {
        await this.$http({
          url: this.$http.adornUrl("/product/category/delete"),
          method: "delete",
          data: this.$http.adornData(checkNodeIds, false),
        });
        this.fetchData();
        this.$message({
          message: "删除成功",
          type: "success",
        });
      });
    },
    // 构建修改数据时需要的配置
    async buildEdit({ catId }) {
      this.dislogType = 1;
      const { data } = await this.$http({
        url: this.$http.adornUrl(`/product/category/info/${catId}`),
        method: "get",
        data: this.$http.adornData(data, false),
      });
      this.category = data.data;
      this.dialogFormVisible = true;
    },
    // 修改数据
    async edit() {
      await this.$http({
        url: this.$http.adornUrl("/product/category/update"),
        method: "put",
        data: this.$http.adornData(this.category, false),
      });
    },
    // 提交对话框信息
    async submitDialogInfo() {
      console.log(this.category);
      if (this.dislogType == 0) {
        // 添加分类
        await this.append();
      } else if (this.dislogType == 1) {
        // 修改分类
        await this.edit();
      }
      this.fetchData();
      this.defaultExpandedKeys = [this.category.parentCid];
      this.dialogFormVisible = false;
      this.$message({
        message: `${this.dislogType == 0 ? "添加" : "修改"}成功`,
        type: "success",
      });
    },
    // 是否允许拖拽
    allowDrop(draggingNode, targetNode, type) {
      // console.log(...arguments);
      // 拖拽节点 + 目标节点的深度不能超过 3
      if (type === "inner") {
        // 拖拽节点成为子节点 ==> targetNode.level + draggingNode 节点的深度
        return targetNode.level + this.countNodeDepth(draggingNode) <= 3;
      } else {
        // 拖拽节点成为兄弟节点 ==> targetNode.parent.level + draggingNode 节点的深度
        return targetNode.parent.level + this.countNodeDepth(draggingNode) <= 3;
      }
    },
    // 计算节点深度
    countNodeDepth(node) {
      if (!node) {
        return 0;
      }
      let maxChildDepth = 0,
        childDepth;
      for (let i = 0; i < node.childNodes.length; i++) {
        childDepth = this.countNodeDepth(node.childNodes[i]);
        if (maxChildDepth < childDepth) {
          maxChildDepth = childDepth;
        }
      }
      return 1 + maxChildDepth;
    },
    // 拖拽成功
    nodeDrop(draggingNode, targetNode, type) {
      let peerNodes; // 同辈节点
      let parentCid; // 父分类 id
      let catLevel; // 分类层级
      if (type === "inner") {
        // 修改同辈的 sort => targetNode.childNodes
        peerNodes = targetNode.childNodes;
        // 确定父类的id
        parentCid = targetNode.data.catId;
        // 调整之后的分类层级
        catLevel = targetNode.level + 1;
      } else {
        // 修改同辈的 sort => targetNode.parent.childNodes
        peerNodes = targetNode.parent.childNodes;
        parentCid = targetNode.parent.data.catId || 0;
        catLevel = targetNode.level;
      }

      // 修改同辈节点的 sort 排序顺序
      peerNodes.forEach(({ data }, idx) => {
        this.dropNodeData.set(data.catId, {
          ...this.dropNodeData.get(data.catId),
          sort: idx,
        });
      });

      // 如果调整之后的分类节点层级没有改变的话，不需要调整，否之，还需要调整其子分类的节点层级
      if (catLevel !== draggingNode.level) {
        // 修改子分类的 level
        this.dropNodeData.set(draggingNode.data.catId, {
          ...this.dropNodeData.get(draggingNode.data.catId),
          catLevel,
        });
        draggingNode.childNodes.forEach(({ data }) => {
          this.dropNodeData.set(data.catId, {
            ...this.dropNodeData.get(data.catId),
            catLevel: catLevel + 1,
          });
        });
      }

      // 设置新的父分类id
      this.dropNodeData.set(draggingNode.data.catId, {
        ...this.dropNodeData.get(draggingNode.data.catId),
        parentCid,
      });
    },
    // 保存拖拽信息
    async saveDropNode() {
      // 映射为数组
      const dropNodeDataArray = Array.from(this.dropNodeData).map(
        ([key, value]) => {
          value.catId = key;
          return value;
        }
      );
      // 提交修改请求
      await this.$http({
        url: this.$http.adornUrl("/product/category/update/batch"),
        method: "put",
        data: this.$http.adornData(dropNodeDataArray, false),
      });
      this.fetchData();
      this.draggable = false
      Object.assign(this.dropNodeData, this.$options.data().dropNodeData)
      this.$message({
        message: "修改成功",
        type: "success",
      });
    },
  },
  //生命周期 - 创建完成（可以访问当前this实例）
  created() {},
  //生命周期 - 挂载完成（可以访问DOM元素）
  mounted() {
    this.fetchData();
  },
  beforeCreate() {}, //生命周期 - 创建之前
  beforeMount() {}, //生命周期 - 挂载之前
  beforeUpdate() {}, //生命周期 - 更新之前
  updated() {}, //生命周期 - 更新之后
  beforeDestroy() {}, //生命周期 - 销毁之前
  destroyed() {}, //生命周期 - 销毁完成
  activated() {}, //如果页面有keep-alive缓存功能，这个函数会触发
};
</script>
<style scoped>
/* .custom-tree-node {
    flex: 1;
    display: flex;
    align-items: center;
    justify-content: space-between;
    font-size: 14px;
    padding-right: 8px;
  } */
/*@import url(); 引入公共css类*/
</style>