
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
/// \file   MainWindow.cpp
/// \author Uwe Kindler
/// \date   13.02.2018
/// \brief  Implementation of CMainWindow demo class
//============================================================================


//============================================================================
//                                   INCLUDES
//============================================================================
#include <MainWindow.h>
#include "ui_mainwindow.h"

#include <iostream>

#include <QTime>
#include <QLabel>
#include <QTextEdit>
#include <QCalendarWidget>
#include <QFrame>
#include <QTreeView>
#include <QFileSystemModel>
#include <QBoxLayout>
#include <QSettings>
#include <QDockWidget>
#include <QDebug>
#include <QResizeEvent>
#include <QAction>
#include <QWidgetAction>
#include <QComboBox>
#include <QInputDialog>
#include <QRubberBand>
#include <QPlainTextEdit>
#include <QTableWidget>
#include <QScreen>
#include <QStyle>
#include <QMessageBox>
#include <QMenu>
#include <QToolButton>
#include <QToolBar>
#include <QPointer>
#include <QMap>
#include <QElapsedTimer>
#include <QQuickWidget>


#if QT_VERSION >= QT_VERSION_CHECK(5, 10, 0)
#include <QRandomGenerator>
#endif

#ifdef Q_OS_WIN
#if QT_VERSION < QT_VERSION_CHECK(6, 0, 0)
#include <QAxWidget>
#endif
#endif

#include "DockAreaTabBar.h"
#include "DockAreaTitleBar.h"
#include "DockAreaWidget.h"
#include "DockComponentsFactory.h"
#include "DockManager.h"
#include "DockSplitter.h"
#include "DockWidget.h"
#include "FloatingDockContainer.h"




static QIcon svgIcon(const QString& File)
{
	// This is a workaround, because in item views SVG icons are not
	// properly scaled and look blurry or pixelate
	QIcon SvgIcon(File);
	SvgIcon.addPixmap(SvgIcon.pixmap(92));
	return SvgIcon;
}




//============================================================================

struct MainWindowPrivate
{
	CMainWindow* _this;
	Ui::MainWindow ui;

	ads::CDockManager* DockManager = nullptr;
	QPointer<ads::CDockWidget> LastDockedEditor;
	QPointer<ads::CDockWidget> LastCreatedFloatingEditor;

	MainWindowPrivate(CMainWindow* _public) : _this(_public) {}

	//初始化相关接口
	void createActions();
	void saveState();
	void restoreState();
    void createContent();


	//QTreeView
	ads::CDockWidget* createFileSystemTreeDockWidget()
	{
		static int FileSystemCount = 0;
		QTreeView* w = new QTreeView();
		w->setFrameShape(QFrame::NoFrame);
		QFileSystemModel* m = new QFileSystemModel(w);
		m->setRootPath(QDir::currentPath());
		w->setModel(m);
		w->setRootIndex(m->index(QDir::currentPath()));
		ads::CDockWidget* DockWidget = DockManager->createDockWidget(QString("Filesystemjy %1")
			.arg(FileSystemCount++));
		DockWidget->setWidget(w);
		DockWidget->setIcon(svgIcon(":/adsdemo/images/folder_open.svg"));
		//ui.menuView->addAction(DockWidget->toggleViewAction());
		// We disable focus to test focus highlighting if the dock widget content
		// does not support focus
		w->setFocusPolicy(Qt::NoFocus);
		auto ToolBar = DockWidget->createDefaultToolBar();
		ToolBar->addAction(ui.actionSaveState);
		ToolBar->addAction(ui.actionRestoreState);
		return DockWidget;
	}

