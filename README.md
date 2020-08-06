過去のコミットから特定のファイルを除外したいとき

「コードレビューをしてもらう際に修正ファイルと同じコミットにビルドファイルが含まれていて、ビルドファイルはビルドファイル単体のコミットで管理したい。」という場合の解決方法

#### やりたいこと

1stコミットに含まれる、bulid.txtを1stコミットから外したい

```
3rd コミット:
　toto.txt

2nd コミット:
　piyopiyo.txt

1st コミット:
　hogehoge.txt
　fugafuga.txt
　build.txt ← このファイルをコミットから外したい
```

#### 現状を確認する

```
$ git log --graph --pretty="%h %s" --stat

* 0c2ad68 3rd コミット
|
|  toto.txt | 0
|  1 file changed, 0 insertions(+), 0 deletions(-)
* df807a1 2nd コミット
|
|  piyopiyo.txt | 0
|  1 file changed, 0 insertions(+), 0 deletions(-)
* 6d7e47e 1st コミット
|
|  build.txt    | 0
|  fugafuga.txt | 0
|  hogehoge.txt | 0
|  3 files changed, 0 insertions(+), 0 deletions(-)
* 912382d git diffを追記
|
|  index.html | 1 +
|  1 file changed, 1 insertion(+)
```

#### git rebaseを行う

除外したいファイル(build.txt)が含まれているコミットの一つ前のコミットIDを指定してgit rebaseを行う

```
$ git rebase -i 912382d
```

エディタが起動するので、除外したいファイル(build.txt)が含まれているコミットを `edit` に変更して保存・終了

```
edit 6d7e47e 1st コミット
pick df807a1 2nd コミット
pick 0c2ad68 3rd コミット

# Rebase 912382d..0c2ad68 onto 912382d (3 commands)
#
# Commands:
# p, pick <commit> = use commit
# r, reword <commit> = use commit, but edit the commit message
# e, edit <commit> = use commit, but stop for amending
# s, squash <commit> = use commit, but meld into previous commit
# f, fixup <commit> = like "squash", but discard this commit's log message
# x, exec <command> = run command (the rest of the line) using shell
# b, break = stop here (continue rebase later with 'git rebase --continue')
# d, drop <commit> = remove commit
# l, label <label> = label current HEAD with a name
# t, reset <label> = reset HEAD to a label
# m, merge [-C <commit> | -c <commit>] <label> [# <oneline>]
# .       create a merge commit using the original merge commit's
# .       message (or the oneline, if no original merge commit was
# .       specified). Use -c <commit> to reword the commit message.
#
# These lines can be re-ordered; they are executed from top to bottom.
#
# If you remove a line here THAT COMMIT WILL BE LOST.
#
# However, if you remove everything, the rebase will be aborted.
#
# Note that empty commits are commented out
```

#### git rm でファイルの削除

エディタで保存・終了したら、下記のようになり指定したコミットまで戻っていることが確認できる

```
Stopped at 6d7e47e...  1st コミット
$ git rebase -i 912382d
You can amend the commit now, with

  git commit --amend

Once you are satisfied with your changes, run

  git rebase --continue

$ git log 
commit 6d7e47e100e483887cc498086b86ebc2d52fedc0 (HEAD)
Author: PHPer-sugiyama
Date:   Thu Aug 6 19:55:03 2020 +0900

    1st コミット

commit 912382d749bd3e17a993b1939c8b7690ee4b6d4c (origin/master, origin/HEAD, master)
Author: PHPer-sugiyama
Date:   Sat Aug 10 18:48:14 2019 +0900

    git diffを追記
```

`git rm` コマンドで除外したいファイルを指定したあとに `git commit --amend` でコミットの修正

```
$ git rm build.txt
$ git commit --amend
```

エディタが立ち上がるので、コメントを修正する場合は修正して保存・終了

```
1st コミット(build.txtファイルを除外)

# Please enter the commit message for your changes. Lines starting
# with '#' will be ignored, and an empty message aborts the commit.
#
# Date:      Thu Aug 6 19:55:03 2020 +0900
#
# interactive rebase in progress; onto 912382d
# Last command done (1 command done):
#    edit 6d7e47e 1st コミット
# Next commands to do (2 remaining commands):
#    pick df807a1 2nd コミット
#    pick 0c2ad68 3rd コミット
# You are currently splitting a commit while rebasing branch 'article' on '912382d'.
#
# Changes to be committed:
#       new file:   build.txt
#       new file:   fugafuga.txt
#       new file:   hogehoge.txt
#
# Changes not staged for commit:
#       deleted:    build.txt
```

#### git rebaseの継続

ファイルを除外することができたので、`git rebase --continue` でリベースを続行

```
$ git rebase --continue
uccessfully rebased and updated

$ git log --graph --pretty="%h %s" --stat

* 0c2ad68 3rd コミット
|
|  toto.txt | 0
|  1 file changed, 0 insertions(+), 0 deletions(-)
* df807a1 2nd コミット
|
|  piyopiyo.txt | 0
|  1 file changed, 0 insertions(+), 0 deletions(-)
* 6d7e47e 1st コミット(build.txtを除外)
|
|  fugafuga.txt | 0
|  hogehoge.txt | 0
|  3 files changed, 0 insertions(+), 0 deletions(-)
* 912382d git diffを追記
|
|  index.html | 1 +
|  1 file changed, 1 insertion(+)
```

これで無事に1stコミットからbuild.txtファイルを除外できる
