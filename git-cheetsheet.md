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
