#### windows运行wsl编译出来的aosp编译出来的镜像文件方案：
编译是lunch选择<br>
sdk_phone64_x86_64-trunk_staging-eng<br>
编译成功后，powershell里设置设置环境变量<br>
$env:ANDROID_PRODUCT_OUT="\\wsl.localhost\Ubuntu-22.04\home\lingh\android\aosp\out\target\product\vsoc_x86_64"<br>
指向镜像文件的输出目录，然后运行一下命令启动emulator<br>
C:\Users\lingh\AppData\Local\Android\Sdk\emulator\emulator.exe -sysdir $env:ANDROID_PRODUCT_OUT -kernel $env:ANDROID_PRODUCT_OUT\kernel -data $env:ANDROID_PRODUCT_OUT\userdata.img -ramdisk $env:ANDROID_PRODUCT_OUT/ramdisk-qemu.img<br>
其中-sysdir、-kernel、-ramdisk是必须参数，-ramdisk指向带有qemu的镜像<br>


#### AndroidStudio模拟器启动失败原因排查<br>
可在以下目录的日志中找到启动命令及失败原因<br>
C:\Users\lingh\AppData\Local\Google\AndroidStudio2025.1.2\log\idea.log<br>

#### 可用以下命令挂载img镜像到指定目录后查看其内容
sudo mount -t erofs out/target/product/<device>/system.img /mnt/system<br>
sudo umount /mnt/system<br>

#### Android13自带类库可以在以下文件查看
prebuilts\sdk\current\support\Android.bp

#### android.bp编译时部分系统接口或字段无法编译通过的问题
如WifiConfiguration.INVALID_RSSI字段、ConnectivityManager.getTetherableUsbRegexs()，它们在源码中存在，但在framework.jar中又会被移除掉，导致设置了platform_apis:true也无法编译通过，或者即使通过static_libs提供jar包也会因为ramework.jar优先级更高而被覆盖无作用。<br><br>
解决方法是找到包含该类的java_sdk_library模块，比如其模块名称为framework-wifi，则在自己的模块中通过libs（不是static_libs）引入，引入名称为framework-wifi.impl（需加上.impl才能访问到隐藏接口），同时libs引入framework-res，framework（需保证其排在framework-wifi.impl后，优先级才不会覆盖framework-wifi.impl），同时设置sdk_version: "core_platform"（不能是platform_apis: true）,这样即可访问隐藏接口并编译通过。如果提示framework-wifi.impl对当前模块不可见，则修改framework-wifi.impl模块中的impl_library_visibility属性内容。<br><br>
<img width="821" height="596" alt="image" src="https://github.com/user-attachments/assets/a4012050-e93a-43de-93c4-9df418318b75" /><br><br>
<img width="662" height="236" alt="image" src="https://github.com/user-attachments/assets/ca4b5283-ba59-4b2a-a729-61f11e1ecc81" />

