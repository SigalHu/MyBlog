升级了`Markdown Preview Enhanced`之后发现原来的styles.less中对该插件字体大小的设置失效了。

记得之前是`[Ctrl]+[Shift]+i`打开开发者工具，然后定位到`Markdown Preview Enhanced`的预览窗口并进行字体调整，再将相关修改写入到styles.less文件中。
```less
.markdown-preview-enhanced[for="preview"] {
  // background-color: #444;
  font-size: 23px;
}
```
然而升级之后这个方法行不通了，毕竟我对前端只是略微了解，折腾了半天，搞不定，也不想继续浪费时间了，所以确定直接修改该插件的css。

1. 选择`Packages->Settings View->Manage Packages->Markdown Preview Enhanced`，将`Preview Theme`设置为`github-dark.css`
2. 点击`View Code`，打开文件`node_modules\@shd101wyy\mume\styles\preview.css`
3. 将`.preview-container .mume[for="preview"] {...;font-size:16px;...}`修改为`.preview-container .mume[for="preview"] {...;font-size:23px;...}`，保存修改
4. 重新触发`Markdown Preview Enhanced`，修改生效

**注意：**`Preview Theme`的其他主题可能需要进一步修改路径`node_modules\@shd101wyy\mume\styles\preview_theme`下相关主题的css。

**另外：** 如果哪位知道怎么通过修改styles.less来改变`Markdown Preview Enhanced`的字体大小，可以在下面留言，感激不尽~
