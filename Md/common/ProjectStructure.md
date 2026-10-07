A. 项目结构说明





A1 概述





A1.1 所有 HBuilderX 项目都必须遵循此项目结构规范。

A1.2 标准项目结构分为两层：项目层和共享层。





A2 项目层 (Project Level)





A2.1 每个项目都有自己的根目录，目录结构如下：





A2.1.1 pages/

页面文件目录。





A2.1.2 uni_modules/

uni-app 插件模块目录。





A2.1.3 md/

项目文档目录，供项目内的 agent 使用。包含 main/、chief-of-staff/、common/、uni-agent/、reviewer/、todo/、knowledgebase/ 等子目录。





A2.1.4 autotest/

自动化测试相关文件目录。





A2.1.5 static/

静态资源文件目录。





A2.1.6 server/

服务端相关文件目录。





A2.1.7 node_modules/

npm 依赖包目录。





A2.1.8 non-git/

不需要 git 同步的文件目录，如截图、临时文件等。





A2.1.9 unpackage/

编译输出目录。





A2.2 md/ 子目录说明





A2.2.1 main/

main agent 专属档案。





A2.2.2 chief-of-staff/

chief-of-staff agent（项目经理）专属档案。





A2.2.3 common/

所有 agent 都需要读取的通用档案。





A2.2.4 uni-agent/

uni-agent（程序员）专属档案。





A2.2.5 reviewer/

reviewer（测试）专属档案。





A2.2.6 todo/

工作任务的 md 文件，命名规范为 todo001.md、todo002.md...。





A2.2.7 knowledgebase/

知识库 md 文件，命名规范为 kb001.md、kb002.md...。





A3 共享层 (Shared Level)





A3.1 hbxCommonMd01 是跨项目共享的文件夹。

A3.2 所有 HBuilderX 项目都应能访问 hbxCommonMd01 的内容。

A3.3 hbxCommonMd01 包含：





A3.3.1 Md/

各类型 agent 的通用模板和配置。





A3.3.2 Skills/

共享的 skills，如 uniautotest、forward-uniagent、crossagentcommunication 等。





A3.3.3 NamingSpacingAndOtherRules.md

隔行规矩文档，所有 md 文件必须遵守。





A4 路径约定





A4.1 项目路径由人类在分配任务时提供。

A4.2 共享层路径由人类配置到各 agent 的环境中。

A4.3 引用共享层时使用相对路径，不使用绝对路径。
