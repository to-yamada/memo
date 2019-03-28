# Windows でのシンボリックリンク作成

ディレクトリに対して。

```
> mklink /D リンク ターゲット
```

例) `%USERPROFILE%` に clone した nvim の設定ファイルへリンクを作成。
```
> mklink /D %LOCALAPPDATA%\nvim %USERPROFILE%\dotfiles\.config\nvim
```

例) `%USERPROFILE%` に clone した vim の設定ファイルへリンクを作成。
```
> mklink /D %USERPROFILE%\vimfiles %USERPROFILE%\dotfiles\.vim
```
