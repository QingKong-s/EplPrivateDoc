# 易语言源码文件结构

## 结构概览

文件由一个文件头和若干段组成。

## 文件头

文件头总为`"CNWTEPRG"`的ASCII编码，共`8`字节。

## 段

段由段头和段体组成，段头的定义如下：

|大小|含义|备注|
|-|-|-|
|4|幻数|`0x15117319`|
|4|检验和|TODO : 计算方式|
|4|标识||
|30|段名||
|2|对齐空隙||
|4|段索引||
|4|是否为重要数据段|因某种原因无法识别段时是否可跳过本段|
|4|数据校验和||
|4|段尺寸||
|40|保留||

共100字节。

标识字段唯一标识了某个段，还决定如何读取段名，标识字段的最高字节为序号，定义如下：

|值|含义|
|-|-|
|1|用户信息段|
|2|系统信息段|
|3|程序段|
|4|程序资源段|
|5|可丢失程序段|
|8|初始模块段|
|9|编辑信息段2|
|11|辅助信息段2|
|12|TODO 依赖信息段|
|13|易包信息段|
|14|编辑过滤器信息段|

其中段名由段名的GB2312编码配合标识加密而来，加解密算法如下：

```C
char Name[30];
for (size_t i = 0; i < 30; ++i) {
	Name[i] = SecName[i] ^ Key[(i + 1) % 4];
}
```

算法结束后`Name`中的内容变为段名的GB2312编码。

### 用户信息段



### 系统信息段

该段为固定的60字节，定义如下：

|大小|含义|备注|
|-|-|-|
|2|易语言主版本号||
|2|易语言次版本号||
|4|未知||
|4|语言|1 = GBK；2 = ASCII；3 = BIG5；4 = Shift JIS|
|2|源代码主版本||
|2|源代码次版本||
|4|类型|1 = 易源码；3 = 易模块|
|4|未知||
|4|编译为|0 = Windows窗体程序；<br>1 = Windows控制台程序；<br>2 = Windows动态链接库；<br>0x3E8 = Windows易语言模块；<br>0x2710 = Linux控制台程序；<br>0x2AF8 = Linux易语言模块|
|4 * 8|未知||

### 程序资源段

该段由窗口开始：

|大小|含义|备注|
|-|-|-|
|4|长度||
|`n1 * 4`|未知|`n1` = 长度 / 8|
|`n1 * 4`|未知|`n1` = 长度 / 8|

上述结构后跟每个窗口的数据：

|大小|含义|备注|
|-|-|-|
|4|未知||
|4|对应程序集||
|4|名称字符串长度||
|`n1`|名称字符串|`n1` = 名称字符串长度|
|4|备注字符串长度||
|`n2`|备注字符串|`n2` = 备注字符串长度|

每个窗口数据后跟该窗口所有控件数据：

|大小|含义|备注|
|-|-|-|
|4|控件数||
|4|控件数据长度||
|`n1 * 4`|ID||
|`n1 * 4`|此部分数据的尾部到指定控件数据的偏移||

在每个偏移量处有：

|大小|含义|备注|
|-|-|-|
|4|数据长度|不包括此字段本身|
|4|数据类型|`65539` = 菜单|
|20|保留||
|`n1`|名称|以NULL结尾的字符串|
|`n2`|备注|以NULL结尾的字符串|
|4|CWnd指针||
|4|左边||
|4|顶边||
|4|宽度||
|4|高度||
|4|*未知*||
|4|父窗口ID||
|4|子窗口个数||
|`n3 * 4`|子窗口ID||
|4|光标资源长度||
|`n4`|光标||
|`n5`|标记字符串||
|4|*未知*|似乎总为0|
|4|标志|一组位标志<br>`1` = 可视；<br>`2` = 禁止；<br>`4` = 可停留焦点；<br>`16` = 锁定(供设计器使用)|
|4|停留顺序||
|4|事件数||
|`n6 * 4`|TODO|`n6` = 事件数|
|`n6 * 4`|TODO|`n6` = 事件数|
|20|*未知*||
|`n7`|控件序列化数据||



## 控件序列化数据结构

### 系统核心支持库

#### 按钮

|大小|含义|备注|
|-|-|-|
|4|版本|当前为`1`|
|4|类型||
|4|横向对齐方式||
|4|纵向对齐方式||
|4|图片数据长度||
|`n1`|图片数据||
|4|字体数据长度||
|`n2`|字体|`n2` = 字体数据长度，若`n2 == 60`，则指示`LOGFONTA`|
|4|标题字符串长度||
|`n3`|标题字符串||

#### 编辑框

|大小|含义|备注|
|-|-|-|
|4|版本|`2`|
|4|边框||
|4|文本颜色||
|4|背景颜色||
|4|字体数据长度||
|`n2`|字体|`n2` = 字体数据长度，若`n2 == 60`，则指示`LOGFONTA`|
|4|最大允许长度||
|4|是否允许多行||
|4|滚动条||
|4|对齐方式||
|4|输入方式||
|2|密码遮盖字符||
|4|转换方式||
|4|调节器方式||
|4|调节器底限值||
|4|调节器上限值||
|4|起始选择位置||
|4|被选择字符数||
|4|*未知*||
|4|内容字符串长度||
|`n1`|内容字符串||
|4|数据源||
|4|数据列||

#### 标签

