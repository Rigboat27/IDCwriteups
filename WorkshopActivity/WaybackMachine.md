
# Wayback Machine

**Category:**  Git
**Difficulty:** Medium (but i'm dumb)
(I was not present at the workshop as I had a MA101 quiz the next day)
I started this challenge with a single file: `activity.tar.gz`. The prompt mentioned something about the "Wayback Machine" and "Git".

### The Archive That Wouldn't Open

My first step was the most obvious one. I have a `.tar.gz` file, so I use the `tar` command to open it.

```bash
tar -xvf activity.tar.gz
```

**Output:**

```
tar: This does not look like a tar archive
tar: Skipping to next header
tar: Exiting with failure status
```

The file extension says it is a tarball, but `tar` says it isn't. I thought it wasn't a file at all but just a text file with a URL inside (thinking back to that Wayback Machine hint).

I decided to check what the file actually was using the `file` command.

```bash
file activity.tar.gz
```

**Output:** `activity.tar.gz: gzip compressed data`

Okay, so it is a gzip file. Why did `tar` fail? I decided to uncompress it manually using `gunzip` to see what was inside.

```bash
mv activity.tar.gz layer1.gz
gunzip layer1.gz
```

This gave me a file named `layer1`. I checked it again.

```bash
file layer1
```

**Output:** `layer1: gzip compressed data`

This is where I realized I was being trolled. I unzipped the file, and the result was another gzip file. It wasn't a corrupted archive; it was a file inside a file.

I unzipped it again.

```bash
mv layer1 layer2.gz
gunzip layer2.gz
file layer2
```

**Output:** `layer2: POSIX tar archive`

t was a Tar archive wrapped in two layers of Gzip. I extracted this final layer.

```bash
tar -xvf layer2
```

It created a folder called `activity`. I was in.

### Investigating the Repository

I navigated into the folder `activity` and listed the files.

```bash
ls -la
```

I saw a `.git` directory, which confirmed this was a Git repository. I also saw a `README.md`. I cat'd the readme to see if the flag was just sitting there.

```bash
cat README.md
```

**Output:**

```
# Welcome to Git & Github Workshop

Somethings change, but some don't, right?
```

"Somethings change." That is a clear hint to look at the history.

My first instinct was to run `git log`, but I was worried. Since I extracted this repo raw from a tarball, sometimes the permissions get messed up, or `HEAD` isn't pointing where you think it is.

So instead of trusting the active branch, I went straight to the raw logs. Git keeps a plain-text log of every action in `.git/logs`.

```bash
cd .git/logs
cat HEAD
```

This printed a big list of commit hashes and messages. I scanned through them, looking for anything suspicious. Most messages were boring ("init repo", "add commit push"), but one stood out immediately.

```
3213d768cb13c1656e7522fbf7b344063b43de2c ... commit: see
```

A commit message that just says "see"? And right after it, a commit says "not worthy", and then "original state". This told me a story: someone added a secret file in the "see" commit, realized it was a mistake, and then deleted it in "original state".

The flag wasn't in the current files. It was buried in the past, inside that "see" commit.

### Mining the Git Objects

I needed to see what files existed at that specific point in time. I grabbed the hash for the "see" commit: `3213d768cb13c1656e7522fbf7b344063b43de2c`.

I used a command called `git cat-file`. This is a plumbing command that lets you read the raw data of any object in the Git database.

First, I checked the commit itself to find the "Tree" (the folder structure).

```bash
git cat-file -p 3213d768cb13c1656e7522fbf7b344063b43de2c
```

**Output:**

```
tree 586b7da45f452ad9f8c1f212356de1f7cc327653
parent 1ad3b5...
author Jayesh Puri...
committer Jayesh Puri...

see
```

Okay, the commit points to a Tree with hash `425095...`. I inspected that Tree to see what files were inside.

```bash
git cat-file -p 586b7da45f452ad9f8c1f212356de1f7cc327653
```

**Output:**

```
100644 blob 6975c8f545a33777c3d7457550a08785264d14b8    README.md
100644 blob c8950bb19850438e0b948e910acfef19d084584b    vic.py

```

Both of these were decoys. Outputs:
```
# Welcome to Git & Github Workshop

This was left on purpose. :rofl:    print("this is not that special") 

```
and
```
print("this is not that special") 

```

I realized I might be looking at the wrong commit. I went back to my manual exploration. I  grew weary at this point and decided to bruteforce it a bit. I looked into the `.git/objects` folder.

I learnt that Git stores files in subdirectories based on the first two characters of the hash. I saw a folder named `e1`. I checked inside and saw a file starting with `6e41...`.

After "cat-file"ing almost every object, finally:

```bash
git cat-file -p e16e41f08c0681ff83bf77bcf2d18ff617f81c5b
```

**Output:**

```
print("GIT{secrets_told_can_always_be_retrived}")
```

### Conclusion

I successfully recovered the flag: `GIT{secrets_told_can_always_be_retrived}`

The "Wayback Machine" hint was definitely a metaphor for looking into the past history of the repository. The real challenge was realizing that the `.tar.gz` was double-compressed and then knowing how to dig through Git's internal object database to find deleted files.

 `git cat-file -p` is my key learning, as well as repeated archivals and gunzip for the same.