	//QCalendarWidget
	ads::CDockWidget* createCalendarDockWidget()
	{
		static int CalendarCount = 0;
		QCalendarWidget* w = new QCalendarWidget();
		ads::CDockWidget* DockWidget = DockManager->createDockWidget(QString("Calendar %1").arg(CalendarCount++));
		// The following lines are for testing the setWidget() and takeWidget()
		// functionality
		DockWidget->setWidget(w);
		DockWidget->setWidget(w); // what happens if we set a widget if a widget is already set
		DockWidget->takeWidget(); // we remove the widget
		DockWidget->setWidget(w); // and set the widget again - there should be no error
		DockWidget->setToggleViewActionMode(ads::CDockWidget::ActionModeShow);
		DockWidget->setIcon(svgIcon(":/adsdemo/images/date_range.svg"));
		//ui.menuView->addAction(DockWidget->toggleViewAction());
		auto ToolBar = DockWidget->createDefaultToolBar();
		ToolBar->addAction(ui.actionSaveState);
		ToolBar->addAction(ui.actionRestoreState);
		// For testing all calendar dock widgets have a the tool button style
		// Qt::ToolButtonTextUnderIcon
		DockWidget->setToolBarStyleSource(ads::CDockWidget::ToolBarStyleFromDockWidget);
        DockWidget->setToolBarStyle(Qt::ToolButtonTextBesideIcon,
                                    ads::CDockWidget::StateFloating);
		return DockWidget;
	}

	//QLabel
	ads::CDockWidget* createLongTextLabelDockWidget()
	{
		static int LabelCount = 0;
		QLabel* l = new QLabel();
		l->setWordWrap(true);
		l->setAlignment(Qt::AlignTop | Qt::AlignLeft);
		l->setText(QString("Label %1 %2 - Lorem ipsum dolor sit amet, consectetuer adipiscing elit. "
			"Aenean commodo ligula eget dolor. Aenean massa. Cum sociis natoque "
			"penatibus et magnis dis parturient montes, nascetur ridiculus mus. "
			"Donec quam felis, ultricies nec, pellentesque eu, pretium quis, sem. "
			"Nulla consequat massa quis enim. Donec pede justo, fringilla vel, "
			"aliquet nec, vulputate eget, arcu. In enim justo, rhoncus ut, "
			"imperdiet a, venenatis vitae, justo. Nullam dictum felis eu pede "
			"mollis pretium. Integer tincidunt. Cras dapibus. Vivamus elementum "
			"semper nisi. Aenean vulputate eleifend tellus. Aenean leo ligula, "
			"porttitor eu, consequat vitae, eleifend ac, enim. Aliquam lorem ante, "
			"dapibus in, viverra quis, feugiat a, tellus. Phasellus viverra nulla "
			"ut metus varius laoreet.")
			.arg(LabelCount)
			.arg(QTime::currentTime().toString("hh:mm:ss:zzz")));

		ads::CDockWidget* DockWidget = DockManager->createDockWidget(QString("Label %1").arg(LabelCount++));
		DockWidget->setWidget(l);
		DockWidget->setIcon(svgIcon(":/adsdemo/images/font_download.svg"));
		//ui.menuView->addAction(DockWidget->toggleViewAction());
		return DockWidget;
	}

	//QPlainTextEdit
	ads::CDockWidget* createEditorWidget()
	{
		static int EditorCount = 0;
		QPlainTextEdit* w = new QPlainTextEdit();
		w->setPlaceholderText("This is an editor. If you close the editor, it will be "
			"deleted. Enter your text here.");
		w->setStyleSheet("border: none");
		ads::CDockWidget* DockWidget = DockManager->createDockWidget(QString("Editor %1").arg(EditorCount++));
		DockWidget->setWidget(w);
		DockWidget->setIcon(svgIcon(":/adsdemo/images/edit.svg"));
		DockWidget->setFeature(ads::CDockWidget::CustomCloseHandling, true);
		//ui.menuView->addAction(DockWidget->toggleViewAction());

		QMenu* OptionsMenu = new QMenu(DockWidget);
		OptionsMenu->setTitle(QObject::tr("Options"));
		OptionsMenu->setToolTip(OptionsMenu->title());
		OptionsMenu->setIcon(svgIcon(":/adsdemo/images/custom-menu-button.svg"));
		auto MenuAction = OptionsMenu->menuAction();
		// The object name of the action will be set for the QToolButton that
		// is created in the dock area title bar. You can use this name for CSS
		// styling
		MenuAction->setObjectName("optionsMenu");
		DockWidget->setTitleBarActions({OptionsMenu->menuAction()});
		auto a = OptionsMenu->addAction(QObject::tr("Clear Editor"));
		w->connect(a, SIGNAL(triggered()), SLOT(clear()));

		return DockWidget;
	}








};




