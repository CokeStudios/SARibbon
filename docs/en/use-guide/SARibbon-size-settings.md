# Ribbon Size Settings

SARibbon allows you to finely adjust the height and spacing of various ribbon parts to accommodate different interface layouts and high-resolution display requirements.

## Core Features

- ✅ **Global height control**: independently set the height of the title bar, tab bar, and Category area
- ✅ **Panel-level size adjustment**: unified setting of panel title height, panel spacing, and button icon sizes
- ✅ **Dynamic icon calculation**: icon sizes are automatically calculated based on Category height and text height, no need to manually specify each button
- ✅ **Button aspect ratio constraint**: control the maximum button width through a maximum aspect ratio factor to prevent excessive stretching
- ✅ **Large button minimum width control**: control the minimum width of short-text large buttons through a minimum width ratio factor for a more compact layout

## Complete API Summary

All size settings are managed uniformly through `SARibbonBar`, no need to directly manipulate individual Panels:

| Method | Return Value | Description |
|--------|-------------|-------------|
| `setTitleBarHeight(h)` | void | Set the title bar height (visible only in loose mode) |
| `titleBarHeight()` | int | Get the title bar height |
| `setTabBarHeight(h)` | void | Set the tab bar height |
| `tabBarHeight()` | int | Get the tab bar height |
| `setCategoryHeight(h)` | void | Set the total height of the Category panel content area |
| `categoryHeight()` | int | Get the Category height |
| `setPanelTitleHeight(h)` | void | Set the height of the panel bottom title area |
| `panelTitleHeight()` | int | Get the panel title height |
| `setPanelSpacing(n)` | void | Set the horizontal spacing between adjacent panels |
| `panelSpacing()` | int | Get the panel spacing |
| `setPanelToolButtonIconSize(s, l)` | void | Set the small icon and large icon sizes for tool buttons |
| `panelToolButtonIconSize()` | QPair<QSize, QSize> | Get the tool button icon size pair |
| `setPanelLargeIconSize(size)` | void | Set the large icon size independently |
| `panelLargeIconSize()` | QSize | Get the large icon size |
| `setPanelSmallIconSize(size)` | void | Set the small icon size independently |
| `panelSmallIconSize()` | QSize | Get the small icon size |
| `setButtonMaximumAspectRatio(r)` | void | Set the maximum button aspect ratio (controls maximum width) |
| `buttonMaximumAspectRatio()` | qreal | Get the maximum button aspect ratio |
| `setLargeButtonMinimumWidthRatio(r)` | void | Set the minimum width ratio of large buttons (controls minimum width, compact layout) |
| `largeButtonMinimumWidthRatio()` | qreal | Get the minimum width ratio of large buttons |
| `normalModeMainBarHeight()` | int | Get the main bar height in normal mode |
| `minimumModeMainBarHeight()` | int | Get the main bar height in minimum mode |

## Size Relationship Explanation

The vertical height of the Ribbon is composed of multiple layers stacked together. Understanding these relationships helps in configuring sizes appropriately:

```
┌─────────────────────────────────────┐  ← Title bar height (setTitleBarHeight)
│  Title Bar (visible only in loose mode) │
├─────────────────────────────────────┤  ← Tab bar height (setTabBarHeight)
│  Tab Bar                            │
├─────────────────────────────────────┤  ← Category height (setCategoryHeight)
│  Category content area              │
│  ┌─────────┬─────────┬─────────┐    │     Contains: panel content + panel title
│  │ Panel 1 │ Panel 2 │ Panel 3 │    │     Panel title height controlled by setPanelTitleHeight
│  └─────────┴─────────┴─────────┘    │
└─────────────────────────────────────┘
```

**Total height formula** (loose mode):
> Ribbon total height = Title bar height + Tab bar height + Category height

**Total height formula** (compact mode):
> Ribbon total height = Tab bar height + Category height (compact mode has no title bar)

**Total height formula** (single-row mode):
> Ribbon total height = Tab bar height + Category height (in single-row mode, Category height contains only one row of buttons; panel titles are hidden)

The Category height internally includes the panel content area and the panel title area. The panel title height is controlled separately through `setPanelTitleHeight` and is displayed at the bottom of the panel by default.

## Usage Example

