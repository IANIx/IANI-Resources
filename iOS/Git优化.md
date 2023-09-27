# Git优化

经常会碰到项目clone时很慢，下载下来文件很大。解析文件大小后，其实项目本身代码没有多少，`.git`文件就有500多兆。

但是其实项目提交记录并不是很多，经过一番查询发现是旧的历史提交文件过大，虽然后面删除了，但是`.git`会在记录中保存这些文件。

## 清理在历史提交中记录的大文件

### 首先查询大文件信息

`git verify-pack -v .git/objects/pack/pack-xxx.idx | sort -k 3 -n | tail -3` (xxx你的.git的pack目录文件)

查询结果如下

```shell
~/Desktop/** master*
❯ git verify-pack -v .git/objects/pack/pack-d29d9a7191eb895a9cdd2a133b85f5c665927113.idx | sort -k 3 -n | tail -3
f32582cdbf1e151f889daea3fcd37423c2d1ed49 blob   91796352 41252977 626967740
b31dd041e4d98c912d613fce7ce8590732ea0eaf blob   92350608 40223625 672593001
c75277368267b9457ff4a2ad813d11d1da8da175 blob   94722952 38906290 260868505
```

然后在根据上面的文件名的**编码ID**：f32582cdbf1e151f889daea3fcd37423c2d1ed49 查询出大文件的名称：

```shell
~/Desktop/** master* 21s
❯ git rev-list --objects --all | grep f32582cdbf1e151f889daea3fcd37423c2d1ed49

f32582cdbf1e151f889daea3fcd37423c2d1ed49 **/Class/Vendors/GrowingIO/libGrowing.a

```

可以看到是一个放在项目中的三方静态库，再后面开发中这个库早已经不用了，但是确一直在`git` 记录中，每次`clone`都会下载这些无用文件。



除了单个文件名查询外，也可以一次查询多个文件。

`git rev-list --objects --all | grep "$(git verify-pack -v .git/objects/pack/*.idx | sort -k 3 -n | tail -10 | awk '{print$1}')"` #查询前10个大文件

```shell
~/Desktop/** master*
❯ git rev-list --objects --all | grep "$(git verify-pack -v .git/objects/pack/*.idx | sort -k 3 -n | tail -10 | awk '{print$1}')"

45a7d93709777b9c65123d78450decfeae4b1948 **/Class/Vendors/JPush/lib/jcore-noidfa-ios-2.4.0.a
c253dad711a68f22e3f03a816b1ac60fec03e7e2 **/Class/Vendors/baidu/BaiduMap/BaiduMapAPI_Map.framework/BaiduMapAPI_Map
e97fcd562245668d144fa338133e777b73f62353 **/Class/Vendors/baidu/BaiduMap/thirdlibs/libcrypto.a
c75277368267b9457ff4a2ad813d11d1da8da175 **/Class/Vendors/GDTAd/lib/libGDTMobSDK.a
ead6c1fb28a0e79de9ab22d984242d0fddefb40b **/Class/Vendors/GrowingIO/GrowingAutoTrackKit.framework/GrowingAutoTrackKit
c296bcc509848960c9bcc18be4f65a6d84b9589d **/Class/Vendors/UPPay/libUMSPosPayOnly.a
f32582cdbf1e151f889daea3fcd37423c2d1ed49 **/Class/Vendors/GrowingIO/libGrowing.a
b31dd041e4d98c912d613fce7ce8590732ea0eaf **/Class/Vendors/baidu/BaiduMap/BaiduMapAPI_Map.framework/BaiduMapAPI_Map
bff7e4454b76825500a09a1e943e6c30990bd41c **/Class/Vendors/baidu/BaiduMap/BaiduMapAPI_Map.framework/BaiduMapAPI_Map
afc8ea948d3da13405ecfb0138970b7d15559469 **/Class/Vendors/baidu/BaiduMap/BaiduMapAPI_Map.framework/BaiduMapAPI_Map


```



### 在git记录中删除这些文件的提交记录

查询这些无用的大文件信息后，接下来要把这些文件从历史记录的所有 tree 中移除，执行命令从历史中删除指定的大文件：

`git filter-branch --force --index-filter "git rm -rf --cached --ignore-unmatch 文件/文件夹" --prune-empty --tag-name-filter cat -- --all` #文件/文件夹 是通过上面查询出来的大文件路径和名称

`git filter-branch` 是一个用于过滤和转换 Git 存储库历史记录的工具，它可以帮助您删除不需要的文件、分支、提交等，并重写存储库的历史记录。

**注意：使用 `git filter-branch` 可能会导致历史记录的重写，这可能对与存储库协作的其他人造成问题。在执行任何 `git filter-branch` 操作之前，请务必备份存储库。**

这些文件都是最早的时候本地导入的外部库文件，现在都是pod管理，所以可以直接把`Vendors`文件移除掉。

```shell
~/Desktop/** master* 10s
❯ git filter-branch --force --index-filter "git rm -rf --cached --ignore-unmatch **/Class/Vendors/*" --prune-empty --tag-name-filter cat -- --all 

.....
Ref 'refs/heads/master' was rewritten

```

清除完毕检查历史记录都没问题后，记得推送到远程仓库

```shell
~/Desktop/** master* 10s
❯ git push origin --force --all 
```

