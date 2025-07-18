# jj-todo

A todo-list wrapper for [jj-vcs](https://github.com/jj-vcs/jj)

## Motivation

jj is a fantastic version control system which I've been happily using for some time. After using the [squash workflow](https://steveklabnik.github.io/jujutsu-tutorial/real-world-workflows/the-squash-workflow.html), I realized I could make a bunch of temporary changes after the described change and use it like a todo list, then squash them down when I was finished.

Todo lists work best when you can check items off, and see how much stuff you've accomplished, and this simple wrapper bash script hooks into those motivating qualities to make the workflow even better!

## Contrived Example

For sake of a contrived example, let's add a post to a hypothetical blog engine. And again, for the sake of argument, the change I want to eventually make is, "Create Post schema backend".

I find really granular todo lists helpful for keeping myself on track, so I might make the following todo lists for each of those chages:

* Create Post schema backend
  - [ ] migration
  - [ ] schema
  - [ ] changeset
  - [ ] boundary functions
 
With `jj-todo` I can manage this todo list right in `jj` as I work, without having to jump into some other todo list app. I will check off the todo items until I'm all done and ready to squash them all down into the "real" change.

I will quickly introduce the commands and then walk you through an example.

### `jt new` makes new todo changes

`jt new` is a wrapper for `jj new` and `jj describe` to make todo-style changes quickly. It takes whatever you pass to it, appends `"- [ ] "` to it, and creates a new change.

### `jt log` views todos

`jt log` is a wrapper for `jj log` with a custom template to make clean a clean todo list. Any arguments you pass to it will be passed along to `jj log`. 

### `jt toggle` toggles a todo item done or not

`jt toggle` is a wrapper for `jj describe` that changes `"- [ ]"` to `"- [x]"` and back again. Yes, all it does is alter the message by one character. Do not underestimate the power of that dopamine hit when you complete a todo, and the motivation factor when you see your todo list being checked off.

### Let's make some todos

Here's a how you'd put it all together in a terminal, with some of the output truncated

```shell
# First let's make the "real" change
$ jj new -m "Create Post schema backend"

# Now let's quickly make some todo items
$ jt new migration
$ jt n schema
$ jt n changeset
$ jt n boundary functions

# Let's see the todo list so far
$ jt log
@  mwm ○ - [ ] boundary functions
○  kzp ○ - [ ] changeset
○  nkz ○ - [ ] schema
○  rnx ○ - [ ] migration
○  zwv ○ Create Post schema backend
◆  zzz ○

# If we want, we could also create the other change and todos but I'll skip that for brevity
#
# Let's get to work instead on the migration
$ jj edit r
Working copy  (@) now at: rnxoxyxq 7574136f (empty) - [ ] migration

# Make a migration
$ echo 'create table("posts")' >migration.exs

# Nice work, let's mark that done!
$ jt toggle
# And check our progress... "l" is an alias for log
$ jt l
○  mwm ○ - [ ] boundary functions
○  kzp ○ - [ ] changeset
○  nkz ○ - [ ] schema
@  rnx   - [x] migration
○  zwv ○ Create Post schema backend
◆  zzz ○

# Do some more work...
$ jj edit n
$ echo 'schema("posts")' >posts/post.ex
$ jt t # "t" is an alias for toggle
$ jj edit k
$ echo 'def changeset(%Post{}, attrs)' >>posts/post.ex
$ jt t

# Such productivity!
$ jt l
○  mwm ○ - [ ] boundary functions
@  kzp   - [x] changeset
○  nkz   - [x] schema
○  rnx   - [x] migration
○  zwv ○ Create Post schema backend
◆  zzz ○

# Let's finish up...
$ jj edit m
$ echo 'def list_posts()' >posts.ex
$ jt t
$ jt l
@  mwm   - [x] boundary functions
○  kzp   - [x] changeset
○  nkz   - [x] schema
○  rnx   - [x] migration
○  zwv ○ Create Post schema backend
◆  zzz ○

# And squash!
$ jj squash --from zw..m --into zw

# Et voila
$ jj log
@  pylmzuxk rschenk 2025-07-17 20:47:46 8a484fc9
│  (empty) (no description set)
○  zwvyouou rschenk 2025-07-17 20:47:43 70d74463
│  Create Post schema backend
◆  zzzzzzzz root() 00000000
```

## Documentation

### `jt new` or `jt n`

Makes a change with the provided description preceeded by `"- [ ] "`

If the current jj change has a description, creates a new change after the current one.

If the current jj change has no description, sets the description to the message you provide.

#### Examples:

```shell
$ jt new do the things
# if the current change has a description,
# equivalent to jj new -m "- [ ] do the things"

$ jt new do the things
# if the current change has NO description,
# equivalent to jj describe -m "- [ ] do the things"
```

Quickly blast out a todo list:

```shell
$ jt n fix the bug
$ jt n update the tests
$ jt n touch up docs
```

Insert a todo before another:

```shell
$ jj new --before xyz
$ jt n oops forgot that thing
```

### `jt toggle` or `jt t`

Toggles a todo item done by changing `"- [ ]"` to `"- [x]"` (and vice versa)

It will only alter the description if it starts with `"- [ ]"` or `"- [x]"`. All other descriptions will be unaltered/ignored.

#### Examples:

```shell
$ jt log
○  tqs ○ - [ ] fix event handler
@  rws   - [ ] ui matches mockup
○  mmy   - [x] update validations

$ jt toggle
$ jt log
○  tqs ○ - [ ] fix event handler
@  rws   - [x] ui matches mockup   # <-- Notice it checked this one
○  mmy   - [x] update validations
```

### `jt log` or `jt l`

A wrapper for `jj log` with a more streamlined template.

Any args you pass will be forwarded along to `jj log`.

#### Examples:

```shell
$ jt log
$ jt log --reversed --limit 10
```
