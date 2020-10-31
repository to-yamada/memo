# Windows でのシンボリックリンク作成

ディレクトリに対して。

```
> mklink /D リンク ターゲット
```

例) `%USERPROFILE%` に clone した vim の設定ファイルへリンクを作成。
```
> mklink /D %USERPROFILE%\vimfiles %USERPROFILE%\dotfiles\.vim
```

例) `%USERPROFILE%` に clone した volt へリンクを作成。
```
> mklink /D %USERPROFILE%\volt %USERPROFILE%\dotfiles\.volt
```
