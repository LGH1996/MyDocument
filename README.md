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
