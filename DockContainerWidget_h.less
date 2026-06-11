#ifndef DockContainerWidgetH
#define DockContainerWidgetH
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
/// \file   DockContainerWidget.h
/// \author Uwe Kindler
/// \date   24.02.2017
/// \brief  Declaration of CDockContainerWidget class
//============================================================================


//============================================================================
//                                   INCLUDES
//============================================================================
#include <QFrame>

#include "ads_globals.h"
#include "AutoHideTab.h"
#include "DockWidget.h"

QT_FORWARD_DECLARE_CLASS(QXmlStreamWriter)


namespace ads
{
class DockContainerWidgetPrivate;
class CDockAreaWidget;
class CDockWidget;
class CDockManager;
struct DockManagerPrivate;
class CFloatingDockContainer;
struct FloatingDockContainerPrivate;
class CFloatingDragPreview;
struct FloatingDragPreviewPrivate;
class CDockingStateReader;
class CAutoHideSideBar;
class CAutoHideTab;
class CDockSplitter;
struct AutoHideTabPrivate;
struct AutoHideDockContainerPrivate;


/*

+----------+-----------------------+----------+
|  (0,0)   |       (0,1)           |  (0,2)   |
|  empty   |   Top SideBar         |  empty   |
+----------+-----------------------+----------+
|  (1,0)   |       (1,1)           |  (1,2)   |
| LeftBar  |   RootSplitter        | RightBar |
+----------+-----------------------+----------+
|  (2,0)   |       (2,1)           |  (2,2)   |
|  empty   |  Bottom SideBar       |  empty   |
+----------+-----------------------+----------+

*/

class ADS_EXPORT CDockContainerWidget : public QFrame
{
	Q_OBJECT
private:
	DockContainerWidgetPrivate* d; ///< private data (pimpl)
    friend class DockContainerWidgetPrivate;
	friend class CDockManager;
	friend struct DockManagerPrivate;
	friend class CDockAreaWidget;
	friend struct DockAreaWidgetPrivate;
	friend class CFloatingDockContainer;
	friend struct FloatingDockContainerPrivate;
	friend class CDockWidget;
	friend class CFloatingDragPreview;
	friend struct FloatingDragPreviewPrivate;
	friend CAutoHideDockContainer;
	friend CAutoHideTab;
	friend AutoHideTabPrivate;
	friend AutoHideDockContainerPrivate;
	friend CAutoHideSideBar;

	//构造初始化
public:
    CDockContainerWidget(CDockManager* DockManager, QWidget* parent = 0);
    ~CDockContainerWidget() override;
protected:
    void createRootSplitter();
    void createSideTabBarWidgets();
    CDockSplitter* rootSplitter() const;

	//添加布局主线
public:
    CDockAreaWidget* addDockWidget(DockWidgetArea area, CDockWidget* Dockwidget, CDockAreaWidget* DockAreaWidget = nullptr,int Index = -1);
    void removeDockWidget(CDockWidget* Dockwidget);
protected:
    CAutoHideDockContainer* createAndSetupAutoHideContainer( SideBarLocation area, CDockWidget* DockWidget, int TabIndex = -1);
    void addDockArea(CDockAreaWidget* DockAreaWidget, DockWidgetArea area = ads::CenterDockWidgetArea);
    void removeDockArea(CDockAreaWidget* area);
    QList<QPointer<CDockAreaWidget>> removeAllDockAreas();

	//拖放与移动主线
    void dropFloatingWidget(CFloatingDockContainer* FloatingWidget, const QPoint& TargetPos);
    void dropWidget(QWidget* Widget, DockWidgetArea DropArea, CDockAreaWidget* TargetAreaWidget, int TabIndex = -1);
public:
    CDockAreaWidget* dockAreaAt(const QPoint& GlobalPos) const;

	//保存/恢复状态
protected:
    void saveState(QXmlStreamWriter& Stream) const;
    bool restoreState(CDockingStateReader& Stream, bool Testing);

	//dock area/dock widget 查询接口
public:
    CDockAreaWidget* dockArea(int Index) const;
    QList<CDockAreaWidget*> openedDockAreas() const;
    QList<CDockWidget*> openedDockWidgets() const;
    bool hasOpenDockAreas() const;
    int dockAreaCount() const;
    int visibleDockAreaCount() const;
protected:
    CDockAreaWidget* lastAddedDockAreaWidget(DockWidgetArea area) const;

	//top-level/floating/container相关
public:
    bool isFloating() const;
    CFloatingDockContainer* floatingWidget() const;
    bool hasTopLevelDockWidget() const;
    CDockManager* dockManager() const;
protected:
    CDockWidget* topLevelDockWidget() const;
    CDockAreaWidget* topLevelDockArea() const;
    QList<CDockWidget*> dockWidgets() const;

	//auto-hide相关接口
public:
    CAutoHideSideBar* autoHideSideBar(SideBarLocation area) const;
    QList<CAutoHideDockContainer*> autoHideWidgets() const;
protected:
    void registerAutoHideWidget(CAutoHideDockContainer* AutoHideWidget);
    void removeAutoHideWidget(CAutoHideDockContainer* AutoHideWidget);
    void handleAutoHideWidgetEvent(QEvent* e, QWidget* w);

	//容器运行时机制
    virtual bool event(QEvent* e) override;
public:
    virtual unsigned int zOrderIndex() const;
    bool isInFrontOf(CDockContainerWidget* Other) const;

	//容器状态与能力查询
protected:
    void updateSplitterHandles(QSplitter* splitter);
public:
    QRect contentRect() const;
    QRect contentRectGlobal() const;
    CDockWidget::DockWidgetFeatures features() const;
    void closeOtherAreas(CDockAreaWidget* KeepOpenArea);
    void dumpLayout();
private Q_SLOTS:
    void removeFromDockManager();

Q_SIGNALS:
    void dockAreasAdded();
    void dockAreasRemoved();
    void autoHideWidgetCreated(ads::CAutoHideDockContainer* AutoHideWidget);
    void dockAreaViewToggled(ads::CDockAreaWidget* DockArea, bool Open);



}; // class DockContainerWidget
} // namespace ads
//-----------------------------------------------------------------------------
#endif // DockContainerWidgetH
