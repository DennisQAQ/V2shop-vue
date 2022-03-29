<!--  -->
<template>
  <div>
    <el-tree
      :data="menus"
      :props="defaultProps"
      show-checkbox
      :expand-on-click-node="false"
    >
      <span class="custom-tree-node" slot-scope="{ node, data }">
        <span>{{ node.label }}</span>
        <span>
          <el-button
            v-if="node.level <= 2"
            type="text"
            size="mini"
            @click="() => append(data)"
            >添加</el-button
          ><el-button type="text" size="mini" @click="() => edit(data)"
            >修改</el-button
          >
          <el-button
            v-if="node.childNodes.length == 0"
            type="text"
            size="mini"
            @click="() => remove(node, data)"
            >Delete</el-button
          >
        </span>
      </span>
    </el-tree>
    <el-dialog :title="title" :visible.sync="dialogFormVisible" :close-on-click-modal="false">
      <el-form :model="category">
        <el-form-item label="分类名称">
          <el-input v-model="category.name" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="图标">
          <el-input v-model="category.icon" autocomplete="off"></el-input>
        </el-form-item>
        <el-form-item label="计量单位">
          <el-input
            v-model="category.productUnit"
            autocomplete="off"
          ></el-input>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="dialogFormVisible = false">取 消</el-button>
        <el-button type="primary" @click="submitData()">确 定</el-button>
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
    return {
      menus: [],
      title: "",
      dialogType: "", //apped or edit 当调用不同的方法时，判断打开何种对话框。
      defaultProps: {
        children: "children",
        label: "name",
      },
      //展开菜单的id
      expandedKey: [],
      //添加分类对话框默认关闭
      dialogFormVisible: false,
      category: {
        name: "", //名称
        parentCid: 0, //父类ID
        catLevel: 0, //分类级别
        showStatus: 1, //商品状态
        sort: 0, //商品排序
        productUnit: "", //商品计量单位
        icon: "", //图标
        catId: null, //分类ID
      },
    };
  },

  //监听属性 类似于data概念
  computed: {},
  //监控data中的数据变化
  watch: {},
  //方法集合
  methods: {
    append(data) {
      console.log("添加数据", data);
      //调用修改方法时，打开添加对话框
      this.dialogType = "add";
      //显示对话框
      this.dialogFormVisible = true;
      //显示对话名称 添加分类
      this.title = "添加分类";
      //父id为当前点击数据id
      this.category.parentCid = data.catId;
      //显示层级为当前点击数据下一级
      this.category.catLevel = data.catLevel * 1 + 1;
      this.category.catId = null;
      this.category.name = "";
      this.category.icon = "";
      this.category.productUnit = "";
      this.category.sort = 0;
      this.category.showStatus = 1;
    },
    addCategory() {
      this.$http({
        url: this.$http.adornUrl("/product/category/save"),
        method: "post",
        data: this.$http.adornData(this.category, false),
      }).then(({ data }) => {
        this.$message({
          message: "菜单保存成功",
          type: "success",
        });
        //关闭对话框
        this.dialogFormVisible = false;
        //刷新出新的菜单
        this.getMenus();
        //设置需要默认展开的菜单
        this.expandedKey = [this.category.parentCid];
      });
    },

    remove(node, data) {
      //讲删除id传入数组
      var ids = [data.catId];
      //删除前弹出确认框
      this.$confirm(`是否删除【${data.name}】菜单?`, "提示", {
        confirmButtonText: "确定",
        cancelButtonText: "取消",
        type: "warning",
      })
        .then(() => {
          this.$http({
            url: this.$http.adornUrl("/product/category/delete"),
            method: "post",
            data: this.$http.adornData(ids, false),
          }).then(({ data }) => {
            //成功删除后弹出提示
            this.$message({
              message: "菜单删除成功",
              type: "success",
            });
            //关闭对话框
            this.dialogFormVisible = false;
            //刷新出新的菜单
            this.getMenus();
            //设置需要默认展开的菜单，使得删除后当前目录处于展开的状态
            this.expandedKey = [node.parent.data.catId];
          });
        })
        .catch(() => {});

      console.log("remove", node, data);
    },
    //编写edit方法，将参数data传入并再控制台打印出所选的数据
    edit(data) {
      //显示对话名称 添加分类
      this.title = "修改分类";
      //调用修改方法时，打开修改对话框
      this.dialogType = "edit",
       console.log("要修改的数据", data);
      //调用edit方法时将dialogFormVisib属性修改为true，以弹出对话框。
      this.dialogFormVisible = true;
      //发送请求获取最新节点的数据
      this.$http({
        url: this.$http.adornUrl(`/product/category/info/${data.catId}`),
        method: "get",
      }).then(({ data }) => {
        console.log("最新数据",data);
        //将传入要的修改参数data值赋值给绑定的catagory.catId
        this.category.catId = data.data.catId;
        //将传入要的修改参数data值赋值给绑定的catagory.name用于表单数据回显
        this.category.name = data.data.name;
        this.category.productUnit = data.data.productUnit;
        this.category.icon = data.data.icon;
        this.category.parentCid=data.data.parentCid;
        this.category.catLevel =data.data.catLevel;
        this.category.showStatus =data.data.showStatus
        this.category.sort=data.data.sort;

      });
    },
    //
    editCategory() {
        var {catId,name,icon,productUnit}=this.category;
      this.$http({
      url: this.$http.adornUrl("/product/category/update"),
      method: 'post',
      data: this.$http.adornData(catId,name,icon,productUnit, false)
      }).then(({ data }) => {
        this.$message({
          message: "菜单修改成功",
          type: "success",
        });
        //关闭对话框
        this.dialogFormVisible = false;
        //刷新出新的菜单
        this.getMenus();
        //设置需要默认展开的菜单
        this.expandedKey = [this.category.parentCid];
      });
    },
    getMenus() {
      this.$http({
        url: this.$http.adornUrl("/product/category/list/Tree"),
        method: "get",
      }).then(({ data }) => {
        console.log("成功获取到菜单数据。。。", data.data),
          (this.menus = data.data);
      });
    },
    //判断当前调用方法是添加还修改，以打开对应的对话框
    submitData() {
     this.dialogType=="add"?this.addCategory():this.editCategory();
    },
  },
  //生命周期 - 创建完成（可以访问当前this实例）
  created() {},
  //生命周期 - 挂载完成（可以访问DOM元素）
  mounted() {
    this.getMenus();
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
<style lang='scss' scoped>
//@import url(); 引入公共css类
</style>