<!--  -->
<template>
  <el-tree
    :data="data"
    :props="defaultProps"
    :expand-on-click-node="false"
    @node-click="nodeClick"
    node-key="catId"
    ref="categoryTree"
  >
  </el-tree>
</template>

<script>
//这里可以导入其他文件（比如：组件，工具js，第三方插件js，json文件，图片文件等等）
//例如：import 《组件名称》 from '《组件路径》';

export default {
  props: {
    data: {
      default: () => [],
    },
    defaultProps: {
      default: () => {
        return {
          children: "children",
          label: "name",
        };
      },
    },
  },
  //import引入的组件需要注入到对象中才能使用
  components: {},
  data() {
    //这里存放数据
    return {};
  },
  //监听属性 类似于data概念
  computed: {},
  //监控data中的数据变化
  watch: {},
  //方法集合
  methods: {
    nodeClick(data, node) {
      console.log(data, node);
      this.$emit("node-click", data.catId, node.level);
    },
  },
};
</script>
<style scoped>
/*@import url(); 引入公共css类*/
</style>