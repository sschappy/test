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
/// \file   DockContainerWidget.cpp
/// \author Uwe Kindler
/// \date   24.02.2017
/// \brief  Implementation of CDockContainerWidget class
//============================================================================


//============================================================================
//                                   INCLUDES
//============================================================================
#include "DockContainerWidget.h"

#include <QEvent>
#include <QList>
#include <QGridLayout>
#include <QPointer>
#include <QVariant>
#include <QDebug>
#include <QXmlStreamWriter>
#include <QAbstractButton>
#include <QLabel>
#include <QTimer>
#include <QMetaObject>
#include <QMetaType>
#include <QApplication>

#include "DockManager.h"
#include "DockAreaWidget.h"
#include "DockWidget.h"
#include "DockingStateReader.h"
#include "FloatingDockContainer.h"
#include "DockOverlay.h"
#include "ads_globals.h"
#include "DockSplitter.h"
#include "DockWidgetTab.h"
#include "DockAreaTitleBar.h"
#include "DockFocusController.h"
#include "AutoHideDockContainer.h"
#include "AutoHideSideBar.h"
#include "AutoHideTab.h"

#include <functional>
#include <iostream>

#if QT_VERSION < 0x050900

inline char toHexLower(uint value)
{
    return "0123456789abcdef"[value & 0xF];
}

QByteArray qByteArrayToHex(const QByteArray& src, char separator)
{
    if(src.size() == 0)
        return QByteArray();

    const int length = separator ? (src.size() * 3 - 1) : (src.size() * 2);
    QByteArray hex(length, Qt::Uninitialized);
    char *hexData = hex.data();
    const uchar *data = reinterpret_cast<const uchar *>(src.data());
    for (int i = 0, o = 0; i < src.size(); ++i) {
        hexData[o++] = toHexLower(data[i] >> 4);
        hexData[o++] = toHexLower(data[i] & 0xf);

        if ((separator) && (o < length))
            hexData[o++] = separator;
    }
    return hex;
}
#endif

namespace ads
{
static unsigned int zOrderCounter = 0;

enum eDropMode
{
	DropModeIntoArea,///< drop widget into a dock area
	DropModeIntoContainer,///< drop into container
	DropModeInvalid///< invalid mode - do not drop
};

/**
 * Converts dock area ID to an index for array access
 */
static int areaIdToIndex(DockWidgetArea area)
{
	switch (area)
	{
	case LeftDockWidgetArea: return 0;
	case RightDockWidgetArea: return 1;
	case TopDockWidgetArea: return 2;
	case BottomDockWidgetArea: return 3;
	case CenterDockWidgetArea: return 4;
	default:
		return 4;
	}
}

/**
 * Helper function to ease insertion of dock area into splitter
 */
static void insertWidgetIntoSplitter(QSplitter* Splitter, QWidget* widget, bool Append)
{
	if (Append)
	{
		Splitter->addWidget(widget);
	}
	else
	{
		Splitter->insertWidget(0, widget);
	}
}


/************************************************************************************************************************************************/

class DockContainerWidgetPrivate
{
public:
	CDockContainerWidget* _this;

	QPointer<CDockManager> DockManager;
    bool isFloating = false;
    QGridLayout* Layout = nullptr;
    CDockSplitter* RootSplitter = nullptr;
    QMap<SideBarLocation, CAutoHideSideBar*> SideTabBarWidgets;
    CDockAreaWidget* TopLevelDockArea = nullptr;

	QList<QPointer<CDockAreaWidget>> DockAreas;
    CDockAreaWidget* LastAddedAreaCache[5];

	int VisibleDockAreaCount = -1;

	QList<CAutoHideDockContainer*> AutoHideWidgets;
    CAutoHideTab* DelayedAutoHideTab;
    bool DelayedAutoHideShow = false;
    QTimer DelayedAutoHideTimer;

	unsigned int zOrderIndex = 0;


	/*********函数***********/
    DockContainerWidgetPrivate(CDockContainerWidget* _public);
	CDockSplitter* newSplitter(Qt::Orientation orientation, QWidget* parent = nullptr)
    {
        CDockSplitter* s = new CDockSplitter(orientation, parent);
        s->setOpaqueResize(
            CDockManager::testConfigFlag(CDockManager::OpaqueSplitterResize));
        s->setChildrenCollapsible(false);
        return s;
    }
	//核心
    CDockAreaWidget* addDockWidgetToDockArea(DockWidgetArea area, CDockWidget* Dockwidget, CDockAreaWidget* TargetDockArea, int Index = -1);
    CDockAreaWidget* addDockWidgetToContainer(DockWidgetArea area, CDockWidget* Dockwidget);
    void updateSplitterHandles(QSplitter* splitter);
    bool widgetResizesWithContainer(QWidget* widget);
    void adjustSplitterSizesOnInsertion(QSplitter* Splitter, qreal LastRatio = 1.0)
    {
        int AreaSize = (Splitter->orientation() == Qt::Horizontal) ?
                           Splitter->width() :
                           Splitter->height();
        auto SplitterSizes = Splitter->sizes();
		//最后一个比重是LastRatio  其他的是1。从而算出总比重
        qreal TotRatio = SplitterSizes.size() - 1.0 + LastRatio;
        for (int i = 0; i < SplitterSizes.size() - 1; i++)
        {
            SplitterSizes[i] = AreaSize / TotRatio;
        }
        SplitterSizes.back() = AreaSize * LastRatio / TotRatio;
        Splitter->setSizes(SplitterSizes);
    }
    void addDockAreasToList(const QList<CDockAreaWidget*> NewDockAreas);
    void appendDockAreas(const QList<CDockAreaWidget*> NewDockAreas);
    void onDockAreaViewToggled(bool Visible)
    {
        CDockAreaWidget* DockArea =
            qobject_cast<CDockAreaWidget*>(_this->sender());
        VisibleDockAreaCount += Visible ? 1 : -1;
        onVisibleDockAreaCountChanged();
        Q_EMIT _this->dockAreaViewToggled(DockArea, Visible);
    }
    void emitDockAreasAdded()
    {
        onVisibleDockAreaCountChanged();
        Q_EMIT _this->dockAreasAdded();
    }
    void onVisibleDockAreaCountChanged();
    void addDockArea(CDockAreaWidget* NewDockWidget, DockWidgetArea area = CenterDockWidgetArea);
    void emitDockAreasRemoved()
    {
        onVisibleDockAreaCountChanged();
        Q_EMIT _this->dockAreasRemoved();
    }

    void dropIntoSection(CFloatingDockContainer* FloatingWidget, CDockAreaWidget* TargetArea, DockWidgetArea area,int TabIndex = 0);
    void dropIntoCenterOfSection(CFloatingDockContainer* FloatingWidget, CDockAreaWidget* TargetArea, int TabIndex = 0);
    void dropIntoAutoHideSideBar(CFloatingDockContainer* FloatingWidget, DockWidgetArea area);
    void dropIntoContainer(CFloatingDockContainer* FloatingWidget, DockWidgetArea area);
    void moveToNewSection(QWidget* Widget, CDockAreaWidget* TargetArea, DockWidgetArea area, int TabIndex = 0);
    void moveIntoCenterOfSection(QWidget* Widget, CDockAreaWidget* TargetArea, int TabIndex = 0);
    void moveToAutoHideSideBar(QWidget* Widget, DockWidgetArea area, int TabIndex = TabDefaultInsertIndex);
    void moveToContainer(QWidget* Widgett, DockWidgetArea area);

    void saveChildNodesState(QXmlStreamWriter& Stream, QWidget* Widget);
    void saveAutoHideWidgetsState(QXmlStreamWriter& Stream);
    bool restoreChildNodes(CDockingStateReader& Stream, QWidget*& CreatedWidget, bool Testing);
    bool restoreSplitter(CDockingStateReader& Stream, QWidget*& CreatedWidget, bool Testing);
    bool restoreDockArea(CDockingStateReader& Stream, QWidget*& CreatedWidget, bool Testing);
    bool restoreSideBar(CDockingStateReader& Stream, QWidget*& CreatedWidget, bool Testing);

