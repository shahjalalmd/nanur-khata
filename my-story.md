# Nanu's Recipe Book: My Git Story

**Repo:** https://github.com/shahjalalmd/nanur-khata

Nanu's handwritten recipe book is tearing, so I decided to digitize it with the help of my cousin Mitu in Chattogram. Nanu has one rule: no recipe can ever be lost. That is exactly what Git is for. This is the story of how I did it.

> Fill in anything marked **[ ]** with your own details. Replace each screenshot placeholder with your real image from the `screenshots/` folder.

---

## Chapter 1: The First Recipe

I started by telling Git who I am, then made a home for the book.

```bash
git config --global user.name "shahjalalmd"
git config --global user.email "mdjalal6791@gmail.com"
mkdir nanur-khata
cd nanur-khata
git init
```

Nanu said, "Winter is coming, write the bhapa pitha first." I created `bhapa-pitha.md`, checked `git status` (the file showed up as untracked, in red), and staged it with `git add`.

```bash
git status
git add bhapa-pitha.md
```



Then Nanu corrected me: "One and a half cups of gur, not two!" I edited the file and compared the two kinds of diff.

```bash
git diff
git diff --staged
```


**The difference:** `git diff` shows changes that are not staged yet, comparing my working folder to the staging area. `git diff --staged` shows what is already staged and will go into the next commit, comparing the staging area to the last commit. So after I edited the file, `git diff` showed Nanu's correction, and `git diff --staged` still showed my old version until I ran `git add` again.

I staged the fix and committed:

```bash
git add bhapa-pitha.md
git commit -m "Add bhapa pitha recipe with corrected gur quantity"
```

(commit `4ff44bd`)

Then I made a mistake on purpose: I staged `bazar-list.txt` by accident and took it back out without deleting it.

```bash
git add bazar-list.txt
git reset bazar-list.txt
git status
```


The file went back to "untracked" and was still on my disk, so nothing was lost.

---

## Chapter 2: The Secret Spice

Nanu's secret masala blend must never reach GitHub. I created the secret file, some `.tmp` files and a `draft/` folder:

```bash
echo "Nanu's secret blend" > secret-masala.txt
touch notes.tmp test.tmp
mkdir draft
echo "work in progress" > draft/halua-draft.md
```

Then I wrote a `.gitignore` to keep all of them out:

```bash
printf "secret-masala.txt\n*.tmp\ndraft/\n" > .gitignore
git status
```

As proof, `git status` listed only `.gitignore` and `bazar-list.txt`. The secret file, the `.tmp` files and `draft/` were all invisible to Git.


```bash
git add .gitignore bazar-list.txt
git commit -m "Add .gitignore to protect Nanu's secret masala and add bazar list"
git ls-files
```

(commit `cdf483a`)

`git ls-files` shows only `.gitignore`, `bazar-list.txt` and `bhapa-pitha.md`, so the secret masala is not tracked.

---

## Chapter 3: The Eid Special

I wanted to test the Eid recipes without touching the main book, so I made a branch and switched to it:

```bash
git branch eid-special
git checkout eid-special
```

I added two recipes in two separate commits:

```bash
git add biryani.md
git commit -m "Add Eid biryani recipe"          # 1556fbc
git add shemai.md
git commit -m "Add Eid shemai recipe"           # db8156f
```


Then I went back to `main` and checked: **[write what you saw: were biryani.md and shemai.md there before the merge?]** Files created on a branch only exist on that branch until it is merged. After that I merged:

```bash
git checkout main
git merge eid-special
git log --oneline
```


### Challenge: a merge conflict on purpose

I created two branches that changed the same line of the same file in different ways.

```bash
git checkout -b conflict-test
echo "Use 1 cup gur for a lighter taste" >> bhapa-pitha.md
git commit -am "Make pitha lighter on conflict-test branch"     # b3be6b8

git checkout main
echo "Use 2 cups gur for a sweeter taste" >> bhapa-pitha.md
git commit -am "Make pitha sweeter on main"                     # 7fd1551

git merge conflict-test
```

Git stopped and reported a CONFLICT, because it could not decide which version of the line to keep. It put markers in the file:

```
<<<<<<< HEAD
(my version from main)
=======
(the version from the other branch)
>>>>>>> conflict-test
```


**How I resolved it:** I opened the file, decided on the final wording, deleted the three marker lines, then staged and committed:

```bash
git add bhapa-pitha.md
git commit -m "Resolve gur conflict between main and conflict-test"      # 2732d2d
```

