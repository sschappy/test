#ifndef DockManagerH
#define DockManagerH
/*******************************************************************************
** Qt Advanced Docking System
** Copyright (C) 2017 Uwe Kindler
**
** This library is free software; you can redistribute it and/or
** modify it under the terms of the GNU Lesser General Public
** License as published by the Free Software Foundation; either
** version 2.1 of the License, or (at your option) any later version.
**
** This library is distributed in the hope that it will be useful,
** but WITHOUT ANY WARRANTY; without even the implied warranty of
** MERCHANTABILITY or FITNESS FOR A PARTICULAR PURPOSE.  See the GNU
** Lesser General Public License for more details.
**
** You should have received a copy of the GNU Lesser General Public
** License along with this library; If not, see <http://www.gnu.org/licenses/>.
******************************************************************************/


//============================================================================
/// \file   DockManager.h
/// \author Uwe Kindler
/// \date   26.02.2017
/// \brief  Declaration of CDockManager class
//============================================================================


//============================================================================
//                                   INCLUDES
//============================================================================
#include "ads_globals.h"
#include "DockContainerWidget.h"
#include "DockWidget.h"
#include "FloatingDockContainer.h"


QT_FORWARD_DECLARE_CLASS(QSettings)
QT_FORWARD_DECLARE_CLASS(QMenu)

namespace ads
{
struct DockManagerPrivate;
class CFloatingDockContainer;
struct FloatingDockContainerPrivate;
class CDockContainerWidget;
class DockContainerWidgetPrivate;
class CDockOverlay;
class CDockAreaTabBar;
class CDockWidgetTab;
struct DockWidgetTabPrivate;
struct DockAreaWidgetPrivate;
class CIconProvider;
class CDockComponentsFactory;
class CDockFocusController;
class CAutoHideSideBar;
class CAutoHideTab;
struct AutoHideTabPrivate;


/*
CDockManager
  ├─ CDockContainerWidget (主容器)
  │    ├─ CDockAreaWidget
  │    │    ├─ CDockAreaTitleBar
  │    │    │    └─ CDockAreaTabBar
  │    │    │         └─ CDockWidgetTab
  │    │    └─ CDockWidget
  │    │         └─ 你的内容控件(QWidget)
  │    └─ CDockSplitter
  │
  ├─ CFloatingDockContainer
  │    └─ CDockContainerWidget
  │         └─ CDockAreaWidget
  │              └─ CDockWidget
  │
  ├─ CAutoHideSideBar
  │    └─ CAutoHideTab
  │
  ├─ CAutoHideDockContainer
  │    └─ CDockWidget
  │
  ├─ CDockOverlay
  ├─ CDockFocusController
  └─ CDockComponentsFactory

*/



class ADS_EXPORT CDockManager : public CDockContainerWidget
{
	Q_OBJECT
private:
	DockManagerPrivate* d; ///< private data (pimpl)
	friend struct DockManagerPrivate;
	friend class CFloatingDockContainer;
	friend struct FloatingDockContainerPrivate;
	friend class CDockContainerWidget;
	friend class DockContainerWidgetPrivate;
	friend class CDockAreaTabBar;
	friend class CDockWidgetTab;
	friend struct DockAreaWidgetPrivate;
	friend struct DockWidgetTabPrivate;
	friend class CFloatingDragPreview;
	friend struct FloatingDragPreviewPrivate;
	friend class CDockAreaTitleBar;
	friend class CAutoHideDockContainer;
	friend CAutoHideSideBar;
	friend CAutoHideTab;
	friend AutoHideTabPrivate;

public:
    using Super = CDockContainerWidget;

    enum eViewMenuInsertionOrder
    {
        MenuSortedByInsertion,
        MenuAlphabeticallySorted
    };

