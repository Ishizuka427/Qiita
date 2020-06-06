# 環境
macOS
spacemacs

# 参照
SpacemacsのTheme一覧: [Spacemacs Themes](https://themegallery.robdor.com/
)
twilight-bright-themeのGitHubリポジトリ: [jimeh/twilight-bright-theme.el](https://github.com/jimeh/twilight-bright-theme.el)

# Step1. 既存のThemeを変更してみる

```
M-x load-theme
```

変更したいThemeを選びます。

# Step2. 外部のThemeをインストールして適用してみる

1. [Spacemacs Themes](https://themegallery.robdor.com/
) (SpacemacsのTheme一覧) から、使用したいThemeを選びます。
2. `twilight-bright` を使いたいため、[twilight-bright-themeのGitHubリポジトリ](https://github.com/jimeh/twilight-bright-theme.el)を見つけます。 

3. `.spacemacs.d`以下の`init.el`を開きます。

4. `dotspacemacs-additional-packages`に`twilight-bright-theme`を追記して
`dotspacemacs-themes`に`twilight-bright`を追記します。

※`dotspacemacs-themes`は上から順に読み込まれるため、先頭に記述してください。

```emacs-lisp:init.el
dotspacemacs-additional-packages '(twilight-bright-theme
                                      )
dotspacemacs-themes '(twilight-bright
                         spacemacs-light
                         spacemacs-dark
                         )
```

emacsを再起動すると読み込まれます。
