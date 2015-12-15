# Launcher3 Source Code Study ☮
## Questions
0. The database structure ?
1. The logic structure ?
2. Where is the code adding APP from all apps to Workspace ?

## Classes
0. ShortcutInfo: Represents a launchable icon on the workspaces and in folders.
0. LauncherAppWidgetInfo: Represents a widget (either instantiated or about to be) in the Launcher.
0. LauncherAppsCompat: user management ?
0. UserManagerCompat: user management ?
0. [LauncherApps](https://developer.android.com/reference/android/content/pm/LauncherApps.html)
0. AllAppsGridAdapter

## Launcher.java
### onCreate()函数
0. 每次旋转屏幕都会重新create这个activity，onCreate()都会被调用。
> [Android Dev Guide: Handling Runtime Changes](http://developer.android.com/guide/topics/resources/runtime-changes.html)
> Some device configurations can change during runtime (such as screen orientation, keyboard availability, and language). 
> When such a change occurs, Android restarts the running Activity (onDestroy() is called, followed by onCreate()).

	```java
	onCreate()
		|--> setupViews()
		|--> mDeviceProfile.layout(this)
		|--> mModel.startLoader(mWorkspace.getRestorePage())
							|--> mLoaderTask = new LoaderTask(mApp.getContext(), loadFlags)
							|--> sWorker.post(mLoaderTask)
	```

1. setupViews()
2. mDeviceProfile.layout()
3. mModel.startLoader()
## LauncherModel.java (MVC)
0. LauncherModel.LoaderTask.loadWorkspace()
把所有的workspace上的items从数据库加载到sBgItemsIdMap
```java
loadWorkspace()
	|--> LauncherAppState.getLauncherProvider().loadDefaultFavoritesIfNecessary()
	|--> Uri contentUri = LauncherSettings.Favorites.CONTENT_URI
	|--> Cursor c = contentResolver.query(contentUri, null, null, null, null)
```
1. LauncherModel.LoaderTask.bindWorkspace()
把加载进来的workspace上的item绑定到view
## Workspace.java
0. Workspace.Workspace(Context context, AttributeSet attrs, int defStyle)
```
Workspace(Context context, AttributeSet attrs, int defStyle)
		|
		|--> initWorkspace()
```
1. Workspace.acceptDrop()
2. Workspace.onDrop()

## 添加和删除APP
0. install app: InstallShortcutReceiver, 这个receiver是在AndroidManifest.xml里面注册的，监听"com.android.launcher.action.INSTALL_SHORTCUT"这个action。
1. remove app: LauncherModel.PackageUpdatedTask
##APP长按处理
0. com.android.launcher3.Launcher#onLongClick 处理：
	1. 调用onLongClickAllAppsButton(v)处理all APPs按钮的长按
	1. showOverviewMode(true)，workspace上的长按或hotseat上空白处的长按
	1. mWorkspace.startReordering(v)，overview模式时候的长按
	1. mWorkspace.startDrag(longClickCellInfo)，除了folder以外的其他item的长按
	1. **folder的长按在哪里处理？** com.android.launcher3.Folder#onLongClick
0. worxlauncher trusted apps 管理
	1. load时，初始化，add到数据库和从数据库remove时修改
	1. com.android.launcher3.LauncherModel#addItemToDatabase