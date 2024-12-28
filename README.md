# Homebrew tap repository

## 1. 创建 `tap`

使用 `brew tap-new` 命令即可。详细说明见 `brew tap-new -h`。

`formula` 可以放在 `Formula` 或 `HomebrewFormula` 文件夹下，也可以放在仓库根目录下。

`formula` 尽量不要与 [`homebrew-core`](https://github.com/Homebrew/homebrew-core) 仓库中的重名，以避免冲突。

## 2. Formula

### 2.1. General

```ruby
desc "Remote terminal with IP roaming"
homepage "https://mistertea.github.io/EternalTerminal/"
url "https://github.com/MisterTea/EternalTerminal/archive/et-v6.2.9.tar.gz"
head "https://github.com/MisterTea/EternalTerminal.git"   # 开发版本源代码地址 (可选)
sha256 "13bfb2722b011b5f0a28fa619508deca96deec9eee5e42b922add0c166d8185a"
version "6.2.9"   # 显式指定版本号 (如果 URL 自动提取, 则此行可省略?)
revision 2        # 修复配置选项的修订版本
```

如果 `head` 需要指定特定分支或标签：

```ruby
head "https://github.com/example/software.git", branch: "develop"
head "https://github.com/example/software.git", tag: "v2.0-beta"
```

### 2.2. `keg_only`

用于定义某个 `formula` 是否应安装为「仅存放在 keg 中」，换句话说，设置了 `keg_only` 的 `formula` 的二进制文件、库和其他资源不会直接添加到系统的全局路径中，需要用户手动指定路径（`$(brew --prefix)/Cellar/<formula>/<version>/bin/<executable>`）来使用它们。

语法：

```ruby
keg_only reason, explanation
```

- `reason`：符号或字符串，用于说明为什么这个 `formula` 是 `keg`-only 的。Homebrew 提供了一些常见的预定义理由。

- `explanation`（可选）：提供额外的说明或上下文，帮助用户理解为什么这个 `formula` 是 keg-`only` 的。

#### 2.2.1. 常见的 `keg_only` 理由

Homebrew 提供了一些预定义的符号，可以作为 `reason` 参数。例如：

1. `:provided_by_macos`

    用于指示该 `formula` 提供的功能或库已经由 macOS 原生提供，因此避免覆盖系统版本。

    示例：

    ```ruby
    keg_only :provided_by_macos, "macOS 提供了类似的功能"
    ```

2. `:shadowed_by_macos`

    类似于 `:provided_by_macos`，但用于描述 macOS 原生的工具更常用。

3. `:versioned_formula`

    用于处理特定版本的 `formula`。例如，如果用户安装了 Python 3.8，但系统默认使用 Python 3.11。

    示例：

    ```shell
    keg_only :versioned_formula, "这个版本和默认版本冲突"
    ```

4. `:conflicts_with_system`

    如果某个 `formula` 与系统软件直接冲突，可以使用此选项。

5. 自定义理由

    可以提供一个自定义字符串来说明具体原因。

    示例：

    ```ruby
    keg_only "自定义原因", "防止覆盖自定义编译的版本"
    ```

示例：

```ruby
class MyFormula < Formula
  desc "Example formula for keg_only usage"
  homepage "https://example.com"
  url "https://example.com/myformula-1.0.0.tar.gz"
  sha256 "abc123..."

  keg_only :provided_by_macos, "macOS 提供了类似的工具"

  def install
    bin.install "my_executable"
  end
end
```

安装该 `formula` 后，Homebrew 会提醒：

```text
This formula is keg-only, which means it was not symlinked into /usr/local, because macOS 提供了类似的工具.
```

### 2.3. `conflicts_with`

如果需要指定 `formula` 与其他软件包存在冲突，可以使用 `conflicts_with` 指令：`conflicts_with <FORMULA>, because: <REASON>`。例如：

```ruby
conflicts_with "ex-vi",
because: "vim and ex-vi both install bin/ex and bin/view"

conflicts_with "macvim",
because: "vim and macvim both install vi* binaries"
```

### 2.4. `depends_on`

基础用法：

1. 构建时依赖（`build`）

    `build` 类型的依赖只在构建软件包时需要，在运行时不再需要：

    ```ruby
    depends_on "cmake" => :build
    ```

2. 运行时依赖（`runtime`）

    `runtime` 类型的依赖在软件运行时必须存在（这是默认类型，因此可以省略 `:runtime`）。

    ```ruby
    depends_on "openssl@3"

    # 等价于
    depends_on "openssl@3" => :runtime
    ```

3. 测试时依赖（`test`）

    `test` 类型的依赖只在测试阶段使用。

    ```ruby
    depends_on "wget" => :test
    ```

4. 多类型依赖

    一个依赖可以同时用于多个阶段，例如：

    ```ruby
    depends_on "autoconf" => [:build, :test]
    ```

根据条件指定依赖：

1. 操作系统条件

    ```ruby
    depends_on "libomp" if OS.mac?
    depends_on "gcc" if OS.linux?
    ```

2. 选项条件

    如果 formula 允许用户通过选项选择不同的功能，可以根据选项条件添加依赖：

    ```ruby
    option "with-feature", "Enable some feature"

    depends_on "some_dependency" if build.with? "feature"
    ```

3. formula 版本条件

    可以根据软件包的版本指定依赖，例如：

    ```ruby
    depends_on "python@3.9" if MacOS.version <= :catalina
    ```

特殊依赖：

1. 使用系统工具

    有时软件需要系统中的工具，可以使用 `:macos` 或 `:linux` 系统依赖：

    ```ruby
    depends_on :xcode => :build   # 需要 Xcode
    depends_on :macos             # 仅支持 macOS
    depends_on :linux             # 仅支持 Linux
    ```

2. Java 依赖

    如果软件需要 Java 运行环境，可以使用以下声明：

    ```ruby
    depends_on :java => "11"   # 需要 Java 11
    ```

3. 特定编译器

    如果软件需要特定版本的编译器，可以声明如下：

    ```ruby
    depends_on "gcc"   # 需要 GCC 编译器
    ```

## 3. 安装

### 3.1. 远程仓库

当仓库托管在 GitHub 时，可以直接使用 `brew install user/repo/formula` 安装。安装前，Homebrew 会自动执行：

```shell
brew tap github.com/user/homebrew-repo
```

`user/repo/formula` 指向 `github.com/user/homebrew-repo/**/formula.rb` 文件。

如果只需要添加 `tap` 而不需要安装：

```shell
# 如果托管在 GitHub 上, 可以这样写 (其他情况要写完整的 URL)
brew tap user/repo

# 其等价于
brew tap github.com/user/homebrew-repo
```

添加 `tap` 后，可以用 `brew install foo` 命令安装，前提是 [`homebrew-core`](https://github.com/Homebrew/homebrew-core) 仓库中没有同名软件包。如果存在同名软件包，需要用 `brew install user/repo/foo` 命令。

如果是在 [`brew bundle`](https://github.com/Homebrew/homebrew-bundle) 的 `Brewfile` 中：

```ruby
tap "user/repo"
brew "<formula>"
```

### 3.2. 本地文件

```shell
brew install -s ./path/to/formula.rb
```

## 4. 相关说明

[How to Create and Maintain a Tap](https://docs.brew.sh/How-to-Create-and-Maintain-a-Tap)

[Taps (Third-Party Repositories)](https://docs.brew.sh/Taps)

[brew(1) – The Missing Package Manager for macOS (or Linux)](https://docs.brew.sh/Manpage)