> Mistake worth telling: at one point I ran `git branch -M main` while I was on `conflict-test`. That renamed the wrong branch and I got confused about which branch was which. I fixed it by reading `git log --oneline` on each branch and `git branch -a`, then recreated the `conflict-test` branch at the right commit. Lesson: always check the branch name in the prompt before renaming.

**[Optional: if you redid the conflict with halua.md to get a real merge commit, describe it here.]**

---

## Chapter 4: Mitu Joins In

Mitu wants to write in the book from Chattogram, so I put the repo on GitHub.

```bash
git remote add origin https://github.com/shahjalalmd/nanur-khata.git
git push -u origin main
git push origin eid-special conflict-test
```

> My first push failed with "Repository not found" because I had not created the repo on GitHub yet. After creating it, the push worked.


### Playing Mitu

```bash
cd ~/Downloads
git clone https://github.com/shahjalalmd/nanur-khata.git mitu-khata
cd mitu-khata
printf "# Chittagong Mezban\n" > chittagong-mezban.md
git add chittagong-mezban.md
git commit -m "Add Chittagong mezban recipe"
git push
```


### Back in my folder: fetch, then merge

```bash
cd ~/Downloads/nanur-khata
git fetch origin
git log --oneline origin/main
git merge origin/main
```

After `fetch`, Mitu's commit was visible in `origin/main`, but `chittagong-mezban.md` was not in my folder yet. After `merge`, the file appeared.


### Mitu pushes again, this time I use pull

```bash
# in mitu-khata:
printf "# Panta Ilish\n" > panta-ilish.md
git add panta-ilish.md
git commit -m "Add panta ilish recipe"
git push

# in nanur-khata:
git pull
```


### Question: fetch + merge vs pull

`git fetch` only downloads the new commits from the remote. It does not change my working files, so I can look at what Mitu did (`git log origin/main`, `git diff origin/main`) before deciding. `git merge origin/main` then brings those changes into my branch. `git pull` does both steps in a single command. It is faster, but I do not get a chance to review the changes first, and it can surprise me with a merge or a conflict.

---

## Chapter 5: Half-Cooked Halua and Time Travel

### Task A: stash

Halfway through the halua recipe, Nanu called: "You forgot cardamom in the biryani!" I did not want to commit half-finished work, so I stashed it.

```bash
printf "# Halua\n- 1 cup suji\n- 1 cup sugar\n" > halua.md
git add halua.md
git stash
echo "- 4 cardamom pods" >> biryani.md
git commit -am "Add missing cardamom to biryani"
git stash pop
```


The stash put my halua work in a safe drawer, the working folder went back to clean, I fixed and committed the biryani, and `stash pop` brought the halua work back.

### Task B: organizing with git mv and git rm

```bash
mkdir pitha main-dish dessert
git mv bhapa-pitha.md pitha/
git mv biryani.md main-dish/
git mv chittagong-mezban.md main-dish/
git mv shemai.md dessert/
git commit -m "Organize recipes into category folders"

git rm old-recipe.md
git commit -m "Remove old recipe note"
git log --stat -M
```


`-M` makes Git detect renames, so the moved files show up as renamed (R100) instead of one file deleted and another added.

### Task C: git show

```bash
git log --oneline
git show 1556fbc
```


`git show` displays the commit's author, date, message and the exact lines it changed.

### Task D: reset --hard on a test branch

```bash
git checkout -b test-reset
git log --oneline
git reset --hard HEAD~3
git log --oneline
git checkout main
```


**What I lost on `test-reset`:** the last 3 commits and the files and changes inside them (**[list the exact commits/files you saw disappear]**). They were still safe on `main`, and `git reflog` could bring them back.

### Question: why are rebase and reset --hard dangerous on a shared branch?

Both rewrite history. On a shared branch my teammates already have the old commits. After I rewrite them, my history and theirs no longer match, so I would have to `git push --force`, which can overwrite their work. They would end up with duplicate commits or confusing conflicts when they pull. That is why I only practiced `reset --hard` on a throwaway test branch and never on `main`.

---

## Final Check

```bash
git log --oneline --graph --all
git branch -a
git ls-files | grep secret
git log --oneline | wc -l
```


- Total commits: **[ ]** (at least 10)
- Branches: `main`, `eid-special`, `conflict-test`, `test-reset` (at least 2)
- `git ls-files | grep secret` printed nothing, so `secret-masala.txt` was never committed. Nanu's secret is safe.

## What I learned

Git is like Nanu's recipe book with a time machine: I can try new things on a branch, undo mistakes, work with Mitu from another city, and never lose a recipe.
