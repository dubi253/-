# VS Code 使用 Prettier 进行代码格式化指南

Prettier 是一款当下极其流行且“强主见”（opinionated）的代码格式化工具。它通过解析代码并根据自身规则重新约束输出，从而保证整个团队代码风格的绝对一致。支持 JavaScript、TypeScript、HTML、CSS、JSON、Markdown 等众多语言。

本指南将带你在 VS Code 中安装和配置 Prettier。

---

## 1. 安装 Prettier 插件

在离线环境中，我们需要通过 `.vsix` 离线包来手动安装 Prettier 插件：

1. 提前准备好 `esbenp.prettier-vscode` 插件的 `.vsix` 离线安装包，并将其导入到你的无网开发环境设备中。
2. 打开 VS Code，点击左侧活动栏的 **扩展 (Extensions)** 图标（或按下 `Ctrl+Shift+X`）。
3. 点击扩展面板侧边栏右上角的 **...** (更多操作) 图标。
4. 在下拉出的菜单中选择 **从 VSIX 安装... (Install from VSIX...)**。
5. 在弹出的文件系统选择框中，找到并选中你刚才导入的 `.vsix` 插件文件，点击安装。等待右下角提示安装完成即可。

---

## 2. 配置默认格式化设置

安装完成后，你需要明确告诉 VS Code 优先使用 Prettier 作为默认的格式化工具。

**方法 A：通过图形界面配置**

- **打开 VS Code 设置**：快捷键 `Ctrl+,`（macOS 为 `Cmd+,`）。
- **设置默认格式化工具**：在搜索框中输入 `Default Formatter`，在下拉菜单中找到并选择 `Prettier - Code formatter`。

**方法 B：通过 settings.json 配置（推荐）**

如果你更喜欢直接修改配置文件，可以打开 VS Code 的 `settings.json`，在里面添加或修改以下代码：

```json
{
  // 设置默认的格式化工具为 Prettier
  "editor.defaultFormatter": "esbenp.prettier-vscode",
  // 针对特定语言可以单独配置（可选）
  "[javascript]": {
    "editor.defaultFormatter": "esbenp.prettier-vscode"
  }
}
```

---

## 3. 手动激活格式化与自动格式化 (可选)

在配置好默认格式化工具后，你可以根据个人习惯选择“手动”或“自动”格式化代码。

### 3.1 手动激活格式化（常规推荐）

你可以在编写代码时，通过以下快捷键随时手动触发格式化：

- **Windows / Linux**: 按下 `Shift + Alt + F`
- **macOS**: 按下 `Shift + Option + F`
- **右键菜单**: 在代码编辑区右键，选择 **格式化文档 (Format Document)**。

### 3.2 自动格式化（保存时格式化，按需可选）

如果你希望每次执行保存操作（如按下 `Ctrl+S`）时，VS Code 自动为你整理代码，可以开启并配置“保存时格式化”功能。

> **⚠️ 重点警告（与“自动保存”的冲突）：**  
> 如果你之前开启了 VS Code 的 **自动保存** 功能，且模式为 `afterDelay`（延迟自动保存），**不建议**开启自动格式化！因为一旦你打字时有短暂亦或细微的停顿，代码就会立刻被静默保存并强制格式化，这会导致输入框乱跳和代码闪烁。  
> **如何解决？** 如果你坚持想要同时拥有“自动保存”和这是“自动格式化”，请前去设置面板，将 `files.autoSave`（自动保存模式）修改为 `onFocusChange` (失去焦点时保存) 或 `onWindowChange` (切换窗口时保存)。

如果你确信自己的保存策略适合开启此功能，可以通过如下方式开启：

**方法 A：通过图形界面配置**

- **开启保存时格式化**: 在设置（`Ctrl+,`）搜索框中输入 `Format On Save`，并勾选 `Editor: Format On Save` 选项。

**方法 B：通过 settings.json 配置**

```json
{
  "editor.formatOnSave": true
}
```

---

## 4. 自定义 Prettier 格式化规则

虽然 Prettier 强调无需配置直接可用，但你仍可以根据团队喜好微调规则。通常推荐在项目根目录创建一个 `.prettierrc` 文件来统一定义规则，这样所有参与项目的开发者行为都会一致。

在项目根目录新建 `.prettierrc` 文件，填入常见配置（按需修改）：

```json
{
  "printWidth": 80, // 单行代码最大长度（推荐80或100）
  "tabWidth": 2, // 缩进级别（2个空格）
  "useTabs": false, // 是否使用制表符（通常为false，即使用空格）
  "semi": true, // 语句末尾是否添加分号
  "singleQuote": true, // 是否使用单引号（JS/TS中常开启）
  "trailingComma": "es5", // 多行对象、数组时结尾是否加逗号
  "bracketSpacing": true, // 对象字面量括号内前后是否有空格 { foo: bar }
  "arrowParens": "always" // 箭头函数单参数时是否加括号 (x) => x
}
```

> **注意**：一旦项目包含 `.prettierrc` 文件，VS Code 的 Prettier 插件将优先读取该文件中的规则，自动忽略编辑器内的个人全局设置。

---

## 5. 忽略不需要格式化的文件

有时你不希望 Prettier 去修改某些大文件或自动生成的文件（如 `node_modules`, `dist` 或压缩后的代码文件），你可以在项目根目录下创建一个 `.prettierignore` 文件，语法与 `.gitignore` 类似：

```text
# .prettierignore
node_modules/
dist/
build/
*.min.js
```

## 6. 与 ESLint 的冲突处理建议 (进阶)

如果在前端项目里同时使用了 ESLint 进行代码检查及格式约束，它俩的规则可能发生冲突（比如 ESLint 要求不加分号，而 Prettier 默认加）。

目前社区的最佳实践是：**让 Prettier 专职负责格式化，让 ESLint 专职负责代码质量检查（语法与逻辑报错）**。
解决方案是安装 `eslint-config-prettier`：
禁用所有与格式化有关的 ESLint 规则（将其覆盖为 Prettier 的规则），只保留逻辑检查规则。这样两者合作将毫无违和感。