//============初始构建工具栏================================================================
void MainWindowPrivate::createActions()
{
    ui.toolBar->setToolButtonStyle(Qt::ToolButtonTextUnderIcon);
	//保存和还原
	ui.toolBar->addAction(ui.actionSaveState);
	ui.actionSaveState->setIcon(svgIcon(":/adsdemo/images/save.svg"));
	ui.toolBar->addAction(ui.actionRestoreState);
	ui.actionRestoreState->setIcon(svgIcon(":/adsdemo/images/restore.svg"));

	ui.toolBar->addSeparator();
	//锁定：禁止拖拽
	QAction* a = ui.toolBar->addAction("Lock Workspace");
	a->setIcon(svgIcon(":/adsdemo/images/lock_outline.svg"));
	a->setCheckable(true);
	a->setChecked(false);
	QObject::connect(a, &QAction::triggered, _this, &CMainWindow::lockWorkspace);



	ui.toolBar->addSeparator();

	//创建editor
	a = ui.toolBar->addAction("Create Editor");
	a->setProperty("Floating", false);
	a->setToolTip("Creates a editor tab and inserts it as second tab into an area");
	a->setIcon(svgIcon(":/adsdemo/images/tab.svg"));
	a->setProperty("Tabbed", true);
	_this->connect(a, SIGNAL(triggered()), SLOT(createEditor()));



}

//============保存和还原布局状态等================================================================
void MainWindowPrivate::saveState()
{
	QSettings Settings("Settings.ini", QSettings::IniFormat);
	Settings.setValue("mainWindow/Geometry", _this->saveGeometry());
	Settings.setValue("mainWindow/State", _this->saveState());
	Settings.setValue("mainWindow/DockingState", DockManager->saveState());
}
void MainWindowPrivate::restoreState()
{
	QSettings Settings("Settings.ini", QSettings::IniFormat);
	_this->restoreGeometry(Settings.value("mainWindow/Geometry").toByteArray());
	_this->restoreState(Settings.value("mainWindow/State").toByteArray());
	DockManager->restoreState(Settings.value("mainWindow/DockingState").toByteArray());
}

