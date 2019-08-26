# git cheetsheet

#### CHECK THE COMMITS BETWEEN `tag` AND `master`. 
```bash
git log --pretty=format:"%h%x09%an%x09%ad%x09%s" tag..master
```

#### Check for specific commit `tag` AND `master`.
```bash
git log --pretty=format:"%h%x09%an%x09%ad%x09%s" tag..master | grep 1d6fe8c
```

#### Check number of commits between `tag` AND `master`
```bash
git log --pretty=format:"%h%x09%an%x09%ad%x09%s" tag..master | wc -l
```

#### Undo last commit to local/remote
```bash
git reset HEAD^ --hard
git push origin -f
```

#### Rebase keeping own commits.

```bash
git rebase -i 
```
<!--stackedit_data:
eyJoaXN0b3J5IjpbMjExMTAyODk3NCwxOTcwNzI1MzY1XX0=
-->

#### Stash work alias

```bash
alias gstash='git stash'
```

#### List stashes alias

```bash
alias glist='git stash list'
```

#### Check stash function
```bash
gshow () {
    num=$1
    git stash show stash@{${num:=0}}
```

#### Restore stash # function
```bash
grestore () {
    num=$1
    git stash apply stash@{${num:=0}}
    git stash drop stash@{${num:=0}}
}
```
