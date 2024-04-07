# 1 mongodb 构建[可选]

https://www.runoob.com/mongodb/mongodb-tutorial.html

1.1 mongodb docker拉取及配置

1.2 如何将json文件存入mongdodb

1.3 该如何存入mongodb，使用什么样的可扩展的数据结构，一个记录一条库还是增量

1.4 存入mongdb使用什么样的java api进行聚合查询对比差异点

# 2 json对比 后端实现

参考资料：

1sdk：https://github.com/LianjiaTech/json-diff

2设计逻辑：https://blog.csdn.net/hi_bigbai/article/details/128162687

3 后续前端展示jsondiff：https://blog.csdn.net/Revivedsun/article/details/132388214?utm_medium=distribute.pc_relevant.none-task-blog-2~default~baidujs_baidulandingword~default-4-132388214-blog-132551076.235^v43^pc_blog_bottom_relevance_base4&spm=1001.2101.3001.4242.3&utm_relevant_index=7

https://blog.csdn.net/qq_44604364/article/details/129629542?spm=1001.2101.3001.6650.1&utm_medium=distribute.pc_relevant.none-task-blog-2%7Edefault%7ECTRLIST%7ERate-1-129629542-blog-132551076.235%5Ev43%5Epc_blog_bottom_relevance_base4&depth_1-utm_source=distribute.pc_relevant.none-task-blog-2%7Edefault%7ECTRLIST%7ERate-1-129629542-blog-132551076.235%5Ev43%5Epc_blog_bottom_relevance_base4&utm_relevant_index=2

3 其他：
判读json相等：https://gitee.com/codeleep/json-diff#2%E6%89%A7%E8%A1%8C%E6%B5%8B%E8%AF%95

github:搜索--对比工具

web网站--在线json对比工具

## 2.1 功能点

### 2.1.1对比新增add、delete接口

### 2.1.2全量对比变化输出对比json文件[后续实现可选]

## 2.2  后端实现

### 2.2.1  整合项目dependency

将sdk项目中的depency整合到ruoyi-admin中

### 2.2.2  使用ruoyi自动化生成curd表[后续需要表实现 此处暂时未详细设计表的结构]

### 2.2.3   复制sdk项目的com/ke/diff到若依形成sdk

将com.ke.diff中的ke改为ruoyi再移动到ruoyi 的admin下

### 2.2.4   在原来的项目接口下暂不用数据库，增加对比controller实现[由于暂用接口，未分离至impl里面]

当前由于复用原来的项目，使用mapper record中的taskname 和master作为2个文件路径

```java
    @PostMapping("/Jsoncompare")
    public AjaxResult Jsoncompare(@RequestBody Record record)
    {
        // 1 此处后续需要增加json文件存在判读,修改mapper获取方式
        String jsonFilePath = record.getTaskname();
        String jsonFilePath2 = record.getMaster();

        String jsonString1 = null;
        String jsonString2 = null;

        JsonElement jsonElement1 = null;
        JsonElement jsonElement2 = null;

        try (BufferedReader reader = new BufferedReader(new FileReader(jsonFilePath))) {
            jsonString1 = reader.lines().collect(Collectors.joining(System.lineSeparator()));
            JsonParser parser = new JsonParser();
            jsonElement1 = parser.parse(jsonString1);
        } catch (IOException e) {
            e.printStackTrace();
        }
        try (BufferedReader reader = new BufferedReader(new FileReader(jsonFilePath2))) {
            jsonString2 = reader.lines().collect(Collectors.joining(System.lineSeparator()));
            JsonParser parser = new JsonParser();
            jsonElement2 = parser.parse(jsonString2);
        } catch (IOException e) {
            e.printStackTrace();
        }

        JsonObject jsonObject1 = jsonElement1.getAsJsonObject();
        JsonObject jsonObject2 = jsonElement2.getAsJsonObject();

        // 2此处需要改成对应忽略的json路径
        JsonElement serviceNameElement1 = jsonObject1.get("API");
        JsonElement serviceNameElement2 = jsonObject2.get("API");
        List<String> noiseList = Lists.newArrayList("parmas","httpmethod");
        LinkedList<String> specailPath = Lists.newLinkedList(Lists.newArrayList("path"));
        List<Result> diff = new Diff().withNoisePahList(noiseList).withSpecialPath(specailPath).withAlgorithmEnum(AlgorithmEnum.MOST_COMMONLY_USED).diffElement(serviceNameElement1, serviceNameElement2);

        List<String> deletecontroller=new ArrayList<>();//最终输出的列表
        List<String> addcontroller =new ArrayList<>();
        for (Result result : diff) {
            if(result.getDiffType()=="MODIFY"){
                deletecontroller.add(result.getLeft().toString());
                addcontroller.add(result.getRight().toString());
            }
            else{
                String str = result.getRightPath();
                // 移除第一个和最后一个字符（即方括号）
                String numberStr = str.substring(1, str.length() - 1);
                int number = Integer.parseInt(numberStr);
                addcontroller.add(serviceNameElement2.getAsJsonArray().get(number).getAsJsonObject().get("path").getAsString());
            }
        }

        List<List<String>> listOfLists = new ArrayList<>(); //泛型列表返回
        listOfLists.add(deletecontroller);
        listOfLists.add(addcontroller);
        System.out.println("success");

        return success(listOfLists);
    }
```