//=============构建主体布局===============================================================
void MainWindowPrivate::createContent()
{
    //1，创建一个日历，不可关闭，放到container左侧,只允许在左上右放置
    auto DockWidget = createCalendarDockWidget();
    DockWidget->setFeature(ads::CDockWidget::DockWidgetClosable, false);
    auto SpecialDockArea =
        DockManager->addDockWidget(ads::LeftDockWidgetArea, DockWidget);
    SpecialDockArea->setAllowedAreas({ads::LeftDockWidgetArea,
                                      ads::RightDockWidgetArea,
                                      ads::TopDockWidgetArea});

	// 2，创建一个label，不可聚焦，放到container左侧
    DockWidget = createLongTextLabelDockWidget();
    DockWidget->setFeature(ads::CDockWidget::DockWidgetFocusable, false);
    DockManager->addDockWidget(ads::LeftDockWidgetArea, DockWidget);

	// 3，创建一个树，不可浮动，放到container底部
    auto FileSystemWidget = createFileSystemTreeDockWidget();
    FileSystemWidget->setFeature(ads::CDockWidget::DockWidgetFloatable, false);
    DockManager->addDockWidget(ads::BottomDockWidgetArea, FileSystemWidget);



    // 4，创建一个日历，放到container中间（底部）
    DockWidget = createCalendarDockWidget();
    DockWidget->setTabToolTip(QString("Tab ToolTip\nHodie est dies magna"));//tab缝隙hover展示
    DockWidget->setWindowTitle(QString("My " + DockWidget->windowTitle()));
    auto DockArea =
        DockManager->addDockWidget(ads::CenterDockWidgetArea, DockWidget);//CenterDockWidgetArea和BottomDockWidgetArea等价
	// 4.1，创建一个按钮
    auto CustomButton = new QToolButton(DockArea);
    CustomButton->setToolTip(QObject::tr("Create Editor"));
    CustomButton->setIcon(svgIcon(":/adsdemo/images/plus.svg"));
    CustomButton->setAutoRaise(true);
    // 4.2，把按钮插入到标题栏tab右侧
    auto TitleBar = DockArea->titleBar();
    int Index = TitleBar->indexOf(TitleBar->tabBar());
    TitleBar->insertWidget(Index + 1, CustomButton);
    QObject::connect(CustomButton, &QToolButton::clicked, [DockArea, this]() {
        auto DockWidget = createEditorWidget();
        DockWidget->setFeature(ads::CDockWidget::DockWidgetDeleteOnClose, true);
		//语义化接口：等价于addDockWidget(ads::CenterDockWidgetArea, Dockwidget, DockAreaWidget, Index);
        DockManager->addDockWidgetTabToArea(DockWidget, DockArea);
        _this->connect(DockWidget, SIGNAL(closeRequested()),
                       SLOT(onEditorCloseRequested()));
    });





    // 5，创建一个label，放到container右侧
    auto RighDockArea = DockManager->addDockWidget(
        ads::RightDockWidgetArea, createLongTextLabelDockWidget());
    // 6，创建一个label，不可固定（Pin固定侧边栏功能），放到RighDockArea上方
    DockWidget = createLongTextLabelDockWidget();
    DockWidget->setFeature(ads::CDockWidget::DockWidgetPinnable, false);
    DockManager->addDockWidget(ads::TopDockWidgetArea, DockWidget, RighDockArea);
    // 7，创建一个label，放到RighDockArea底部
    auto BottomDockArea = DockManager->addDockWidget(
        ads::BottomDockWidgetArea, createLongTextLabelDockWidget(), RighDockArea);
    // 8，创建一个label，放到RighDockArea中心（Tab形式插入）
    DockManager->addDockWidget(ads::CenterDockWidgetArea,
                               createLongTextLabelDockWidget(), RighDockArea);
	// 9，创建一个label，放到BottomDockArea中心（Tab形式插入）
    auto LabelDockWidget = createLongTextLabelDockWidget();
    DockManager->addDockWidget(ads::CenterDockWidgetArea, LabelDockWidget,
                               BottomDockArea);
    //设置自定义处理关闭事件
    LabelDockWidget->setFeature(ads::CDockWidget::CustomCloseHandling, true);
    LabelDockWidget->setWindowTitle(LabelDockWidget->windowTitle()
                                    + " [Custom Close]");
    QObject::connect(LabelDockWidget, &ads::CDockWidget::closeRequested,
                     [LabelDockWidget, this]() {
                         int Result = QMessageBox::question(
                             _this, "Custom Close Request",
                             "Do you really want to close this dock widget?");
                         if (QMessageBox::Yes == Result)
                         {
                             LabelDockWidget->closeDockWidget();
                         }
                     });


	//注释的两个构建悬浮窗口的代码

    //// Test hidden floating dock widget
    // DockWidget = createLongTextLabelDockWidget();
    // DockManager->addDockWidgetFloating(DockWidget);
    // DockWidget->toggleView(false);
	// 
    //// Test visible floating dock widget
    // DockWidget = createCalendarDockWidget();
    // DockManager->addDockWidgetFloating(DockWidget);
    // DockWidget->setWindowTitle(QString("My " + DockWidget->windowTitle()));


}





//============================================================================
CMainWindow::CMainWindow(QWidget *parent) :
	QMainWindow(parent),
	d(new MainWindowPrivate(this))
{
	using namespace ads;
	d->ui.setupUi(this);
	setWindowTitle(QApplication::instance()->applicationName());
	d->createActions();


	//当前获得焦点的 dock widget/tab 会有更明显的视觉状
    CDockManager::setConfigFlag(CDockManager::FocusHighlighting, true);
	// 整个自动隐藏机制
    CDockManager::setAutoHideConfigFlags({CDockManager::DefaultAutoHideConfig});


	d->DockManager = new CDockManager(this);
	//浮动状态下的 dock widget toolbar只显示图标，不显示文字
	d->DockManager->setDockWidgetToolBarStyle(Qt::ToolButtonIconOnly, ads::CDockWidget::StateFloating);



	//整个 demo 布局搭建的核心函数
	d->createContent();
    resize(1280, 720);
	//根据主屏幕可用区域，把主窗口居中。
    setGeometry(QStyle::alignedRect(
        Qt::LeftToRight, Qt::AlignCenter, frameSize(),
        QGuiApplication::primaryScreen()->availableGeometry()));

	//恢复布局
	//d->restoreState();


}
CMainWindow::~CMainWindow()
{
	delete d;
}
void CMainWindow::closeEvent(QCloseEvent* event)
{
	d->saveState();
    delete d->DockManager;
    d->DockManager = nullptr;
	QMainWindow::closeEvent(event);
}