The following example demonstrates how to create a compact-style Ribbon interface and adjust various sizes:

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

        // Use compact three-row mode
        ribbon->setRibbonStyle(SARibbonBar::RibbonStyleCompactThreeRow);

        // Adjust heights of each part for a more compact interface
        ribbon->setTabBarHeight(28);
        ribbon->setCategoryHeight(80);
        ribbon->setPanelTitleHeight(16);

        // Set icon sizes (small icon, large icon)
        ribbon->setPanelToolButtonIconSize(QSize(16, 16), QSize(24, 24));

        // Set panel spacing for better visual separation
        ribbon->setPanelSpacing(4);

        // Add category and panel
        SARibbonCategory* cat = ribbon->addCategoryPage(tr("&File"));
        SARibbonPanel* panel = cat->addPanel(tr("Operations"));
        panel->addLargeAction(new QAction(tr("New"), this));
        panel->addSmallAction(new QAction(tr("Open"), this));
    }
};
```

Result: a compact yet fully functional interface with appropriate panel spacing and icon sizes suited for high-resolution displays.

## Large Button Minimum Width (Compact Layout)

The minimum width of large buttons (buttons created by `addLargeAction`) defaults to **button height × 0.75**.
When the button text is short (e.g., two characters), the text width is smaller than this minimum, so the button
is stretched to the minimum width and looks wider than necessary.

Use `setLargeButtonMinimumWidthRatio` to lower this ratio so short-text buttons shrink to a width close to the icon/text:

```cpp
SARibbonBar* ribbon = ribbonBar();

// Globally compact: minimum width = button height × 0.4
ribbon->setLargeButtonMinimumWidthRatio(0.4);

// Remove the height-based lower bound entirely; only the icon width (plus margins) is kept
ribbon->setLargeButtonMinimumWidthRatio(0.0);
```

**Parameter description**:

| Value | Behavior |
|-------|----------|
| `> 0` | Minimum width = button height × ratio (default 0.75) |
| `<= 0` | No height-based constraint; minimum width = large icon width + margins |

- Only affects large buttons; small/medium buttons are not affected
- The default value 0.75 is fully backward compatible with previous versions
- It can also be set per `SARibbonCategory` / `SARibbonPanel` (e.g., `panel->setLargeButtonMinimumWidthRatio(0.3)`)
  to affect only the buttons within that scope

!!! tip "Effect Reference"
    With the default font in three-row mode, a large button labeled with two characters is about 59px wide by default
    (ratio 0.75); setting the ratio to 0.3 or 0 shrinks it to about 34px, close to the icon width, making the layout
    noticeably more compact.

!!! example "Interactive Debugging"
    The "Large Button Min Width Ratio" spin box in the settings dock of `example/MainWindowExample` adjusts this ratio in real time.

!!! info "Dynamic Icon Calculation Logic"
    SARibbon's icon sizes are not set to fixed values like traditional toolbars, but are dynamically calculated based on the RibbonBar's `CategoryHeight` and text height.
    The layout automatically determines the icon size. To make icons larger: increase `CategoryHeight`, or call `setEnableWordWrap(false)` to disable text wrapping and free up more icon space.

!!! tip "4K/Multi-Monitor Adaptation Suggestions"
    In 4K or multi-monitor environments, it is recommended to call `SARibbonBar::initHighDpi()` to enable high DPI support, and use `setPanelToolButtonIconSize` to appropriately increase icon sizes.
    You can also combine this with `QApplication::setAttribute(Qt::AA_EnableHighDpiScaling)` to ensure the ribbon displays consistently across different DPI monitors.

!!! example "Interactive Debugging"
    Run the **Size** tab in the `example/MainWindowExample` example to dynamically adjust component sizes and see the effects in real time.
    ![example-size](../../assets/pic/example-size.png)
## Gallery Stretch Factor

When a panel contains multiple Galleries, the extra width is **shared equally** by default. To distribute it by weight, set a stretch factor on each Gallery:

```cpp
SARibbonGallery* g1 = panel->addGallery();
SARibbonGallery* g2 = panel->addGallery();
g1->setStretchFactor(2);  // final width ratio of g1 : g2 is approximately 2 : 1
g2->setStretchFactor(1);
```

Rules:

- The factor defaults to `0`, which keeps the legacy equal-share behavior — if no factor is ever set, the layout is identical to previous versions;
- As soon as one Gallery has a factor greater than 0, weighted mode kicks in: the final width ratio approximates the weight ratio, and Galleries with a factor of `0` keep their base width and opt out of the extra distribution;
- The weighted result is constrained by the column maximum width (`QWidget::setMaximumWidth`) and minimum widths, so the actual ratio deviates slightly;
- Changing the factor automatically re-lays out the containing panel; no manual call is needed.

!!! note
    Run the **Size** tab in `example/MainWindowExample` to verify interactively.
