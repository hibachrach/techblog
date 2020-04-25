% A Hierarchy of Documentation
% Harry Bachrach
% Apr 24 2020

"Where should this information live?" is a common question I encounter daily as
a software engineer. I (alone or in consultation with others) will make a
decision and that information needs to persist...somewhere. The goal of course
is to prevent some future engineer (often myself) from encountering some code
and asking "why?" or, even worse, "what?".

I've found there's a sort of hierarchy to the accessibility
of various information that I found useful in sharing. 

In order of most accessible to least, the hierarchy is as follows:

1. Code (identifiers)
2. In-code comments
3. Commit messages
4. Pull/merge requests & issue trackers

This list may be different for other people and may change in time as tools
get better, but for now, this has held true for me.

I'm sure that similar things have also been said before. Shoulders of giants and
all that---would love to read any previous writings on similar topics!

So! Let's get into some of the reasoning...

## Code (Identifiers)

### Accessibility

By far, the most important place to put information is in the code itself. Why?

* It can be easily searched with tools like `grep`, `ag`, `ripgrep`, or your
  favorite file searcher
* It (generally) follows a predictable structure and connections between
  components can be traced (e.g. through the call stack)
* It is _very_ unlikely to be discarded accidentally

### Responsibilities

Because of their inherent visibility, identifiers[^identifiers_meaning] should
always be the first place to put information, especially

* What values represent
* What functions do & how they do it
* (Sparingly) when functions are invoked[^identifiers_when]

### Identifiers and Code Simplification

I'm going to intentionally give you no context for the following code:

```javascript
if (
  userSession['signed']['admin']['update'] ||
  AdminOverrides[userSession['signed']['id']]
) {
  repository.delete();
  return render({
    status: 200,
    body: {
      notice: 'Repository successfully deleted.'
    }
  });
} else {
  return render({
    status: 403,
    body: {
      errors: ['You are not allowed to perform that action.']
    }
  });
}
```

Okay, pretty straightforward. But consider this version instead:

```javascript
const userHasUpdateAccess = (
  userSession['signed']['admin']['update'] ||
  AdminOverrides.update[userSession['signed']['id']]
);

if (userHasAdminUpdateAccess) {
  repository.delete();
  return render({
    status: STATUS_CODES.ok,
    body: {
      notice: 'Repository successfully deleted.'
    }
  });
} else {
  return render({
    status: STATUS_CODES.unauthorized,
    body: {
      errors: ['You are not allowed to perform that action.']
    }
  });
}
```

We've made a few updates. First, we've pulled the condition out into its own
variable. What that's done is document the intention of what the condition is
*supposed* to mean. This has a few benefits:

* **Each line is doing less, cognitively.** That means you have to think about
  less when reading. You're going to read code way more than you're going to
  write it---why make it harder for yourself?[^code_boring_good]
* **Pulling that connection out and labeling it makes it harder to miss when
  changes happen.** At some point, how we determine whether a user has
  update access will probably change. Additionally, the body of the first branch
  of the `if` may get much larger. It's always easier to keep things tidy one
  piece at a time (as opposed to going back and cleaning it up later)!

The other two changes are to replace the HTTP status codes (which are
essentially [magic numbers][magic_numbers]) with references to a global
`STATUS_CODES` object. While HTTP status codes are fairly well known (e.g. a
"404" is meaningful to many that have never done web development), they're
likely not going to be better known than English.

Generally, breaking things down like this makes the answer to "why?" very
obvious.

### Caveat: Enigmatic language syntax

Unlike libraries which can often be worked around, specific hard-to-parse
features of a language's syntax may be unavoidable. For example, most languages
have regular expressions which are infamous for how easy they are to mess up.
Consider the following regular expression in JavaScript:

```javascript
const IS_VALID_EMAIL_REGEX = /^[^@]+@[^@]+$/;
```

There's not really much more we can do in terms of adding/changing identifiers
to clarify how the regular expression determines which email addresses are valid
and which aren't--you just have to know the specific syntax of (JavaScript)
regular expressions. In cases like these, an in-code comment (which we'll talk
about more in the next section) is the next best place to put this information:

