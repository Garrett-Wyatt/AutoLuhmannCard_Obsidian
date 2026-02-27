---
banner: "![[header.png]]"
---
# 影子索引 (AI-Only Reference)

> [!IMPORTANT]
> 这是一个自动生成的索引文件，用于 Text Generator 插件的 context 注入。请勿手动修改。


```dataviewjs
// 1. 获取所有卡片数据
const pages = dv.pages('"Zettelkasten"')
    .where(p => p.luhmann_id != null)
    .sort(p => p.luhmann_id, 'asc');

// 2. 生成供 AI 阅读的纯文本索引（核心是加入 summary）
let indexContent = "### 📥 卢曼卡片盒索引总览\n";
indexContent += "格式：[编号] | [逻辑标题] | [核心摘要] | [[物理路径]]\n\n";

pages.forEach(p => {
    // 过滤掉 summary 中的换行符，防止破坏索引结构
    const cleanSummary = p.summary ? p.summary.replace(/\n/g, " ") : "暂无摘要";
    
    indexContent += `${p.luhmann_id} | ${p.file_name} | ${cleanSummary} | [[${p.file.name}]]\n`;
});

// 3. 将生成的索引渲染在当前页面
dv.paragraph(indexContent);
```
