```dataviewjs
const pages = dv.pages('"Zettelkasten"').where(p => p.luhmann_id != null);

// --- 1. 每小时随机回看 ---
const hourSeed = Math.floor(Date.now() / 3600000);
const luckyPage = pages[hourSeed % pages.length];
if (luckyPage) {
    dv.header(3, "🎲 随机回看");
    dv.el("div", `> [!quote] ${luckyPage.file_name}\n> **摘要**：${luckyPage.summary || "暂无摘要"}\n> [[${luckyPage.file.path}|点此复习]]`);
}

// --- 2. 总览统计：提取章节主题 (核心标签) ---
const roots = pages.groupBy(p => Math.floor(parseFloat(p.luhmann_id)));
dv.header(2, "📊 库概况与章节主题");

const summaryData = roots.sort(g => g.key, 'asc').map(g => {
    // 统计本章标签频率，排除通用标签
    const tagCounts = {};
    g.rows.flatMap(p => p.tags || []).forEach(t => {
        if (!["#zettel/growth", "#zettelkasten"].includes(t)) {
            tagCounts[t] = (tagCounts[t] || 0) + 1;
        }
    });
    const topTags = Object.entries(tagCounts).sort((a,b) => b[1]-a[1]).slice(0, 2).map(e => e[0]);
    
    return [
        `第 ${g.key} 章`,
        topTags.length > 0 ? topTags.join(" ") : "💡 探索中",
        g.rows.length + " 张",
        moment(Math.max(...g.rows.map(p => p.file.mday))).format("YYYY-MM-DD")
    ];
});
dv.table(["章节", "核心主题 (Top 标签)", "卡片数", "最后更新"], summaryData);

// --- 3. 章节详情：高鲁棒性关联逻辑 ---
roots.forEach(g => {
    dv.header(4, `📂 节点 ${g.key} 详情`);
    dv.table(["标题", "编号", "摘要", "溯源", "标签", "File"], 
        g.rows.sort(p => p.luhmann_id, 'asc').map(p => {
            let upLinkDisplay = "🚩 根节点"; // 默认设定为根节点
            
            // 1. 只有当 up_link 存在，且不是空字符串、不是空数组时才进入逻辑
            if (p.up_link && String(p.up_link).trim() !== "" && String(p.up_link) !== "[[ ]]") {
                const targetPath = (typeof p.up_link === 'object') ? p.up_link.path : p.up_link;
                const parent = dv.page(targetPath);
                
                if (parent) {
                    // 2. 如果找到了父页面，显示漂亮链接
                    upLinkDisplay = dv.fileLink(parent.file.path, false, parent.file_name || "未命名上级");
                } else if (p.luhmann_id && String(p.luhmann_id).includes('.')) {
                    // 3. 如果没找到父页面，但编号是 1.1 这种带点的，说明它是子节点但链接断了
                    upLinkDisplay = "⚠️ 关联失效";
                } else {
                    // 4. 其他情况（比如编号就是 1），依然视为根节点
                    upLinkDisplay = "🚩 根节点";
                }
            }

            return [
                p.file_name, 
                p.luhmann_id, 
                p.summary, 
                upLinkDisplay, 
                p.tags, 
                p.file.link
            ];
        })
    );
});
```

