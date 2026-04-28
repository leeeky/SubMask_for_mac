# SubMask
macOS计算子网掩码
# 子网计算器应用架构文档

## 1. 项目概述

本应用是一个基于 macOS 风格设计的专业子网计算工具，采用 SwiftUI 构建，旨在为网络工程师和学生提供直观、准确的子网划分和计算功能。应用严格遵循 MVVM 架构模式，确保代码的可维护性和可扩展性。

## 2. 核心功能

*   **IP 地址输入与验证**: 支持标准点分十进制 IP 地址输入，实时进行格式校验（范围 0-255，无非法前导零）。
*   **子网掩码调整**: 提供滑块（Slider）和文本框两种方式调整 CIDR（0-32 位），支持实时联动。
*   **智能联动**:
    *   在 IP 输入框中输入 CIDR 后缀（如 `/24`）会自动更新滑块。
    *   在 IP 输入框中输入子网掩码（如 `/255.255.255.0`）会自动识别并更新 CIDR。
    *   在 IP 输入框中输入子网掩码（如 ` 255.255.255.0`）会自动识别并更新 CIDR。
    *   拖动滑块会自动更新 IP 输入框中的 CIDR 后缀。
*   **实时计算**:
    *   网络地址 (Network Address)
    *   广播地址 (Broadcast Address)
    *   子网掩码 (Subnet Mask)
    *   CIDR 格式 (Classless Inter-Domain Routing)
    *   子网掩码格式 (Dotted Decimal Subnet Mask)
    *   通配符掩码 (Wildcard Mask)
    *   可用 IP 范围 (First/Last Usable)
    *   可用主机总数 (Total Hosts)
    *   IP 类别识别 (A/B/C/D/E 类)
*   **二进制可视化**: 直观展示 IP 地址的二进制形式，高亮显示网络位与主机位。
*   **深色模式**: 支持一键切换亮色/深色主题。
*   **快速选择**: 提供 A、B、C 类私有地址的快速预设。

## 3. 架构设计 (MVVM)

### 3.1 Model (模型层)

**`SubnetEngine.swift`**
*   **职责**: 纯逻辑计算引擎，无状态，线程安全。
*   **核心结构**: `SubnetResult` 结构体，封装所有计算结果。
*   **关键方法**:
    *   `calculate(ip:cidr:) -> SubnetResult`: 核心计算入口。
    *   `ipToInt`, `intToIp`: IP 字符串与 32 位整数互转。
    *   `validateIP`: 严格的正则表达式逻辑验证。
    *   `maskToCidr`: 将掩码字符串转换为 CIDR 整数。

### 3.2 ViewModel (视图模型层)

**`CalculatorViewModel.swift`**
*   **职责**: 管理应用状态，连接 View 与 Model。
*   **状态**:
    *   `@Published var ipAddress`: 当前输入的 IP 字符串。
    *   `@Published var cidr`: 当前 CIDR 值 (Double 类型适配 Slider)。
    *   `@Published var result`: 当前计算结果。
*   **逻辑**:
    *   监听 `ipAddress` 和 `cidr` 的变化，自动触发 `SubnetEngine.calculate`。
    *   执行 CIDR 范围限制 (0-32)。

### 3.3 View (视图层)

**`ContentView.swift`**
*   **职责**: 主界面容器，协调左右分栏布局。
*   **结构**: 左侧 `InputSection`，右侧 `ResultCard` 网格 + `BinaryVisualizer`。

**`InputSection.swift`**
*   **职责**: 处理用户输入。
*   **特性**:
    *   包含 IP 文本框、CIDR 滑块、CIDR 数字输入框。
    *   实现复杂的双向绑定逻辑，处理输入冲突和格式同步。
    *   包含错误提示气泡（当 CIDR > 32 时）。

**`ResultCard.swift`**
*   **职责**: 展示单个计算项。
*   **特性**: 支持点击复制，鼠标悬停效果。

**`BinaryVisualizer.swift`**
*   **职责**: 二进制位展示。
*   **特性**: 动态根据 CIDR 高亮网络位颜色。

## 4. 关键逻辑细节

### 4.1 特殊子网处理
*   **/32 子网**: 视为单机 IP。
    *   网络地址 = 广播地址 = IP 本身。
    *   可用主机数 = 1。
*   **/31 子网**: 点对点链路 (RFC 3021)。
    *   可用主机数 = 2。
    *   First Usable = Network Address。
    *   Last Usable = Broadcast Address。

### 4.2 交互优化
*   **防抖与联动**: 输入框与滑块的更新循环被精心处理，确保用户输入 CIDR 后缀时滑块更新，而滑块拖动时后缀更新，互不干扰且逻辑自洽。
*   **视觉反馈**: IP 格式错误时输入框边框变红并显示警告图标。

## 5. 文件结构

```
SubnetCalculatorApp/
├── SubnetCalculatorApp.swift  // 应用入口
├── ContentView.swift          // 主视图
├── CalculatorViewModel.swift  // 视图模型
├── SubnetEngine.swift         // 核心算法
└── Components/
    ├── InputSection.swift     // 输入区域组件
    ├── ResultCard.swift       // 结果卡片组件
    └── BinaryVisualizer.swift // 二进制可视化组件
```

## 6. 技术栈
*   **语言**: Swift 5
*   **UI 框架**: SwiftUI
*   **平台**: macOS
*   **设计风格**: macOS Native (半透明背景、圆角、Monospaced 字体)
