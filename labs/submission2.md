# Lab 2 — Version Control & Advanced Git

## Task 1 — Git Object Model Exploration

### What each object represents

* Blob: stores the content of a file (bytes). It does not store filename or path.
* Tree: stores a directory snapshot: filenames + file modes + pointers to blobs (files) and other trees (subdirectories).
* Commit: stores metadata (author, message, timestamp) and points to a single root tree (snapshot) plus parent commit(s).

### Analysis: how Git stores repository data

Git is a content-addressable object store: each object (blob/tree/commit) is identified by its hash.
A commit represents a full snapshot via a root tree; trees reference blobs/trees, so the repository history is a chain of snapshots linked by parent pointers.

### Commands and outputs

**Create a sample commit**
```bash
echo "Test content" > test.txt
git add test.txt
git commit -m "Add test file"
git log --oneline -1
````

```text
31c58a9 (HEAD -> feature/lab2) Add test file
```

**Inspect tree entries referenced by HEAD**

```bash
git ls-tree HEAD
```

```text
100644 blob a38dbc03f5f790c14c955f7cc235f89bef651095    .gitignore
100644 blob 6e60bebec0724892a7c82c52183d0a7b467cb6bb    README.md
040000 tree a1061247fd38ef2a568735939f86af7b1000f83c    app
040000 tree 47f8b14e105dc6c7b484342b01f7d9e03c452bb4    labs
040000 tree d3fb3722b7a867a83efde73c57c49b5ab3e62c63    lectures
100644 blob 418a98ced2ac70b5bdee0be9732ecdaae7264515    test.txt
```

**Inspect blob content (test.txt)**

```bash
git cat-file -p 418a98ced2ac70b5bdee0be9732ecdaae7264515
```

```text
Test content
```

**Inspect commit object (HEAD)**

```bash
git cat-file -p HEAD
```

```text
tree bc3fdac9ed1c923ddb4d97cc1cbc9eb2f76da041
parent 31c58a917df7998afb6953b5a13b4e7705612974
author Batsiev Oleg <122930346+zv3zdochka@users.noreply.github.com> 1770464616 +0300
committer Batsiev Oleg <122930346+zv3zdochka@users.noreply.github.com> 1770464616 +0300
(gpgsig omitted)
rename 1 lab screenshots
```

**Inspect tree object content**

```bash
git cat-file -p bc3fdac9ed1c923ddb4d97cc1cbc9eb2f76da041
```

```text
100644 blob a38dbc03f5f790c14c955f7cc235f89bef651095    .gitignore
100644 blob 6e60bebec0724892a7c82c52183d0a7b467cb6bb    README.md
040000 tree a1061247fd38ef2a568735939f86af7b1000f83c    app
040000 tree 31b249783e5b1d21eb16fea27bbb1c4a975dea70    labs
040000 tree d3fb3722b7a867a83efde73c57c49b5ab3e62c63    lectures
100644 blob 418a98ced2ac70b5bdee0be9732ecdaae7264515    test.txt
```

### Screenshots
![Task 1 — cat-file](./screenshots/lab_2_1.png)
![Task 1 — ls-tree ](./screenshots/lab_2_2.png)
![Task 1 — (cat-file -p HEAD)](./screenshots/lab_2_3.png)














## Task 2 — Reset and Reflog Recovery

## Task 3 — Visualize Commit History

## Task 4 — Tagging Commits

## Task 5 — git switch vs git checkout vs git restore

## Task 6 — GitHub Community Engagement

## Challenges & Solutions
