# SARibbon User Guide

- ✅ **Quick start**: Static embedding needs only 2 files, 5 lines of code to create a Ribbon interface
- ✅ **MFC-style naming**: Category/Panel/Action naming follows MFC Ribbon conventions
- ✅ **Contextual tabs**: SARibbonContextCategory condition-based show/hide for specific feature groups
- ✅ **Gallery widget**: Grid-style display for large icon option sets
- ✅ **Customization persistence**: User-customizable UI with XML save/load configuration
- ✅ **12-step documentation**: From import to advanced, covering all core features

---

SARibbon is a Qt library for creating modern Ribbon interfaces, with a style similar to Microsoft Office or WPS. It is designed for complex desktop applications, effectively organizing a large number of functions, and is commonly used in the interface development of industrial software.

Before starting coding, you need to integrate the SARibbon library into your Qt project. The simplest way is **static embedding**, which is to directly copy the source files `SARibbon.h` and `SARibbon.cpp` into your project.

## Quick Start

```cpp
#include "SARibbon.h"
#include <QApplication>

int main(int argc, char* argv[])
{
#if (QT_VERSION >= QT_VERSION_CHECK(5, 6, 0))
    QApplication::setAttribute(Qt::AA_EnableHighDpiScaling);
    QApplication::setAttribute(Qt::AA_UseHighDpiPixmaps);
#endif
    SARibbonBar::initHighDpi();
    QApplication a(argc, argv);
    
    SARibbonMainWindow w;
    w.setWindowTitle("SARibbon Quick Start");
    
    SARibbonBar* ribbon = w.ribbonBar();
    SARibbonCategory* cat = ribbon->addCategoryPage("Home");
    SARibbonPanel* panel = cat->addPanel("Actions");
    panel->addLargeAction(new QAction(QIcon(":/icon.svg"), "Click Me", &w));
    
    w.show();
    return a.exec();
}
```

## Documentation Reading Guide

| Topic | Document | Description |
|-------|----------|-------------|
| Building | [Build Instructions](../build-guide/build-instructions.md) | How to build SARibbon (CMake/QMake) |
| Integration | [Importing the Library](./import-SARibbon.md) | Static/dynamic integration into your project |
| Window Setup | [Create a Ribbon-Style Window](./create-ribbon-style-window.md) | SARibbonMainWindow / SARibbonWidget usage |
| UI Creation | [Creating the Ribbon UI](./create-ribbon-ui.md) | Category, Panel, Action, Gallery, etc. |
| Layout | [Ribbon Layout Options](./layout-of-SARibbon.md) | Loose/Compact, 2-row/3-row/SingleRow modes |
| Theming | [Ribbon Themes](./SARibbon-theme.md) | Built-in themes and custom QSS styling |
| Customization | [User-Configurable Ribbon](./persistence-configuration-ribbon.md) | Runtime customization and XML persistence |

## Differences between Ribbon interface and traditional menubar+toolbar

The traditional menubar+toolbar cannot be directly converted into a ribbon interface. Ribbon is not just a toolbar with `QToolBar`. Compared with the traditional menu bar and toolbar, it has the following characteristics:

- The button rendering method of Ribbon has an obvious change, making it impossible to directly use ToolButton for simulation. SARibbon uses `SARibbonToolButton` to re-layout and render the buttons for Ribbon.
- Ribbon also has a special type of tab called `Context Category`. For example, when you select a picture in Office Word, a "Picture Editing" tab will automatically appear, providing picture-specific functions such as cropping and rotating. This tab will automatically hide when the selection is canceled.
- The Ribbon interface comes with some special controls, such as Gallery (the style selection in Word is a Gallery control).

## Terminology

| Term | SARibbon Class | Description |
|------|---------------|-------------|
| Ribbon Bar | `SARibbonBar` | The main Ribbon control at the top of the window |
| Category | `SARibbonCategory` | A tab page, equivalent to a functional group |
| Panel | `SARibbonPanel` | A group of related actions within a Category |
| Tool Button | `SARibbonToolButton` | Ribbon-specific button with custom painting |
| Context Category | `SARibbonContextCategory` | Conditional tab that appears based on context |
| Gallery | `SARibbonGallery` | Grid-style visual selector (e.g., styles in Word) |
| Quick Access Bar | `SARibbonQuickAccessBar` | Toolbar at the very top for frequently used actions |
| Application Button | `SARibbonApplicationButton` | The "File" button at the top-left corner |

## Automation Testing Integration (Button Identification Convention)

