# 记一记是一个类似备忘录笔记的uniapp项目  
## 目前功能有：  
* 笔记可进行新增、更新以及删除笔记。  
* 笔记内有样式选择，可添加图片保存。  
* 待办可进行添加、勾选当前待办可把当前待办划掉。  
* 通过在搜索框输入关键字筛选出含有相关文字的笔记或者待办内容。  

## 暂无长按删除功能：  
* 小程序端导航栏左上角的返回无添加事件；  
* 待办事件勾选后无后续将已完成事件隐藏或者放在最后，  
* 以及取消勾选后把当前事件放回当前时间的数组中置顶；  

## index部分  
### 笔记列表页面  
**默认笔记列表**  
<img src="https://github.com/nzcBevis/rememberIt/raw/0428fd93a32a4314d3c8caa9b59875918c958c02/static/images/screenshot/defaultNoteList.jpg" alt="默认笔记列表" width="200" heigth="300">  
**新增笔记列表**  
<img src="https://github.com/nzcBevis/rememberIt/raw/0428fd93a32a4314d3c8caa9b59875918c958c02/static/images/screenshot/newNoteList.jpg" alt="新增笔记列表" width="200" heigth="300">  

### 待办列表页面  
**默认待办列表**  
<img src="https://github.com/nzcBevis/rememberIt/raw/0428fd93a32a4314d3c8caa9b59875918c958c02/static/images/screenshot/defaultToDoList.jpg" alt="默认待办列表" width="200" heigth="300">  
**新增及改变状态时待办列表**  
<img src="https://github.com/nzcBevis/rememberIt/raw/0428fd93a32a4314d3c8caa9b59875918c958c02/static/images/screenshot/newToDoList%26changestatus.jpg" alt="新增及改变状态时待办列表" width="200" heigth="300">  

## note部分  
### 笔记内容  
**默认笔记内容**  
<img src="https://github.com/nzcBevis/rememberIt/raw/0428fd93a32a4314d3c8caa9b59875918c958c02/static/images/screenshot/defaultNote.jpg" alt="默认笔记内容" width="200" heigth="300">  
**新增笔记内容**   
<img src="https://github.com/nzcBevis/rememberIt/raw/0428fd93a32a4314d3c8caa9b59875918c958c02/static/images/screenshot/addNote.jpg" alt="新增笔记内容" width="200" heigth="300">  

## 待办内容  
### 因待办仅需添加文字，所以待办放在组件addIcon中  
**新增待办**  
<img src="https://github.com/nzcBevis/rememberIt/raw/0428fd93a32a4314d3c8caa9b59875918c958c02/static/images/screenshot/addToDo.jpg" alt="新增待办内容" width="200" heigth="300">  

## 搜索  
### 进行搜索  
**搜索笔记**  
<img src="https://github.com/nzcBevis/rememberIt/raw/b1256d62b41e5fcbf1c3d4a982b1b972c762542d/static/images/screenshot/searchNoteList.jpg" alt="搜索笔记" width="200" heigth="300">  
**搜索待办**  
<img src="https://github.com/nzcBevis/rememberIt/raw/b1256d62b41e5fcbf1c3d4a982b1b972c762542d/static/images/screenshot/searchToDoList.jpg" alt="搜索待办" width="200" heigth="300">  