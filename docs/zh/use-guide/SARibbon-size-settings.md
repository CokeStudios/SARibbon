# Ribbon尺寸设置

SARibbon 允许您精细调整 ribbon 各个部分的高度和间距，以适应不同的界面布局和高分辨率显示器需求。

## 核心特性

- ✅ **全局高度控制**：可独立设置标题栏、标签栏、Category 区域的高度
- ✅ **面板级尺寸调节**：支持面板标题高度、面板间距、按钮图标大小的统一设置
- ✅ **动态图标计算**：图标大小根据 Category 高度和文字高度自动计算，无需手动指定每个按钮
- ✅ **按钮宽高比限制**：通过最大宽高比系数控制按钮的最大宽度，防止过度拉伸
- ✅ **大按钮最小宽度控制**：通过最小宽度比例系数控制短文字大按钮的最小宽度，让按钮布局更紧凑

## 完整API摘要

所有尺寸设置通过 `SARibbonBar` 统一管理，无需直接操作单个 Panel：

| 方法 | 返回值 | 说明 |
|------|--------|------|
| `setTitleBarHeight(h)` | void | 设置标题栏高度（仅宽松模式可见） |
| `titleBarHeight()` | int | 获取标题栏高度 |
| `setTabBarHeight(h)` | void | 设置标签栏高度 |
| `tabBarHeight()` | int | 获取标签栏高度 |
| `setCategoryHeight(h)` | void | 设置 Category 面板内容区域总高度 |
| `categoryHeight()` | int | 获取 Category 高度 |
| `setPanelTitleHeight(h)` | void | 设置面板底部标题区域高度 |
| `panelTitleHeight()` | int | 获取面板标题高度 |
| `setPanelSpacing(n)` | void | 设置相邻面板间的水平间距 |
| `panelSpacing()` | int | 获取面板间距 |
| `setPanelToolButtonIconSize(s, l)` | void | 设置工具按钮的小图标和大图标尺寸 |
| `panelToolButtonIconSize()` | QPair<QSize, QSize> | 获取工具按钮图标尺寸对 |
| `setPanelLargeIconSize(size)` | void | 单独设置大图标尺寸 |
| `panelLargeIconSize()` | QSize | 获取大图标尺寸 |
| `setPanelSmallIconSize(size)` | void | 单独设置小图标尺寸 |
| `panelSmallIconSize()` | QSize | 获取小图标尺寸 |
| `setButtonMaximumAspectRatio(r)` | void | 设置按钮最大宽高比（控制最大宽度） |
| `buttonMaximumAspectRatio()` | qreal | 获取按钮最大宽高比 |
| `setLargeButtonMinimumWidthRatio(r)` | void | 设置大按钮最小宽度比例（控制最小宽度，紧凑布局） |
| `largeButtonMinimumWidthRatio()` | qreal | 获取大按钮最小宽度比例 |
| `normalModeMainBarHeight()` | int | 获取正常模式下的主栏高度 |
| `minimumModeMainBarHeight()` | int | 获取最小模式下的主栏高度 |

## 尺寸关系说明

Ribbon 的垂直高度由多个层级叠加而成，理解这些关系有助于合理配置尺寸：

```mermaid
block-beta
    columns 1
    block:title["TitleBar — setTitleBarHeight(h)（仅宽松模式可见）"]
        title_inner["标题栏区域"]
    end
    block:tab["TabBar — setTabBarHeight(h)"]
        tab_inner["标签栏区域"]
    end
    block:cat["Category — setCategoryHeight(h)"]
        columns 3
        p1["Panel 1"]
        p2["Panel 2"]
        p3["Panel 3"]
        pt["面板标题 — setPanelTitleHeight(h)"]:3
    end

    style title fill:#dae8fc,stroke:#6c8ebf
    style tab fill:#fff2cc,stroke:#d6b656
    style cat fill:#d5e8d4,stroke:#82b366
    style pt fill:#f8cecc,stroke:#b85450
```

Ribbon 各层级的层级关系和包含结构如下：

```
┌─────────────────────────────────────┐  ← 标题栏高度 (setTitleBarHeight)
│  标题栏 (仅宽松模式可见)              │
├─────────────────────────────────────┤  ← 标签栏高度 (setTabBarHeight)
│  标签栏 (Tab Bar)                    │
├─────────────────────────────────────┤  ← Category高度 (setCategoryHeight)
│  Category 内容区域                    │
│  ┌─────────┬─────────┬─────────┐    │     包含: 面板内容 + 面板标题
│  │ Panel 1 │ Panel 2 │ Panel 3 │    │     面板标题高度由 setPanelTitleHeight 控制
│  └─────────┴─────────┴─────────┘    │
└─────────────────────────────────────┘
```

**总高度公式**（宽松模式）：
> Ribbon总高度 = 标题栏高度 + 标签栏高度 + Category高度

**总高度公式**（紧凑模式）：
> Ribbon总高度 = 标签栏高度 + Category高度（紧凑模式无标题栏）

**总高度公式**（单行模式）：
> Ribbon总高度 = 标签栏高度 + Category高度（单行模式下 Category 高度仅为一行按钮高度，Panel 标题隐藏）

Category 高度内部包含面板内容区域和面板标题区域。面板标题高度通过 `setPanelTitleHeight` 单独控制，默认显示在面板底部。

## 使用示例

