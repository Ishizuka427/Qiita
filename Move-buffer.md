バッファの移動を上下左右自由に行いたかったので設定を編集しました。
ショートカットを追加します。

```console
~/.spacemacs
```

```lisp
(defun dotspacemacs/user-config ()
  "Configuration function for user code.
This function is called at the very end of Spacemacs initialization after
layers configuration.
This is the place where most of your configurations should be done. Unless it is
explicitly specified that a variable should be set before a package is loaded,
you should place your code here."
  (bind-keys*
   ;; move buffer
   ("C-t h" . windmove-left)
   ("C-t j" . windmove-down)
   ("C-t k" . windmove-up)
   ("C-t l" . windmove-right)
   ("C-t C-h" . windmove-left)
   ("C-t C-j" . windmove-down)
   ("C-t C-k" . windmove-up)
   ("C-t C-l" . windmove-right)
   )
  )
```

これで設定したキーバインド通りにショートカットが効き
上下左右のバッファ移動が可能になります。
