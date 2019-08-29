### 解决win10与deepin双系统无法共享蓝牙鼠标的问题


### 背景
> 新买的荣耀Magicbook pro笔记本感觉非常nice，还送了非常好看的蓝牙鼠标，哈哈哈。心血来潮装了最好看的国产linux系统Deepin，玩了一会切换到window的时候鼠标不好使了，删除鼠标设备后重新配对又好使了，可是等我切换到deepin的时候，鼠标又不好使，没又办法，只好重新连接。。。。。我好无奈。
> 
>难道就要这样妥协吗？？？怎么可能，不折腾一下心是不会死的。。
>
> 不懂就要问度娘嘛，嘻嘻O(∩_∩)O

### 原因
> 在经过一番仔细搜索之后，我发现了问题的根本所在。
> 
> 荣耀鼠标的蓝牙每次连接都会随机生成一个对接码。也就是说先和win10对接后，会生成对接码123，但是切换到deepin后再连接，鼠标就会存储第二次对接的对接码456，那么等你再次回到win10系统之后，鼠标存储的对接码456和原来系统存储的对接码123不匹配，就会无法连接。
> 
> 嗦嘎。找到了原因可是如何解决呢？？？
> 
### 解决方案
> 有没有办法让两个对接码一致呢？？？又在度娘那撒了一会娇O(∩_∩)O哈哈~，度娘又告诉我好多消息。
> 
> 既然对接码不一致造成了这种问题，那将电脑记录的对接码修改成一样的，问题不就解决了吗？？关键是怎么修改，改哪一个？在哪里改？
> 

1. 首先要让Deepin连接蓝牙鼠标，在linux上生成配对的基本信息，后期拿windows的信息去覆盖掉。

2. 然后，要让windows连接鼠标，生成一个对接码，这些信息会存储在注册表里，不过一般情况下访问注册表看不见这些信息，需要下载[PSTools](https://technet.microsoft.com/en-us/sysinternals/bb897553)。https://technet.microsoft.com/en-us/sysinternals/bb897553

3. 再然后将压缩包解压到D盘（为例，随便哪里都可以）
 
4. 接着以管理员身份打开cmd，打开方式多种多样，可以win+X，再按A，也可以在小娜搜索框搜索cmd，右键点击以管理员身份运行，
 
5. 在命令行输入 以下命令打开注册表
  `
 cd D：/PSTools/
 Psexec64.exe -s -i regedit 
 `
6. 在注册表的HKEY_LOCAL_MACHINE\SYSTEM\CurrentControlSet\services\BTHPORT\Parameters\Keys\下有一串字符，是电脑蓝牙的MAC地址，再下一层又有一串字符获几串字符，是你连接过的所有蓝牙设备的MAC地址，如果有多个字符串，搞不清楚哪个是鼠标的，可以暂时把其他配对设备删除掉，只留下鼠标的。到此处你就可以看到好多连接的信息。数据里面没有被括号括起来的值是16进制的，括号里面的10进制的，我们需要记录下一些数据。
*    【蓝牙设备的MAC地址】
*    【EDIV】                             记录10进制的数据
*    【ERand】                           记录10进制的数据
*    【IRK】                                记录16进制的数据
*    【LTK】                                记录16进制的数据
 
最好是手机拍照拍下来。省的到时候搞不好怀疑记错了反复重启。

7. 重启进入Deepin，以管理员身份（非管理员身份进不去）进入/var/lib/bluetooth/XX:XX:XX:XX:XX:XX（电脑蓝牙地址）/YY:YY:YY:YY:YY:YY（鼠标蓝牙地址）目录  
 
8. 强调一下，这个YY：YY：YY：YY：YY：YY（鼠标蓝牙MAC地址）跟windows上的不一样，需要修改一下。打开终端输入mv  oldfile  newfile
`
mv YY：YY：YY：YY：YY：Y1  YY：YY：YY：YY：YY：Y2
`
为什么不直接重命名，因为直接重命名的话冒号就没了。。。。。bug吧 

9. 再打开YY:YY:YY:YY:YY:YY文件夹下的info文件进行修改。

         [IdentityResolvingKey]        就是记录的【IRK】
         Key=21d43414b9cd1dd244185f468df0ad57
        [LongTermKey]                 就是记录的 【LTK】        
        Key=ebfad27beadc8958d6f046165c1daf72
        Authenticated=0
        EncSize=16
        EDiv=50307                    这就是记录的【EDiv】         
        Rand=182778928298143522       这就是记录的【Rand】         

10. 修改完以后保存文件。
 终端输入
 `
service bluetooth restart
`
11. 重启蓝牙服务，或者重新开关蓝牙，或者重启电脑。之后就可以不配对连接鼠标，双系统共用了。