```javascript
// Matches one or more non-`@` characters, folloed by a `@` character, followed
// by one or more non-`@` characters.
const IS_VALID_EMAIL_REGEX = /^[^@]+@[^@]+$/;
```

## In-code comments

### Accessibility

One of the points made about code also apply to in-code comments: they are
easily searchable using a file searcher. Also, they do not need any further
seeking out than code,[^dimmed_comments] unlike something like commit messages.

However, as they are in a non-programming language, they can be a bit less
predictable in terms of structure---you'll have a harder of a time breaking an
English sentence into an abstract syntax tree than compared with something like
JavaScript!

Additionally, in-code comments are related to their subjects
typically by proximity alone. If someone moves that code without taking the
comment with it, the comment ceases to be helpful (or, worse, becomes
confusing/misleading). We talk about this more in the ["Pitfall"
section](#common-pitfall-poor-comment-location) below.

### Responsibilities

Comments, while still being very visible, afford a much greater level of
flexibility in comparison with code identifiers for documenting behavior,
decisions, etc. Comments are where you put

* Explanations of why a section of code is the way that it is
* Explanations of why a section of code _isn't_ written another way **if one may
  be tempted to rewrite it** 
* Contextual information for where a code snippet came from if possibly helpful
  to future readers[^snippet_came_from]
* Explanations of [enigmatic language syntax](#caveat-enigmatic-language-syntax)
* (Sparingly) warnings and assumptions[^warnings_and_assumptions]

### Common Pitfall: Poor Comment Location

For example, say we have this method:

```javascript
// Must be done in reverse order to be done in O(N) time due to
// Kleppner's Law of Stupid Data Structures
function smooshify(newElements) {
  for (let i = newElements.length - 1; i >= 0; i--) {
    this.flarbo.insert(newElements[i])
  }
}
```

At some later point, it turns out are some `null` elements that need to be
screened out:[^null_screening]

```javascript
// Must be done in reverse order to be done in O(N) time due to
// Kleppner's Law of Stupid Data Structures
function smooshify(newElements) {
  const nonNullNewElements = newElements.withoutNulls();
  for (let i = newElements.length - 1; i >= 0; i--) {
    this.flarbo.insert(nonNullNewElements[i])
  }
}
```

With the code as it is now, another dev may stumble along one day and ask "Is
`withoutNulls` iterating in reverse order?" While this may be a bit of a trivial
example, these things can lead to a lot more headache down the line

The only way to truly avoid mistakes like this is by is by keeping a vigilant
eye on comments. Some techniques I've found to be helpful:

* Keep comments as closely positioned to the code they're describing as
  possible. E.g. if talking about code in the first branch of an `if` statement,
  put that comment **in the first branch**, not above the entire statement:

  ```javascript
  // comment about why `floarp` should be used --- BAD
  if (somethingOrOther()) {
    // comment about why `floarp` should be used --- GOOD
    floarp();
  } else {
    blarg();
  }
  ```
* Make references to specific identifiers in comments as these tend to garner
  more attention and help establish a more explicit relationship between the
  comment and the code

## Commit Messages

Most code that I've interacted with is version-controlled with something like
Git, and if not Git, some other version-control software like Mercurial. I'm
going to be referring to Git exclusively in this section, but I'm sure that
there are similar tools for other version-control software.

### Accessibility

Unlike code or in-code comments, commit messages are not immediately visible
when browsing a codebase. However, they can be made much more easily accessible
with tools like [Fugitive][fugitive] (for Vim) or
[GitLens][gitlens]
(for VSCode). However, they can be searched--just run `git log` and press
<kbd>/</kbd> and _BAM_! You can search through all your commit history (thanks
to [`less`][less])

Unfortunately, commit messages can sometimes be squashed or lost during a
rebase, so the information placed there may not be preserved forever, though
this mostly depends upon the other collaborators working on the repository.

### Responsibilities

Much has been written about [how to write good commit
messages][writing_good_commits], so I won't be too
comprehensive here. However, I'd say the things I'm talking about here are, in
my opinion, the most important things about writing a useful commit message.  As
much as I like consistency, the grammatical aspects of commit message guidelines
are not nearly as helpful as guidelines on the _content_ of the message (as long
as it's readable!).

Commit messages are all about change. A commit itself is an object representing
a change to a codebase and commit messages ought to do the following:

* Briefly summarize what changed, from a high level
* Explain why the changes in the commit are necessary
* Explain why other possible strategies were _not_ attempted, if relevant

These are much easier to accomplish when the commits you make are small and
focused (within reason).[^atomic_commits]

### Tip: using references to other commits

While commit messages allow you to tell a story, any single commit rarely allows
you to tell the whole story. If referencing earlier changes, I highly encourage
you to make such references by using the commit hash. This will enable those
reading those commit messages later to gain the same context you had when
writing the message in the first place with a simple `git show`. A word of
warning however: commit hashes will change when you rebase or amend a commit, so
if referencing an earlier commit in a commit message, make sure to update that
hash if the earlier commit is ever altered.

## Pull/merge requests & issue trackers

These are your GitHubs, your Asanas, your Jiras. I'm lumping issue trackers and
hosted version control together because some of these services function as both
(e.g.  GitHub).

### Accessibility

While there's a large amount of variance here, I always have found messages and
comments to get lost somehow, even when generally good search functionality is
present.  Some services are better than others at this, but even in GitHub
(which automatically doubly-link issues to pull requests), comments will be
collapsed or hidden in large pull requests, making them a pain to excavate.

### Responsibilities

These services are all about facilitating conversation and communication. As
such, their responsibilities are focused on those topics:[^use_of_trackers]

* Documenting why certain changes weren't done and/or aren't worth doing
  * However, these should instead be put in the relevant commit messages if they
    are related to certain changes that _did_ make it into the codebase
* Containing or summarizing deliberation about proposed changes, even when that
  deliberation happens in real life or on something like Slack

## Conclusion

Hope you found this helpful in answering the question "where do I put this
information?". If you have any thoughts, [I'm on Twitter][twitter_profile].

[^code_boring_good]: [This is not a new idea][code_boring_good_search].
[^identifiers_when]: I'll probably make another post about this, but I've found
  answering the question of "when" to be a bit of a slippery slope, especially
  in function names. When you answer the question of "when", you are often doing
  so at the expense of "what". Who hasn't written a function like `onClick` or
  `after_save` that spans 20+ lines, accomplishing many disparate tasks? One
  technique I've employed when these are unavoidable (as these identifiers are
  often dictated by some external API) is to push all sense of "how" out of the
  bodies of these functions---they merely list off the things to do and the "how"
  is pushed into other functions. If you read on, we're trying to lighten
  cognitive load here!
[^identifiers_meaning]: Identifiers include variable names, function names,
  class names/types, operators---basically anything that has a name that you can
  control!
[^null_screening]: If this were real code, I would hope that someone would also
  leave a comment explaining why we are doing this screening!
[^warnings_and_assumptions]: If possible, these should be replaced with good identifier naming, type
    checking, validations, etc.
[^snippet_came_from]: I most often use this for adaptations of code from StackOverflow or blog posts.
[^dimmed_comments]: However, they may be slightly dimmed depending on the code editor.
[^atomic_commits]: I found [this
  article][atomic_commit_article] a great summary of
  the how/why of keeping commits small
[^use_of_trackers]: Of course, these services have usages beyond the topics
  addressed in this post like: 
  * centralizing the agreed upon priority of upcoming work
  * establishing the owner(s) of specific tasks
  * informing those outside the dev team about the progress of tasks, etc.

[magic_numbers]: https://en.wikipedia.org/wiki/Magic_number_(programming)
[twitter_profile]: https://twitter.com/HarryB
[atomic_commit_article]: https://www.freshconsulting.com/atomic-commits/
[code_boring_good_search]: https://www.google.com/search?q=code+boring+good
[fugitive]: https://github.com/tpope/vim-fugitive
[gitlens]: https://marketplace.visualstudio.com/items?itemName=eamodio.gitlens
[less]: https://en.wikipedia.org/wiki/Less_(Unix)
[writing_good_commits]: https://chris.beams.io/posts/git-commit/