    /**
     * These global configuration flags configure some global dock manager
     * settings.
     * Set the dock manager flags, before you create the dock manager instance.
     */
    enum eConfigFlag
    {
        ActiveTabHasCloseButton =
            0x0001,  //!< If this flag is set, the active tab in a tab area has a close button
        DockAreaHasCloseButton =
            0x0002,  //!< If the flag is set each dock area has a close button
        DockAreaCloseButtonClosesTab =
            0x0004,  //!< If the flag is set, the dock area close button closes the active tab, if not set, it closes the complete dock area
        OpaqueSplitterResize =
            0x0008,  //!< See QSplitter::setOpaqueResize() documentation
        XmlAutoFormattingEnabled =
            0x0010,  //!< If enabled, the XML writer automatically adds line-breaks and indentation to empty sections between elements (ignorable whitespace).
        XmlCompressionEnabled =
            0x0020,  //!< If enabled, the XML output will be compressed and is not human readable anymore
        TabCloseButtonIsToolButton =
            0x0040,  //! If enabled the tab close buttons will be QToolButtons instead of QPushButtons - disabled by default
        AllTabsHaveCloseButton =
            0x0080,  //!< if this flag is set, then all tabs that are closable show a close button
        RetainTabSizeWhenCloseButtonHidden =
            0x0100,  //!< if this flag is set, the space for the close button is reserved even if the close button is not visible
        DragPreviewIsDynamic = 0x0400,  ///< If opaque undocking is disabled, this
                                        ///< flag defines the behavior of the drag
                                        ///< preview window, if this flag is
                                        ///< enabled, the preview will be adjusted
                                        ///< dynamically to the drop area
        DragPreviewShowsContentPixmap = 0x0800,  ///< If opaque undocking is
                                                 ///< disabled, the created drag
                                                 ///< preview window shows a copy
                                                 ///< of the content of the dock
                                                 ///< widget / dock are that is
                                                 ///< dragged
        DragPreviewHasWindowFrame = 0x1000,  ///< If opaque undocking is disabled,
                                             ///< then this flag configures if the
                                             ///< drag preview is frameless or
                                             ///< looks like a real window
        AlwaysShowTabs = 0x2000,  ///< If this option is enabled, the tab of a
                                  ///< dock widget is always displayed - even if
                                  ///< it is the only visible dock widget in a
                                  ///< floating widget.
        DockAreaHasUndockButton =
            0x4000,  //!< If the flag is set each dock area has an undock button
        DockAreaHasTabsMenuButton =
            0x8000,  //!< If the flag is set each dock area has a tabs menu button
        DockAreaHideDisabledButtons =
            0x10000,  //!< If the flag is set disabled dock area buttons will not appear on the toolbar at all (enabling them will bring them back)
        DockAreaDynamicTabsMenuButtonVisibility =
            0x20000,  //!< If the flag is set, the tabs menu button will be shown only when it is required - that means, if the tabs are elided. If the tabs are not elided, it is hidden
        FloatingContainerHasWidgetTitle =
            0x40000,  //!< If set, the Floating Widget window title reflects the title of the current dock widget otherwise it displays the title set with `CDockManager::setFloatingContainersTitle` or application name as window title
        FloatingContainerHasWidgetIcon =
            0x80000,  //!< If set, the Floating Widget icon reflects the icon of the current dock widget otherwise it displays application icon
        HideSingleCentralWidgetTitleBar =
            0x100000,  //!< If there is only one single visible dock widget in the main dock container (the dock manager) and if this flag is set, then the titlebar of this dock widget will be hidden
        //!< this only makes sense for non draggable and non floatable widgets and enables the creation of some kind of "central" widget

        FocusHighlighting =
            0x200000,  //!< enables styling of focused dock widget tabs or floating widget titlebar
        EqualSplitOnInsertion =
            0x400000,  ///!< if enabled, the space is equally distributed to all widgets in a  splitter

