# branch rename

## 첫번 째 방법

```shell
git checkout branch-name
```

```shell
git branch -m new-branch-name
```

Then, you can use **git status** to see your new branch name.

## 두번 째 방법

### Step 1: Make sure you are in the master/main branch

If you are not in the master/main branch, then you will need to run  git
checkout master or git checkout main.

### Step 2: Use the -m flag to rename the branch

```shell
git branch -m old-branch new-branch
```

This is what it would look like to rename the test-branch to test-branch2.

To see your new branch name, you can run git branch which will list all of
your branches.