以下示例展示如何创建一个紧凑风格的 Ribbon 界面并调整各项尺寸：

```cpp
#include "SARibbonMainWindow.h"
#include "SARibbonBar.h"
#include "SARibbonCategory.h"
#include "SARibbonPanel.h"

class MainWindow : public SARibbonMainWindow {
public:
    MainWindow(QWidget* parent = nullptr) : SARibbonMainWindow(parent)
    {
        SARibbonBar* ribbon = ribbonBar();

        // 使用紧凑三行模式
        ribbon->setRibbonStyle(SARibbonBar::RibbonStyleCompactThreeRow);

        // 调整各部分高度，使界面更紧凑
        ribbon->setTabBarHeight(28);
        ribbon->setCategoryHeight(80);
        ribbon->setPanelTitleHeight(16);

        // 设置图标尺寸（小图标、大图标）
        ribbon->setPanelToolButtonIconSize(QSize(16, 16), QSize(24, 24));

        // 设置面板间距，增加视觉分隔
        ribbon->setPanelSpacing(4);

        // 添加分类和面板
        SARibbonCategory* cat = ribbon->addCategoryPage(tr("&File"));
        SARibbonPanel* panel = cat->addPanel(tr("Operations"));
        panel->addLargeAction(new QAction(tr("New"), this));
        panel->addSmallAction(new QAction(tr("Open"), this));
    }
};
```

运行效果：界面紧凑但功能完整，各面板间距适当，图标大小适合高分辨率显示。

## 大按钮最小宽度（紧凑布局）

大按钮（`addLargeAction` 生成的按钮）的最小宽度默认为 **按钮高度 × 0.75**。当按钮文字很短（如两个汉字）时，
文字宽度会小于这个最小值，按钮被撑到最小宽度，导致按钮显得过宽、布局不够紧凑。

通过 `setLargeButtonMinimumWidthRatio` 可以调小这个比例，让短文字按钮收缩到贴近文字/图标的宽度：

```cpp
SARibbonBar* ribbon = ribbonBar();

// 全局紧凑：最小宽度 = 按钮高度 × 0.4
ribbon->setLargeButtonMinimumWidthRatio(0.4);

// 完全取消按高度计算的下限，仅以 icon 宽度（加边距）作为最小宽度
ribbon->setLargeButtonMinimumWidthRatio(0.0);
```

**参数说明**：

| 取值 | 行为 |
|------|------|
| `> 0` | 最小宽度 = 按钮高度 × 比例（默认 0.75） |
| `<= 0` | 取消高度比例约束，最小宽度 = 大图标宽度 + 边距 |

- 仅对大按钮生效，小按钮/中按钮不受影响
- 默认值 0.75，与旧版本行为完全一致
- 也可在 `SARibbonCategory` / `SARibbonPanel` 级别单独设置（如 `panel->setLargeButtonMinimumWidthRatio(0.3)`），
  仅影响该范围内的按钮

!!! tip "效果参考"
    以默认字体下三行模式为例："打开"两字大按钮默认宽度约 59px（比例 0.75），设为 0.3 或 0 后收缩到约 34px，
    与图标宽度相当，布局明显更紧凑。

!!! example "交互式调试"
    运行 `example/MainWindowExample` 示例左侧 dock 的参数设置区，"Large Button Min Width Ratio" 数值框可实时调整此比例并查看效果。

!!! info "图标动态计算逻辑"
    SARibbon 的图标大小并非像传统工具栏那样设置固定值，而是根据 RibbonBar 的 `CategoryHeight` 和文字高度动态计算。
    布局时会自动决定图标大小。想让图标变大：增大 `CategoryHeight`，或调用 `setEnableWordWrap(false)` 关闭文字换行以腾出更多图标空间。

!!! tip "4K/多显示器适配建议"
    在 4K 或多显示器环境中，建议调用 SARibbonBar::initHighDpi() 启用高 DPI 支持，并通过 setPanelToolButtonIconSize 适当增大图标尺寸。
    同时可配合 QApplication::setAttribute(Qt::AA_EnableHighDpiScaling) 使用，确保 ribbon 在不同 DPI 显示器上显示一致。

!!! example "交互式调试"
    运行 `example/MainWindowExample` 示例的 **Size** 标签页，可动态调整各部件尺寸并实时查看效果。
    ![example-size](../../assets/pic/example-size.png)

## Gallery 拉伸系数

一个面板里放多个 Gallery 时，默认对额外宽度**均分**。如需按权重分配，可对每个 Gallery 设置拉伸系数：

```cpp
SARibbonGallery* g1 = panel->addGallery();
SARibbonGallery* g2 = panel->addGallery();
g1->setStretchFactor(2);  // g1 : g2 的最终宽度比约为 2 : 1
g2->setStretchFactor(1);
```

规则：

- 系数默认 `0`，表示沿用既有均分行为——不设置任何系数时布局与旧版本完全一致；
- 只要有一个 Gallery 设置了大于 0 的系数，就进入加权模式：最终宽度比接近权重比，系数为 `0` 的 Gallery 保持基础宽度不参与增量分配；
- 加权结果受列最大宽度（`QWidget::setMaximumWidth`）与最小宽度约束，实际比例会有少量偏差；
- 系数变化会自动触发所在面板重新布局，无需手动调用。

!!! note
    运行 `example/MainWindowExample` 的 **Size** 标签页可配合验证。