        FloatingContainerForceNativeTitleBar =
            0x800000,  //!< Linux only ! Forces all FloatingContainer to use the native title bar. This might break docking for FloatinContainer on some Window Managers (like Kwin/KDE).
        //!< If neither this nor FloatingContainerForceCustomTitleBar is set (the default) native titlebars are used except on known bad systems.
        //! Users can overwrite this by setting the environment variable ADS_UseNativeTitle to "1" or "0".
        FloatingContainerForceQWidgetTitleBar =
            0x1000000,  //!< Linux only ! Forces all FloatingContainer to use a QWidget based title bar.
        //!< If neither this nor FloatingContainerForceNativeTitleBar is set (the default) native titlebars are used except on known bad systems.
        //! Users can overwrite this by setting the environment variable ADS_UseNativeTitle to "1" or "0".
        MiddleMouseButtonClosesTab =
            0x2000000,  //! If the flag is set, the user can use the mouse middle button to close the tab under the mouse
        DisableTabTextEliding =
            0x4000000,  //! Set this flag to disable eliding of tab texts in dock area tabs
        ShowTabTextOnlyForActiveTab =
            0x8000000,  //! Set this flag to show label texts in dock area tabs only for active tabs
        DoubleClickUndocksWidget =
            0x10000000,  //!< If the flag is set, a double click on a tab undocks the widget
        TabsAtBottom =
            0x20000000,  //!< If the flag is set, tabs will be shown at the bottom instead of in the title bar.
        UseNativeWindows =
            0x40000000,  //!< If the flag is set, windows for the dock and area widgets will be native.
        DisableStylesheet =
            0x80000000,  //!< If the flag is set, the dock manager will not apply the default stylesheet

        DefaultDockAreaButtons = DockAreaHasCloseButton | DockAreaHasUndockButton
                                 | DockAreaHasTabsMenuButton,  ///< default
                                                               ///< configuration
                                                               ///< of dock area
                                                               ///< title bar
                                                               ///< buttons

        DefaultBaseConfig = DefaultDockAreaButtons | ActiveTabHasCloseButton
                            | XmlCompressionEnabled
                            | FloatingContainerHasWidgetTitle
                            | DoubleClickUndocksWidget,  ///< default base
                                                         ///< configuration
                                                         ///< settings

        DefaultOpaqueConfig = DefaultBaseConfig | OpaqueSplitterResize
                              | DragPreviewShowsContentPixmap,  ///< the default
                                                                ///< configuration
                                                                ///< for non
                                                                ///< opaque
                                                                ///< operations

        DefaultNonOpaqueConfig = DefaultBaseConfig
                                 | DragPreviewShowsContentPixmap,  ///< the
                                                                   ///< default
                                                                   ///< configuration
                                                                   ///< for non
                                                                   ///< opaque
                                                                   ///< operations

        NonOpaqueWithWindowFrame = DefaultNonOpaqueConfig
                                   | DragPreviewHasWindowFrame  ///< the default
                                                                ///< configuration
                                                                ///< for non
                                                                ///< opaque
                                                                ///< operations
                                                                ///< that show a
                                                                ///< real window
                                                                ///< with frame
    };
    Q_DECLARE_FLAGS(ConfigFlags, eConfigFlag)

    /**
     * These global configuration flags configure some dock manager auto hide
     * settings
     * Set the dock manager flags, before you create the dock manager instance.
     */
    enum eAutoHideFlag
    {
        AutoHideFeatureEnabled = 0x01,  //!< enables / disables auto hide feature
        DockAreaHasAutoHideButton =
            0x02,  //!< If the flag is set each dock area has a auto hide menu button
        AutoHideButtonTogglesArea =
            0x04,  //!< If the flag is set, the auto hide button enables auto hiding for all dock widgets in an area, if disabled, only the current dock widget will be toggled
        AutoHideButtonCheckable =
            0x08,  //!< If the flag is set, the auto hide button will be checked and unchecked depending on the auto hide state. Mainly for styling purposes.
        AutoHideSideBarsIconOnly = 0x10,  ///< show only icons in auto hide side
                                          ///< tab - if a tab has no icon, then
                                          ///< the text will be shown
        AutoHideShowOnMouseOver = 0x20,   ///< show the auto hide window on mouse
                                         ///< over tab and hide it if mouse leaves
                                         ///< auto hide container
        AutoHideCloseButtonCollapsesDock = 0x40,  ///< Close button of an auto
                                                  ///< hide container collapses
                                                  ///< the dock instead of hiding
                                                  ///< it completely
        AutoHideHasCloseButton = 0x80,  //< If the flag is set an auto hide title
                                        //bar has a close button
        AutoHideHasMinimizeButton = 0x100,  ///< if this flag is set, the auto
                                            ///< hide title bar has a minimize
                                            ///< button to collapse the dock
                                            ///< widget
        AutoHideOpenOnDragHover = 0x200,  ///< if this flag is set, dragging hover
                                          ///< the tab bar will open the dock
        AutoHideCloseOnOutsideMouseClick = 0x400,  ///< if this flag is set, the
                                                   ///< auto hide dock container
                                                   ///< will collapse if the user
                                                   ///< clicks outside of the
                                                   ///< container, if not set, the
                                                   ///< auto hide container can be
                                                   ///< closed only via click on
                                                   ///< sidebar tab

