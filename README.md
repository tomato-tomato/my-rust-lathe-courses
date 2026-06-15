# 使用Lathe工具学习Rust

一个使用 [Lathe](https://github.com/devenjarvis/lathehttps://github.com/devenjarvis/lathe) 来学习Rust的个人仓库记录。本仓库包含将 Lathe 提炼成无 cli 版本的 skills、skills的使用说明、通过 kimi网页版规划的Rust学习路线、通过该skills生成的对应课程 以及 相关学习代码的记录。

本仓库中的 skills 通过 Qwen 3.7max 读取 Lathe 中的代码改写生成，可直接复制到QoderWork 的skills文件夹中，直接在 QoderWork 中使用。

非常感谢 Lathe 这个工具 🙏 ！终于让我找到适合自己的学习 Rust 的方法。

## 本仓库介绍

基本结构

```
my-rust-lathe-courses/
├── .git/
├── .gitmodules          
├── skills/              ← 根据lathe生成的skills
├── docs/                ← skills使用文档、rust学习相关文档
├── tutorials/           ← skills生成的课程
└── code/
    └── todo-cli-rust/   ← 课程对应代码。submodule，内部有独立的 .git
```

[Lathe 技能在QoderWork中的使用教程](./docs/skills-tutorial.md) 

目前整理的一些 [Rust 学习路径](./docs/learn-rust-way.md) 

## 一些经历、牢骚😂

身处 AI 爆发式增长的大环境，就一直都在寻找怎样用AI工具来学习Rust的方法。受限于Anthropic 对国内用户的各种限制，还有使用国外各种优秀大模型需要面临的各种各样的问题。所以把大部分关注点聚焦于可以在国内使用的，好用的，且最好是不用花费太多的AI工具。

国内确实发现一些专门针对学习这个领域的一些探索性的AI相关的产品，比如清华大学的开源项目 [openMAIC](https://open.maic.chat/) ，但是这个项目感觉目前更适合专注于学习某个知识点，这种课堂的模式更适合学生群体。再比如 千问小课堂（当前仅在手机端应用找到入口，网页端以及桌面应用端都找不到😂），也是类似讲课的模式，更适合学生群体。

很早也关注过Google的实验室项目 —— Learn your way。想给它上传Rust官网那套教程文档，让它生成学习课程，可是就是一直让我填一个表格，无法进入使用😩。后来偶然发现了 网页版 Gemini 的学习模式，一下子打开新世界。采用问答的形式，及时反馈很有成就感，问题讲解的通俗易懂，体验感很好可以当一个在线家教使用。

对于Rust的学习尝试过看官方的学习文档，结果看不下去。通过 Rustlings学习，总是学完现在的前面的又都忘了。花钱买课程，跟看文档一样还是看学不下去。所以一直想找些能通过做项目来学习Rust的方法，问了各种 AI，就是给你列出一些学习项目，就没一个系统性的东西。

直到前些天逛 [Zeli](https://zeli.app/zh) 遇到了 [Lathe](https://github.com/devenjarvis/lathe) 这个项目（再次感谢编写这个项目的大佬🙏）， 看了这个工具的介绍惊为天人啊😲！感觉这就是我要找的东西。所以又开始问各种AI，这个工具在国内有啥好的使用媒介。

最开始想法是使用 DeepSeek v4（因为便宜😁）相关的agent工具，看看能不能直接结合 Lathe 这个仓库中的使用教程使用。结果就是感觉有点麻烦，就想着能不能把这个提炼成skills来用，问了几个AI得到了肯定答复。然后就是找有免费额度的国内AI工具，选择了 QoderWork。 有免费额度，能用，操作比较方便，能直接查看生成的教程。找 kimi 总结了些 Rust学习项目，然后这样学起来了😄

但愿这次自己能坚持学下去吧😤！💪加油