//============保存和还原布局状态等================================================================
void CMainWindow::on_actionSaveState_triggered(bool)
{
	qDebug() << "MainWindow::on_actionSaveState_triggered";
	d->saveState();
}
void CMainWindow::on_actionRestoreState_triggered(bool)
{
	qDebug() << "MainWindow::on_actionRestoreState_triggered";
	d->restoreState();
}

//=============锁定：进制拖拽等===============================================================
void CMainWindow::lockWorkspace(bool Value)
{
    if (Value)
    {
        d->DockManager->lockDockWidgetFeaturesGlobally();
    }
    else
    {
        d->DockManager->lockDockWidgetFeaturesGlobally(
            ads::CDockWidget::NoDockWidgetFeatures);
    }
}

//=============创建editor===============================================================
void CMainWindow::createEditor()
{
    QObject* Sender = sender();
    QVariant vFloating = Sender->property("Floating");
    bool Floating = vFloating.isValid() ? vFloating.toBool() : true;
    QVariant vTabbed = Sender->property("Tabbed");
    bool Tabbed = vTabbed.isValid() ? vTabbed.toBool() : true;
    auto DockWidget = d->createEditorWidget();
    DockWidget->setFeature(ads::CDockWidget::DockWidgetDeleteOnClose, true);
    DockWidget->setFeature(ads::CDockWidget::DockWidgetForceCloseWithArea, true);
    connect(DockWidget, SIGNAL(closeRequested()), SLOT(onEditorCloseRequested()));

    if (Floating)
    {
        auto FloatingWidget = d->DockManager->addDockWidgetFloating(DockWidget);
        FloatingWidget->move(QPoint(20, 20));
        d->LastCreatedFloatingEditor = DockWidget;
        d->LastDockedEditor.clear();
        return;
    }

    ads::CDockAreaWidget* EditorArea =
        d->LastDockedEditor ? d->LastDockedEditor->dockAreaWidget() : nullptr;
    if (EditorArea)
    {
        if (Tabbed)
        {
            // Test inserting the dock widget tab at a given position instead
            // of appending it. This function inserts the new dock widget as
            // first tab
            d->DockManager->addDockWidgetTabToArea(DockWidget, EditorArea, 0);
        }
        else
        {
            d->DockManager->setConfigFlag(
                ads::CDockManager::EqualSplitOnInsertion, true);
            d->DockManager->addDockWidget(ads::RightDockWidgetArea, DockWidget,
                                          EditorArea);
        }
    }
    else
    {
        if (d->LastCreatedFloatingEditor)
        {
            d->DockManager->addDockWidget(
                ads::RightDockWidgetArea, DockWidget,
                d->LastCreatedFloatingEditor->dockAreaWidget());
        }
        else
        {
            d->DockManager->addDockWidget(ads::TopDockWidgetArea, DockWidget);
        }
    }
    d->LastDockedEditor = DockWidget;
}

//==============自定义关闭槽函数==============================================================
void CMainWindow::onEditorCloseRequested()
{
	auto DockWidget = qobject_cast<ads::CDockWidget*>(sender());
	int Result = QMessageBox::question(this, "Close Editor", QString("Editor %1 "
		"contains unsaved changes? Would you like to close it?")
		.arg(DockWidget->windowTitle()));
	if (QMessageBox::Yes == Result)
	{
		DockWidget->closeDockWidget();
	}
}