        DefaultAutoHideConfig = AutoHideFeatureEnabled | DockAreaHasAutoHideButton
                                | AutoHideHasMinimizeButton
                                | AutoHideCloseOnOutsideMouseClick

    };
    Q_DECLARE_FLAGS(AutoHideFlags, eAutoHideFlag)

    /**
     * Global configuration parameters that you can set via setConfigParam()
     */
    enum eConfigParam
    {
        AutoHideOpenOnDragHoverDelay_ms,  ///< Delay in ms before the dock opens
                                          ///< on drag hover if
                                          ///< AutoHideOpenOnDragHover flag is set
        ConfigParamCount  // just a delimiter to count number of config params
    };


	/************************************************************************************************************************/

public:
    CDockManager(QWidget* parent = nullptr);
    virtual ~CDockManager() override;

    //添加操作
    CDockAreaWidget* addDockWidget(DockWidgetArea area, CDockWidget* Dockwidget, CDockAreaWidget* DockAreaWidget = nullptr, int Index = -1);
    CFloatingDockContainer* addDockWidgetFloating(CDockWidget* Dockwidget);
    CDockAreaWidget* addDockWidgetTabToArea(CDockWidget* Dockwidget,CDockAreaWidget* DockAreaWidget, int Index = -1);
    CAutoHideDockContainer* addAutoHideDockWidget(SideBarLocation Location, CDockWidget* Dockwidget);
    CAutoHideDockContainer* addAutoHideDockWidgetToContainer(SideBarLocation Location, CDockWidget* Dockwidget, CDockContainerWidget* DockContainerWidget);
    CDockAreaWidget* addDockWidgetTab(DockWidgetArea area, CDockWidget* Dockwidget);
    CDockAreaWidget* addDockWidgetToContainer(DockWidgetArea area, CDockWidget* Dockwidget, CDockContainerWidget* DockContainerWidget);

    //状态保存/恢复
    QByteArray saveState(int version = 0) const;
    bool restoreState(const QByteArray& state, int version = 0);
    bool isRestoringState() const;

 	//Perspective 机制
    void addPerspective(const QString& UniquePrespectiveName);
public Q_SLOTS:
    void openPerspective(const QString& PerspectiveName);
public:
    void removePerspective(const QString& Name);
    void removePerspectives(const QStringList& Names);
    QStringList perspectiveNames() const;
    void savePerspectives(QSettings& Settings) const;
    void loadPerspectives(QSettings& Settings);

    //dock 管理查询接口
    CDockWidget* findDockWidget(const QString& ObjectName) const;
    void removeDockWidget(CDockWidget* Dockwidget);
    QMap<QString, CDockWidget*> dockWidgetsMap() const;
    const QList<CDockContainerWidget*> dockContainers() const;
    const QList<CFloatingDockContainer*> floatingWidgets() const;

    //central widget 相关
    CDockWidget* centralWidget() const;
    CDockAreaWidget* setCentralWidget(CDockWidget* widget);

    //菜单与视图动作
    QAction* addToggleViewActionToMenu(QAction* ToggleViewAction,const QString& Group = QString(), const QIcon& GroupIcon = QIcon());
    QMenu* viewMenu() const;
    void setViewMenuInsertionOrder(eViewMenuInsertionOrder Order);

    //事件与窗口状态
    bool eventFilter(QObject* obj, QEvent* e) override;
    bool isLeavingMinimizedState() const;
public Q_SLOTS:
    void endLeavingMinimizedState();
protected:
    virtual void showEvent(QShowEvent* event) override;
    void restoreHiddenFloatingWidgets();

    //floating/container 注册机制
    void registerFloatingWidget(CFloatingDockContainer* FloatingWidget);
    void removeFloatingWidget(CFloatingDockContainer* FloatingWidget);
    void registerDockContainer(CDockContainerWidget* DockContainer);
    void removeDockContainer(CDockContainerWidget* DockContainer);
public:
    unsigned int zOrderIndex() const override;