When using automation testing tools such as Squish, TestComplete, or uiautomator, it is recommended to locate Ribbon buttons by **objectName**.

### Convention

- The `objectName` of a panel button (`SARibbonToolButton`) is automatically inherited from the `QAction` it carries:
    1. If the `QAction` has an `objectName`, the button uses it (**values explicitly set by the user take priority and are never overwritten**);
    2. Otherwise, if the action has text, the button falls back to `QAction::text()` (note: multiple actions with the same text produce duplicate names; setting objectName explicitly is recommended for automation);
- The button's `accessibleName` (used by screen readers / assistive technology) is also auto-filled from the action text;
- The action key assigned by `SARibbonActionsManager` can also be used for identification (see [Interface Customization and Persistence](persistence-configuration-ribbon.md)).

### Example

```cpp
QAction* saveAction = new QAction(QIcon(":/save.png"), tr("Save"), this);
saveAction->setObjectName("actionSave");  // automation tools locate the button by this name
panel->addLargeAction(saveAction);
```

Lookup example on the Squish side:

```python
# Find by name (recommended)
saveButton = waitForObject({"objectName": "actionSave", "type": "SARibbonToolButton"})
# Hierarchy path + name (to handle duplicate names from text fallback)
btn = waitForObject({"container": ribbonPanel, "objectName": "Save"})
```

### Recommended Rules

- Use only letters, digits, and underscores in objectName; avoid spaces and non-ASCII characters (some tools have escaping difficulties);
- Name the actions that need automation coverage centrally, in `main()` or in the window constructor; do not rely on the text fallback.

## Placing Custom Widgets in Galleries and Panels

Gallery items are modeled on `QAction` (icon + text), which is **not** suitable for hosting interactive widgets such as `QCheckBox` directly. There are two recommended ways to place custom widgets:

### Way 1: `SARibbonPanel::addWidget` — show the widget alongside the Gallery

The widget is carried by a `QWidgetAction` and participates in the panel layout as a small item:

```cpp
SARibbonPanel* panel = category->addPanel(tr("gallery widgets"));
SARibbonGallery* gallery = panel->addGallery();

QCheckBox* checkBox = new QCheckBox(tr("enable preview"), panel);
panel->addSmallWidget(checkBox);           // added as a small item

QComboBox* combo = new QComboBox(panel);
combo->addItems({ tr("option 1"), tr("option 2") });
panel->addSmallWidget(combo);
```

### Way 2: put widgets into the Gallery popup viewport

The popup window (opened by the "more" button at the bottom-right of the Gallery) is managed by `SARibbonGalleryViewport`, and arbitrary widgets can be added to it (grouped by title):

```cpp
QWidget* custom = new QWidget(gallery->getPopupViewPort());
QVBoxLayout* lay = new QVBoxLayout(custom);
lay->addWidget(new QCheckBox(tr("checkbox in popup"), custom));
lay->addWidget(new QComboBox(custom));
lay->addStretch();
gallery->getPopupViewPort()->addWidget(custom, tr("custom widgets"));  // second argument is the group title
```

### Key Constraints

- **Size**: widgets inside the panel are constrained by the row height (about one button height in single-row mode); taller widgets get compressed; `sizeHint` determines the reserved width;
- **Ownership and release**: widgets in Way 1 are carried by a `QWidgetAction` and are **not** deleted on removal (the parent is explicitly set to the panel) — manage their lifetime yourself; widgets in Way 2 are managed by the viewport's content layout;
- **`Qt::WA_LayoutUsesWidgetRect`**: already set by the framework when creating panel items; no manual handling is needed;
- See the **gallery widgets** panel in the **other** tab of `example/MainWindowExample` for a complete runnable example (demonstrating both ways).

## Reordering Actions in Button Groups and the Quick Access Bar

`SARibbonButtonGroupWidget` and `SARibbonQuickAccessBar` both inherit `QToolBar`; use the native Qt interfaces for insertion and reordering (no extra API needed):

```cpp
SARibbonQuickAccessBar* quickBar = ribbonBar()->quickAccessBar();
// append
quickBar->addAction(action);
// insert before beforeAction (prepend or middle insert)
quickBar->insertAction(beforeAction, action);
// remove
quickBar->removeAction(action);
// move = remove + re-insert (custom widgets carried by QWidgetAction keep their state)
quickBar->removeAction(action);
quickBar->insertAction(targetAction, action);
```

The ordering semantics of `insertAction(before, ...)` and the state preservation of custom-widget actions (`QWidgetAction`) across moves are locked by the regression test `tests/SARibbonButtonGroupWidgetTest.cpp`.
