#[Android in the Workplace and in Education](https://developer.android.com/about/versions/android-5.0.html#Enterprise)
0. **DevicePolicyManager: manage users, switch users, hide/unhide APPs !!!**
http://developer.android.com/reference/android/app/admin/DevicePolicyManager.html
	1. [DevicePolicyManager.createUser()](http://developer.android.com/reference/android/app/admin/DevicePolicyManager.html#createUser(android.content.ComponentName, java.lang.String))
	1. [DevicePolicyManager.switchUser()](http://developer.android.com/reference/android/app/admin/DevicePolicyManager.html#switchUser(android.content.ComponentName,%20android.os.UserHandle))
0. [Device Owner](https://developer.android.com/about/versions/android-5.0.html#DeviceOwner):A device owner is a specialized type of device administrator. 
Android device administration includes two new types of device administrators for enterprises:
	- Profile owner—Designed for bring your own device (BYOD) environments
		> A Device Policy Client (**DPC**) app typically functions as the profile owner. The DPC app is typically provided by an enterprise mobility management (**EMM**) partner, such as Google Apps Device Policy.
		> The EMM has control **only over the managed profile (not personal space)** with some exceptions, such as enforcing > the lock screen.
		https://source.android.com/devices/tech/admin/managed-profiles.html#device_administration
	- Device Owner—Designed for corp-liable environments
		- The device owner can be set only in an unprovisioned device:
			- Can be provisioned only at initial device setup
			- Enforced disclosure always displayed in quick-settings
		- Device owners can conduct some tasks profile owners cannot, and here are a few examples:
			- Wipe device data
			- Disable Wi-Fi/ BT
			- Control setGlobalSetting
			- setLockTaskPackages (the ability to whitelist packages that can pin themselves to the foreground)
			- Set DISALLOW_MOUNT_PHYSICAL_MEDIA (FALSE by default. When TRUE, physical media, both portable and adoptable, cannot be mounted.)
0. [Provisioning for Device Administration](https://source.android.com/devices/tech/admin/provision.html)
	1. __Managed Provisioning__: this is a setup wizard for managed profiles. 设置managed profile时候的setup wizard.
	In this flow, managed provisioning triggers device encryption. The framework copies the EMM app into the managed profile as part of managed provisioning. The instance of the EMM app inside of the managed profile gets a callback from the framework when provisioning is done.
	The EMM can then add accounts and enforce policies; it then calls setProfileEnabled(), which makes the launcher icons visible.
	设置过程：
		- Encrypts the device(时间较长，约十几到几十分钟，电池电量要至少80%)
		- Creates the managed profile
		- Disables non-required applications
		- Sets the enterprise mobility management (EMM) app as profile owner
		（然后是EMM app的需要做的过程）
		- Adds user accounts
		- Enforces device compliance
		- Enables any additional system applications
	1. __Profile Owner Provisioning__: Mobile Device Management (MDM) applications trigger the creation of the managed profile by sending an intent with action:
	```java
	DevicePolicyManager.ACTION_PROVISION_MANAGED_PROFILE
	```
	1. __Device Owner Provisioning via NFC__
	1. __Device Owner Provisioning with Activation Code__
0. [Setting up Device Testing](https://source.android.com/devices/tech/admin/testing-setup.html)
	1. Essential elements: These are the essential elements that must exist for OEM devices to ensure minimal support for managed profiles:
	```
	- Profile Owner as described in Ensuring Compatibility with Managed Profiles
	- Device Owner
	- Activation Code Provisioning
	```
	See [Implementing Device Administration](https://source.android.com/devices/tech/admin/implement.html) for the complete list of requirements.
	**注意：在[Set up the device owner for testing](https://source.android.com/devices/tech/admin/testing-setup.html#set_up_the_device_owner_for_testing)中的测试方法要求设备上的系统是『userdebug』或『eng』编译的镜像，所以对于stock的设备不适用。因为stock设备在setup wizard阶段是无法使用adb shell的！**
	__ 用这种方法：http://stackoverflow.com/a/27909315/1054982 __
	1. DPM 命令的使用方法  [Android Shell command DPM : Device Policy Manager](http://florent-dupont.blogspot.fr/2015/01/android-shell-command-dpm-device-policy.html). 
		- 此人的另一篇文章：[10 things to know about Device Owner Apps in Android 5](http://florent-dupont.blogspot.com/2015/02/10-things-to-know-about-device-owner.html)
		- **注意：**
			- Important thing to know : It has to be done on an unprovisioned device , on the FIRST step, once your device is booted (if your device is already provisionned, go to Settings > Backup & Reset > Factory data reset to reset it. Beware : all your data will be lost. **必须要factory reset机器才能设置device owner！**). Once the NFC message is delivered, the second step will ask you to setup the Wifi connection. Once the Wifi connection is validated, the APK is donwloaded and your device is provisioned. 
			- once the Device Owner application is set, it cannot be unset with the dpm command. You’ll need to programmatically use the DevicePolicyManager.clearDeviceOwnerApp() method or factory reset your device. **一旦设置device owner APP，用户无法将其移除或卸载。除非factory reset，或者device owner APP自己调用clearDeviceOwnerApp()把自己移除**
			- “Device owner can only be set on an unprovisioned device, unless it was initiated by “adb”, in which case we allow it if no account is associated with the device” says the source code. So, make sure you don’t have any account (like Gmail) associated to your current user set before using the dpm command. **如果已经在设备上添加账户（如Google account），那么无法从adb设置device owner，必须要先将添加的账户删除**
			- Bad thing is that you cannot, programmatically and silently, install new applications for a user. To do this, you’ll need to have the permission INSTALL_PACKAGES, and this permission is not given for third party apps.**Device Owner APP没有删除其他APP的权限，只能隐藏**