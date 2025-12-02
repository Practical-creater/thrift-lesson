## 本项目流程
- [[#构建 match. thrift]]
- [[#通过 match.thrift 接口构建 match_server]]
- [[#通过 match.thrift 接口构建 match_client ]] 
- [[#match_server v2.0]]
- [[#构建 save.thrift 接口]]
- [[#通过 save.thrift 接口构建 save_client]]
- [[#match_server v3.0]]
- [[#match_server v4.0 ]]
- [[#match_server v5.0 ]]



### 构建 match. thrift
1. 创建 thrift 文件夹，以保存各种.thrift 文件，在文件夹下创建 match.thrift 文件。
``` thrift
namespace cpp match_service

//接口传入的用户信息
struct User {                                  
    1: i32 id,
    2: string name,
    3: i32 score
}

//接口需要的服务方法
service Match {                                    
    i32 add_user(1: User user, 2: string info),    //user: 用户信息
    i32 remove_user(1: User user, 2: string info), //info: 附加信息
}
```

*  小结
	- 在 Thrift 的 [.thrift](https://thrift.apache.org/docs/types) 文件中，可以定义以下内容：

	- 命名空间（Namespace）：指定当前 `.thrift` 文件所属的命名空间。命名空间可以防止不同服务或数据类型之间的冲突。
	- 数据类型（Data Types）：定义结构化数据类型，包括基本数据类型（如整数、字符串、布尔值）和复杂数据类型（如结构体、枚举、集合、映射等）。
	- 服务（Services）：定义服务接口和方法。每个服务包含一个或多个方法，每个方法包含输入参数和返回类型。
	- 异常（Exceptions）：定义可能在服务调用过程中抛出的异常。
	- 注释（Comments）：提供对代码的说明和文档。可以使用单行注释（`//`）或多行注释（`/* */`）。




### 通过 match.thrift 接口构建 match_server
补充说明：在自己的环境中（macbookpro 2021）还需要通过brew安装thrift环境。
`brew install thrift` 
完成安装后，输入一下命令查看版本：
`thrift --version`
如果输出了类似 Thrift version 0.16.0 (或更高版本) 的字样，说明编译器安装成功了。 此时就可以运行教程里的 thrift --gen cpp match.thrift 命令了.
为了取消VScode中编辑器的警告，需要作如下编辑，首先打开命令面板，搜索框输入`C/C++: Edit Configurations (JSON)`, 添加**homebrew**路径
```json
{
    "configurations": [
        {
            "name": "Mac", // 找到你的 Mac 配置
            "includePath": [
                // 保持原有的路径，例如：
                "${workspaceFolder}/**",
                // ---------- 添加这行 ----------
                "/opt/homebrew/include"
                // -----------------------------
            ],
            "defines": [],
            "macFrameworkPath": [],
            "compilerPath": "/usr/bin/clang", // 或你的编译器路径
            "cStandard": "c17",
            "cppStandard": "c++17",
            "intelliSenseMode": "macos-clang-arm64" 
        }
    ],
    "version": 4
}
```


1. 在 thrift 同层文件夹，创建 match_system 文件夹，再创建 src 子文件夹，表示源代码文件夹。在 src 文件夹下执行 thrift 指令 `thrift --gen cpp ../../thrift/match.thrift` 生成 gen-cpp 文件夹。
2. 把生成的 gen-cpp 文件夹改名为 match.server，并把其中的 Match_server.skeleton.cpp 移动到 src 下，并改名为 main.cpp。
3. 为使编译成功，把 main.cpp 内的 add_user 和 remove_user 方法加上 `return 0;`。并且修改头文件路径 `Match.h` 为 `match_server/Match.h`。
   我在VScode环境下运行会有警告，由于我使用的是 Apple Silicon (arm64_sequoia)，Homebrew 將 Thrift 庫安裝到了 /opt/homebrew 路徑下。编译C++ 服務端時，編譯器 (g++) 不會自動知道去哪裡找這些庫和頭文件。编译时要加上`-I`参数。`g++ -c main.cpp match_server/*.cpp -I/opt/homebrew/include`
   链接时，加上 `-L` 参数（L for Library）`g++ *.o -o main -L/opt/homebrew/lib -lthrift`

為了避免編譯時出現 file not found 或 Undefined symbols 錯誤，請確保在編譯和鏈接時，將 /opt/homebrew/ 路徑包含進去。
4. 在 src 路径下编译所有 cpp 文件，`g++ -c main.cpp match_server/*.cpp`，链接时引入 thrift 库，`g++ *.o -o main -lthrift`，生成可执行文件 main。 


- 小结
	- `thrift --gen <language> <Thrift filename>` 是 Thrift 编译器的命令行工具，用于根据指定的 Thrift 文件生成特定语言的代码。执行这个命令后，Thrift 编译器会解析指定的 Thrift 文件，并根据文件中定义的结构体、枚举、服务等内容，生成相应语言的代码文件。生成的代码包含了数据结构的定义、序列化和反序列化方法，以及用于与其他服务进行通信的客户端和服务端代码。
	- 把文件夹等文件改名移动是为了后续方便管理项目，并且在引入头文件和编译时要注意修改路径。
	- 一般情况下 git 只需上传 .h 和 .cpp 文件，可执行文件和 . o 等中间文件无需上传。
