# GitBash终端中文问乱码解决

1. 鼠标右键 GitBash 窗口选择 `Optons...` 选项

2. 选择弹出框中左侧菜单项中的 `Text` 选项

3. 在右侧属性框中设置属性：

   - **`Local`**：设置为 `zh_CN`
  
   - **`Character set`**：设置为 `UTF-8`

4. 点击 `Apply` 应用, 再点击 `Save` **保存**

5. 最后在 GitBash 中执行命令: `git config --global core.quotepath false`