## 2.3 前端实现

前端的实现主要靠文心一言，样式布局调整的时候要注意后续如何将样式随着视口高度随时调整[后续改进]

**下面的!!!!!部分需要适配下**

```vue
<template>  
  <div>  
    <el-container style="height: 100vh;">  <!-- 100vh 视界高度 -->  
      <el-header style="height: 200px;"> <!-- 将高度设置为更大的值，防止header 中的路径、按钮高度不够-->    
        <el-row type="flex" justify="center" align="middle" class="header-row">   <!-- 采用flex布局+span占比方法--> 
          <el-col :span="24" class="header-text-container">  
            <span class="header-text">json接口在线对比</span>  
          </el-col>  
        </el-row>  
        <el-row type="flex" justify="center" align="middle" class="input-row" style="margin-top: 20px;">  
          <el-col :span="8">  
            <el-input
              placeholder="请输入：json1的服务器路径，如:/tmp/1.json"
              v-model="input1"
              clearable>
            </el-input> 
          </el-col>  
          <el-col :span="8">  
            <el-input
              placeholder="请输入：json2的服务器路径，如:/tmp/2.json"
              v-model="input2"
              clearable>
            </el-input>  
          </el-col>  
        </el-row>  
        <el-row type="flex" justify="center" align="middle" class="button-row" style="margin-top: 20px;">  
          <el-button type="primary" @click="submitPath" round>提交对比</el-button>  
        </el-row>  
      </el-header>  
      <el-main class="main-content" style="padding-top: 150px;">  <!-- el-main 距离head保持一定距离，防止被覆盖，后续可以ui继续优化--> 
        <div class="main-content" style="border-radius: 20px; border: 3px solid #555;">  
        <el-row type="flex" justify="center" class="content-row">  <!-- content-row占比一定要css一定要100%，不然都挤到左边了（文心一言）--> 
          <el-col :span="11" class="column-container">  
            <div class="column-title">删除接口列表</div>  
            <el-table v-loading="loading" :data="tableList1" style="width: 100%; border-radius: 10px;">  <!-- width一定要css一定要100%，不然都挤到左边了（文心一言）--> 
              <el-table-column prop="index" label="编号" width="60"></el-table-column>  
              <el-table-column prop="content" label="接口" width="600"></el-table-column>  
            </el-table>  
          </el-col>  
          <el-col :span="11" class="column-container">  
            <div class="column-title">新增接口列表</div>  
            <el-table v-loading="loading" :data="tableList2" style="width: 100%; border-radius: 10px;">  
              <el-table-column prop="index" label="编号" width="60"></el-table-column>  
              <el-table-column prop="content" label="接口" width="600"></el-table-column>  
            </el-table>  
          </el-col>  
        </el-row>  
      </div>  
      </el-main>  
    </el-container>  
  </div>  
</template>  
  
<script>  
import { Jsoncompare } from "@/api/sectool/record";
export default {  
  name: 'MyLayout',  
  data() {  
    return {  
      tableData1: [  
        { index: 1, content: '内容1' },  
        { index: 2, content: '内容2' },  
        { index: 2, content: '内容2' },  
        { index: 2, content: '内容2' },  
        { index: 2, content: '内容2' },  
        { index: 2, content: '内容2' },  
        { index: 2, content: '内容2' },  
        { index: 2, content: '内容2' },  
        { index: 2, content: '内容2' },  
        { index: 2, content: '内容2' },  
        { index: 2, content: '内容2' },  
        { index: 2, content: '内容2' },  
        { index: 2, content: '内容2' },  
        { index: 2, content: '内容2' },  
        
        // ...mock数据参考 
      ],  
      tableData2: [  
        { index: 1, content: '内容A' },  
        { index: 2, content: '内容B' },  
        // ...mock数据参考 
      ],
      input1: '',  
      input2: '',

      loading:false, 

      adaptivejson: { //暂为了发送请求适配下，后续修改成自己的mapper传送！！！！！！！
        taskname: null,
        master: null,
      },

      tableList1:[],
      tableList2:[],
    };  
  } ,
  methods:{
    submitPath(){
      this.loading=true;
      console.log(this.input1);
      console.log(this.input2);
      this.adaptivejson.taskname=this.input1; //暂时适配下，后续修改成自己的mapper传送！！！！！
      this.adaptivejson.master=this.input2; //暂时适配下，后续修改成自己的mapper传送！！！！！
      Jsoncompare(this.adaptivejson).then(response => {
        console.log(response);
        // 下面为了先将后端泛型分别处理，然后将List<string>类型转为类似json格式，有index和content2个部分
        this.tableList1 = response.data[0].map((content, index) => ({ // 映射为带有索引的对象数组  
            index: index + 1, // 索引通常从1开始，根据需要调整  
            content: content,  
          }));  
        this.tableList2 = response.data[1].map((content, index) => ({ // 映射为带有索引的对象数组  
            index: index + 1, // 索引通常从1开始，根据需要调整  
            content: content,  
          })); 
        console.log(this.tableList1);
        console.log(this.tableList2);
        this.loading = false;
      });
    }
  } ,
};  
</script>  
  
<style scoped>  
.header-row {  
  /* 根据需要调header高度 */  
}  
  
.header-text-container {  
  display: flex;  
  justify-content: center;  
  align-items: center;  
}  
  
.header-text {  
  font-weight: bold;  
  font-size: 3em; /* 根据需要调字体大小 */  
}  
  
.input-row {  
  /* 可添加额外的样式来调整输入框行的外观 */  
}  
  
.button-row {  
  /* 可添加额外的样式来调整按钮行的外观 */  
}  
  
.main-content {  
  display: flex;  
  justify-content: center;  
  align-items: center;  
  flex-direction: column; /* 垂直居中 */  
  padding: 20px; /* 为内容添加内边距 */  
}  
  
.rounded-main {  
  border: 3px solid #333; /* 深灰色边框，线条稍微粗一点 */  
  border-radius: 20px; /* 大圆角边框 */  
  overflow: hidden; /* 隐藏超出边框的内容 */  
}  
  
.column-content {  
  padding: 20px; /* 列内容内边距 */  
}  
  
.column-title {  
  font-weight: bold;  
  font-size: 2em; /* 比header-text稍小 */  
  text-align: center; /* 居中显示 */  
  margin-bottom: 20px; /* 与下方表格的间距 */  
}  
  
/* 表格样式 */  
/* 设置表格样式 */  
.el-table {  
  width: 100%; /* 确保表格宽度填满列容器 */  
  margin-bottom: 20px; /* 设置表格之间的间距 */  
  border-radius: 10px; /* 设置表格圆角 */  
}  
  
.el-table__header-wrapper {  
  background-color: #f5f7fa; /* 固定表头背景色，可以根据需要调整 */  
}  

.el-container {  
  display: flex;  
  flex-direction: column;  
  height: 100vh; /* 设置容器高度为视口高度 */  
  width: 100vw;
}  
  
.el-header {  
  /* 设置 header 的高度 */  
  height: 80px;  
}  
  
.el-main {  
  /* el-main 会自动占据剩余空间 */  
  flex: 1;  
}

.content-row {  
  width: 100%; /* 使内容行占据整个main-content的宽度 */  
} 

.column-container {  
  display: flex;  
  flex-direction: column;  
  align-items: center;  
}  
</style>

```

```vue
// 新增json对比扫描，后续路径需要修改下,在导出的js下面 需要将record路径去掉
export function Jsoncompare(data) {
  return request({
    url: '/sectool/record/Jsoncompare',
    method: 'post',
    data: data
  })
}
```

