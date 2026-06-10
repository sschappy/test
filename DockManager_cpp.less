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
/// \file   DockManager.cpp
/// \author Uwe Kindler
/// \date   26.02.2017
/// \brief  Implementation of CDockManager class
//============================================================================


//============================================================================
//                                   INCLUDES
//============================================================================
#include <AutoHideDockContainer.h>
#include "DockWidgetTab.h"
#include "DockManager.h"

#include <algorithm>
#include <iostream>

#include <QMainWindow>
#include <QList>
#include <QMap>
#include <QVariant>
#include <QDebug>
#include <QFile>
#include <QDialog>
#include <QAction>
#include <QXmlStreamWriter>
#include <QSettings>
#include <QMenu>
#include <QApplication>
#include <QWindow>
#include <QWindowStateChangeEvent>
#include <QVector>

#include "FloatingDockContainer.h"
#include "DockOverlay.h"
#include "DockWidget.h"
#include "ads_globals.h"
#include "DockAreaWidget.h"
#include "IconProvider.h"
#include "DockingStateReader.h"
#include "DockAreaTitleBar.h"
#include "DockFocusController.h"
#include "DockSplitter.h"
#include "DockComponentsFactory.h"


#if defined(Q_OS_UNIX) && !defined(Q_OS_MACOS)
#include "linux/FloatingWidgetTitleBar.h"
#endif


/**
 * Initializes the resources specified by the .qrc file with the specified base
 * name. Normally, when resources are built as part of the application, the
 * resources are loaded automatically at startup. The Q_INIT_RESOURCE() macro
 * is necessary on some platforms for resources stored in a static library.
 * Because GCC causes a linker error if we put Q_INIT_RESOURCE into the
 * loadStyleSheet() function, we place it into a function outside of the ads
 * namespace
 */
static void initResource()
{
	Q_INIT_RESOURCE(ads);
}


namespace ads
{
/**
 * Internal file version in case the structure changes internally
 */
enum eStateFileVersion
{
	InitialVersion = 0,      //!< InitialVersion
	Version1 = 1,            //!< Version1
	CurrentVersion = Version1//!< CurrentVersion
};

static CDockManager::ConfigFlags StaticConfigFlags = CDockManager::DefaultNonOpaqueConfig;
static CDockManager::AutoHideFlags StaticAutoHideConfigFlags; // auto hide feature is disabled by default
static QVector<QVariant> StaticConfigParams(CDockManager::ConfigParamCount);

static QString FloatingContainersTitle;


/**********************************************************************************************************************************/

struct DockManagerPrivate
{
	//构造函数内创建
	CDockManager* _this;
    QMenu* ViewMenu;
    CDockOverlay* ContainerOverlay;
    CDockOverlay* DockAreaOverlay;
    QList<CDockContainerWidget*> Containers;
    QList<QPointer<CFloatingDockContainer>> FloatingWidgets;
    CDockFocusController* FocusController = nullptr;

	QMap<QString, CDockWidget*> DockWidgetsMap;
    QVector<CFloatingDockContainer*> UninitializedFloatingWidgets;

    CDockWidget* CentralWidget = nullptr;
	bool RestoringState = false;

	QMap<QString, QByteArray> Perspectives;

	QMap<QString, QMenu*> ViewMenuGroups;
    CDockManager::eViewMenuInsertionOrder MenuInsertionOrder = CDockManager::MenuAlphabeticallySorted;

    bool IsLeavingMinimized = false;
    QList<QPointer<CFloatingDockContainer>> HiddenFloatingWidgets;

    Qt::ToolButtonStyle ToolBarStyleDocked = Qt::ToolButtonIconOnly;
    Qt::ToolButtonStyle ToolBarStyleFloating = Qt::ToolButtonTextUnderIcon;
    QSize ToolBarIconSizeDocked = QSize(16, 16);
    QSize ToolBarIconSizeFloating = QSize(24, 24);
    CDockWidget::DockWidgetFeatures LockedDockWidgetFeatures;

	QSharedPointer<ads::CDockComponentsFactory> ComponentFactory{ ads::CDockComponentsFactory::factory()};


