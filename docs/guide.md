# Workflow Guide 

For sake of a contrived example, let's add a post feature to a hypothetical blog engine. And again, for the sake of argument, the change/commit I want to eventually make is, "Create Post backend".

I find really granular todo lists helpful for keeping myself on track, so I might make the following todo lists for each of those chages:

* Create Post backend
  - [ ] make a database migration
  - [ ] write a model file
  - [ ] validations
  - [ ] tests

With `jj-todo` I can manage this todo list right in `jj` as I work, without having to jump into some other todo list app. I will check off the todo items until I'm all done and ready to squash them all down into the "real" change.

## Let's write a todo list

Here's a how you'd put it all together in a terminal, with some of the output truncated.

The first thing I do is create the "real" change, just like in the squash workflow. I am not going to edit this change, it's going to sit there as a placeholder as we work in other changes, then finally we'll squash all of our changes into the "real" one in one whack.

```shell
# First let's make the "real" change
$ jj new -m "Create Post backend"

# And here is our repo:
$ jj log
@  wllrxoyu rschenk 2025-07-19 21:44:48 70156d73
│  (empty) Create Post backend
◆  zzzzzzzz root() 00000000
```

Next we will use `jt new` (alias `jt n`) to quickly create a bunch of empty changes, one for each todo list item.

The output below is truncated for clarity, but `jt new` has the same output as `jj new` 

```command
# Now let's quickly make some todo items
$ jt new db migration
$ jt new model
$ jt new validations
$ jt new tests
```

Now let's see them! You could just use `jj log` but, `jt log` (alias `jt l`) provides a much more compact, list-friendly output with just the essentials:

```command
# Let's see the todo list so far
$ jt log
@  lxz ○ - [ ] tests
○  nvq ○ - [ ] validations
○  zts ○ - [ ] model
○  uyu ○ - [ ] db migration
○  wll ○ Create Post backend
◆  zzz ○
```

You will notice that `jt log` outputs with the most recent commits first, just like `jj log`. I'm used to this output, but you might find it strange to read a todo list from bottom-to-top. 

Any arguments you pass to `jt log` will be delegated to `jj log` so you can easily reverse the list to make it look more like a todo list if you like:

```command
$ jt l --reversed
◆  zzz ○
○  wll ○ Create Post backend
○  uyu ○ - [ ] db migration
○  zts ○ - [ ] model
○  nvq ○ - [ ] validations
@  lxz ○ - [ ] tests
```

## Start working

Alright, we have our list, let's implement the database migration. Use `jj edit` to start working in that change.

```command
$ jj edit u
Working copy  (@) now at: uyuyplzq 48b6ffe8 (empty) - [ ] db migration
Parent commit (@-)      : wllrxoyu 70156d73 (empty) Create Post backend


$ echo 'migration' >migration.example
```

Nice work! Let's mark that item done with `jt toggle` (alias `jt t`)...

```command
$ jt toggle
Rebased 3 descendant commits
Working copy  (@) now at: uyuyplzq b6048002  - [x] db migration
Parent commit (@-)      : wllrxoyu 70156d73 (empty) Create Post backend
```

And check our progress:

```command
$ jt l
○  lxz ○ - [ ] tests
○  nvq ○ - [ ] validations
○  zts ○ - [ ] model
@  uyu   - [x] db migration
○  wll ○ Create Post backend
◆  zzz ○
```

So what exactly did `jt toggle`do? All it did was re-write the change description to change `- [ ]` to `- [x]`. That's it. 

But don't underestimate the power of that small dopamine hit you get from crossing off the item. It also helps to keep track of what's in your todo list that's done vs pending, and helps give you a motivation-boosting visual of how much progress you're making.

## Do some more work

Let's pretend that we're going to implement the model and validations:

```command
$ jj edit zt
$ echo 'model' >model.example
$ jt t
$ jt l
○  lxz ○ - [ ] tests
○  nvq ○ - [ ] validations
@  zts   - [x] model
○  uyu   - [x] db migration
○  wll ○ Create Post backend
◆  zzz ○

# do validations next
$ jj edit @+
$ echo "validate" >> model.example
$ jt t
$ jt l
○  lxz ○ - [ ] tests
@  nvq   - [x] validations
○  zts   - [x] model
○  uyu   - [x] db migration
○  wll ○ Create Post backend
◆  zzz ○
```

## Appending todo list items

We're almost done, but let's say we forgot something. We should probably add this to the admin dashboard too. I'm in the flow and don't want to do that right away, but don't want to forget either, so I'll add it to the end.

This uses a different workflow for `jt new`. If `jt new` detects that the current change has a descrition, it delegates to `jj new`, but if the current change is empty and has no description, it will use `jj describe` instead. Let's see how that works by adding a new change at the end of our list.

```command
$ jt l
○  lxz ○ - [ ] tests
@  nvq   - [x] validations
○  zts   - [x] model
○  uyu   - [x] db migration
○  wll ○ Create Post backend
◆  zzz ○

$ jj new --after l
Working copy  (@) now at: xxlmtxso c5c672ef (empty) (no description set)
Parent commit (@-)      : lxzkqksr 2bd93608 (empty) - [ ] tests

$ jt new admin dashboard
Working copy  (@) now at: xxlmtxso bf5d9c95 (empty) - [ ] admin dashboard
Parent commit (@-)      : lxzkqksr 2bd93608 (empty) - [ ] tests

$ jt l
@  xxl ○ - [ ] admin dashboard
○  lxz ○ - [ ] tests
○  nvq   - [x] validations
○  zts   - [x] model
○  uyu   - [x] db migration
○  wll ○ Create Post backend
◆  zzz ○
```

Awesome, now we can resume where we left off with our tests

```command
$ jj edit l
Working copy  (@) now at: lxzkqksr 2bd93608 (empty) - [ ] tests
Parent commit (@-)      : nvqmorwt 1ca94b13 - [x] validations

$ touch test.example
$ jt t
$ jtl
○  xxl ○ - [ ] admin dashboard
@  lxz   - [x] tests
○  nvq   - [x] validations
○  zts   - [x] model
○  uyu   - [x] db migration
○  wll ○ Create Post backend
◆  zzz ○
```

## Inserting items mid-list

Ah crap, I forgot to sanitize our parameters! And since this is a contrived example, I want to do that right after the validations and before the tests! 

No problem!

```command
$ jt l
○  xxl ○ - [ ] admin dashboard
@  lxz   - [x] tests
○  nvq   - [x] validations
○  zts   - [x] model
○  uyu   - [x] db migration
○  wll ○ Create Post backend
◆  zzz ○

$ jj new --before l
$ jt n sanitize params

$ jt l
○  xxl ○ - [ ] admin dashboard
○  lxz   - [x] tests
@  zyo ○ - [ ] sanitize params
○  nvq   - [x] validations
○  zts   - [x] model
○  uyu   - [x] db migration
○  wll ○ Create Post backend
◆  zzz ○

```

## Finish and squash

Let's knock out the last two items on the list and squash it down.

```command
$ echo 'sane params' >>model.example
$ jt t

$ jj edit x
$ echo 'admin post dashboard' >admin.example
$ jt t
$ jt l
@  xxl   - [x] admin dashboard
○  lxz   - [x] tests
○  zyo   - [x] sanitize params
○  nvq   - [x] validations
○  zts   - [x] model
○  uyu   - [x] db migration
○  wll ○ Create Post backend
◆  zzz ○
```

All that's left to do now is squash all our todo changes into the "real" change

```command
$ jj squash --from w..x --into w

# Et voila
$ jj log
@  slyxwtmx rschenk 2025-07-19 22:24:04 a557ad9e
│  (empty) (no description set)
○  wllrxoyu rschenk 2025-07-19 22:24:00 0eb2d77b
│  Create Post backend
◆  zzzzzzzz root() 00000000
```