    //overlay/focus/relocation机制：拖拽时 overlay 高亮，拖拽完成后的通知，焦点高亮机制
protected:
    CDockOverlay* containerOverlay() const;
    CDockOverlay* dockAreaOverlay() const;
    void notifyWidgetOrAreaRelocation(QWidget* RelocatedWidget);
    void notifyFloatingWidgetDrop(CFloatingDockContainer* FloatingWidget);
    CDockFocusController* dockFocusController() const;
public Q_SLOTS:
    void setDockWidgetFocused(CDockWidget* DockWidget);
public:
    CDockWidget* focusedDockWidget() const;
    template<class QWidgetPtr>
    static void setWidgetFocus(QWidgetPtr widget)
    {
        if (!CDockManager::testConfigFlag(CDockManager::FocusHighlighting))
        {
            return;
        }

        widget->setFocus(Qt::OtherFocusReason);
    }

    //splitter/toolbar/全局锁
    QList<int> splitterSizes(CDockAreaWidget* ContainedArea) const;
    void setSplitterSizes(CDockAreaWidget* ContainedArea, const QList<int>& sizes);
    void setDockWidgetToolBarStyle(Qt::ToolButtonStyle Style, CDockWidget::eState State);
    Qt::ToolButtonStyle dockWidgetToolBarStyle(CDockWidget::eState State) const;
    void setDockWidgetToolBarIconSize(const QSize& IconSize, CDockWidget::eState State);
    QSize dockWidgetToolBarIconSize(CDockWidget::eState State) const;
    void lockDockWidgetFeaturesGlobally(CDockWidget::DockWidgetFeatures Features =  CDockWidget::GloballyLockableFeatures);
    CDockWidget::DockWidgetFeatures globallyLockedDockWidgetFeatures() const;

    //静态全局配置
    static ConfigFlags configFlags();
    static AutoHideFlags autoHideConfigFlags();
    static void setConfigFlags(const ConfigFlags Flags);
    static void setAutoHideConfigFlags(const AutoHideFlags Flags);
    static void setConfigFlag(eConfigFlag Flag, bool On = true);
    static void setAutoHideConfigFlag(eAutoHideFlag Flag, bool On = true);
    static bool testConfigFlag(eConfigFlag Flag);
    static bool testAutoHideConfigFlag(eAutoHideFlag Flag);
    static void setConfigParam(eConfigParam Param, QVariant Value);
    static QVariant configParam(eConfigParam Param, QVariant Default);
    static int startDragDistance();
    static CIconProvider& iconProvider();
    static void setFloatingContainersTitle(const QString& Title);
    static QString floatingContainersTitle();

    //窗口显隐控制
public Q_SLOTS:
    void hideManagerAndFloatingWidgets();
    void raise();

    //组件工厂机制
public:
    CDockWidget* createDockWidget(const QString& title, QWidget* parent = nullptr);
    QSharedPointer<ads::CDockComponentsFactory> componentsFactory() const;
    void setComponentsFactory(ads::CDockComponentsFactory* Factory);
    void setComponentsFactory(QSharedPointer<ads::CDockComponentsFactory>);

Q_SIGNALS:
    void dockWidgetAdded(ads::CDockWidget* DockWidget);

    void restoringState();
    void stateRestored();

    void perspectiveListChanged();
    void openingPerspective(const QString& PerspectiveName);
    void perspectiveOpened(const QString& PerspectiveName);
    void perspectivesRemoved();
    void perspectiveListLoaded();

    void dockWidgetAboutToBeRemoved(ads::CDockWidget* DockWidget);
    void dockWidgetRemoved(ads::CDockWidget* DockWidget);

    void floatingWidgetCreated(ads::CFloatingDockContainer* FloatingWidget);

    //外部触发信号
    void dockAreaCreated(ads::CDockAreaWidget* DockArea);
    void focusedDockWidgetChanged(ads::CDockWidget* old, ads::CDockWidget* now);



}; // class DockManager
} // namespace ads

Q_DECLARE_OPERATORS_FOR_FLAGS(ads::CDockManager::ConfigFlags)
//-----------------------------------------------------------------------------
#endif // DockManagerH
