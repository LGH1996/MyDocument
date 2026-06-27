#### windows运行wsl编译出来的aosp编译出来的镜像文件方案：
编译是lunch选择<br>
sdk_phone64_x86_64-trunk_staging-eng<br>
编译成功后，powershell里设置设置环境变量<br>
$env:ANDROID_PRODUCT_OUT="\\wsl.localhost\Ubuntu\home\lingh\aosp\out\target\product\emu64x"<br>
指向镜像文件的输出目录，然后运行一下命令启动emulator<br>
D:\Users\80420416\AppData\Local\Android\Sdk\emulator\emulator.exe -sysdir $env:ANDROID_PRODUCT_OUT\ -kernel $env:ANDROID_PRODUCT_OUT\kernel-ranchu -system $env:ANDROID_PRODUCT_OUT\system-qemu.img -vendor $env:ANDROID_PRODUCT_OUT\vendor-qemu.img -ramdisk  $env:ANDROID_PRODUCT_OUT\ramdisk-qemu.img  -data $env:ANDROID_PRODUCT_OUT\userdata-qemu.img  -writable-system -shell -selinux disabled -vsync-rate 60 -skin 1080x2400<br>
其中-sysdir是必须参数，其他的看情况<br>


#### AndroidStudio模拟器启动失败原因排查<br>
可在以下目录的日志中找到启动命令及失败原因<br>
C:\Users\lingh\AppData\Local\Google\AndroidStudio2025.1.2\log\idea.log<br>

#### 可用以下命令挂载img镜像到指定目录后查看其内容
通过file命令，如果system.img是sparse格式，则需通过以下命令安装simg2img，把sparse格式转为EROFS格式<br>
sudo apt-get install android-sdk-libsparse-utils<br>
sudo simg2img system.img system-raw.img<br>
sudo mount -t erofs system-row.img mountdir<br>
sudo umount mountdir

#### Android13自带类库可以在以下文件查看
prebuilts\sdk\current\support\Android.bp

#### android.bp编译时部分系统接口或字段无法编译通过的问题
如WifiConfiguration.INVALID_RSSI字段、ConnectivityManager.getTetherableUsbRegexs()，它们在源码中存在，但在framework.jar中又会被移除掉，导致设置了platform_apis:true也无法编译通过，或者即使通过static_libs提供jar包也会因为ramework.jar优先级更高而被覆盖无作用。<br><br>
解决方法是找到包含该类的java_sdk_library模块，比如其模块名称为framework-wifi，则在自己的模块中通过libs（不是static_libs）引入，引入名称为framework-wifi.impl（需加上.impl才能访问到隐藏接口），同时libs引入framework-res，framework（需保证其排在framework-wifi.impl后，优先级才不会覆盖framework-wifi.impl），同时设置sdk_version: "core_platform"（不能是platform_apis: true）,这样即可访问隐藏接口并编译通过。如果提示framework-wifi.impl对当前模块不可见，则修改framework-wifi.impl模块中的impl_library_visibility属性内容。<br><br>
< img width="821" height="596" alt="image" src="https://github.com/user-attachments/assets/a4012050-e93a-43de-93c4-9df418318b75" /><br><br>
< img width="662" height="236" alt="image" src="https://github.com/user-attachments/assets/ca4b5283-ba59-4b2a-a729-61f11e1ecc81" />


#### 执行git fetch --unshallow报以下错误
error: Could not read 6dc40025a3874522742e6a25a20ba37e834e959c<br>
fatal: Failed to traverse parents of commit b4dad600a484149402d551225fcec3261d823ea8<br>
error: remote did not send all necessary objects<br>
#### 解决方法为依次执行以下命令
git fsck --full<br>
git reflog expire --stale-fix --all<br>
git gc --prune=now<br>


#### AndroidStudio里debug无法显示attach进程时，需设置下一下属性：
setprop ro.debuggable 1；<br>
setprop persist.debug.dalvik.vm.jdwp.enabled 1；<br>
setprop persist.debug.ptrace.enabled 1；<br>
如果debug时经常因为input despatch time导致anr，可设置ro.hw_timeout_multiplier属性延迟默认timeout时间，设置后killall system_server重启下服务即可生效；<br>

#### adb shell debuggerd -b <pid> 打印当前进程的 Native 堆栈；


#### native打印堆栈：
#include <utils/CallStack.h><br>
CallStack::logStack("TAG");<br>
如果打印输出CallStack::logStackInternal not linked，无具体堆栈信息，Android.bp或Android.mk加入<br>
shared_libs: [<br>
    "libutilscallstack", // 添加这一行<br>
], 或 LOCAL_SHARED_LIBRARIES += libutilscallstack重新编译；<br>

#### 解析堆栈行号：
~/erhai/source/sys/prebuilts/clang/host/linux-x86/clang-r547379/bin/llvm-symbolizer --obj="/work/source/sys/out/target/product/emu64x/symbols/system/bin/surfaceflinger" -C -f -i  0x0000000000385984 <br>

#### 挂载usb设备到wsl：
安装usbipd-win：https://github.com/dorssel/usbipd-win/releases；<br>
usbipd list 命令查看可用usb设备；<br>
usbipd bind --force --busid 1-11 命令bind设备；<br>
usbipd attach --wsl --busid 1-11 命令attach设备到wsl；<br>
如遇权限问题，可能需要以管理员身份运行powershell；<br>
#### wsl里挂载调试服务器的Samba目录：
mkdir newDir 命令创建挂载目录；<br>
sudo mount -t drvfs "//服务器地址/data/dir" newDir/ 命令挂载服务器目录到本地；<br>
