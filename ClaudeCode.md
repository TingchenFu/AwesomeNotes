## Notes from official claude courses

+ run "/init" to create CLAUDE.md in the working directory. 还有另外两个CLAUDE.md 分别是 CLAUDE.local.md 和~/.claude/CLAUDE.md; 前者是personalized claude文件 后者是全局的claude文件
+ Use "think", "think more", "think a lot", "think longer", "ultrathink" as key words to trigger thinking with different thinking token budgets; "shift"+"tab" for plan mode
+ Esc 中断claude; Esc+Esc 两次可以对话回滚到之前的问题; /compact 压缩对话 /clear 开始新的对话
+ 在/claude/commands/ 下面创建{command_name}.md 可以使用自定义的名为command_name的指令


### MCP

+ MCP 的定义是communication layer that provides Claude with context and tools without requiring you to write a bunch of tedious integration code 相当于调用第三方service(例如Github)的工具集合 定义了一系列的tool schema 和function. 注意定义的并不是工具本身而是工具的范式, 或者换句话说developer's app 与service provider 进行通讯的时候应该遵循怎样的协议怎样的接口类型规范等等
+ MCP 的workflow: <img width="1920" height="1080" alt="image" src="https://github.com/user-attachments/assets/97697a2f-e427-4c19-ba47-6000313dfcce" />
+ 其中MCP server 暴露三类原语(tool, resource 以及 prompt) 给MCP client; MCP server 并不与our server 直接接触, 而是完全由MCP client封装起来; 这三类原语的功能定位有所差异 但核心目的都是把处理MCP server 所具有的资源 这些资源包括已有的数据文件(对应resource) 已有的函数(对应tool) 以及已有的template(对应prompt)
+ 一般来说，MCP 开发要么是作为App developer来开发Client 要么是作为serivce provider 去开发Server 而不会同时开发两者
