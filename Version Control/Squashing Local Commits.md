Squashing Local Commits

# Squashing All Local Commits Into One

## Start the Rebase
```powershell
git rebase -i master
```

## Edit Text

Set **pick** for the **oldest commit**

Set **squash** for **all other commits**

```bash
pick 1914635 change 1
squash 6929064 change 2
squash 65677aa change 3
squash 510a97b change 4

# or you can use p and s for pick and squash
# p 1914635 change 1
# s 6929064 change 2
# s 65677aa change 3
# s 510a97b change 4
```