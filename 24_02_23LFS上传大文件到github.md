使用方法很简单，在确保配置好git的情况下，首先安装lfs

```
git lfs install
```

然后track大文件

```
git lfs track "*.psd"
```

确保track .gitattributes

```
git add .gitattributes
```

接下来走正常的git流程就行

```
git add file.psd
git commit -m 'commit psd'
git push -u orgin main
```

