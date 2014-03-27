Git Internals
=============

What you will understand by the end of this talk.  

  - **git init:**     start the base of a repository
  - **git add:**      add files to the staging area
  - **git commit:**   wrap the staging area in a commit
  - **git status:**   show staging area and current direcory
  - **git log:**      show logs from previous commits 
  - **git branch:**   create branches 
  - **git checkout:** move HEAD
  - **git reset:**    generic move branch command
  - **git clone:**    clones a remote repository
  - **git push:**     sends objects to a remote
  - **git pull:**     request new objects from a remote

We're going to start by making a directory to put the repo in.

```
mkdir git-internals
cd git-internals
```

Now let's make this a git repository.

`git init`

But what did that really do?

`ls -a`

So we see that it made a .git directory. Nothing magical. A git repo is simply a structured filesystem.

From now on ---|--- represents a blank repository.

<pre>
---|---
</pre>

Let's add something to this repo to add content.

```
echo "content"  file
git add file
```

What did that command do?

<pre>
   *
---|---
</pre>

The conetent has been staged and added to the repository.

Now we should commit it.

`git commit -m "commited content"`

<pre>
   a (master)(HEAD)
---|---
</pre>

The first commit in a repo is special in that it creates the branch master when you do so.

Lets do that again.

```
echo "more" >> content
git add content
```

<pre>
   *
   |
   a (master)(HEAD)
---|---
</pre>

`git commit -m "added more content"`

<pre>
   b (master)(HEAD)
   |
   a
---|---
</pre>

As you can see the id of the new commit is different from the previous; commit ids can never collide.
lets branch

`git branch feature`

<pre>
   b (master) (feature*) (HEAD)
   |
   a
---|---
</pre>

Now that we have multiple branches we should commit again to see what happens.


```
echo "branchs!" >> content
git add content
```

<pre>
   *
   |
   b (master) (feature*) (HEAD)
   |
   a
---|---
</pre>

`git commit -m "added more BRANCHES"`

<pre>
   c (feature*) (HEAD)
   |
   b (master)
   |
   a
---|---
</pre>

Now that we have tested our feature we should merge back into master. First we have to move onto master though.

`git checkout master`

<pre>
   c (feature)
   |
   b (master*) (HEAD)
   |
   a
---|---
</pre>

Now we merge in changes from feature now.

`git merge feature`

<pre>
   c (feature) (master*) (HEAD)
   |
   b
   |
   a
---|---
</pre>

We should move back a commit to make the demonstration cleaner

`git reset --hard HEAD^` HEAD^ means one commit behind HEAD

<pre>
   c (feature)
   |
   b (master*) (HEAD)
   |
   a
---|---
</pre>

Now lets make a commit on master

```
echo "new commit on master" > master_stuff
git add master_stuff
```

<pre>
(feature)c   *
          \ /
           b  (master*) (HEAD)
           |
           a
        ---|---
</pre>

`git commit -m "commit on master"`

<pre>
(feature)c   d (master*) (HEAD)
          \ /
           b
           |
           a
        ---|---
</pre>

`git checkout feature`

<pre>
(feature*)(HEAD)c   d (master)
                 \ /
                  b
                  |
                  a
               ---|---
</pre>

`git merge master`

<pre>
   e (feature*)(HEAD)
  / \
 c   d (master)
  \ /
   b
   |
   a
---|---
</pre>

`git checkout master`

<pre>
   e (feature)
  / \
 c   d (master*)(HEAD)
  \ /
   b
   |
   a
---|---
</pre>

`git tag v1.0`

<pre>
   e (feature)
  / \
 c   d (master*)(HEAD) [v1.0]
  \ /
   b
   |
   a
---|---
</pre>


Start networking.

<pre>
Main:
   e (feature)
  / \
 c   d (master*)(HEAD) [v1.0]
  \ /
   b
   |
   a
---|---
</pre>

```
cd ..
git clone --bare git-internals server
cd server
ls
```

This is what github looks like

<pre>
Main:                        Server:
   e (feature)                   e (feature)
  / \                           / \
 c   d (master*)(HEAD) [v1.0]  c   d (master*)(HEAD) [v1.0]
  \ /                           \ /
   b                             b
   |                             |
   a                             a
---|---                       ---|---
</pre>


```
cd ..
git clone server peer
cd peer
```

<pre>
Main:                        Server:                        Peer:
   e (feature)                   e (feature)                   e (feature)(origin/feature)
  / \                           / \                           / \
 c   d (master*)(HEAD) [v1.0]  c   d (master*)(HEAD) [v1.0]  c   d (master*)(origin/master)(HEAD)(origin/HEAD) [v1.0]
  \ /                           \ /                           \ /
   b                             b                             b
   |                             |                             |
   a                             a                             a
---|---                       ---|---                       ---|---
</pre>


```
git checkout feature
echo "stuff from peer" > peer_stuff
git add peer_stuff
```

<pre>
Main:                        Server:                        Peer:
                                                               *
                                                               |
   e (feature)                   e (feature)                   e (feature*)(origin/feature)(HEAD)
  / \                           / \                           / \
 c   d (master*)(HEAD) [v1.0]  c   d (master*)(HEAD) [v1.0]  c   d (master)(origin/master)(origin/HEAD) [v1.0]
  \ /                           \ /                           \ /
   b                             b                             b
   |                             |                             |
   a                             a                             a
---|---                       ---|---                       ---|---
</pre>

`git commit -m "made code changes"`

<pre>
Main:                        Server:                        Peer:
                                                               f (feature*)(HEAD)
                                                               |
   e (feature)                   e (feature)                   e (origin/feature)
  / \                           / \                           / \
 c   d (master*)(HEAD) [v1.0]  c   d (master*)(HEAD) [v1.0]  c   d (master)(origin/master)(origin/HEAD) [v1.0]
  \ /                           \ /                           \ /
   b                             b                             b
   |                             |                             |
   a                             a                             a
---|---                       ---|---                       ---|---
</pre>


```
cd ../git-internals
git remote add o server
git checkout feature
```

<pre>
Main:                                 Server:                        Peer:
                                                                        f (feature*)(HEAD)
                                                                        |
   e (feature*)(o/feature)(HEAD)          e (feature)                   e (origin/feature)
  / \                                    / \                           / \
 c   d (master) (o/master)(o/HEAD)[v1.0]c   d (master*)(HEAD) [v1.0]  c   d (master)(origin/master)(origin/HEAD) [v1.0]
  \ /                                    \ /                           \ /
   b                                      b                             b
   |                                      |                             |
   a                                      a                             a
---|---                                ---|---                       ---|---
</pre>

```
echo "more original code on feature" >> content
git add content
git commit -m "original edit"
```
<pre>
Main:                                 Server:                        Peer:
   g (feature*)(HEAD)                                                   f (feature*)(HEAD)
   |                                                                    |
   e (o/feature)                          e (feature)                   e (origin/feature)
  / \                                    / \                           / \
 c   d (master)(o/master)(o/HEAD)[v1.0] c   d (master*)(HEAD) [v1.0]  c   d (master)(origin/master)(origin/HEAD) [v1.0]
  \ /                                    \ /                           \ /
   b                                      b                             b
   |                                      |                             |
   a                                      a                             a
---|---                                ---|---                       ---|---
</pre>