|大小|含义|备注|
|-|-|-|
|4|版本|`2`|
|4|效果||
|4|渐变边框宽度||
|4|边框||
|4|渐变边框颜色1||
|4|渐变边框颜色2||
|4|渐变边框颜色3||
|4|文本颜色||
|4|背景颜色||
|4|底图方式||
|4|渐变背景方式||
|4|渐变背景颜色1||
|4|渐变背景颜色2||
|4|渐变背景颜色3||
|4|横向对齐方式||
|4|纵向对齐方式||
|4|是否自动折行||
|4|字体数据长度||
|`n1`|字体|若`n1 == 60`，则指示`LOGFONTA`|
|4|图片数据长度||
|`n2`|图片数据||
|4|数据源||
|4|数据列||

#### 表格

#### 超级链接框

|大小|含义|备注|
|-|-|-|
|4|版本|当前为`1`|
|4|类型||
|4|文本颜色||
|4|访问后的颜色||
|4|热点颜色||
|4|背景颜色||
|4|边框||
|4|字体数据长度||
|`n1`|字体|若`n1 == 60`，则指示`LOGFONTA`|
|4|标题字符串长度||
|`n2`|标题字符串||
|4|电子信箱地址长度||
|`n3`|电子信箱地址字符串||
|4|Internet地址长度||
|`n4`|Internet地址字符串||

#### 打印机

#### 单选框

|大小|含义|备注|
|-|-|-|
|4|版本|当前为`1`|
|4|按钮形式||
|4|平面||
|4|标题居左||
|4|横向对齐方式||
|4|纵向对齐方式||
|4|文本颜色||
|4|背景颜色||
|4|选中||
|4|图片数据长度||
|`n1`|图片数据||
|4|字体数据长度||
|`n2`|字体|若`n2 == 60`，则指示`LOGFONTA`|
|4|标题字符串长度||
|`n3`|标题字符串||

#### 调节器

|大小|含义|备注|
|-|-|-|
|4|版本|当前为`1`|
|4|方向||
|4|热点跟踪||

#### 端口

#### 分组框

|大小|含义|备注|
|-|-|-|
|4|版本|当前为`1`|
|4|对齐方式||
|4|文本颜色||
|4|背景颜色||
|4|字体数据长度||
|`n1`|字体|若`n1 == 60`，则指示`LOGFONTA`|
|4|标题字符串长度||
|`n2`|标题字符串||

#### 服务器

#### 横向滚动条

|大小|含义|备注|
|-|-|-|
|4|版本|当前为`1`|
|4|最小位置||
|4|最大位置||
|4|页改变值||
|4|行改变值||
|4|位置||
|4|允许拖动跟踪||

#### 滑块条

|大小|含义|备注|
|-|-|-|
|4|版本|当前为`1`|
|4|边框||
|4|方向||
|4|刻度类型||
|4|单位刻度值||
|4|允许选择||
|4|首选择位置||
|4|选择长度||
|4|页改变值||
|4|行改变值||
|4|最小位置||
|4|最大位置||
|4|位置||

#### 画板

|大小|含义|备注|
|-|-|-|
|4|版本|当前为`1`|
|4|边框||
|4|画板背景色||
|4|底图方式||
|4|自动重画||
|4|绘画单位||
|4|画笔类型||
|4|画出方式||
|4|画笔粗细||
|4|画笔颜色||
|4|刷子类型||
|4|刷子颜色||
|4|文本颜色||
|4|文本背景色||
|4|字体数据长度||
|`n1`|字体|若`n1 == 60`，则指示`LOGFONTA`|
|4|图片数据长度||
|`n1`|图片数据||

#### 进度条

|大小|含义|备注|
|-|-|-|
|4|版本|当前为`1`|
|4|边框||
|4|方向||
|4|显示方式||
|4|最小位置||
|4|最大位置||
|4|位置||

#### 客户

#### 列表框

|大小|含义|备注|
|-|-|-|
|4|版本|`3`|
|4|边框||
|4|自动排序||
|4|多列||
|4|行间距||
|4|文本颜色||
|4|背景颜色||
|4|允许选择多项||
|4|现行选中项||
|4|字体数据长度||
|`n1`|字体|若`n1 == 60`，则指示`LOGFONTA`|
|4|表项数值字节数||
|`n2 * 4`|表项数值||
|2|表项数||
|`n3`|表项||
|4|图片数据长度||
|`n1`|图片数据||
|4|数据源||
|4|数据列||

其中每个表项的结构如下：

|大小|含义|备注|
|-|-|-|
|4|标题字符串长度||
|`n2`|标题字符串||

#### 目录框

|大小|含义|备注|
|-|-|-|
|4|版本|当前为`1`|
|4|边框||
|4|背景颜色||
|4|字体数据长度||
|`n1`|字体|若`n1 == 60`，则指示`LOGFONTA`|

#### 驱动器框

|大小|含义|备注|
|-|-|-|
|4|版本|当前为`1`|
|1|*未知*|可能作用为供运行时保存当前驱动器|
|4|类型||
|4|文本颜色||
|4|背景颜色||
|4|字体数据长度||
|`n1`|字体|若`n1 == 60`，则指示`LOGFONTA`|

#### 日期框

|大小|含义|备注|
|-|-|-|
|4|版本|`2`|
|4|允许编辑||
|4|附件类型||
|8|今天||
|8|最小日期||
|8|最大日期||
|4|边框||
|4|字体数据长度||
|`n1`|字体|若`n1 == 60`，则指示`LOGFONTA`|
|4|数据源||
|4|数据列||

#### 时钟

|大小|含义|备注|
|-|-|-|
|4|版本|当前为`1`|
|4|时钟周期||

#### 数据报

#### 数据库提供者

#### 数据源

#### 通用对话框


|大小|含义|备注|
|-|-|-|
|4|版本|当前为`1`|
|4|||
|4|||
|4|||
|4|||
|4|||
|4|||
|4|||
|4|||
|4|||
|4|||
|4|||
|4|||
|4|||
|4|||
|4|||