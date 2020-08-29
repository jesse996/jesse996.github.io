# Win10微软拼音添加小鹤双拼以及其他配置


首先打开注册表，找到这个路径

```
计算机计算机\HKEY_CURRENT_USER\Software\Microsoft\InputMethod\Settings\CHS
```

然后新建一个名为 `UserDefinedDoublePinyinScheme0`的字符串值，数值数据为

```
小鹤双拼*2*^*iuvdjhcwfg^xmlnpbksqszxkrltvyovt
```

然后找到微软拼音的配置页面，把新出现的小鹤双拼设置为默认值。设置默认模式为英文。

直接修改注册表的最大意义在于，不需要自己一个个去定义按键和音节的对应关系了。

(可选)设置不让所有程序窗口共用输入法（时间和语言 → 语言 → 键盘）。