    void dumpRecursive(int level, QWidget* widget);
    // 未被使用的函数
    eDropMode getDropMode(const QPoint& TargetPos);
    int& visibleDockAreaCount()
    {
        // Lazy initialisation - we initialize the VisibleDockAreaCount variable
        // on first use
        initVisibleDockAreaCount();
        return VisibleDockAreaCount;
    }
	void initVisibleDockAreaCount()
	{
		if (VisibleDockAreaCount > -1)
		{
			return;
		}

		VisibleDockAreaCount = 0;
		for (auto DockArea : DockAreas)
		{
			if (!DockArea)
			{
				continue;
			}
			VisibleDockAreaCount += (DockArea->isHidden() ? 0 : 1);
		}
	}

}; 

DockContainerWidgetPrivate::DockContainerWidgetPrivate( CDockContainerWidget* _public)
    : _this(_public)
{
    std::fill(std::begin(LastAddedAreaCache), std::end(LastAddedAreaCache),
              nullptr);
    //auto-hide 标签的“延时自动打开”交互
    DelayedAutoHideTimer.setSingleShot(true);
    DelayedAutoHideTimer.setInterval(500);
    //CDockManager::AutoHideShowOnMouseOver 启动鼠标悬停自动展开后 延迟展开
    QObject::connect(&DelayedAutoHideTimer, &QTimer::timeout, [this]() {
        auto GlobalPos = DelayedAutoHideTab->mapToGlobal(QPoint(0, 0));
        qApp->sendEvent(DelayedAutoHideTab,
                        new QMouseEvent(QEvent::MouseButtonPress, QPoint(0, 0),
                                        GlobalPos, Qt::LeftButton,
                                        {Qt::LeftButton}, Qt::NoModifier));
    });
}

CDockAreaWidget* DockContainerWidgetPrivate::addDockWidgetToDockArea( DockWidgetArea area, CDockWidget* Dockwidget, CDockAreaWidget* TargetDockArea, int Index)
{
	//插入到这个已有 area 内部，作为 tab
    if (CenterDockWidgetArea == area)
    {
        TargetDockArea->insertDockWidget(Index, Dockwidget);
        TargetDockArea->updateTitleBarVisibility();
        return TargetDockArea;
    }

	//相对这个已有 area 左右上下切分出一个新 area，再把 dock 放进去
    CDockAreaWidget* NewDockArea = new CDockAreaWidget(DockManager, _this);
    NewDockArea->addDockWidget(Dockwidget);
    auto InsertParam = internal::dockAreaInsertParameters(area);

    auto TargetAreaSplitter = TargetDockArea->parentSplitter();
    int index = TargetAreaSplitter->indexOf(TargetDockArea);
    if (TargetAreaSplitter->orientation() == InsertParam.orientation())
    {
        ADS_PRINT(
            "TargetAreaSplitter->orientation() == InsertParam.orientation()");
        TargetAreaSplitter->insertWidget(index + InsertParam.insertOffset(),
                                         NewDockArea);
        updateSplitterHandles(TargetAreaSplitter);
        // do nothing, if flag is not enabled
        if (CDockManager::testConfigFlag(CDockManager::EqualSplitOnInsertion))
        {
            adjustSplitterSizesOnInsertion(TargetAreaSplitter);
        }
    }
    else
    {
        ADS_PRINT(
            "TargetAreaSplitter->orientation() != InsertParam.orientation()");
        auto TargetAreaSizes = TargetAreaSplitter->sizes();
        // 构建一个新的splitter然后把目标 area 和新 area 都放进去
        auto NewSplitter = newSplitter(InsertParam.orientation());
        NewSplitter->addWidget(TargetDockArea);
        insertWidgetIntoSplitter(NewSplitter, NewDockArea, InsertParam.append());
        updateSplitterHandles(NewSplitter);
		//把新的splitter插入到原来目标area的位置
        TargetAreaSplitter->insertWidget(index, NewSplitter);
        updateSplitterHandles(TargetAreaSplitter);
        if (CDockManager::testConfigFlag(CDockManager::EqualSplitOnInsertion))
        {
            TargetAreaSplitter->setSizes(TargetAreaSizes);
            adjustSplitterSizesOnInsertion(NewSplitter);
        }
    }

    addDockAreasToList({NewDockArea});
    return NewDockArea;
}
void DockContainerWidgetPrivate::updateSplitterHandles(QSplitter* splitter)
{
    if (!DockManager->centralWidget() || !splitter)
    {
        return;
    }

    for (int i = 0; i < splitter->count(); ++i)
    {
        splitter->setStretchFactor(
            i, widgetResizesWithContainer(splitter->widget(i)) ? 1 : 0);
    }
}
bool DockContainerWidgetPrivate::widgetResizesWithContainer(QWidget* widget)
{
    if (!DockManager->centralWidget())
    {
        return true;
    }

    auto Area = qobject_cast<CDockAreaWidget*>(widget);
    if (Area)
    {
        return Area->isCentralWidgetArea();
    }

    auto innerSplitter = qobject_cast<CDockSplitter*>(widget);
    if (innerSplitter)
    {
        return innerSplitter->isResizingWithContainer();
    }

    return false;
}
void DockContainerWidgetPrivate::addDockAreasToList( const QList<CDockAreaWidget*> NewDockAreas)
{
    int CountBefore = DockAreas.count();
    int NewAreaCount = NewDockAreas.count();
    appendDockAreas(NewDockAreas);
    // If the user dropped a floating widget that contains only one single
    // visible dock area, then its title bar button TitleBarButtonUndock is
    // likely hidden. We need to ensure, that it is visible
    for (auto DockArea : NewDockAreas)
    {
        DockArea->titleBarButton(TitleBarButtonClose)->setVisible(true);
        DockArea->titleBarButton(TitleBarButtonAutoHide)->setVisible(true);
        DockArea->titleBarButton(TitleBarButtonUndock)->setVisible(true);
    }

    // We need to ensure, that the dock area title bar is visible. The title bar
    // is invisible, if the dock are is a single dock area in a floating widget.
    if (1 == CountBefore)
    {
        DockAreas.at(0)->updateTitleBarVisibility();
    }

    if (1 == NewAreaCount)
    {
        DockAreas.last()->updateTitleBarVisibility();
    }

    emitDockAreasAdded();
}
void DockContainerWidgetPrivate::appendDockAreas( const QList<CDockAreaWidget*> NewDockAreas)
{
    for (auto* newDockArea : NewDockAreas)
    {
        DockAreas.append(newDockArea);
    }
    for (auto DockArea : NewDockAreas)
    {
        /*
		std::bind(&DockContainerWidgetPrivate::onDockAreaViewToggled, this,
          std::placeholders::_1)

        把 DockContainerWidgetPrivate 对象的成员函数 onDockAreaViewToggled
        绑定成一个可调用对象，并把信号传来的第一个参数转发进去

		等价于：
        [this](auto arg1) {
			this->onDockAreaViewToggled(arg1);
		}
        注意：这里的 this 是指向 DockContainerWidgetPrivate 对象的指针，而不是
        CDockContainerWidget 对象的指针，因为 onDockAreaViewToggled 是
        DockContainerWidgetPrivate 的成员函数。

		*/
        QObject::connect(
            DockArea, &CDockAreaWidget::viewToggled, _this,
            std::bind(&DockContainerWidgetPrivate::onDockAreaViewToggled, this,
                      std::placeholders::_1));
    }
}
void DockContainerWidgetPrivate::onVisibleDockAreaCountChanged()
{
    auto TopLevelDockArea = _this->topLevelDockArea();

    if (TopLevelDockArea)
    {
        this->TopLevelDockArea = TopLevelDockArea;
        TopLevelDockArea->updateTitleBarButtonVisibility(true);
    }
    else if (this->TopLevelDockArea)
    {
        this->TopLevelDockArea->updateTitleBarButtonVisibility(false);
        this->TopLevelDockArea = nullptr;
    }
}
void DockContainerWidgetPrivate::addDockArea(CDockAreaWidget* NewDockArea, DockWidgetArea area)
{
    auto InsertParam = internal::dockAreaInsertParameters(area);
    // As long as we have only one dock area in the splitter we can adjust
    // its orientation
    if (DockAreas.count() <= 1)
    {
        RootSplitter->setOrientation(InsertParam.orientation());
    }

    QSplitter* Splitter = RootSplitter;
    if (Splitter->orientation() == InsertParam.orientation())
    {
        insertWidgetIntoSplitter(Splitter, NewDockArea, InsertParam.append());
        updateSplitterHandles(Splitter);
        if (Splitter->isHidden())
        {
            Splitter->show();
        }
    }
    else
    {
        auto NewSplitter = newSplitter(InsertParam.orientation());
        if (InsertParam.append())
        {
            QLayoutItem* li = Layout->replaceWidget(Splitter, NewSplitter);
            NewSplitter->addWidget(Splitter);
            NewSplitter->addWidget(NewDockArea);
            updateSplitterHandles(NewSplitter);
            delete li;
        }
        else
        {
            NewSplitter->addWidget(NewDockArea);
            QLayoutItem* li = Layout->replaceWidget(Splitter, NewSplitter);
            NewSplitter->addWidget(Splitter);
            updateSplitterHandles(NewSplitter);
            delete li;
        }
        RootSplitter = NewSplitter;
    }

    addDockAreasToList({NewDockArea});
}
CDockAreaWidget* DockContainerWidgetPrivate::addDockWidgetToContainer(DockWidgetArea area, CDockWidget* Dockwidget)
{
    CDockAreaWidget* NewDockArea = new CDockAreaWidget(DockManager, _this);
    NewDockArea->addDockWidget(Dockwidget);
    addDockArea(NewDockArea, area);
    NewDockArea->updateTitleBarVisibility();
    LastAddedAreaCache[areaIdToIndex(area)] = NewDockArea;
    return NewDockArea;
}

void DockContainerWidgetPrivate::dropIntoSection(CFloatingDockContainer* FloatingWidget, CDockAreaWidget* TargetArea, DockWidgetArea area, int TabIndex)
{
    // 如果是中心区域，直接走 tab 合并
    if (CenterDockWidgetArea == area)
    {
        dropIntoCenterOfSection(FloatingWidget, TargetArea, TabIndex);
        return;
    }

	//进入真正的“切分区域”逻辑
    CDockContainerWidget* FloatingContainer = FloatingWidget->dockContainer();
	//抽象的 Left/Right/Top/Bottom 转成具体插入参数
    auto InsertParam = internal::dockAreaInsertParameters(area);
	// floating 容器里这次会被迁入目标 container 的所有 CDockAreaWidget
    auto NewDockAreas = FloatingContainer->findChildren<CDockAreaWidget*>(
        QString(), Qt::FindChildrenRecursively);
    auto TargetAreaSplitter = TargetArea->parentSplitter();
	//TargetArea 在父 splitter 中的位置
    int AreaIndex = TargetAreaSplitter->indexOf(TargetArea);
	// floating 容器内部的根 splitter
    auto FloatingSplitter = FloatingContainer->rootSplitter();
	//目标父 splitter 的方向和这次插入方向一致
    if (TargetAreaSplitter->orientation() == InsertParam.orientation())
    {
        auto Sizes = TargetAreaSplitter->sizes();
        int TargetAreaSize = (InsertParam.orientation() == Qt::Horizontal) ?
                                 TargetArea->width() :
                                 TargetArea->height();
        bool AdjustSplitterSizes = true;
        if ((FloatingSplitter->orientation() != InsertParam.orientation())
            && FloatingSplitter->count() > 1)
        {
			//如果被拖进来的 floating 布局本身是一棵“方向不匹配、而且不止一个 child”的 splitter 树，
			// 那就不要拆开它，直接把整个 FloatingSplitter 当成一个整体插进目标父 splitter。
            TargetAreaSplitter->insertWidget(
                AreaIndex + InsertParam.insertOffset(), FloatingSplitter);
            updateSplitterHandles(TargetAreaSplitter);
        }
        else
        {
			//如果 floating 的根 splitter 结构简单，或者方向匹配，就把它的 child 一个一个拆下来，直接插入目标父 splitter
            AdjustSplitterSizes = (FloatingSplitter->count() == 1);
            int InsertIndex = AreaIndex + InsertParam.insertOffset();
            while (FloatingSplitter->count())
            {
                TargetAreaSplitter->insertWidget(InsertIndex++,
                                                 FloatingSplitter->widget(0));
                updateSplitterHandles(TargetAreaSplitter);
            }
        }

        if (AdjustSplitterSizes)
        {
            int Size = (TargetAreaSize - TargetAreaSplitter->handleWidth()) / 2;
            Sizes[AreaIndex] = Size;
            Sizes.insert(AreaIndex, Size);
            TargetAreaSplitter->setSizes(Sizes);
        }
    }
	//目标父 splitter 方向和插入方向不一致
    else
    {
        QSplitter* NewSplitter = newSplitter(InsertParam.orientation());
        int TargetAreaSize = (InsertParam.orientation() == Qt::Horizontal) ?
                                 TargetArea->width() :
                                 TargetArea->height();
        bool AdjustSplitterSizes = true;
        if ((FloatingSplitter->orientation() != InsertParam.orientation())
            && FloatingSplitter->count() > 1)
        {
			//把整个 FloatingSplitter 当成一个整体 child 放进 NewSplitter
            NewSplitter->addWidget(FloatingSplitter);
            updateSplitterHandles(NewSplitter);
        }
        else
        {
			//逐个 child 加入 NewSplitter
            AdjustSplitterSizes = (FloatingSplitter->count() == 1);
            while (FloatingSplitter->count())
            {
                NewSplitter->addWidget(FloatingSplitter->widget(0));
                updateSplitterHandles(NewSplitter);
            }
        }

        // Save the sizes before insertion and restore it later to prevent
        // shrinking of existing area
        auto Sizes = TargetAreaSplitter->sizes();
		//把 TargetArea 插进 NewSplitter
        insertWidgetIntoSplitter(NewSplitter, TargetArea, !InsertParam.append());
        updateSplitterHandles(NewSplitter);
        if (AdjustSplitterSizes)
        {
            int Size = TargetAreaSize / 2;
            NewSplitter->setSizes({Size, Size});
        }
        TargetAreaSplitter->insertWidget(AreaIndex, NewSplitter);
        TargetAreaSplitter->setSizes(Sizes);
        updateSplitterHandles(TargetAreaSplitter);
    }
	//把新迁入的 dock areas 纳入当前 container 管理
    addDockAreasToList(NewDockAreas);
    _this->dumpLayout();
}
void DockContainerWidgetPrivate::dropIntoCenterOfSection(CFloatingDockContainer* FloatingWidget, CDockAreaWidget* TargetArea, int TabIndex)
{
    CDockContainerWidget* FloatingContainer = FloatingWidget->dockContainer();
    auto NewDockWidgets = FloatingContainer->dockWidgets();
    auto TopLevelDockArea = FloatingContainer->topLevelDockArea();
    int NewCurrentIndex = -1;
    TabIndex = qMax(0, TabIndex);

    // If the floating widget contains only one single dock are, then the
    // current dock widget of the dock area will also be the future current
    // dock widget in the drop area.
    if (TopLevelDockArea)
    {
		//判断这个 floating 容器当前是不是只有一个唯一可见的顶层 dock area
        /*
		如果它只有一个 area，那么：

		这个 area 现在有哪个 tab 是 current
		将来 drop 到 TargetArea 后，也应该尽量保持那个 tab 成为当前项
		*/
        NewCurrentIndex = TopLevelDockArea->currentIndex();
    }

    for (int i = 0; i < NewDockWidgets.count(); ++i)
    {
        CDockWidget* DockWidget = NewDockWidgets[i];
        TargetArea->insertDockWidget(TabIndex + i, DockWidget, false);
        // If the floating widget contains multiple visible dock areas, then we
        // simply pick the first visible open dock widget and make it
        // the current one.
        if (NewCurrentIndex < 0 && !DockWidget->isClosed())
        {
            NewCurrentIndex = i;
        }
    }
    TargetArea->setCurrentIndex(NewCurrentIndex + TabIndex);
    TargetArea->updateTitleBarVisibility();
    return;
}
void DockContainerWidgetPrivate::dropIntoAutoHideSideBar(CFloatingDockContainer* FloatingWidget, DockWidgetArea area)
{
    auto SideBarLocation = internal::toSideBarLocation(area);
    auto NewDockAreas = FloatingWidget->findChildren<CDockAreaWidget*>(
        QString(), Qt::FindChildrenRecursively);
	//拿到当前鼠标对应的 side bar tab 插入位置
    int TabIndex = DockManager->containerOverlay()->tabIndexUnderCursor();
    for (auto DockArea : NewDockAreas)
    {
        auto DockWidgets = DockArea->dockWidgets();
        for (auto DockWidget : DockWidgets)
        {
            _this->createAndSetupAutoHideContainer(SideBarLocation, DockWidget,
                                                   TabIndex++);
        }
    }
}
void DockContainerWidgetPrivate::dropIntoContainer(CFloatingDockContainer* FloatingWidget, DockWidgetArea area)
{
    auto InsertParam = internal::dockAreaInsertParameters(area);
    CDockContainerWidget* FloatingDockContainer = FloatingWidget->dockContainer();
    auto NewDockAreas = FloatingDockContainer->findChildren<CDockAreaWidget*>(
        QString(), Qt::FindChildrenRecursively);
	//把当前 container 的根 splitter 取出来
    auto Splitter = RootSplitter;

	//当前 container 里 area 很少，根 splitter 方向还可以自由调整
    if (DockAreas.count() <= 1)
    {
        Splitter->setOrientation(InsertParam.orientation());
    }
    else if (Splitter->orientation() != InsertParam.orientation())
    {
		//如果这次 drop 方向和它不匹配，就新建一层更高的 splitter，把旧 root 整体包进去
        auto NewSplitter = newSplitter(InsertParam.orientation());
        QLayoutItem* li = Layout->replaceWidget(Splitter, NewSplitter);
        NewSplitter->addWidget(Splitter);
        updateSplitterHandles(NewSplitter);
        Splitter = NewSplitter;
        delete li;
    }

    // Now we can insert the floating widget content into this container
    auto FloatingSplitter = FloatingDockContainer->rootSplitter();
    if (FloatingSplitter->count() == 1)
    {
		//如果被拖进来的 floating 内容实际上非常简单，根 splitter 下面只有一个 child，那就没必要保留这个 floating splitter 外壳。
		//直接把它唯一那个 child 挂进当前目标 splitter 就行
        insertWidgetIntoSplitter(Splitter, FloatingSplitter->widget(0),
                                 InsertParam.append());
        updateSplitterHandles(Splitter);
    }
    else if (FloatingSplitter->orientation() == InsertParam.orientation())
    {
		//如果 floating 根 splitter 本身方向和当前目标 splitter 一致，那就把 floating 里的 child 一个一个拆出来，平铺插入当前 splitter
        int InsertIndex = InsertParam.append() ? Splitter->count() : 0;
        while (FloatingSplitter->count())
        {
            Splitter->insertWidget(InsertIndex++, FloatingSplitter->widget(0));
            updateSplitterHandles(Splitter);
        }
    }
    else
    {
		//如果 floating 里是一棵复杂子树，而且它的根 splitter 方向和目标方向不同，
		// 那就不要拆它，直接把整个 FloatingSplitter 当成一个整体 child 挂进目标 splitter
        insertWidgetIntoSplitter(Splitter, FloatingSplitter,
                                 InsertParam.append());
    }

    RootSplitter = Splitter;
    addDockAreasToList(NewDockAreas);

    // If we dropped the floating widget into the main dock container that does
    // not contain any dock widgets, then splitter is invisible and we need to
    // show it to display the docked widgets
    if (!Splitter->isVisible())
    {
        Splitter->show();
    }
    _this->dumpLayout();
}
void DockContainerWidgetPrivate::moveToNewSection(QWidget* Widget,CDockAreaWidget* TargetArea, DockWidgetArea area,int TabIndex)
{
    // Dropping into center means all dock widgets in the dropped floating
    // widget will become tabs of the drop area
    if (CenterDockWidgetArea == area)
    {
        //如果目标是 TargetArea 的中心区域，那就不需要切 splitter，直接把它插成 tab
        moveIntoCenterOfSection(Widget, TargetArea, TabIndex);
        return;
    }

    //判断拖进来的到底是单个 dock widget 还是整个 dock area
    CDockWidget* DroppedDockWidget = qobject_cast<CDockWidget*>(Widget);
    CDockAreaWidget* DroppedDockArea = qobject_cast<CDockAreaWidget*>(Widget);
    CDockAreaWidget* NewDockArea;
    if (DroppedDockWidget)
    {
        //它本身还不是一个 area，必须先把它装进一个新的 CDockAreaWidget
        NewDockArea = new CDockAreaWidget(DockManager, _this);
        CDockAreaWidget* OldDockArea = DroppedDockWidget->dockAreaWidget();
        if (OldDockArea)
        {
            OldDockArea->removeDockWidget(DroppedDockWidget);
        }
        NewDockArea->addDockWidget(DroppedDockWidget);
    }
    else
    {
        //它本来就是一个完整区域，可以整体挪走，不必再包装
        DroppedDockArea->dockContainer()->removeDockArea(DroppedDockArea);
        NewDockArea = DroppedDockArea;
    }

    auto InsertParam = internal::dockAreaInsertParameters(area);
    auto TargetAreaSplitter = TargetArea->parentSplitter();
    int AreaIndex = TargetAreaSplitter->indexOf(TargetArea);
    auto Sizes = TargetAreaSplitter->sizes();
    //目标父 splitter 的方向和本次插入方向一致
    if (TargetAreaSplitter->orientation() == InsertParam.orientation())
    {
        int TargetAreaSize = (InsertParam.orientation() == Qt::Horizontal) ?
                                 TargetArea->width() :
                                 TargetArea->height();
        TargetAreaSplitter->insertWidget(AreaIndex + InsertParam.insertOffset(),
                                         NewDockArea);
        updateSplitterHandles(TargetAreaSplitter);
        // 目标Area和新加的area一起均分原来目标area的空间
        int Size = (TargetAreaSize - TargetAreaSplitter->handleWidth()) / 2;
        Sizes[AreaIndex] = Size;
        Sizes.insert(AreaIndex, Size);
    }
    else
    {
        //当前父 splitter 方向不对，不能直接插，于是新建一层 splitter。
        int TargetAreaSize = (InsertParam.orientation() == Qt::Horizontal) ?
                                 TargetArea->width() :
                                 TargetArea->height();
        QSplitter* NewSplitter = newSplitter(InsertParam.orientation());
        NewSplitter->addWidget(TargetArea);
        insertWidgetIntoSplitter(NewSplitter, NewDockArea, InsertParam.append());
        updateSplitterHandles(NewSplitter);
        int Size = TargetAreaSize / 2;
        NewSplitter->setSizes({Size, Size});
        TargetAreaSplitter->insertWidget(AreaIndex, NewSplitter);
        updateSplitterHandles(TargetAreaSplitter);
    }
    TargetAreaSplitter->setSizes(Sizes);

    addDockAreasToList({NewDockArea});
}
void DockContainerWidgetPrivate::moveIntoCenterOfSection(QWidget* Widget, CDockAreaWidget* TargetArea, int TabIndex)
{
    //判断传进来的是单个 DockWidget 还是整个 DockArea
    auto DroppedDockWidget = qobject_cast<CDockWidget*>(Widget);
    auto DroppedArea = qobject_cast<CDockAreaWidget*>(Widget);

    TabIndex = qMax(0, TabIndex);
    if (DroppedDockWidget)
    {
        //当前只是把一个单独的 dock widget 并入目标 area
        CDockAreaWidget* OldDockArea = DroppedDockWidget->dockAreaWidget();
        if (OldDockArea == TargetArea)
        {
            return;
        }

        if (OldDockArea)
        {
            OldDockArea->removeDockWidget(DroppedDockWidget);
        }
        TargetArea->insertDockWidget(TabIndex, DroppedDockWidget, true);
    }
    //拖进来的是整个 CDockAreaWidget
    else
    {
        QList<CDockWidget*> NewDockWidgets = DroppedArea->dockWidgets();
        int NewCurrentIndex = DroppedArea->currentIndex();
        for (int i = 0; i < NewDockWidgets.count(); ++i)
        {
            CDockWidget* DockWidget = NewDockWidgets[i];
            TargetArea->insertDockWidget(TabIndex + i, DockWidget, false);
        }
        TargetArea->setCurrentIndex(TabIndex + NewCurrentIndex);
        DroppedArea->dockContainer()->removeDockArea(DroppedArea);
        DroppedArea->deleteLater();
    }

    TargetArea->updateTitleBarVisibility();
    return;
}
void DockContainerWidgetPrivate::moveToAutoHideSideBar(QWidget* Widget, DockWidgetArea area, int TabIndex)
{
    CDockWidget* DroppedDockWidget = qobject_cast<CDockWidget*>(Widget);
    CDockAreaWidget* DroppedDockArea = qobject_cast<CDockAreaWidget*>(Widget);
    auto SideBarLocation = internal::toSideBarLocation(area);

    if (DroppedDockWidget)
    {
        //拖进来的是单个 CDockWidget
        if (_this == DroppedDockWidget->dockContainer())
        {
            //如果这个 dock widget 已经在当前 container 里，那就不用跨 container 搬运，直接把它切换成 auto-hide 状态
            DroppedDockWidget->setAutoHide(true, SideBarLocation, TabIndex);
        }
        else
        {
            //跨 container 时，不能原地转，必须在当前 container 重新创建 auto-hide 宿主
            /*
            用 createAndSetupAutoHideContainer(...)
            因为它会帮你完成一整套动作：

            确保 dock widget 绑定到当前 manager
            找到当前 container 对应的 side bar
            创建 CAutoHideDockContainer
            创建 auto-hide tab
            加入当前边栏结构
            */
            _this->createAndSetupAutoHideContainer(SideBarLocation,
                                                   DroppedDockWidget, TabIndex);
        }
    }
    else
    {
        //拖进来的是整个 CDockAreaWidget
        if (_this == DroppedDockArea->dockContainer())
        {
            //如果整个 dock area 已经属于当前 container，那就直接把这个 area 切换成 auto-hide 状态。
            DroppedDockArea->setAutoHide(true, SideBarLocation, TabIndex);
        }
        else
        {
            //如果这个 dock area 原来属于别的 container，那么不能直接把整个 area 作为一个现成 auto-hide area 整体搬过来，
            // 而是要把其中“打开着的 dock widgets”逐个拆出来，在当前 container 重新创建 auto-hide 容器
            for (const auto DockWidget : DroppedDockArea->openedDockWidgets())
            {
                if (!DockWidget->features().testFlag(
                        CDockWidget::DockWidgetPinnable))
                {
                    continue;
                }

                _this->createAndSetupAutoHideContainer(SideBarLocation,
                                                       DockWidget, TabIndex++);
            }
        }
    }
}
void DockContainerWidgetPrivate::moveToContainer(QWidget* Widget, DockWidgetArea area)
{
    CDockWidget* DroppedDockWidget = qobject_cast<CDockWidget*>(Widget);
    CDockAreaWidget* DroppedDockArea = qobject_cast<CDockAreaWidget*>(Widget);
    CDockAreaWidget* NewDockArea;

    if (DroppedDockWidget)
    {
        //给这个单独的 dock widget 临时造一个区域壳子
        NewDockArea = new CDockAreaWidget(DockManager, _this);
        CDockAreaWidget* OldDockArea = DroppedDockWidget->dockAreaWidget();
        if (OldDockArea)
        {
            OldDockArea->removeDockWidget(DroppedDockWidget);
        }
        NewDockArea->addDockWidget(DroppedDockWidget);
    }
    else
    {
        // We check, if we insert the dropped widget into the same place that
        // it already has and do nothing, if it is the same place. It would
        // also work without this check, but it looks nicer with the check
        // because there will be no layout updates
        auto Splitter = DroppedDockArea->parentSplitter();
        auto InsertParam = internal::dockAreaInsertParameters(area);
        //如果这个 DroppedDockArea 其实已经就在你要放的位置上，那就什么都不做，直接返回
        if (Splitter == RootSplitter
            && InsertParam.orientation() == Splitter->orientation())
        {
            if (InsertParam.append() && Splitter->lastWidget() == DroppedDockArea)
            {
                return;
            }
            else if (!InsertParam.append()
                     && Splitter->firstWidget() == DroppedDockArea)
            {
                return;
            }
        }
        DroppedDockArea->dockContainer()->removeDockArea(DroppedDockArea);
        NewDockArea = DroppedDockArea;
    }

    addDockArea(NewDockArea, area);
    //把这个新插入的 area 记录成该方向上“最近添加的 area”
    LastAddedAreaCache[areaIdToIndex(area)] = NewDockArea;

}

void DockContainerWidgetPrivate::saveChildNodesState(QXmlStreamWriter& s, QWidget* Widget)
{
    QSplitter* Splitter = qobject_cast<QSplitter*>(Widget);
    if (Splitter)
    {
        s.writeStartElement("Splitter");
        s.writeAttribute("Orientation",
                         (Splitter->orientation() == Qt::Horizontal) ? "|" : "-");
        s.writeAttribute("Count", QString::number(Splitter->count()));
        ADS_PRINT("NodeSplitter orient: " << Splitter->orientation()
                                          << " WidgetCont: "
                                          << Splitter->count());
        for (int i = 0; i < Splitter->count(); ++i)
        {
            saveChildNodesState(s, Splitter->widget(i));
        }

        s.writeStartElement("Sizes");
        for (auto Size : Splitter->sizes())
        {
            s.writeCharacters(QString::number(Size) + " ");
        }
        s.writeEndElement();
        s.writeEndElement();
    }
    else
    {
        CDockAreaWidget* DockArea = qobject_cast<CDockAreaWidget*>(Widget);
        if (DockArea)
        {
            DockArea->saveState(s);
        }
    }
}
void DockContainerWidgetPrivate::saveAutoHideWidgetsState(QXmlStreamWriter& s)
{
    for (const auto sideTabBar : SideTabBarWidgets.values())
    {
        if (!sideTabBar->count())
        {
            continue;
        }

        sideTabBar->saveState(s);
    }
}
bool DockContainerWidgetPrivate::restoreChildNodes(CDockingStateReader& s,QWidget*& CreatedWidget, bool Testing)
{
    bool Result = true;
    while (s.readNextStartElement())
    {
        if (s.name() == QLatin1String("Splitter"))
        {
            Result = restoreSplitter(s, CreatedWidget, Testing);
            ADS_PRINT("Splitter");
        }
        else if (s.name() == QLatin1String("Area"))
        {
            Result = restoreDockArea(s, CreatedWidget, Testing);
            ADS_PRINT("DockAreaWidget");
        }
        else if (s.name() == QLatin1String("SideBar"))
        {
            Result = restoreSideBar(s, CreatedWidget, Testing);
            ADS_PRINT("SideBar");
        }
        else
        {
            s.skipCurrentElement();
            ADS_PRINT("Unknown element");
        }
    }

    return Result;
}
bool DockContainerWidgetPrivate::restoreSplitter(CDockingStateReader& s, QWidget*& CreatedWidget, bool Testing)
{
    /*
    恢复一个 <Splitter> 节点，以及它下面整棵 splitter 子树。

    它做的事情包括：

    读当前 splitter 的方向
    读它应有的 child 数量
    递归恢复每个 child
    child 可能还是 splitter
    也可能是 dock area
    读 <Sizes>，恢复各 child 的尺寸
    恢复 splitter 的可见性
    最后把恢复好的 splitter 通过 CreatedWidget 传回上层
    
    */

    bool Ok;
    QString OrientationStr = s.attributes().value("Orientation").toString();

    // Check if the orientation string is right
    if (!OrientationStr.startsWith("|") && !OrientationStr.startsWith("-"))
    {
        return false;
    }

    // The "|" shall indicate a vertical splitter handle which in turn means
    // a Horizontal orientation of the splitter layout.
    bool HorizontalSplitter = OrientationStr.startsWith("|");
    // In version 0 we had a small bug. The "|" indicated a vertical orientation,
    // but this is wrong, because only the splitter handle is vertical, the
    // layout of the splitter is a horizontal layout. We fix this here
    if (s.fileVersion() == 0)
    {
        //旧版本兼容修正
        HorizontalSplitter = !HorizontalSplitter;
    }

    int Orientation = HorizontalSplitter ? Qt::Horizontal : Qt::Vertical;
    //读取它保存时记录的 child 数量
    int WidgetCount = s.attributes().value("Count").toInt(&Ok);
    if (!Ok)
    {
        return false;
    }
    ADS_PRINT("Restore NodeSplitter Orientation: "
              << Orientation << " WidgetCount: " << WidgetCount);
    QSplitter* Splitter = nullptr;
    if (!Testing)
    {
        Splitter = newSplitter(static_cast<Qt::Orientation>(Orientation));
    }
    bool Visible = false;
    QList<int> Sizes;
    //当前 <Splitter> 节点下面，只要还有子节点，就不断读取
    while (s.readNextStartElement())
    {
        QWidget* ChildNode = nullptr;
        bool Result = true;
        if (s.name() == QLatin1String("Splitter"))
        {
            Result = restoreSplitter(s, ChildNode, Testing);
        }
        else if (s.name() == QLatin1String("Area"))
        {
            Result = restoreDockArea(s, ChildNode, Testing);
        }
        else if (s.name() == QLatin1String("Sizes"))
        {
            QString sSizes = s.readElementText().trimmed();
            ADS_PRINT("Sizes: " << sSizes);
            QTextStream TextStream(&sSizes);
            while (!TextStream.atEnd())
            {
                int value;
                TextStream >> value;
                Sizes.append(value);
            }
        }
        else
        {
            s.skipCurrentElement();
        }

        if (!Result)
        {
            return false;
        }

        if (Testing || !ChildNode)
        {
            continue;
        }

        ADS_PRINT("ChildNode isVisible " << ChildNode->isVisible()
                                         << " isVisibleTo "
                                         << ChildNode->isVisibleTo(Splitter));
        Splitter->addWidget(ChildNode);
        Visible |= ChildNode->isVisibleTo(Splitter);
    }
    if (!Testing)
    {
        //更新 splitter 的 stretch/handle 规则
        updateSplitterHandles(Splitter);
    }

    if (Sizes.count() != WidgetCount)
    {
        return false;
    }

    if (!Testing)
    {
        if (!Splitter->count())
        {
            delete Splitter;
            Splitter = nullptr;
        }
        else
        {
            Splitter->setSizes(Sizes);
            Splitter->setVisible(Visible);
        }
        CreatedWidget = Splitter;
    }
    else
    {
        CreatedWidget = nullptr;
    }

    return true;
}
bool DockContainerWidgetPrivate::restoreDockArea(CDockingStateReader& s, QWidget*& CreatedWidget, bool Testing)
{
    CDockAreaWidget* DockArea = nullptr;
    auto Result = CDockAreaWidget::restoreState(s, DockArea, Testing, _this);
    if (Result && DockArea)
    {
        appendDockAreas({DockArea});
    }
    CreatedWidget = DockArea;
    return Result;
}
bool DockContainerWidgetPrivate::restoreSideBar(CDockingStateReader& s, QWidget*& CreatedWidget, bool Testing)
{
    Q_UNUSED(CreatedWidget)
    // Simply ignore side bar auto hide widgets from saved state if
    // auto hide support is disabled
    if (!CDockManager::testAutoHideConfigFlag(
            CDockManager::AutoHideFeatureEnabled))
    {
        //如果整个 auto-hide 功能关闭了，就直接忽略 side bar 恢复
        return true;
    }

    //位置属性
    bool Ok;
    auto Area = (ads::SideBarLocation)s.attributes().value("Area").toInt(&Ok);
    if (!Ok)
    {
        return false;
    }

    while (s.readNextStartElement())
    {
        if (s.name() != QLatin1String("Widget"))
        {
            continue;
        }

        auto Name = s.attributes().value("Name");
        if (Name.isEmpty())
        {
            return false;
        }

        bool Ok;
        bool Closed = s.attributes().value("Closed").toInt(&Ok);
        if (!Ok)
        {
            return false;
        }

        int Size = s.attributes().value("Size").toInt(&Ok);
        if (!Ok)
        {
            return false;
        }
        //跳过当前 <Widget> 元素正文
        s.skipCurrentElement();
        //根据 Name 从 manager 全局表中找到真实 CDockWidget
        CDockWidget* DockWidget = DockManager->findDockWidget(Name.toString());
        if (!DockWidget || Testing)
        {
            continue;
        }

        auto SideBar = _this->autoHideSideBar(Area);
        CAutoHideDockContainer* AutoHideContainer;
        if (DockWidget->isAutoHide())
        {
            //如果这个 dock widget 当前已经在某个 auto-hide 容器里，那就尽量复用现有的 auto-hide 容器，而不是重新创建
            AutoHideContainer = DockWidget->autoHideDockContainer();
            if (AutoHideContainer->autoHideSideBar() != SideBar)
            {
                SideBar->addAutoHideWidget(AutoHideContainer);
            }
        }
        else
        {
            //如果这个 dock widget 当前并不在 auto-hide 状态，那就现在在当前 side bar 上新建一个 auto-hide 容器
            AutoHideContainer = SideBar->insertDockWidget(-1, DockWidget);
        }
        AutoHideContainer->setSize(Size);
        DockWidget->setProperty(internal::ClosedProperty, Closed);
        //清除 DirtyProperty：表示：这个 widget 已经被成功恢复，不要在后续收尾阶段把它当成未分配垃圾处理掉
        DockWidget->setProperty(internal::DirtyProperty, false);
    }

    return true;

}

//本质未被使用
void DockContainerWidgetPrivate::dumpRecursive(int level, QWidget* widget)
{
#if defined(QT_DEBUG)
    QSplitter* Splitter = qobject_cast<QSplitter*>(widget);
    QByteArray buf;
    buf.fill(' ', level * 4);
    if (Splitter)
    {
#    ifdef ADS_DEBUG_PRINT
        qDebug("%sSplitter %s v: %s c: %s", (const char*)buf,
               (Splitter->orientation() == Qt::Vertical) ? "--" : "|",
               Splitter->isHidden() ? " " : "v",
               QString::number(Splitter->count()).toStdString().c_str());
        std::cout << (const char*)buf << "Splitter "
                  << ((Splitter->orientation() == Qt::Vertical) ? "--" : "|")
                  << " " << (Splitter->isHidden() ? " " : "v") << " "
                  << QString::number(Splitter->count()).toStdString()
                  << std::endl;
#    endif
        for (int i = 0; i < Splitter->count(); ++i)
        {
            dumpRecursive(level + 1, Splitter->widget(i));
        }
    }
    else
    {
        CDockAreaWidget* DockArea = qobject_cast<CDockAreaWidget*>(widget);
        if (!DockArea)
        {
            return;
        }
#    ifdef ADS_DEBUG_PRINT
        qDebug("%sDockArea", (const char*)buf);
        std::cout << (const char*)buf << (DockArea->isHidden() ? " " : "v")
                  << (DockArea->openDockWidgetsCount() > 0 ? " " : "c")
                  << " DockArea "
                  << "[hs: " << DockArea->sizePolicy().horizontalStretch()
                  << ", vs: " << DockArea->sizePolicy().verticalStretch() << "]"
                  << std::endl;
        buf.fill(' ', (level + 1) * 4);
        for (int i = 0; i < DockArea->dockWidgetsCount(); ++i)
        {
            std::cout << (const char*)buf
                      << (i == DockArea->currentIndex() ? "*" : " ");
            CDockWidget* DockWidget = DockArea->dockWidget(i);
            std::cout << (DockWidget->isHidden() ? " " : "v");
            std::cout << (DockWidget->isClosed() ? "c" : " ") << " ";
            std::cout << DockWidget->windowTitle().toStdString() << std::endl;
        }
#    endif
    }
#else
    Q_UNUSED(level);
    Q_UNUSED(widget);
#endif
}
eDropMode DockContainerWidgetPrivate::getDropMode(const QPoint& TargetPos)
{
    CDockAreaWidget* DockArea = _this->dockAreaAt(TargetPos);
    auto dropArea = InvalidDockWidgetArea;
    auto ContainerDropArea =
        DockManager->containerOverlay()->dropAreaUnderCursor();

    if (DockArea)
    {
        auto dropOverlay = DockManager->dockAreaOverlay();
        dropOverlay->setAllowedAreas(DockArea->allowedAreas());
        dropArea = dropOverlay->showOverlay(DockArea);
        if (ContainerDropArea != InvalidDockWidgetArea
            && ContainerDropArea != dropArea)
        {
            dropArea = InvalidDockWidgetArea;
        }

        if (dropArea != InvalidDockWidgetArea)
        {
            ADS_PRINT("Dock Area Drop Content: " << dropArea);
            return DropModeIntoArea;
        }
    }

    // mouse is over container
    if (InvalidDockWidgetArea == dropArea)
    {
        dropArea = ContainerDropArea;
        ADS_PRINT("Container Drop Content: " << dropArea);
        if (dropArea != InvalidDockWidgetArea)
        {
            return DropModeIntoContainer;
        }
    }

    return DropModeInvalid;
}



/************************************************************************************************************************************************/

CDockContainerWidget::CDockContainerWidget(CDockManager* DockManager, QWidget *parent) :
	QFrame(parent),
	d(new DockContainerWidgetPrivate(this))
{
	d->DockManager = DockManager;
	d->isFloating = floatingWidget() != nullptr;

	d->Layout = new QGridLayout();
	d->Layout->setContentsMargins(0, 0, 0, 0);
	d->Layout->setSpacing(0);
	d->Layout->setColumnStretch(1, 1);
	d->Layout->setRowStretch(1, 1);
	setLayout(d->Layout);

	// The function d->newSplitter() accesses the config flags from dock
	// manager which in turn requires a properly constructed dock manager.
	// If this dock container is the dock manager, then it is not properly
	// constructed yet because this base class constructor is called before
	// the constructor of the DockManager private class
	if (DockManager != this)
	{
		d->DockManager->registerDockContainer(this);
		createRootSplitter();
		createSideTabBarWidgets();
	}
}
CDockContainerWidget::~CDockContainerWidget()
{
	if (d->DockManager)
	{
		d->DockManager->removeDockContainer(this);
	}

	delete d;
}
void CDockContainerWidget::createRootSplitter()
{
    if (d->RootSplitter)
    {
        return;
    }
    d->RootSplitter = d->newSplitter(Qt::Horizontal);
    d->Layout->addWidget(d->RootSplitter, 1, 1);  // Add it to the center - the 0
                                                  // and 2 indexes are used for
                                                  // the SideTabBar widgets
}
void CDockContainerWidget::createSideTabBarWidgets()
{
    if (!CDockManager::testAutoHideConfigFlag(
            CDockManager::AutoHideFeatureEnabled))
    {
        return;
    }

    {
        auto Area = SideBarLocation::SideBarLeft;
        d->SideTabBarWidgets[Area] = new CAutoHideSideBar(this, Area);
        d->Layout->addWidget(d->SideTabBarWidgets[Area], 1, 0);
    }

    {
        auto Area = SideBarLocation::SideBarRight;
        d->SideTabBarWidgets[Area] = new CAutoHideSideBar(this, Area);
        d->Layout->addWidget(d->SideTabBarWidgets[Area], 1, 2);
    }

    {
        auto Area = SideBarLocation::SideBarBottom;
        d->SideTabBarWidgets[Area] = new CAutoHideSideBar(this, Area);
        d->Layout->addWidget(d->SideTabBarWidgets[Area], 2, 1);
    }

    {
        auto Area = SideBarLocation::SideBarTop;
        d->SideTabBarWidgets[Area] = new CAutoHideSideBar(this, Area);
        d->Layout->addWidget(d->SideTabBarWidgets[Area], 0, 1);
    }
}
CDockSplitter* CDockContainerWidget::rootSplitter() const
{
    return d->RootSplitter;
}


// 添加布局主线
/*
把一个 CDockWidget 放进当前 container，并返回它最终所在的 CDockAreaWidget

如果 widget 原来已经在别的 area 里，要先摘出来
根据参数判断：
是相对整个 container 放
还是相对某个已有 DockArea 放
放完以后还要修正 top-level 状态
*/
CDockAreaWidget* CDockContainerWidget::addDockWidget(DockWidgetArea area, CDockWidget* Dockwidget, CDockAreaWidget* DockAreaWidget,int Index)
{
	
    /*
		当前 container 是否只有一个唯一可见的顶层 dock widget
		如果 container 当前只有一个唯一可见 dock widget
		那就返回它
		否则返回 nullptr

        添加新 dock 之后，container 的“唯一顶层 dock”状态可能会变化
	*/
    auto TopLevelDockWidget = topLevelDockWidget();
	//如果这个 dock 原来已经挂在某个 area 里，那先把它从原位置摘下来
    CDockAreaWidget* OldDockArea = Dockwidget->dockAreaWidget();
    if (OldDockArea)
    {
        OldDockArea->removeDockWidget(Dockwidget);
    }

	//无论这个 dock 之前属于谁，现在都明确归到当前 container 所属的 DockManager
    Dockwidget->setDockManager(d->DockManager);
    CDockAreaWidget* DockArea;
    if (DockAreaWidget)
    {
		//相对某个已有 DockArea 添加
        DockArea =
            d->addDockWidgetToDockArea(area, Dockwidget, DockAreaWidget, Index);
    }
    else
    {
		//相对整个 container 添加
        DockArea = d->addDockWidgetToContainer(area, Dockwidget);
    }

    if (TopLevelDockWidget)
    {
        auto NewTopLevelDockWidget = topLevelDockWidget();
        // If the container contained only one visible dock widget, the we need
        // to emit a top level event for this widget because it is not the one and
        // only visible docked widget anymore
        if (!NewTopLevelDockWidget)
        {
            /*
			只有在添加前存在 TopLevelDockWidget
			添加后又不存在 NewTopLevelDockWidget
			才说明“唯一状态被打破了”
			*/
            CDockWidget::emitTopLevelEventForWidget(TopLevelDockWidget, false);
        }
    }
    return DockArea;
}
void CDockContainerWidget::removeDockWidget(CDockWidget* Dockwidget)
{
    CDockAreaWidget* Area = Dockwidget->dockAreaWidget();
    if (Area)
    {
        Area->removeDockWidget(Dockwidget);
    }
}
CAutoHideDockContainer* CDockContainerWidget::createAndSetupAutoHideContainer( SideBarLocation area, CDockWidget* DockWidget, int TabIndex)
{
    if (!CDockManager::testAutoHideConfigFlag(
            CDockManager::AutoHideFeatureEnabled))
    {
        Q_ASSERT_X(
            false,
            "CDockContainerWidget::createAndInitializeDockWidgetOverlayContainer",
            "Requested area does not exist in config");
        return nullptr;
    }
    if (d->DockManager != DockWidget->dockManager())
    {
        DockWidget->setDockManager(d->DockManager);  // Auto hide Dock Container
                                                     // needs a valid dock manager
    }

    return autoHideSideBar(area)->insertDockWidget(TabIndex, DockWidget);
}
//按照给定方向 area，把一个新的 CDockAreaWidget 插入到当前 container 的根 splitter 布局树里
void CDockContainerWidget::addDockArea(CDockAreaWidget* DockAreaWidget, DockWidgetArea area)
{
    CDockContainerWidget* Container = DockAreaWidget->dockContainer();
    if (Container && Container != this)
    {
        Container->removeDockArea(DockAreaWidget);
    }

    d->addDockArea(DockAreaWidget, area);
}
//把一个 CDockAreaWidget 从当前 container 的布局树里安全拆出去，并在必要时压缩/重组 splitter 树，最后修正 top-level 状态
void CDockContainerWidget::removeDockArea(CDockAreaWidget* area)
{
    ADS_PRINT("CDockContainerWidget::removeDockArea");
    // 如果这是 auto-hide area，走特殊快速路径
    if (area->isAutoHide())
    {
        area->setAutoHideDockContainer(nullptr);
        return;
    }

    area->disconnect(this);
    d->DockAreas.removeAll(area);
    auto Splitter = area->parentSplitter();

	/*
	因为某些 area 或其中的内容控件，可能已经变成了 native window。
	比如：
	OpenGL
	VTK
	某些特殊渲染控件
	其他调用过 winId() 的内容
	这类控件即使 WA_NativeWindow 不一定显式为 true，也可能已经拿到了真实系统窗口句柄。

    如果对这种 native window 直接 setParent(nullptr) 会怎样
	它可能不会单纯变成“悬空子控件”，而是变成：
	一个不可见但真实存在的顶层 OS 窗口。
	这会带来：
	绘制残影
	黑块
	窗口闪烁
	看不见的悬挂 native window
	*/

	//安全地把 area 从原布局树中脱钩，同时避免 native window 副作用
    if (area->internalWinId())
        area->setParent(d->DockManager);
    else
        area->setParent(nullptr);
	//向上隐藏空 splitter 链
    internal::hideEmptyParentSplitters(Splitter);

    // Remove this area from cached areas
    auto p = std::find(std::begin(d->LastAddedAreaCache),
                       std::end(d->LastAddedAreaCache), area);
    if (p != std::end(d->LastAddedAreaCache))
    {
        *p = nullptr;
    }

    // If splitter has more than 1 widgets, we are finished and can leave
    if (Splitter->count() > 1)
    {
        goto emitAndExit;
    }

    // If this is the RootSplitter we need to remove empty splitters to
    // avoid too many empty splitters
    if (Splitter == d->RootSplitter)
    {
        ADS_PRINT("Removed from RootSplitter");
        // If splitter is empty, we are finished
        if (!Splitter->count())
        {
            Splitter->hide();
            goto emitAndExit;
        }

        QWidget* widget = Splitter->widget(0);
        auto ChildSplitter = qobject_cast<CDockSplitter*>(widget);
        // If the one and only content widget of the splitter is not a splitter
        // then we are finished
        if (!ChildSplitter)
        {
            goto emitAndExit;
        }

        // We replace the superfluous RootSplitter with the ChildSplitter
        ChildSplitter->setParent(nullptr);
        QLayoutItem* li = d->Layout->replaceWidget(Splitter, ChildSplitter);
        d->RootSplitter = ChildSplitter;
        delete li;
        ADS_PRINT("RootSplitter replaced by child splitter");
    }
    else if (Splitter->count() == 1)
    {
		//父 splitter 不再挂这个空壳 Splitter，而是直接挂它唯一剩下的 child
        ADS_PRINT("Replacing splitter with content");
        QSplitter* ParentSplitter = internal::findParent<QSplitter*>(Splitter);
        auto Sizes = ParentSplitter->sizes();
        QWidget* widget = Splitter->widget(0);
        widget->setParent(this);
        internal::replaceSplitterWidget(ParentSplitter, Splitter, widget);
        ParentSplitter->setSizes(Sizes);
    }

    delete Splitter;
    Splitter = nullptr;

emitAndExit:
    updateSplitterHandles(Splitter);
    CDockWidget* TopLevelWidget = topLevelDockWidget();

    // 重新发事件通知
    CDockWidget::emitTopLevelEventForWidget(TopLevelWidget, true);
    dumpLayout();
    d->emitDockAreasRemoved();
}
QList<QPointer<CDockAreaWidget>> CDockContainerWidget::removeAllDockAreas()
{
    auto Result = d->DockAreas;
    d->DockAreas.clear();
    return Result;
}


// 拖放与移动主线
void CDockContainerWidget::dropFloatingWidget(CFloatingDockContainer* FloatingWidget, const QPoint& TargetPos)
{
    ADS_PRINT("CDockContainerWidget::dropFloatingWidget");
	//记录两个“可能受 top-level 状态影响”的 dock widget
    CDockWidget* SingleDroppedDockWidget = FloatingWidget->topLevelDockWidget();
    CDockWidget* SingleDockWidget = topLevelDockWidget();

	//相对于某个具体 DockArea 的落区结果：只有在鼠标真正落在某个 area 上，并且 overlay 判断出有效区域时，它才会变成：Left / Right / Top / Bottom / Center
    auto dropArea = InvalidDockWidgetArea;
	//相对于整个 container 外层 overlay 的落区结果
    auto ContainerDropArea =
        d->DockManager->containerOverlay()->dropAreaUnderCursor();
    bool Dropped = false;
	//判断鼠标当前是不是落在某个现有 DockArea 上
    CDockAreaWidget* DockArea = dockAreaAt(TargetPos);
    // 优先按 DockArea 模式处理
    if (DockArea)
    {
        auto dropOverlay = d->DockManager->dockAreaOverlay();
		//设置这个 area 允许的落区
        dropOverlay->setAllowedAreas(DockArea->allowedAreas());
		//显示该 DockArea 的 overlay 高亮，返回当前鼠标对应的是哪个区域，如果没命中有效区域，则返回InvalidDockWidgetArea
        dropArea = dropOverlay->showOverlay(DockArea);
		//同时给出了结果，但结果不一致，那么 area 级落点作废
        if (ContainerDropArea != InvalidDockWidgetArea
            && ContainerDropArea != dropArea)
        {
            dropArea = InvalidDockWidgetArea;
        }

        if (dropArea != InvalidDockWidgetArea)
        {
            ADS_PRINT("Dock Area Drop Content: " << dropArea);
			//拿到光标下的 tab 索引
            int TabIndex =
                d->DockManager->dockAreaOverlay()->tabIndexUnderCursor();
			//把 floating 容器里的内容，落到某个具体 dock area 上
            d->dropIntoSection(FloatingWidget, DockArea, dropArea, TabIndex);
            Dropped = true;
        }
    }

    //area 级 drop 没有成功但 container 级 overlay 给出了有效结果
    if (InvalidDockWidgetArea == dropArea
        && InvalidDockWidgetArea != ContainerDropArea)
    {
		//当前鼠标落在的是 auto-hide side bar 区
        if (internal::isSideBarArea(ContainerDropArea))
        {
            ADS_PRINT("Container Drop Content: " << ContainerDropArea);
            d->dropIntoAutoHideSideBar(FloatingWidget, ContainerDropArea);
        }
		//当前鼠标落在的是整个 container 的左/右/上/下/中区域
        else
        {
            ADS_PRINT("Container Drop Content: " << ContainerDropArea);
            d->dropIntoContainer(FloatingWidget, ContainerDropArea);
        }
        Dropped = true;
    }

    // 把 floating 容器里的 auto-hide widgets 迁移过来
    for (auto AutohideWidget : FloatingWidget->dockContainer()->autoHideWidgets())
    {
		//dropFloatingWidget(...) 并不是“鼠标每移动一下就调用一次”的试探函数，已经决定把这个 floating widget drop 到当前 container 时，真正执行落地
        /*可以理解为进入这个函数，逻辑上Dropped就应该一直是true的*/
        auto SideBar = autoHideSideBar(AutohideWidget->sideBarLocation());
        SideBar->addAutoHideWidget(AutohideWidget);
    }

    if (Dropped)
    {
        // Fix
        // https://github.com/githubuser0xFFFF/Qt-Advanced-Docking-System/issues/351

		//正式结束 floating widget 的拖放流程
        /*
		清理拖放状态
		关闭/销毁不再需要的浮动壳
		完成内部转移后的收尾
		*/
        FloatingWidget->finishDropOperation();

		//更新 top-level widget 状态
        CDockWidget::emitTopLevelEventForWidget(SingleDroppedDockWidget, false);
        CDockWidget::emitTopLevelEventForWidget(SingleDockWidget, false);
    }
	//拖放完成后，把当前窗口重新激活到前台
    window()->activateWindow();
    if (SingleDroppedDockWidget)
    {
		//通知 manager 有 widget/area 发生了重定位
        d->DockManager->notifyWidgetOrAreaRelocation(SingleDroppedDockWidget);
    }
	//通知 manager 整个 floating widget 完成 drop
    d->DockManager->notifyFloatingWidgetDrop(FloatingWidget);
}
CDockAreaWidget* CDockContainerWidget::dockAreaAt(const QPoint& GlobalPos) const
{
    for (const auto& DockArea : d->DockAreas)
    {
        if (DockArea && DockArea->isVisible()
            && DockArea->rect().contains(DockArea->mapFromGlobal(GlobalPos)))
        {
            return DockArea;
        }
    }

    return nullptr;
}
void CDockContainerWidget::dropWidget(QWidget* Widget, DockWidgetArea DropArea, CDockAreaWidget* TargetAreaWidget, int TabIndex)
{
	//在真正执行 drop 之前，先记住当前 container 里是否存在唯一的 top-level dock widget
    CDockWidget* SingleDockWidget = topLevelDockWidget();
    if (TargetAreaWidget)
    {
		//当前 drop 是相对某个已有 DockArea 做的
        d->moveToNewSection(Widget, TargetAreaWidget, DropArea, TabIndex);
    }
    else if (internal::isSideBarArea(DropArea))
    {
        /*
		* 根据枚举即可判断当前落点区域是不是在侧边栏
			LeftAutoHideArea = 0x20,
			RightAutoHideArea = 0x40,
			TopAutoHideArea = 0x80,
			BottomAutoHideArea = 0x100,
		*/
		//当前 drop 不是落在具体 DockArea 上，而是落在某条 auto-hide 侧边栏区域上
        d->moveToAutoHideSideBar(Widget, DropArea, TabIndex);
    }
    else
    {
		//相对整个 container 的落地
        d->moveToContainer(Widget, DropArea);
    }


	//修正原 top-level widget 的状态
    CDockWidget::emitTopLevelEventForWidget(SingleDockWidget, false);
	//drop 完后把当前窗口激活
    window()->activateWindow();
	//通知 manager：有 widget 或 area 发生了重定位
    d->DockManager->notifyWidgetOrAreaRelocation(Widget);
}


// 保存/恢复状态
void CDockContainerWidget::saveState(QXmlStreamWriter& s) const
{
    ADS_PRINT("CDockContainerWidget::saveState isFloating " << isFloating());

    /*
    保存的内容主要有三类：

    1，这个 container 是不是 floating
    2，如果是 floating，它的窗口几何信息
    3，它内部的布局树
        root splitter
        子 splitter
        dock area
        auto-hide side bar
    
    */

    s.writeStartElement("Container");
    s.writeAttribute("Floating", QString::number(isFloating() ? 1 : 0));
    if (isFloating())
    {
        // floating container 需要额外保存 geometry
        CFloatingDockContainer* FloatingWidget = floatingWidget();
        QByteArray Geometry = FloatingWidget->saveGeometry();

        s.writeTextElement("Geometry", Geometry.toHex(' '));
    }
    //从当前 container 的 RootSplitter 开始，递归把内部布局树保存下来
    d->saveChildNodesState(s, d->RootSplitter);
    //把当前 container 四条 side bar 上的 auto-hide widgets 状态也写进 XML
    d->saveAutoHideWidgetsState(s);
    s.writeEndElement();

}
bool CDockContainerWidget::restoreState(CDockingStateReader& s, bool Testing)
{
    /*
    把当前 XML 读取器所在位置的一个 <Container> 节点，恢复成当前 CDockContainerWidget 的内部布局。

    它恢复的内容包括：

    当前 container 是否是 floating
    如果是 floating，恢复它的窗口 geometry
    恢复 root splitter 及其整棵子树
    恢复其中的 dock areas
    恢复 side bar / auto-hide 结构（这个过程在 restoreChildNodes(...) 下层链路里继续展开）
    */
    bool IsFloating = s.attributes().value("Floating").toInt();
    ADS_PRINT("Restore CDockContainerWidget Floating" << IsFloating);

    //Testing == true   只验证 XML 结构是否合法，不真正修改当前 UI 布局
    QWidget* NewRootSplitter{};
    if (!Testing)
    {
        //如果是真实恢复，先清空和重置当前 container 的内部状态
        d->VisibleDockAreaCount = -1;  // invalidate the dock area count
        d->DockAreas.clear();
        std::fill(std::begin(d->LastAddedAreaCache),
                  std::end(d->LastAddedAreaCache), nullptr);
    }

    if (IsFloating)
    {
        //如果当前 <Container> 是 floating，就先恢复 Geometry
        ADS_PRINT("Restore floating widget");
        if (!s.readNextStartElement() || s.name() != QLatin1String("Geometry"))
        {
            return false;
        }

        QByteArray GeometryString =
            s.readElementText(CDockingStateReader::ErrorOnUnexpectedElement)
                .toLocal8Bit();
        QByteArray Geometry = QByteArray::fromHex(GeometryString);
        if (Geometry.isEmpty())
        {
            return false;
        }

        if (!Testing)
        {
            //如果当前这个 container 是 floating container，那它外层 CFloatingDockContainer 应该已经存在
            CFloatingDockContainer* FloatingWidget = floatingWidget();
            if (FloatingWidget)
            {
                FloatingWidget->restoreGeometry(Geometry);
            }
        }
    }

    //恢复 container 内部子布局树
    if (!d->restoreChildNodes(s, NewRootSplitter, Testing))
    {
        return false;
    }

    if (Testing)
    {
        return true;
    }

    // If the root splitter is empty, rostoreChildNodes returns a 0 pointer
    // and we need to create a new empty root splitter
    if (!NewRootSplitter)
    {
        //如果恢复出来的 root splitter 是空的，就补一个新的空 root
        NewRootSplitter = d->newSplitter(Qt::Horizontal);
    }

    QLayoutItem* li = d->Layout->replaceWidget(d->RootSplitter, NewRootSplitter);
    auto OldRoot = d->RootSplitter;
    d->RootSplitter = qobject_cast<CDockSplitter*>(NewRootSplitter);
    OldRoot->deleteLater();
    delete li;

    return true;
}


// dock area/dock widget 查询接口
CDockAreaWidget* CDockContainerWidget::dockArea(int Index) const
{
    return (Index < dockAreaCount()) ? d->DockAreas[Index] : nullptr;
}
QList<CDockAreaWidget*> CDockContainerWidget::openedDockAreas() const
{
    QList<CDockAreaWidget*> Result;
    for (auto DockArea : d->DockAreas)
    {
        if (DockArea && !DockArea->isHidden())
        {
            Result.append(DockArea);
        }
    }

    return Result;
}
QList<CDockWidget*> CDockContainerWidget::openedDockWidgets() const
{
    QList<CDockWidget*> DockWidgetList;
    for (auto DockArea : d->DockAreas)
    {
        if (DockArea && !DockArea->isHidden())
        {
            DockWidgetList.append(DockArea->openedDockWidgets());
        }
    }

    return DockWidgetList;
}
bool CDockContainerWidget::hasOpenDockAreas() const
{
    for (auto DockArea : d->DockAreas)
    {
        if (DockArea && !DockArea->isHidden())
        {
            return true;
        }
    }

    return false;
}
int CDockContainerWidget::dockAreaCount() const
{
    return d->DockAreas.count();
}
int CDockContainerWidget::visibleDockAreaCount() const
{
    int Result = 0;
    for (auto DockArea : d->DockAreas)
    {
        Result += (!DockArea || DockArea->isHidden()) ? 0 : 1;
    }

    return Result;

    // TODO Cache or precalculate this to speed it up because it is used during
    // movement of floating widget
    // return d->visibleDockAreaCount();
}
CDockAreaWidget* CDockContainerWidget::lastAddedDockAreaWidget( DockWidgetArea area) const
{
    return d->LastAddedAreaCache[areaIdToIndex(area)];
}


// top-level/floating/container相关
bool CDockContainerWidget::isFloating() const
{
    return d->isFloating;
}
CFloatingDockContainer* CDockContainerWidget::floatingWidget() const
{
    return internal::findParent<CFloatingDockContainer*>(this);
}
bool CDockContainerWidget::hasTopLevelDockWidget() const
{
    auto DockAreas = openedDockAreas();
    if (DockAreas.count() != 1)
    {
        return false;
    }

    return DockAreas[0]->openDockWidgetsCount() == 1;
}
CDockManager* CDockContainerWidget::dockManager() const
{
    return d->DockManager;
}
CDockWidget* CDockContainerWidget::topLevelDockWidget() const
{
    auto TopLevelDockArea = topLevelDockArea();
    if (!TopLevelDockArea)
    {
        return nullptr;
    }

    auto DockWidgets = TopLevelDockArea->openedDockWidgets();
    if (DockWidgets.count() != 1)
    {
        return nullptr;
    }

    return DockWidgets[0];
}
CDockAreaWidget* CDockContainerWidget::topLevelDockArea() const
{
    auto DockAreas = openedDockAreas();
    if (DockAreas.count() != 1)
    {
        return nullptr;
    }

    return DockAreas[0];
}
QList<CDockWidget*> CDockContainerWidget::dockWidgets() const
{
    QList<CDockWidget*> Result;
    for (const auto& DockArea : d->DockAreas)
    {
        if (!DockArea)
        {
            continue;
        }
        Result.append(DockArea->dockWidgets());
    }

    return Result;
}


// auto-hide相关接口
CAutoHideSideBar* CDockContainerWidget::autoHideSideBar(SideBarLocation area) const
{
    return d->SideTabBarWidgets[area];
}
QList<CAutoHideDockContainer*> CDockContainerWidget::autoHideWidgets() const
{
    return d->AutoHideWidgets;
}
void CDockContainerWidget::registerAutoHideWidget(CAutoHideDockContainer* AutohideWidget)
{
    d->AutoHideWidgets.append(AutohideWidget);
    Q_EMIT autoHideWidgetCreated(AutohideWidget);
    ADS_PRINT("d->AutoHideWidgets.count() " << d->AutoHideWidgets.count());
}
void CDockContainerWidget::removeAutoHideWidget(CAutoHideDockContainer* AutohideWidget)
{
    d->AutoHideWidgets.removeAll(AutohideWidget);
}
void CDockContainerWidget::handleAutoHideWidgetEvent(QEvent* e, QWidget* w)
{
    //统一处理 auto-hide 标签和 auto-hide 弹出容器的鼠标进入/离开事件，用于触发延迟展开和延迟收起。
    //CAutoHideTab   CAutoHideDockContainer收到 Enter/Leave，会转发给 CDockContainerWidget::handleAutoHideWidgetEvent(...)

    //总开关判断
    if (!CDockManager::testAutoHideConfigFlag(
            CDockManager::AutoHideShowOnMouseOver))
    {
        return;
    }

    //如果现在正处在 restoreState() 恢复界面状态阶段，就不响应这些 hover 事件
    if (dockManager()->isRestoringState())
    {
        return;
    }

    //如果当前事件来自“侧边标签”，就按标签逻辑处理
    auto AutoHideTab = qobject_cast<CAutoHideTab*>(w);
    if (AutoHideTab)
    {
        switch (e->type())
        {
        case QEvent::Enter:
            if (!AutoHideTab->dockWidget()->isVisible())
            {
                d->DelayedAutoHideTab = AutoHideTab;
                d->DelayedAutoHideShow = true;
                d->DelayedAutoHideTimer.start();
            }
            else
            {
                d->DelayedAutoHideTimer.stop();
            }
            break;

        case QEvent::MouseButtonPress: d->DelayedAutoHideTimer.stop(); break;

        case QEvent::Leave:
            if (AutoHideTab->dockWidget()->isVisible())
            {
                d->DelayedAutoHideTab = AutoHideTab;
                d->DelayedAutoHideShow = false;
                d->DelayedAutoHideTimer.start();
            }
            else
            {
                d->DelayedAutoHideTimer.stop();
            }
            break;

        default: break;
        }
        return;
    }

    //这段处理的是“弹出的内容面板壳子”
    auto AutoHideContainer = qobject_cast<CAutoHideDockContainer*>(w);
    if (AutoHideContainer)
    {
        switch (e->type())
        {
        case QEvent::Enter:
        case QEvent::Hide: d->DelayedAutoHideTimer.stop(); break;

        case QEvent::Leave:
            if (AutoHideContainer->isVisible())
            {
                d->DelayedAutoHideTab = AutoHideContainer->autoHideTab();
                d->DelayedAutoHideShow = false;
                d->DelayedAutoHideTimer.start();
            }
            break;

        default: break;
        }
        return;
        return;
    }


}


// 容器运行时机制
bool CDockContainerWidget::event(QEvent* e)
{
    //全局有一个递增计数器 zOrderCounter，谁最新被激活，谁就拿到更大的序号，序号越大，表示它越“靠前”、越“新近活跃”
    /*
    容器A 激活 -> zOrderIndex = 1
    容器B 激活 -> zOrderIndex = 2
    容器A 再次激活 -> zOrderIndex = 3
    */
    bool Result = QWidget::event(e);
    if (e->type() == QEvent::WindowActivate)
    {
        d->zOrderIndex = ++zOrderCounter;
    }
    else if (e->type() == QEvent::Show && !d->zOrderIndex)
    {
        d->zOrderIndex = ++zOrderCounter;
    }

    return Result;
}
unsigned int CDockContainerWidget::zOrderIndex() const
{
    return d->zOrderIndex;
}
bool CDockContainerWidget::isInFrontOf(CDockContainerWidget* Other) const
{
    return this->zOrderIndex() > Other->zOrderIndex();
}


// 容器状态与能力查询
void CDockContainerWidget::updateSplitterHandles(QSplitter* splitter)
{
    d->updateSplitterHandles(splitter);
}
QRect CDockContainerWidget::contentRect() const
{
    //当前 CDockContainerWidget 内部“内容区”的矩形范围
    /*
    这个“内容区”不是整个 container 外框，而是：

    去掉 auto-hide side bar 后
    真正用于放 dock 内容的那块区域
    */
    if (!d->RootSplitter)
    {
        return QRect();
    }

    if (d->RootSplitter->hasVisibleContent())
    {
        return d->RootSplitter->geometry();
    }
    else
    {
        auto ContentRect = this->rect();
        ContentRect.adjust(autoHideSideBar(SideBarLeft)->sizeHint().width(),
                           autoHideSideBar(SideBarTop)->sizeHint().height(),
                           -autoHideSideBar(SideBarRight)->sizeHint().width(),
                           -autoHideSideBar(SideBarBottom)->sizeHint().height());

        return ContentRect;
    }
}
QRect CDockContainerWidget::contentRectGlobal() const
{
    //内容区在全局屏幕坐标系下的矩形
    if (!d->RootSplitter)
    {
        return QRect();
    }
    return internal::globalGeometry(d->RootSplitter);
}
CDockWidget::DockWidgetFeatures CDockContainerWidget::features() const
{
    //把当前 CDockContainerWidget 里所有 DockArea 的能力做一次“交集汇总”，算出整个容器共同支持哪些功能
    CDockWidget::DockWidgetFeatures Features(CDockWidget::AllDockWidgetFeatures);
    for (const auto& DockArea : d->DockAreas)
    {
        if (!DockArea)
        {
            continue;
        }
        Features &= DockArea->features();
    }

    return Features;
}
void CDockContainerWidget::closeOtherAreas(CDockAreaWidget* KeepOpenArea)
{
    //关闭当前 container 里除 KeepOpenArea 之外的其他 dock area，但只关闭“允许关闭且不会触发自定义关闭处理风险”的 area。
    for (const auto& DockArea : d->DockAreas)
    {
        if (!DockArea || DockArea == KeepOpenArea)
        {
            continue;
        }

        //求 area 内所有 dock widget 特性的交集
        if (!DockArea->features(BitwiseAnd)
                 .testFlag(CDockWidget::DockWidgetClosable))
        {
            /*
            如果交集里还保留了 DockWidgetClosable
            说明这个 area 里的所有 dock widget 都可关闭
            才允许继续往下执行关闭
            如果没有，就 continue
            */
            continue;
        }

        // We do not close areas with widgets with custom close handling
        // 
        //BitwiseOr：求 area 内所有 dock widget 特性的并集
        if (DockArea->features(BitwiseOr).testFlag(
                CDockWidget::CustomCloseHandling))
        {
            //只要这个 area 里有任意一个 dock widget 开启了 CustomCloseHandling，整个 area 都不关
            continue;
        }

        DockArea->closeArea();
    }
}
void CDockContainerWidget::dumpLayout()
{
    //把当前 CDockContainerWidget 内部的布局树打印出来，方便开发者查看 splitter / dock area 的嵌套结构
#if (ADS_DEBUG_LEVEL > 0)
    qDebug("\n\nDumping layout --------------------------");
    std::cout << "\n\nDumping layout --------------------------" << std::endl;
    d->dumpRecursive(0, d->RootSplitter);
    qDebug("--------------------------\n\n");
    std::cout << "--------------------------\n\n" << std::endl;
#endif
}
void CDockContainerWidget::removeFromDockManager()
{
    if (d->DockManager)
    {
        d->DockManager->removeDockContainer(this);
        d->DockManager.clear();
    }
}




} // namespace ads

//---------------------------------------------------------------------------
// EOF DockContainerWidget.cpp
