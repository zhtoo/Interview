## 统一编码格式UTF-8

1. File -> setting ->Editor->File Encodings 
将 Global Encoding 和 Project Encoding 设置为UTF-8
2. 在文件窗口右键弹出的对话框中选择当前页面的编码格式（File Encodings）。
3. 控制台编码，需要在项目de gradle中进行修改.

```
tasks.withType(JavaCompile){
    options.encoding = "UTF-8"
}
```
