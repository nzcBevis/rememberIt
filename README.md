# 记一记是一个类似备忘录笔记的uniapp项目
## 目前功能有：
* 笔记可进行新增、更新以及删除笔记。  
* 笔记内有样式选择，可添加图片保存。  
* 待办可进行添加、勾选当前待办可把当前待办划掉。  
* 通过在搜索框输入关键字筛选出含有相关文字的笔记或者待办内容。  

## 暂无长按删除功能；
* 小程序端导航栏左上角的返回无添加事件；  
* 待办事件勾选后无后续将已完成事件隐藏或者放在最后，  
* 以及取消勾选后把当前事件放回当前时间的数组中置顶；  

## index部分  
### 笔记列表页面 
![默认笔记列表](https://github.com/nzcBevis/rememberIt/raw/master/static/images/scrrenshot/defaultNoteList.jpg)
![新增笔记列表](https://github.com/nzcBevis/rememberIt/raw/master/static/images/scrrenshot/newNoteList.jpg)

### 待办列表页面
![默认待办列表](https://github.com/nzcBevis/rememberIt/raw/master/static/images/scrrenshot/defaultToDoList.jpg)
![新增待办列表](https://github.com/nzcBevis/rememberIt/raw/master/static/images/scrrenshot/newToDoList.jpg)
![新增及改变状态时待办列表](https://github.com/nzcBevis/rememberIt/raw/master/static/images/scrrenshot/newToDoList&changestatus.jpg)

## note部分  
### 笔记内容
![默认笔记内容](https://github.com/nzcBevis/rememberIt/raw/master/static/images/scrrenshot/defaultNote.jpg)
![新增笔记内容](https://github.com/nzcBevis/rememberIt/raw/master/static/images/scrrenshot/addNote.jpg)

## 因待办仅需添加文字，所以待办放在组件addIcon中
### 待办内容
![新增待办内容](https://github.com/nzcBevis/rememberIt/raw/master/static/images/scrrenshot/addToDo.jpg)

## 搜索
![搜索](https://github.com/nzcBevis/rememberIt/raw/master/static/images/scrrenshot/search.jpg)