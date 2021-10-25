## Welcome to GitHub Pages

You can use the [editor on GitHub](https://github.com/samxxxxx/samxxxxx.github.io/edit/main/index.md) to maintain and preview the content for your website in Markdown files.

Whenever you commit to this repository, GitHub Pages will run [Jekyll](https://jekyllrb.com/) to rebuild the pages in your site, from the content in your Markdown files.

### Markdown

Markdown is a lightweight and easy-to-use syntax for styling your writing. It includes conventions for

```markdown
Syntax highlighted code block

# Header 1
## Header 2
### Header 3

- Bulleted
- List

1. Numbered
2. List

**Bold** and _Italic_ and `Code` text

[Link](url) and ![Image](src)
```

For more details see [GitHub Flavored Markdown](https://guides.github.com/features/mastering-markdown/).

### Jekyll Themes

Your Pages site will use the layout and styles from the Jekyll theme you have selected in your [repository settings](https://github.com/samxxxxx/samxxxxx.github.io/settings/pages). The name of this theme is saved in the Jekyll `_config.yml` configuration file.

### Support or Contact

Having trouble with Pages? Check out our [documentation](https://docs.github.com/categories/github-pages-basics/) or [contact support](https://support.github.com/contact) and we’ll help you sort it out.

### WCF配置调试日志
```markdown
<system.diagnostics>
<sources>
    <source name="System.ServiceModel"
            switchValue="Information, ActivityTracing"
            propagateActivity="true">
    <listeners>
        <add name="traceListener"
            type="System.Diagnostics.XmlWriterTraceListener"
            initializeData="c:\temp\log\SFC_Traces.svclog"  />
    </listeners>
    </source>
</sources>
</system.diagnostics>
```

### WCF配置knownType
```markdown
<system.runtime.serialization>
<dataContractSerializer>
    <declaredTypes>
    <add type = "Baidu.Modules.DataContract.Equipment.EquimentModel,TopStrong.Modules.DataContract,Version=1.0.0.0,Culture=neutral,PublicKeyToken=null, processorArchitecture=MSIL">
        <knownType type = "Baidu.Modules.DataContract.Equipment.PointCheckStd,TopStrong.Modules.DataContract, Version=1.0.0.0, Culture=neutral, PublicKeyToken=null, processorArchitecture=MSIL"/>
    </add>
    </declaredTypes>
</dataContractSerializer>
</system.runtime.serialization>
```

### DataReader映射到实体类
[查看代码](https://samxxxxx.github.io/2021-10-25/DataReader-Mapping-Model).