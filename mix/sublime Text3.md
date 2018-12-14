## edit markdown
#### install

1.安装Package Control

	keycmd: ctrl+`

	import urllib.request,os; pf = 'Package Control.sublime-package'; ipp = sublime.installed_packages_path(); urllib.request.install_opener( urllib.request.build_opener( urllib.request.ProxyHandler()) ); open(os.path.join(ipp, pf), 'wb').write(urllib.request.urlopen( 'http://sublime.wbond.net/' + pf.replace(' ','%20')).read())

2.安装markdown插件

	keyboard: ctrl_shift_p

输入pcip or install -> Package Control: Install Package

然后安装:

	autosave
	Markdown Editing //md语法高亮
	Markdown Preview //md导出html支持
	or
	MarkdownLivePreview //直接在sublime显示,支持得不是很好

3.配置

配置autosave, 不必0.15 太快了

	Preference->Package Settings->Auto-save->打开Settings-Defualt和Settings-User
	将Default的内容复制粘贴到User里面，然后修改等待时长：
	  	"auto_save_delay_in_seconds": 1,

配置*.md，要实时刷新*.md加入

	<meta http-equiv="refresh" content="0.1">
才能有实时刷新功能

4.预览功能

	keyboard: ctrl_shift_p 输入->mp
	1 选择 MarkdownLivePreview: Edit current file
	2 选择 Markdown Preview: Preview in Browser

#### reference

1. [使用Sublime Text 3进行Markdown 编辑+实时预览](https://blog.csdn.net/github_32886825/article/details/52930195)