    DockManagerPrivate(CDockManager* _public);
    void loadStylesheet();
    bool restoreState(const QByteArray& state, int version);
    bool checkFormat(const QByteArray& state, int version);
    bool restoreStateFromXml(const QByteArray& state, int version, bool Testing = internal::Restore);
    bool restoreContainer(int Index, CDockingStateReader& stream, bool Testing);
    void hideFloatingWidgets()
    {
        // Hide updates of floating widgets from user
        for (auto FloatingWidget : FloatingWidgets)
        {
            if (FloatingWidget)
            {
                FloatingWidget->hide();
            }
        }
    }
    void markDockWidgetsDirty()
    {
        for (auto DockWidget : DockWidgetsMap)
        {
            DockWidget->setProperty(internal::DirtyProperty, true);
        }
    }
    void restoreDockWidgetsOpenState();
    void restoreDockAreasIndices();
    void emitTopLevelEvents();
    void addActionToMenu(QAction* Action, QMenu* Menu, bool InsertSorted);


};

//私有类函数
DockManagerPrivate::DockManagerPrivate(CDockManager* _public) :
	_this(_public)
{
}
void DockManagerPrivate::loadStylesheet()
{
    if (CDockManager::testConfigFlag(CDockManager::DisableStylesheet))
    {
        return;
    }
    initResource();
    QString Result;
    QString FileName = ":ads/stylesheets/";
    FileName += CDockManager::testConfigFlag(CDockManager::FocusHighlighting) ?
                    "focus_highlighting" :
                    "default";

    FileName += ".css";
    QFile StyleSheetFile(FileName);
    StyleSheetFile.open(QIODevice::ReadOnly);
    QTextStream StyleSheetStream(&StyleSheetFile);
    Result = StyleSheetStream.readAll();
    StyleSheetFile.close();
    _this->setStyleSheet(Result);
}
bool DockManagerPrivate::restoreState(const QByteArray& State, int version)
{
    QByteArray state = State.startsWith("<?xml") ? State : qUncompress(State);
    if (!checkFormat(state, version))
    {
        ADS_PRINT("checkFormat: Error checking format!!!!!!!");
        return false;
    }

    // Hide updates of floating widgets from use
    hideFloatingWidgets();
    markDockWidgetsDirty();

    if (!restoreStateFromXml(state, version))
    {
        ADS_PRINT("restoreState: Error restoring state!!!!!!!");
        return false;
    }

    restoreDockWidgetsOpenState();
    restoreDockAreasIndices();
    emitTopLevelEvents();
    _this->dumpLayout();

    return true;
}
bool DockManagerPrivate::checkFormat(const QByteArray& state, int version)
{
    return restoreStateFromXml(state, version, internal::RestoreTesting);
}
bool DockManagerPrivate::restoreStateFromXml(const QByteArray& state, int version, bool Testing)
{
    Q_UNUSED(version);

    if (state.isEmpty())
    {
        return false;
    }
    CDockingStateReader s(state);
    s.readNextStartElement();
    if (s.name() != QLatin1String("QtAdvancedDockingSystem"))
    {
        return false;
    }
    ADS_PRINT(s.attributes().value("Version"));
    bool ok;
    int v = s.attributes().value("Version").toInt(&ok);
    if (!ok || v > CurrentVersion)
    {
        return false;
    }
    s.setFileVersion(v);

    ADS_PRINT(s.attributes().value("UserVersion"));
    // Older files do not support UserVersion but we still want to load them so
    // we first test if the attribute exists
    if (!s.attributes().value("UserVersion").isEmpty())
    {
        v = s.attributes().value("UserVersion").toInt(&ok);
        if (!ok || v != version)
        {
            return false;
        }
    }

    bool Result = true;
#ifdef ADS_DEBUG_PRINT
    int DockContainers = s.attributes().value("Containers").toInt();
#endif
    ADS_PRINT(DockContainers);

    if (CentralWidget)
    {
        const auto CentralWidgetAttribute = s.attributes().value("CentralWidget");
        // If we have a central widget but a state without central widget, then
        // something is wrong.
        if (CentralWidgetAttribute.isEmpty())
        {
            qWarning() << "Dock manager has central widget but saved state does "
                          "not have central widget.";
            return false;
        }

        // If the object name of the central widget does not match the name of the
        // saved central widget, the something is wrong
        if (CentralWidget->objectName() != CentralWidgetAttribute.toString())
        {
            qWarning() << "Object name of central widget does not match name of "
                          "central widget in saved state.";
            return false;
        }
    }

    int DockContainerCount = 0;
    while (s.readNextStartElement())
    {
        if (s.name() == QLatin1String("Container"))
        {
            Result = restoreContainer(DockContainerCount, s, Testing);
            if (!Result)
            {
                break;
            }
            DockContainerCount++;
        }
    }

    if (!Testing)
    {
        // Delete remaining empty floating widgets
        int FloatingWidgetIndex = DockContainerCount - 1;
        for (int i = FloatingWidgetIndex; i < FloatingWidgets.count(); ++i)
        {
            CFloatingDockContainer* floatingWidget = FloatingWidgets[i];
            if (!floatingWidget)
                continue;
            // Use removeFromDockManager() (introduced in upstream commit 544c624)
            // instead of removeDockContainer() so the container's back-pointer to
            // the manager is cleared. Otherwise ~CDockContainerWidget() (called
            // when deleteLater() fires) would try to remove this container a
            // second time, tripping the Q_ASSERT(removed == 1) check in
            // CDockManager::removeDockContainer().
            floatingWidget->dockContainer()->removeFromDockManager();
            floatingWidget->deleteLater();
        }
    }

    return Result;
}
bool DockManagerPrivate::restoreContainer(int Index, CDockingStateReader& stream,bool Testing)
{
    if (Testing)
    {
        Index = 0;
    }

    bool Result = false;
    if (Index >= Containers.count())
    {
        CFloatingDockContainer* FloatingWidget =
            new CFloatingDockContainer(_this);
        Result = FloatingWidget->restoreState(stream, Testing);
    }
    else
    {
        ADS_PRINT("d->Containers[i]->restoreState ");
        auto Container = Containers[Index];
        if (Container->isFloating())
        {
            Result = Container->floatingWidget()->restoreState(stream, Testing);
        }
        else
        {
            Result = Container->restoreState(stream, Testing);
        }
    }

    return Result;
}
void DockManagerPrivate::restoreDockWidgetsOpenState()
{
    // All dock widgets, that have not been processed in the restore state
    // function are invisible to the user now and have no assigned dock area
    // They do not belong to any dock container, until the user toggles the
    // toggle view action the next time
    for (auto DockWidget : DockWidgetsMap)
    {
        if (DockWidget->property(internal::DirtyProperty).toBool())
        {
            // If the DockWidget is an auto hide widget that is not assigned yet,
            // then we need to delete the auto hide container now
            if (DockWidget->isAutoHide())
            {
                DockWidget->autoHideDockContainer()->cleanupAndDelete();
            }
            DockWidget->flagAsUnassigned();
            Q_EMIT DockWidget->viewToggled(false);
        }
        else
        {
            DockWidget->toggleViewInternal(
                !DockWidget->property(internal::ClosedProperty).toBool());
        }
    }
}
void DockManagerPrivate::restoreDockAreasIndices()
{
    // Now all dock areas are properly restored and we setup the index of
    // The dock areas because the previous toggleView() action has changed
    // the dock area index
    int Count = 0;
    for (auto DockContainer : Containers)
    {
        Count++;
        for (int i = 0; i < DockContainer->dockAreaCount(); ++i)
        {
            CDockAreaWidget* DockArea = DockContainer->dockArea(i);
            QString DockWidgetName =
                DockArea->property("currentDockWidget").toString();
            CDockWidget* DockWidget = nullptr;
            if (!DockWidgetName.isEmpty())
            {
                DockWidget = _this->findDockWidget(DockWidgetName);
            }

            if (!DockWidget || DockWidget->isClosed())
            {
                int Index = DockArea->indexOfFirstOpenDockWidget();
                if (Index < 0)
                {
                    continue;
                }
                DockArea->setCurrentIndex(Index);
            }
            else
            {
                DockArea->internalSetCurrentDockWidget(DockWidget);
            }
        }
    }
}
void DockManagerPrivate::emitTopLevelEvents()
{
    // Finally we need to send the topLevelChanged() signals for all dock
    // widgets if top level changed
    for (auto DockContainer : Containers)
    {
        CDockWidget* TopLevelDockWidget = DockContainer->topLevelDockWidget();
        if (TopLevelDockWidget)
        {
            TopLevelDockWidget->emitTopLevelChanged(true);
        }
        else
        {
            for (int i = 0; i < DockContainer->dockAreaCount(); ++i)
            {
                auto DockArea = DockContainer->dockArea(i);
                for (auto DockWidget : DockArea->dockWidgets())
                {
                    DockWidget->emitTopLevelChanged(false);
                }
            }
        }
    }
}
void DockManagerPrivate::addActionToMenu(QAction* Action, QMenu* Menu,bool InsertSorted)
{
    if (InsertSorted)
    {
        auto Actions = Menu->actions();
        auto it = std::find_if(
            Actions.begin(), Actions.end(), [&Action](const QAction* a) {
                return a->text().compare(Action->text(), Qt::CaseInsensitive) > 0;
            });

        if (it == Actions.end())
        {
            Menu->addAction(Action);
        }
        else
        {
            Menu->insertAction(*it, Action);
        }
    }
    else
    {
        Menu->addAction(Action);
    }
}


/**********************************************************************************************************************************/

CDockManager::CDockManager(QWidget *parent) :
	CDockContainerWidget(this, parent),
	d(new DockManagerPrivate(this))
{
	//创建最顶层的 splitter 骨架
	createRootSplitter();
	//创建 auto-hide 用的四个侧边栏基础结构
	createSideTabBarWidgets();
	QMainWindow* MainWindow = qobject_cast<QMainWindow*>(parent);
	if (MainWindow)
	{
		MainWindow->setCentralWidget(this);
	}
	//菜单主要用于收纳所有 dock widget 的
	d->ViewMenu = new QMenu(tr("Show View"), this);
	//ADS 拖拽停靠时你看到的高亮提示
	d->DockAreaOverlay = new CDockOverlay(this, CDockOverlay::ModeDockAreaOverlay);
	d->ContainerOverlay = new CDockOverlay(this, CDockOverlay::ModeContainerOverlay);
	//登记为第一个 dock container
	d->Containers.append(this);
	//给 ADS 自己的内部控件加载默认样式表
	d->loadStylesheet();

	if (CDockManager::testConfigFlag(CDockManager::FocusHighlighting))
	{
		//焦点控制器：跟踪当前哪个 dock widget 获得焦点，驱动 tab 或 floating title bar 的高亮效果
		d->FocusController = new CDockFocusController(this);
	}

	//顶层窗口安装事件过滤器
	window()->installEventFilter(this);


}

//============================================================================
CDockManager::~CDockManager()
{
    // fix memory leaks, see https://github.com/githubuser0xFFFF/Qt-Advanced-Docking-System/issues/307
	std::vector<QPointer<ads::CDockAreaWidget>> areas;
	for (int i = 0; i != dockAreaCount(); ++i)
	{
		areas.push_back( dockArea(i) );
	}
	for ( auto area : areas )
	{
		if (!area || area->dockManager() != this) continue;

		// QPointer delete safety - just in case some dock widget in destruction
		// deletes another related/twin or child dock widget.
		std::vector<QPointer<QWidget>> deleteWidgets;
		for ( auto widget : area->dockWidgets() )
		{
			deleteWidgets.push_back(widget);
		}
		for ( auto ptrWdg : deleteWidgets)
		{
			delete ptrWdg;
		}
	}

	auto FloatingWidgets = d->FloatingWidgets;
	for (auto FloatingWidget : FloatingWidgets)
	{
		FloatingWidget->deleteContent();
		delete FloatingWidget;
	}

	// Delete Dock Widgets before Areas so widgets can access them late (like dtor)
	for ( auto area : areas )
	{
		delete area;
	}

	delete d;
}

// 添加操作
/*
//把一个 CDockWidget 放进 docking 布局中，并返回它最终所在的 CDockAreaWidget
//DockAreaWidget == nullptr：相对整个 container 放置
//DockAreaWidget != nullptr：相对某个已有的 dock area 放置
//index 插入索引，主要在“tab 插入”场景下有意义。
*/
CDockAreaWidget* CDockManager::addDockWidget(DockWidgetArea area, CDockWidget* Dockwidget, CDockAreaWidget* DockAreaWidget, int Index)
{
    d->DockWidgetsMap.insert(Dockwidget->objectName(), Dockwidget);
    auto Container = DockAreaWidget ? DockAreaWidget->dockContainer() : this;
    auto AreaOfAddedDockWidget =
        Container->addDockWidget(area, Dockwidget, DockAreaWidget, Index);
    Q_EMIT dockWidgetAdded(Dockwidget);
    return AreaOfAddedDockWidget;
}
//把 dock widget 直接变成独立浮动窗口
CFloatingDockContainer* CDockManager::addDockWidgetFloating(CDockWidget* Dockwidget)
{
    d->DockWidgetsMap.insert(Dockwidget->objectName(), Dockwidget);
    CDockAreaWidget* OldDockArea = Dockwidget->dockAreaWidget();
    if (OldDockArea)
    {
        OldDockArea->removeDockWidget(Dockwidget);
    }

    Dockwidget->setDockManager(this);
    CFloatingDockContainer* FloatingWidget =
        new CFloatingDockContainer(Dockwidget);
    FloatingWidget->resize(Dockwidget->size());
    if (isVisible())
    {
        FloatingWidget->show();
    }
    else
    {
        d->UninitializedFloatingWidgets.append(FloatingWidget);
    }
    Q_EMIT dockWidgetAdded(Dockwidget);
    return FloatingWidget;
}
//CDockWidget 作为“标签页”插入到某个已有的 CDockAreaWidget 里
CDockAreaWidget* CDockManager::addDockWidgetTabToArea(CDockWidget* Dockwidget, CDockAreaWidget* DockAreaWidget, int Index)
{
    return addDockWidget(ads::CenterDockWidgetArea, Dockwidget, DockAreaWidget,
                         Index);
}
//把一个 CDockWidget 放进某个自动隐藏侧边栏里，并创建对应的 CAutoHideDockContainer
CAutoHideDockContainer* CDockManager::addAutoHideDockWidget( SideBarLocation area, CDockWidget* Dockwidget)
{
    return addAutoHideDockWidgetToContainer(area, Dockwidget, this);
}
CAutoHideDockContainer* CDockManager::addAutoHideDockWidgetToContainer( SideBarLocation area, CDockWidget* Dockwidget,CDockContainerWidget* DockContainerWidget)
{
    d->DockWidgetsMap.insert(Dockwidget->objectName(), Dockwidget);
    auto container =
        DockContainerWidget->createAndSetupAutoHideContainer(area, Dockwidget);
	//创建好后，立即让它处于“收起”状态。
    container->collapseView(true);

    Q_EMIT dockWidgetAdded(Dockwidget);
    return container;
}
CDockAreaWidget* CDockManager::addDockWidgetTab(DockWidgetArea area, CDockWidget* Dockwidget)
{
	//先找“该方位最近添加的 DockArea”
    CDockAreaWidget* AreaWidget = lastAddedDockAreaWidget(area);
    if (AreaWidget)
    {
		//把 DockWidget 作为一个新 tab 加进去
        return addDockWidget(ads::CenterDockWidgetArea, Dockwidget, AreaWidget);
    }
    else
    {
		//新建一个 DockArea 再放进去
        return addDockWidget(area, Dockwidget, nullptr);
    }
}
CDockAreaWidget* CDockManager::addDockWidgetToContainer( DockWidgetArea area, CDockWidget* Dockwidget, CDockContainerWidget* DockContainerWidget)
{
    d->DockWidgetsMap.insert(Dockwidget->objectName(), Dockwidget);
    auto AreaOfAddedDockWidget =
        DockContainerWidget->addDockWidget(area, Dockwidget);
    Q_EMIT dockWidgetAdded(Dockwidget);
    return AreaOfAddedDockWidget;
}


// 状态保存/恢复
QByteArray CDockManager::saveState(int version) const
{
    QByteArray xmldata;
    QXmlStreamWriter s(&xmldata);
    auto ConfigFlags = CDockManager::configFlags();
    s.setAutoFormatting(ConfigFlags.testFlag(XmlAutoFormattingEnabled));
    s.writeStartDocument();
    s.writeStartElement("QtAdvancedDockingSystem");
    s.writeAttribute("Version", QString::number(CurrentVersion));
    s.writeAttribute("UserVersion", QString::number(version));
    s.writeAttribute("Containers", QString::number(d->Containers.count()));
    if (d->CentralWidget)
    {
        s.writeAttribute("CentralWidget", d->CentralWidget->objectName());
    }
    for (auto Container : d->Containers)
    {
        Container->saveState(s);
    }

    s.writeEndElement();
    s.writeEndDocument();

    return ConfigFlags.testFlag(XmlCompressionEnabled) ? qCompress(xmldata, 9) :
                                                         xmldata;
}
bool CDockManager::restoreState(const QByteArray& state, int version)
{
    // Prevent multiple calls as long as state is not restore. This may
    // happen, if QApplication::processEvents() is called somewhere
    if (d->RestoringState)
    {
        return false;
    }

    // We hide the complete dock manager here. Restoring the state means
    // that DockWidgets are removed from the DockArea internal stack layout
    // which in turn  means, that each time a widget is removed the stack
    // will show and raise the next available widget which in turn
    // triggers show events for the dock widgets. To avoid this we hide the
    // dock manager. Because there will be no processing of application
    // events until this function is finished, the user will not see this
    // hiding
    bool IsHidden = this->isHidden();
    if (!IsHidden)
    {
        hide();
    }
    d->RestoringState = true;
    Q_EMIT restoringState();
    bool Result = d->restoreState(state, version);
    d->RestoringState = false;
    if (!IsHidden)
    {
        show();
    }
    Q_EMIT stateRestored();
    return Result;
}
bool CDockManager::isRestoringState() const
{
    return d->RestoringState;
}

// Perspective 机制
void CDockManager::addPerspective(const QString& UniquePrespectiveName)
{
    d->Perspectives.insert(UniquePrespectiveName, saveState());
    Q_EMIT perspectiveListChanged();
}
void CDockManager::openPerspective(const QString& PerspectiveName)
{
    const auto Iterator = d->Perspectives.find(PerspectiveName);
    if (d->Perspectives.end() == Iterator)
    {
        return;
    }

    Q_EMIT openingPerspective(PerspectiveName);
    restoreState(Iterator.value());
    Q_EMIT perspectiveOpened(PerspectiveName);
}
void CDockManager::removePerspective(const QString& Name)
{
    removePerspectives({Name});
}
void CDockManager::removePerspectives(const QStringList& Names)
{
    int Count = 0;
    for (const auto& Name : Names)
    {
        Count += d->Perspectives.remove(Name);
    }

    if (Count)
    {
        Q_EMIT perspectivesRemoved();
        Q_EMIT perspectiveListChanged();
    }
}
QStringList CDockManager::perspectiveNames() const
{
    return d->Perspectives.keys();
}
void CDockManager::savePerspectives(QSettings& Settings) const
{
    Settings.beginWriteArray("Perspectives", d->Perspectives.size());
    int i = 0;
    for (auto it = d->Perspectives.constBegin(); it != d->Perspectives.constEnd();
         ++it)
    {
        Settings.setArrayIndex(i);
        Settings.setValue("Name", it.key());
        Settings.setValue("State", it.value());
        ++i;
    }
    Settings.endArray();
}
void CDockManager::loadPerspectives(QSettings& Settings)
{
    d->Perspectives.clear();
    int Size = Settings.beginReadArray("Perspectives");
    if (!Size)
    {
        Settings.endArray();
        return;
    }

    for (int i = 0; i < Size; ++i)
    {
        Settings.setArrayIndex(i);
        QString Name = Settings.value("Name").toString();
        QByteArray Data = Settings.value("State").toByteArray();
        if (Name.isEmpty() || Data.isEmpty())
        {
            continue;
        }

        d->Perspectives.insert(Name, Data);
    }

    Settings.endArray();
    Q_EMIT perspectiveListChanged();
    Q_EMIT perspectiveListLoaded();
}


// dock 管理查询接口
CDockWidget* CDockManager::findDockWidget(const QString& ObjectName) const
{
    return d->DockWidgetsMap.value(ObjectName, nullptr);
}
void CDockManager::removeDockWidget(CDockWidget* Dockwidget)
{
    Q_EMIT dockWidgetAboutToBeRemoved(Dockwidget);
    d->DockWidgetsMap.remove(Dockwidget->objectName());
    CDockContainerWidget::removeDockWidget(Dockwidget);
    Dockwidget->setDockManager(nullptr);
    Q_EMIT dockWidgetRemoved(Dockwidget);
}
QMap<QString, CDockWidget*> CDockManager::dockWidgetsMap() const
{
    return d->DockWidgetsMap;
}
const QList<CDockContainerWidget*> CDockManager::dockContainers() const
{
    return d->Containers;
}
const QList<CFloatingDockContainer*> CDockManager::floatingWidgets() const
{
    QList<CFloatingDockContainer*> res;
    for (auto& fl : d->FloatingWidgets)
    {
        if (fl)
            res.append(fl);
    }
    return res;
}


// central widget 相关
CDockWidget* CDockManager::centralWidget() const
{
    return d->CentralWidget;
}
CDockAreaWidget* CDockManager::setCentralWidget(CDockWidget* widget)
{
    if (!widget)
    {
        d->CentralWidget = nullptr;
        return nullptr;
    }

    // Setting a new central widget is now allowed if there is already a central
    // widget or if there are already other dock widgets
    if (d->CentralWidget)
    {
        qWarning("Setting a central widget not possible because there is already "
                 "a central widget.");
        return nullptr;
    }

    // Setting a central widget is now allowed if there are already other
    // dock widgets.
    if (!d->DockWidgetsMap.isEmpty())
    {
        qWarning("Setting a central widget not possible - the central widget "
                 "need to be the first "
                 "dock widget that is added to the dock manager.");
        return nullptr;
    }

    widget->setFeature(CDockWidget::DockWidgetClosable, false);
    widget->setFeature(CDockWidget::DockWidgetMovable, false);
    widget->setFeature(CDockWidget::DockWidgetFloatable, false);
    widget->setFeature(CDockWidget::DockWidgetPinnable, false);
    d->CentralWidget = widget;
    CDockAreaWidget* CentralArea = addDockWidget(CenterDockWidgetArea, widget);
    CentralArea->setDockAreaFlag(
        CDockAreaWidget::eDockAreaFlag::HideSingleWidgetTitleBar, true);
    return CentralArea;
}


// 菜单与视图动作
QAction* CDockManager::addToggleViewActionToMenu(QAction* ToggleViewAction, const QString& Group, const QIcon& GroupIcon)
{
    bool AlphabeticallySorted =
        (MenuAlphabeticallySorted == d->MenuInsertionOrder);
    if (!Group.isEmpty())
    {
        QMenu* GroupMenu = d->ViewMenuGroups.value(Group, nullptr);
        if (!GroupMenu)
        {
            GroupMenu = new QMenu(Group, this);
            GroupMenu->setIcon(GroupIcon);
            d->addActionToMenu(GroupMenu->menuAction(), d->ViewMenu,
                               AlphabeticallySorted);
            d->ViewMenuGroups.insert(Group, GroupMenu);
        }
        else if (GroupMenu->icon().isNull() && !GroupIcon.isNull())
        {
            GroupMenu->setIcon(GroupIcon);
        }

        d->addActionToMenu(ToggleViewAction, GroupMenu, AlphabeticallySorted);
        return GroupMenu->menuAction();
    }
    else
    {
        d->addActionToMenu(ToggleViewAction, d->ViewMenu, AlphabeticallySorted);
        return ToggleViewAction;
    }
}
QMenu* CDockManager::viewMenu() const
{
    return d->ViewMenu;
}
void CDockManager::setViewMenuInsertionOrder(eViewMenuInsertionOrder Order)
{
    d->MenuInsertionOrder = Order;
}


// 事件与窗口状态
bool CDockManager::eventFilter(QObject* obj, QEvent* e)
{
    if (e->type() == QEvent::WindowStateChange)
    {
        QWindowStateChangeEvent* ev = static_cast<QWindowStateChangeEvent*>(e);
        if (ev->oldState().testFlag(Qt::WindowMinimized))
        {
            d->IsLeavingMinimized = true;
            QMetaObject::invokeMethod(this, "endLeavingMinimizedState",
                                      Qt::QueuedConnection);
        }
    }
    return Super::eventFilter(obj, e);
}
bool CDockManager::isLeavingMinimizedState() const
{
    return d->IsLeavingMinimized;
}
void CDockManager::endLeavingMinimizedState()
{
    d->IsLeavingMinimized = false;
    this->activateWindow();
}
void CDockManager::showEvent(QShowEvent* event)
{
    Super::showEvent(event);

    //原来可见，但后来因为 manager 被临时隐藏，所以也跟着被隐藏的 floating widgets
    restoreHiddenFloatingWidgets();
    if (d->UninitializedFloatingWidgets.empty())
    {
        return;
    }

    //已经创建了，但因为当时 manager 还没显示，所以不能马上 show 的浮动窗口
    for (auto FloatingWidget : d->UninitializedFloatingWidgets)
    {
        // Check, if someone closed a floating dock widget before the dock
        // manager is shown
        if (FloatingWidget->dockContainer()->hasOpenDockAreas())
        {
            FloatingWidget->show();
        }
    }
    d->UninitializedFloatingWidgets.clear();
}
void CDockManager::restoreHiddenFloatingWidgets()
{
    if (d->HiddenFloatingWidgets.isEmpty())
    {
        return;
    }

    // Restore floating widgets that were hidden upon
    // hideManagerAndFloatingWidgets
    for (auto FloatingWidget : d->HiddenFloatingWidgets)
    {
        bool hasDockWidgetVisible = false;

        // Needed to prevent CFloatingDockContainer being shown empty
        // Could make sense to move this to
        // CFloatingDockContainer::showEvent(QShowEvent *event) if experiencing
        // CFloatingDockContainer being shown empty in other situations, but let's
        // keep it here for now to make sure changes to fix Issue #380 does not
        // impact existing behaviours
        for (auto dockWidget : FloatingWidget->dockWidgets())
        {
            if (dockWidget->toggleViewAction()->isChecked())
            {
                dockWidget->toggleView(true);
                hasDockWidgetVisible = true;
            }
        }

        if (hasDockWidgetVisible)
        {
            FloatingWidget->show();
        }
    }

    d->HiddenFloatingWidgets.clear();
}


// floating/container 注册机制
void CDockManager::registerFloatingWidget(CFloatingDockContainer* FloatingWidget)
{
    d->FloatingWidgets.append(FloatingWidget);
    Q_EMIT floatingWidgetCreated(FloatingWidget);
    ADS_PRINT("d->FloatingWidgets.count() " << d->FloatingWidgets.count());
}
void CDockManager::removeFloatingWidget(CFloatingDockContainer* FloatingWidget)
{
    int removed = d->FloatingWidgets.removeAll(FloatingWidget);
    Q_ASSERT(removed == 1);
}
void CDockManager::registerDockContainer(CDockContainerWidget* DockContainer)
{
    d->Containers.append(DockContainer);
}
void CDockManager::removeDockContainer(CDockContainerWidget* DockContainer)
{
    if (this != DockContainer)
    {
        int removed = d->Containers.removeAll(DockContainer);
        Q_ASSERT(removed == 1);
    }
}
unsigned int CDockManager::zOrderIndex() const
{
    return 0;
}


// overlay / focus / relocation 机制
CDockOverlay* CDockManager::containerOverlay() const
{
    return d->ContainerOverlay;
}
CDockOverlay* CDockManager::dockAreaOverlay() const
{
    return d->DockAreaOverlay;
}
void CDockManager::notifyWidgetOrAreaRelocation(QWidget* DroppedWidget)
{
    if (d->FocusController)
    {
        d->FocusController->notifyWidgetOrAreaRelocation(DroppedWidget);
    }
}
void CDockManager::notifyFloatingWidgetDrop(CFloatingDockContainer* FloatingWidget)
{
    if (d->FocusController)
    {
        d->FocusController->notifyFloatingWidgetDrop(FloatingWidget);
    }
}
CDockFocusController* CDockManager::dockFocusController() const
{
    return d->FocusController;
}
void CDockManager::setDockWidgetFocused(CDockWidget* DockWidget)
{
    if (d->FocusController)
    {
        d->FocusController->setDockWidgetFocused(DockWidget);
    }
}
CDockWidget* CDockManager::focusedDockWidget() const
{
    if (!d->FocusController)
    {
        return nullptr;
    }
    else
    {
        return d->FocusController->focusedDockWidget();
    }
}


// splitter/toolbar/全局锁
QList<int> CDockManager::splitterSizes(CDockAreaWidget* ContainedArea) const
{
    if (ContainedArea)
    {
        auto Splitter = ContainedArea->parentSplitter();
        if (Splitter)
        {
            return Splitter->sizes();
        }
    }
    return QList<int>();
}
void CDockManager::setSplitterSizes(CDockAreaWidget* ContainedArea, const QList<int>& sizes)
{
    if (!ContainedArea)
    {
        return;
    }

    auto Splitter = ContainedArea->parentSplitter();
    if (Splitter && Splitter->count() == sizes.count())
    {
        Splitter->setSizes(sizes);
    }
}
void CDockManager::setDockWidgetToolBarStyle(Qt::ToolButtonStyle Style, CDockWidget::eState State)
{
    if (CDockWidget::StateFloating == State)
    {
        d->ToolBarStyleFloating = Style;
    }
    else
    {
        d->ToolBarStyleDocked = Style;
    }
}
Qt::ToolButtonStyle CDockManager::dockWidgetToolBarStyle(CDockWidget::eState State) const
{
    if (CDockWidget::StateFloating == State)
    {
        return d->ToolBarStyleFloating;
    }
    else
    {
        return d->ToolBarStyleDocked;
    }
}
void CDockManager::setDockWidgetToolBarIconSize(const QSize& IconSize, CDockWidget::eState State)
{
    if (CDockWidget::StateFloating == State)
    {
        d->ToolBarIconSizeFloating = IconSize;
    }
    else
    {
        d->ToolBarIconSizeDocked = IconSize;
    }
}
QSize CDockManager::dockWidgetToolBarIconSize(CDockWidget::eState State) const
{
    if (CDockWidget::StateFloating == State)
    {
        return d->ToolBarIconSizeFloating;
    }
    else
    {
        return d->ToolBarIconSizeDocked;
    }
}
void CDockManager::lockDockWidgetFeaturesGlobally(CDockWidget::DockWidgetFeatures Value)
{
    // Limit the features to CDockWidget::GloballyLockableFeatures
    //位与操作，丢弃不可全局锁定的特性位（如 Focusable、DeleteOnClose），防止误传
    Value &= CDockWidget::GloballyLockableFeatures;
    if (d->LockedDockWidgetFeatures == Value)
    {
        return;
    }

    d->LockedDockWidgetFeatures = Value;
    // Call the notifyFeaturesChanged() function for all dock widgets to update
    // the state of the close and detach buttons
    for (auto DockWidget : d->DockWidgetsMap)
    {
        DockWidget->notifyFeaturesChanged();
    }
}
CDockWidget::DockWidgetFeatures CDockManager::globallyLockedDockWidgetFeatures()const
{
    return d->LockedDockWidgetFeatures;
}


// 静态全局配置
CDockManager::ConfigFlags CDockManager::configFlags()
{
    return StaticConfigFlags;
}
CDockManager::AutoHideFlags CDockManager::autoHideConfigFlags()
{
    return StaticAutoHideConfigFlags;
}
void CDockManager::setConfigFlags(const ConfigFlags Flags)
{
    StaticConfigFlags = Flags;
}
void CDockManager::setAutoHideConfigFlags(const AutoHideFlags Flags)
{
    StaticAutoHideConfigFlags = Flags;
}
void CDockManager::setConfigFlag(eConfigFlag Flag, bool On)
{
    internal::setFlag(StaticConfigFlags, Flag, On);
}
void CDockManager::setAutoHideConfigFlag(eAutoHideFlag Flag, bool On)
{
    internal::setFlag(StaticAutoHideConfigFlags, Flag, On);
}
bool CDockManager::testConfigFlag(eConfigFlag Flag)
{
    return configFlags().testFlag(Flag);
}
bool CDockManager::testAutoHideConfigFlag(eAutoHideFlag Flag)
{
    return autoHideConfigFlags().testFlag(Flag);
}
void CDockManager::setConfigParam(CDockManager::eConfigParam Param, QVariant Value)
{
    StaticConfigParams[Param] = Value;
}
QVariant CDockManager::configParam(eConfigParam Param, QVariant Default)
{
    return StaticConfigParams[Param].isValid() ? StaticConfigParams[Param] :
                                                 Default;
}
int CDockManager::startDragDistance()
{
    return QApplication::startDragDistance() * 1.5;
}
CIconProvider& CDockManager::iconProvider()
{
    static CIconProvider Instance;
    return Instance;
}
void CDockManager::setFloatingContainersTitle(const QString& Title)
{
    FloatingContainersTitle = Title;
}
QString CDockManager::floatingContainersTitle()
{
    if (FloatingContainersTitle.isEmpty())
        return qApp->applicationDisplayName();

    return FloatingContainersTitle;
}


// 窗口显隐控制
void CDockManager::hideManagerAndFloatingWidgets()
{
    hide();

    d->HiddenFloatingWidgets.clear();
    // Hide updates of floating widgets from user
    for (auto FloatingWidget : d->FloatingWidgets)
    {
        if (FloatingWidget->isVisible())
        {
            QList<CDockWidget*> VisibleWidgets;
            for (auto dockWidget : FloatingWidget->dockWidgets())
            {
                if (dockWidget->toggleViewAction()->isChecked())
                    VisibleWidgets.push_back(dockWidget);
            }

            // save as floating widget to be shown when CDockManager will be shown
            // back
            d->HiddenFloatingWidgets.push_back(FloatingWidget);
            FloatingWidget->hide();

            // hiding floating widget automatically marked contained CDockWidgets
            // as hidden but they must remain marked as visible as we want them to
            // be restored visible when CDockManager will be shown back
            for (auto dockWidget : VisibleWidgets)
            {
                dockWidget->toggleViewAction()->setChecked(true);
            }
        }
    }
}
void CDockManager::raise()
{
    if (parentWidget())
    {
        parentWidget()->raise();
    }
}


// 组件工厂机制
CDockWidget* CDockManager::createDockWidget(const QString &title, QWidget* parent)
{
	return new CDockWidget(this, title, parent);
}
QSharedPointer<ads::CDockComponentsFactory> CDockManager::componentsFactory() const
{
    return d->ComponentFactory;
}
void CDockManager::setComponentsFactory(ads::CDockComponentsFactory* factory)
{
    setComponentsFactory(QSharedPointer<ads::CDockComponentsFactory>(factory));
}
void CDockManager::setComponentsFactory(QSharedPointer<ads::CDockComponentsFactory> factory)
{
    d->ComponentFactory = factory;
}






} // namespace ads

//---------------------------------------------------------------------------
// EOF DockManager.cpp
