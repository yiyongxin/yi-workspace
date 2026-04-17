# yi-workspace

版本：v0.0.1  
更新日期：2026-04-17

## 1. 架构总览

```text
┌─────────────────────────────────────────────────────────────┐
│  apps/                    业务服务层（极高频率）              │
│  platform-gateway · lower-platform · superior-platform       │
└─────────────────────────────┬───────────────────────────────┘
                              │
┌─────────────────────────────▼───────────────────────────────┐
│  protocols                共享协议层（高频率）                │
│  跨服务通信协议定义（.fbs）及生成代码                         │
└─────────────────────────────┬───────────────────────────────┘
                              │
┌─────────────────────────────▼───────────────────────────────┐
│  framework/yi-modules     通用模块层（中频率）                │
│  MQ/Redis封装 + 配置管理 + 业务基础（错误码/常量/工具）        │
└─────────────────────────────┬───────────────────────────────┘
                              │
┌─────────────────────────────▼───────────────────────────────┐
│  framework/core        核心框架层（极低频率）              │
│  协程 · 调度器 · 序列化抽象 · 基础类型（Bytes/Units等）       │
└─────────────────────────────────────────────────────────────┘
```

## 2. 仓库清单

| 仓库 | 内容 | 更新频率 | 初始版本 |
| --- | --- | --- | --- |
| yi-core | 协程、调度、基础类型、序列化抽象 | 极低 | v0.0.1 |
| yi-modules | 通用组件 + 业务基础（错误码/常量/工具） | 中 | v0.0.1 |
| protocols | 跨服务共享协议定义（.fbs）及生成代码 | 高 | v0.0.1 |
| platform-gateway | 网关服务 | 极高 | 持续部署 |
| lower-platform | 下级平台服务 | 极高 | 持续部署 |
| superior-platform | 上级平台服务 | 极高 | 持续部署 |

## 3. 工作区目录结构（工作台仓库）

```text
yi-workspace/                       # 工作台仓库（Git 子模块聚合）
├── .gitmodules
├── framework/
│   ├── core/                    # [子模块] L0
│   └── yi-modules/                 # [子模块] L1
├── protocols/                      # [子模块] L2
├── apps/                           # L3 业务服务
│   ├── platform-gateway/           # [子模块]
│   ├── lower-platform/             # [子模块]
│   └── superior-platform/          # [子模块]
└── tools/                          # 可选辅助工具
```

一键克隆：

```bash
git clone --recursive https://github.com/your-org/yi-workspace.git
```

## 4. 业务服务内部结构（以 platform-gateway 为例）

```text
platform-gateway/
├── CMakeLists.txt
├── config/
│   └── config.yaml
├── src/
│   ├── main.cpp
│   ├── gateway_service.cpp
│   ├── handlers/                   # 业务逻辑（协程）
│   └── protocols/                  # 服务专属协议（.fbs + 生成代码）
│       └── schemas/
├── tests/
└── README.md
```

## 5. 依赖引入（业务服务 CMakeLists.txt 核心）

```cmake
cmake_minimum_required(VERSION 3.24)
project(PlatformGateway)

include(FetchContent)

# 本地覆盖根路径（工作台布局）
set(LOCAL_ROOT "${CMAKE_CURRENT_SOURCE_DIR}/../..")

# 引入 L0
FetchContent_Declare(yi_core
    GIT_REPOSITORY https://github.com/your-org/yi-core.git
    GIT_TAG v0.0.1
    SOURCE_DIR ${LOCAL_ROOT}/framework/core
)

# 引入 L1
FetchContent_Declare(yi_modules
    GIT_REPOSITORY https://github.com/your-org/yi-modules.git
    GIT_TAG v0.0.1
    SOURCE_DIR ${LOCAL_ROOT}/framework/yi-modules
)

# 引入 L2
FetchContent_Declare(protocols
    GIT_REPOSITORY https://github.com/your-org/protocols.git
    GIT_TAG v0.0.1
    SOURCE_DIR ${LOCAL_ROOT}/protocols
)

FetchContent_MakeAvailable(yi_core yi_modules protocols)

add_executable(platform_gateway ...)
target_link_libraries(platform_gateway
    PRIVATE
        yi::core
        yi::modules
        yi::protocols
)
```

## 6. 开发工作流

| 场景 | 操作 |
| --- | --- |
| 仅开发业务服务 | 在工作区 apps/ 下对应服务目录执行 CMake 构建，依赖自动从 Git 拉取 |
| 联调 L1/L2 | 修改 framework/yi-modules 或 protocols 源码，业务服务直接重编译生效 |
| 升级依赖版本 | 修改业务服务 CMakeLists.txt 中的 GIT_TAG，提交变更 |
| 切换版本组合 | 在工作台仓库 `git checkout <branch/tag>`，执行 `git submodule update` |
| 发布稳定组合 | 在工作台仓库打标签，记录所有子模块的版本组合快照 |

## 7. 命名约定

| 元素 | 格式 | 示例 |
| --- | --- | --- |
| C++ 命名空间 | yi:: | yi::Task\<T\> |
| 基础类型子空间 | yi::common:: | yi::common::ByteBuffer |
| CMake 目标 | yi::core、yi::modules、yi::protocols | yi::core |
| 头文件包含 | \<yi/模块/头文件.hpp\> | `#include <yi/coro/task.hpp>` |

## 8. 技术栈

- 语言：C++20（协程）
- 构建：CMake 3.24+ + FetchContent
- 序列化：FlatBuffers（.fbs）
- 配置：YAML
- 版本控制：Git（子模块） + 语义化版本

## 9. Agents 与 工具链文档

项目将自定义 agent 文档集中放在 `docs/`，便于团队查阅与维护。当前文档：

- [Agents 说明](docs/agents.md) — 自定义 agent 列表与使用说明
- [代码审查规则](docs/code-review.md) — 初版代码审查清单与建议

建议：将 `docs/` 配置到项目静态站点（例如 MkDocs 或 GitHub Pages）以便更好地展示。
