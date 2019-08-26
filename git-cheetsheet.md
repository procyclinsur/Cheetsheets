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
gstash () { 
    TOP=$(git rev-parse --show-toplevel)
    git add $TOP/*
    git stash
}
```

#### List stashes alias

```bash
glist () {
    git stash list
}
```
#### Restore stash # alias
```bash
grestore () {
    if [ -z $1 ]; then
        echo specify stash number
        exit 1
    fi
    git stash apply stash@{$1}
    git stash drop stash@{$1}
}
```
