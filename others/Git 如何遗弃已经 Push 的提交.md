## Git 如何遗弃已经 Push 的提交

题目看起来很像是提供解决方案的文章，但实际上我并不会给大家直接提供解决方案，我们追求的从来不应该是答案，而是探索的过程。当然，如果你只想查看答案的话，请直接拉到文章最底部。

### 写在前面

相信大家都知道，Git 相比于 SVN，优势不言而喻，以致于现在大多数公司的项目都在采用 Git 进行管理。作为一个开发人员，对 Git 的使用自然应该是得心应手。

如果你还不会使用 Git 的话，那我劝你还是不要声张，好好的去学习一番，再自己弄个实验项目走一下流程，以免遭到同事的鄙视。

每个公司都会有自己不一样的 Git 分支管理规范，特别是在开发人员较多的公司，Git 的分支管理规范就显得更加重要。前面比较出名的 Git Flow 分支管理策略相信不少人都已经了解了，不熟悉的当然也可以去看看：http://nvie.com/posts/a-successful-git-branching-model/

![分支管理](https://upload-images.jianshu.io/upload_images/3994917-84b44e605cc6f095.png)

Git Flow 管理方式把项目分为 5 条线，通常会是下面的管理方式。

- Master：作为稳定主分支，长期有效。不可以在此分支进行任何提交，只能接受从 Hotfix 分支或者 Release 分支发起的 merge request，该分支上的每一个提交都对应一个 Tag。
- Develop：开发主分支，长期有效。不可以在此分支上做任何提交，只接受从 Feature 分支发起的 merge request。所有的 Alpha Release 都应该在这个分支发布。
- Feature：功能分支，生命周期为产品迭代周期，每个分支对应一期的需求。只可以从 Develop 分支进行 Kick Off。可以 merge Release 分支的代码，生命周期结束后，需要 merge 回 Develop 分支。**方式需要采用 merge request。**
- Release：发布分支，声明周期从新需求的预发布到正式发布，每一个分支对应一个新版本的版本号。只可以从 Develop 分支 Kick Off。声明周期结束后，需要 Merge 回 Master 及 Develop 分支，方式同样需要采用 merge request。所有的 Beta Release 均需要在该分支发布。
- Hotfix：热修复分支，生命周期对应一个或者多个需要紧急修复并上线的 Bug，每一个分支对应一个小版本号。只可以从 Master 分支进行 Kick Off。声明周期结束后，需要 merge 回 Master 分支和 Develop 分支，方式当然也是采用 merge request。

实际上，如果你熟悉 Git 的话，你会很快发现上面的管理方式会存在历史提交非常混乱的缺点，但觉得不失为一个 Git 分支管理的经典。实际上，我们可以用 rebase 去替换 merge 让 commit 看起来更加清晰。对 rebase 和 merge 的优劣对比这里暂不做讲解，感兴趣的可以直接 Google 搜索。

下面就给大家分享一下发生在咕咚项目的一次坑爹的 Git 体验。

### 从 git revert 说起

咕咚项目组并没有对开发者限制 Develop 分支和 Master 分支的权限，我们暂时并没有一个专门做代码 Review 和 PR 的角色，其实一定意义上也提现了团队对每个人的信任。

我们依然会基于 Develop 做开发主线，每个需求迭代期，团队成员会从 Develop 拉取自己的分支，并命名于 feture/XX，然后各自在自己的分支上进行开发。

由于大家开发业务上的不同，所以在需求开发完毕，整合代码到 Develop 分支的时候，一般不会出现太多冲突的情况。

而我这边交接一个需求时，采用 merge 的时候出现了一个奇怪的问题，我们姑且来重现一下事故现场。

首先使用 `git branch` 查看一下当前我们的本地分支。

![查看分支](https://upload-images.jianshu.io/upload_images/3994917-81a686501ae2afeb.png)

这里先简单提一下我们要做的操作。

"feature8.28_buyGifts" 是我们同事的分支，基于 "release8.27.0" 拉取，而 "feature8.29.0_nanchen" 是我的分支，基于 "release8.28.0" 分支拉取，所以我这边的分支包含了最新的代码。

现在由于某些原因，我需要把同事的 "feature8.28_buyGifs" 分支代码合并到我的分支上，直接接手他的代码进行开发。

> 就不要吐槽为啥不按照功能搞分支开发了，原因是因为他那边代码基本已经完成，现在只需要少量修改。

所以我们就采用 `git merge <branch>` 命令进行 merge 操作。

![merge](https://upload-images.jianshu.io/upload_images/3994917-de4a932a54324665.png)

我们用 `git status` 更容易看明白冲突了什么。

![](https://upload-images.jianshu.io/upload_images/3994917-c91159f3787e8ca2.png)

可以看到，上面冲突的文件全是和同事开发的需求出现的冲突，所以出现这个冲突其实令人非常懊恼，因为是不可能有其他同事改动到这些文件的。

为了验证自己的想法，我们随意打开一个文件查看。这里就采用 `vim <filename>` 查看第一个文件。

![](https://upload-images.jianshu.io/upload_images/3994917-f988d620cab43fbf.png)

正如我们所想，确实和同事编写的需求 `Presents` 类有关系，但看冲突内容就更一脸懵逼了，因为看起来，这应该是一个不会冲突的 merge。

于是赶紧使用 `git merge --abort` 撤销这次 merge。再在 "origin/feature8.29.0_nanchen" 查看我们刚刚的文件提交历史。

![](https://upload-images.jianshu.io/upload_images/3994917-7d0019d292a08f50.png)

可以很清晰的看到，确实是最近没有任何的修改记录。

**一个 7 个月都没人动的文件，居然 merge 的时候发生了冲突！**这让我一脸懵逼。（手动黑人问号）

使用 `git lg` 查看一下该分支的提交历史，我们希望从中能得到某些思路。

![](https://upload-images.jianshu.io/upload_images/3994917-c82bc8c50bb0a83a.png)

注意其中红框中的 commit，我们这位同事之前想往 "release8.28.0" 合并他分支的代码，后面又因为某些原因，希望撤销这次提交，他采用了 revert 进行处理。**虽然 revert 对文件没有提交记录，但 Git 却认为我们在当前分支更改了这些文件，所以在我们 `git merge` 的时候，Git 认为这是一次冲突，并选择了告知我们。**

如若如我们所想，那我们只需要撤销这次 revert 操作即可。

我们当然知道，可以通过 reset 命令放弃这次提交，但这里后面已经有了非常多的 commit，显然我们这样是不行的，我们需要另辟蹊径。

### 解决方案？

最容易想到的大概就是直接在 merge 的时候解决冲突了，但通过一系列查看以后，我们发现文件改动量非常大，直接解决冲突并非易事。所以我们还是得 **想办法取消掉这次 revert 的 commit，再进行 merge**。

我们知道，代码回滚有三种方式：reset、checkout，还有我们的 revert。直观感受，我们应该在 reset 上想办法。

我们来看看 reset 有些怎样的操作方法。

![](https://upload-images.jianshu.io/upload_images/3994917-a6094e7c28580e87.png)

主要想给大家讲讲：--soft 和 --hard 的区别。

我们经常会用到 `git reset --hard <commit>` 做「毁尸灭迹」的操作，常常爽到不能自已，因为这不仅可以回退到我们想要的版本，而且还「直接丢弃」了后面提交的代码，真正的「毁尸灭迹」级别的操作。

而另外一个 --soft 处理，实际上还具备点人性，虽然同样可以回退到我们想要的版本，但目标版本后面的提交都还会存放在 stage 区域中，以便后面找出证据。

说到这，似乎我们已经有了思路。

1. 使用 `git reset --soft <revert 操作的 commit ID>` 回退到 revert 操作的版本；
2. 使用 `git reset --hard <revert 操作的前一个 commit>` 干掉那次 revert 提交；
3. 最后再把 stage 区域的所有改动汇聚成一个新的提交 commit 到我们的项目仓库中。

**当然，细心的你一定会发现，在第 1 步操作后，我们还必须执行 `git stash` 命令把所有的改动存到暂存区，再在第 2 步操作后使用 `git stash pop` 命令取出来，直接进行第 2 步操作肯定还是会毁灭证据的。**

### 我们后面的提交不见了。

这样似乎可以解决我们的问题，不过有个弊端：**我们后面那么多的提交被合并成一个提交了，以后我们就没办法看到了，万一...**

不少小伙伴会想到进阶方案：

1. 对 "feature8.29.0_nanchen" 的最新代码 checkout -b 一个分支 feature_copy；
2. 然后使用 `git checkout feature8.29.0_nanchen` 回到我们的分支；
3. 然后直接对当前分支 reset 到 revert 的前一个 commit 后，我们采用 cherry-pick 方式进行傻瓜式改写便可以把历史重写了。（谁说的我们不能改写历史？）

### 改写历史？

改写历史？等等，好像还有一个操作：rebase。

rebase 是 Git 的一个神奇的命令，前面我也说了，总会有人不喜欢 merge 之后历史的分叉，这种分叉再汇合后会让结构看起来非常混乱，以致于无法管理。如果你不喜欢 commit 历史出现分叉，那 rebase 绝对是你的救星。

改写历史是 rebase 与生俱来的能力。我们可以用 `git rebase -i <commit>` 进行历史的改写。

我们试试看在我们的项目中直接使用 `git rebase -i <commit>` 会怎样。

![](https://upload-images.jianshu.io/upload_images/3994917-1205e5e0223414bf.png)

我们会拿到分支后面的提交历史，并且前面还有一个 Commands。我们可以从提示中看到，上面全写的 pick 就是代表保持这个提交的意思，edit 代表编辑此次提交...

我们希望删除此次 revert 这次提交，那当然我们最关心的就是 drop 了，**甚至我们可以更加简单粗暴：直接删掉这一行**。

然后我们便开始处理了。

![](https://upload-images.jianshu.io/upload_images/3994917-f9d22eba92500ea3.png)

过程中可能会出现冲突，我们只需要解决就好。

解决掉冲突后，再使用 `git add <filename>` 把它们 merge 进去。

![](https://upload-images.jianshu.io/upload_images/3994917-2032e702850f10ac.png)

oh，我们看到我们已经 rebase 成功了。我们再使用 `git lg` 查看一下提交历史。

![](https://upload-images.jianshu.io/upload_images/3994917-1980d1692a078c41.png)

**我们成功改写了历史！**

历史改写结束，我们还要做我们最开始想做的事情，进行 merge 操作。

![](https://upload-images.jianshu.io/upload_images/3994917-30868d63436765e2.png)

可以看到，这次我们 merge 确实如我们预期的不再发生冲突，方案亲测有效！

### 写在最后

写了这么多，想必大家对解决方案也算比较清楚了。我们主要便是采用 `git rebase -i <>` 操作进入到 commit 历史编辑页面，然后进行历史改写处理！