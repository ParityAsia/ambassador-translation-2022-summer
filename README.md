# Substrate Ambassador Translation Proofread - 2022 Summer

## 流程

[校对任务表](https://docs.google.com/spreadsheets/d/1gwGQDpvdx55_PFxckoOuddMGreHOUKcYLiZofdxyQiw/edit?usp=sharing)

1. 为校对的每一篇开一个新目录。格式 `<ID>-<原文标题>`。`ID` 来源于校对任务表。

2. 里面放三个档案:

    - `1-original.md` - 文章英文原文
    - `2-translated.md` - 天使交付时的中文翻译
    - `3-proofread.md` - 我们校对后的中文翻译

3. 上面每篇 markdown 有些基本 frontmatter 信息。

    **1-original.md**:

    - `title`: 原英文标题
    - `src`: 英文文章链接
    - `author`: 英文文章作者
    - `snapshot-date`: 截取日期

    **2-translated.md**:

    - `title`: 中文翻译标题
    - `src`: 中文翻译链接
    - `author`: 翻译者名称
    - `snapshot-date`: 翻译文章截取日期

    **3-proofread.md**:

    - `proofreader`: 校对者
    - `completion-date`: 校对完成时间

4. 有圖片的話則存到 `./assets` 子目錄裡去。

5. 有些文章可能不是 markdown 格式，这要校对者自行作合理转换。

可参看[第一篇作模板](./01-xcm-the-cross-consensus-message-format)。
