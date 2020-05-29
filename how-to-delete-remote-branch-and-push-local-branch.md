リモートのブランチ名を変えたいけれど**あ゛~~~~~~**ってなったので備忘録です。

リモートのブランチを表示します

```console
$ git branch -r 
  origin/HEAD -> origin/master
  origin/master
  origin/fix_example
```

ローカルのブランチを表示します

```console
$ git branch
* fix_example
  master
```

ローカルのブランチを書き換える必要が生じた！

```console
$ git branch -m 5.fix_example

$ git branch
* 5.fix_example
  master
```

そのままpushしてもローカルのブランチ名は昔のまま

```console
$ git push
fatal: The upstream branch of your current branch does not match
the name of your current branch.  To push to the upstream branch
on the remote, use

    git push origin HEAD:fix_example

To push to the branch of the same name on the remote, use

    git push origin HEAD

```

`.git/config` を編集します。

```
$ vim .git/config
```

**編集前**

```config
~略~
[branch "5.fix_example"]
         remote = origin
         merge = refs/heads/fix_example
```

**編集後**

```config
~略~
[branch "5.fix_example"]
         remote = origin
         merge = refs/heads/5.fix_example
```

揃えた。
あとはpushします。

```
$ git push
```

力技で解決系なので自己責任でお願いします。

