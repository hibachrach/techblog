---
layout: post
title:  JavaScript Improvements
date:   2020-10-07
---

In this article I go over three things that, in my mind, would make JavaScript
better. None are new ideas. This post is an expansion of a tweet I had when I saw
someone asking about improvements for JS. (though probably are impossible for
various reasons). I'm going to be primarily speaking about browsers and the web,
though much of this might apply to Node.js (though I'm not as familiar with that
area so I can't speak on it confidently).

# Versions for JS

Just as a heads up: I'm not really talking about different ECMAScript versions
(e.g. ES6, ES2019, etc.) here--I'm talking about how most programming languages
refer to versions.

Right now, there are two versions of JS: [Strict mode][strict_mode] and ["sloppy
mode"][sloppy_mode]. Feature detection is done dynamically: if a
script makes use of any feature or change in the language that isn't supported
by the environment running it, it will error, either silently or loudly
depending on what that feature is.

To get around this, developers do one or both of the following:

## Strategy 1: Transpile/polyfill to the lowest common denominator

To get around browsers not supporting features, we use tools like [Babel][babel]
to convert JS making use of newer features to JS supported by all or polyfill
them with. However, this
has a few problems:

1. Most of the time, the transpilation increases file size and parsing time.
   [Here][for_of_example] is a dramatic instance of that ([not that this is
   Babel's fault...][babel_for_of_faq]). This is despite the fact that most
   browsers don't need this extra code.
2. Transpilation requires a target platform. For now, it seems like the defacto
   standard is ES5, but this is already changing: websites that can afford to
   drop older browsers like IE11 are doing so (e.g. GitHub). What is the _right_
   target? This might get harder to answer once IE11 disappears and we truly
   only need to support evergreen (i.e. silently auto-updating) browsers.
   Different browsers might implement features in different orders. Automated
   tools like [`browserslist`][browserslist] reduce the impact of this point,
   but they require upkeep. If a website stops development now but stays online,
   its JS bundle won't get any faster despite the herd of browsers moving to
   support the newer features in the source JS.
3. If one takes shortcuts (via options like "loose mode" for various Babel
   transpilations) you could actually be introducing bugs by fragmenting the
   underling semantics of a particular feature (though I admit this problem is
   not super likely).
4. Transpilation does not get around efforts to dramatically evolve the JS
   language (especially those which remove old baggage). Syntax that is
   fundamentally incompatible with old and seldom-used features simply can't be
   introduced because we [can't break the web][dont_break_the_web].
 

## Strategy 2: Offer different bundles to different platforms based on proxies

The idea is that you can identify what a browser might need based on the version
presented in its "user agent" (UA). There's [a whole article on MDN on why this is a
bad idea in general][mdn_user_agent_sniffing]. However, this hasn't stopped
influential companies like Twitter from [doing it][twitter_bundles].

Google Developers instead [encourages using the support for `<script
type="module">` as a discriminating factor][google_bundles]. This seems a bit
better, but of course this is just one test--[Safari is not an evergreen
browser][safari_not_evergreen] and so despite it supporting modules, we can't
rely on this to check for support for "generally new feature" availability in
the medium or long-term.

## How versioning fits in

As I said at the beginning, there already is a versioning scheme for JS. [Strict
mode][strict_mode] changes the behavior of JS scripts in a backwards
incompatible way: if you had a script that worked in "sloppy mode", it might
break in strict mode.

However, it doesn't look like there are any plans to further extend this
approach. When ["#SmooshGate"][smooshgate] (an incident of browsers accidentally
breaking sites relying on old JS extensions by adding incompatible features)
happened, versioning was suggested by [more][versioning_js_comment_1]
[than][versioning_js_comment_2] [one][versioning_js_comment_3]
[person][versioning_js_comment_4]. After all, with versioned JS, the issue
evaporates. Commenters on Hacker News responded to these suggestions, suggesting
that supporting multiple distinct versions [introduces significant complexity
for developers of JS engines][hacker_news_no_js_engine_forks]. One person even
[noted][hacker_news_its_by_design]
> This has been discussed at length and they have decided not to do it. It's not
> a missing feature, it's by design. 

There are other negatives to versioning expounded on in [this wonderful
article][one_javascript], such as the following quoted here:

> * Engines become bloated, because they need to implement the semantics of all
>   versions. The same applies to tools analyzing the language (e.g. style
>   checkers such as JSLint).
> * Programmers need to remember how the versions differ.
> * Code becomes harder to refactor, because you need to take versions into
>   consideration when you move pieces of code.

I can't speak much to the work of maintaining engines--this is done by engineers
far more skilled than myself. My immediate reaction is that managing different
versions might enable stricter handling of various code, leading to
simplification, though that's probably a naive perspective.

On the topic of remembering how versions differ, I would say this is simple in
comparison to the inconsistent mess of browser compatibility, JS transpilation
configuration, and generally frequent change within the ecosystem (though I will
be the first to say that the last point has been fairly exaggerated). In other
languages, versions change, and this is considered business as usual.

With regards to added difficulty in refactoring, I would say that this again is
probably simpler than other things which we do semi-regularly, such as upgrading
major versions of important libraries (e.g. jQuery, webpack), and is likely able
to be automated. Additionally, the difficulty is highly dependent on the
audacity of those at the reins of JavaScript, who, based on the current
environment, seem unlikely to cause unnecessary upset.

# Everything is an expression

The main area where I wish this were the standard within JS is with `if`/`else`,
`try`/`catch`, and `switch`/`case` statements. This is something that I use very
frequently within Ruby

## Example: `if`/`else`

```javascript
const a =
  if (cond) {
    b
  } else {
    c
  };
```

## Example: `try`/`catch`

```javascript
const a =
  try {
    somethingThatMightFail()
  } else {
    fallbackValue
  };
```

## Example: `switch`/`case`

```javascript
const a =
  switch (b) {
    case 'apple' {
      'fruit'
    }
    case 'broccoli' {
      'vegetable'
    }
    default {
      'unknown'
    }
  };
```

Its possible this would need to use different keywords to replace `case` and
`default` for the sake of JS interpreters and maximizing backwards
compatibility because `case`s function as labels.

## Current proposals

To achieve this, `do` expressions were [proposed 2 years
ago][do_expression_proposal], which satisfy the requirements with slightly
more verbose syntax. E.g. for the `if`/`else`, you'd write
```javascript
const a = do {
  if (cond) {
    b
  } else {
    c
  }
};
```

However, the proposal is still at stage 1 of [the 4-stage TC39 (effective JS
steering committee) process][tc39_process], though [it's still being
discussed][do_exp_june_2020_issue]. Some have asked "why do we need the `do`?"
and make the first syntax (without the explicit `do`) part of the language,
though [this can't be done without interfering with existing uncommon language
features][do_exp_labels] (another example of not being able to change syntax due
to "version constraint")

# Improved caching

I would explain this, but there are actually people [already solving this
problem][pika_dev], and they've put together [this article explaining how
caching can be improved around the web][pika_dev_about].

However, there are still unresolved issues here: how does this work for
different browser targets? It's fine if all libraries are only using
browser-supported features, but we all know that that won't be consistent for
all features across all browsers into the future. A lot of this builds on the
issues presented in the section on versioning above: if there are no ways to
talk consistently about versioning, then it's much harder to solve these
problems in an automated way.

[strict_mode]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Strict_mode
[sloppy_mode]: https://developer.mozilla.org/en-US/docs/Glossary/Sloppy_mode
[babel]: https://babeljs.io/
[for_of_example]: https://babeljs.io/repl#?browsers=ie%2011e&build=&builtIns=false&spec=false&loose=false&code_lz=GYewTgBAFAxiB2BnALhAhhEwICMCUEA3gFAC-QA&debug=false&forceAllTransforms=true&shippedProposals=false&circleciRepo=&evaluate=false&fileSize=false&timeTravel=false&sourceType=module&lineWrap=true&presets=es2015&prettier=false&targets=&version=7.11.6&externalPlugins=
[babel_for_of_faq]: https://babeljs.io/docs/en/faq#why-is-the-output-of-forof-so-verbose-and-ugly
[browserslist]: https://github.com/browserslist/browserslist
[mdn_user_agent_sniffing]: https://developer.mozilla.org/en-US/docs/Web/HTTP/Browser_detection_using_the_user_agent
[twitter_bundles]: https://twitter.com/CharlieCroom/status/1291478104016289799
[google_bundles]: https://web.dev/codelab-serve-modern-code/#using-es-modules-with-babel
[safari_not_evergreen]: https://thingsthemselves.com/reminder-safari-is-not-an-evergreen-browser/
[dont_break_the_web]: https://www.w3.org/TR/html-design-principles/#support-existing-content
[hacker_news_no_js_engine_forks]: https://news.ycombinator.com/item?id=17141024
[versioning_js_comment_1]: https://www.reddit.com/r/javascript/comments/8ln83r/at_todays_tc39_meeting_smooshgate_was_resolved_by/dzhvvhy?utm_source=share&utm_medium=web2x&context=3
[versioning_js_comment_2]: https://news.ycombinator.com/item?id=17141522
[versioning_js_comment_3]: https://news.ycombinator.com/item?id=17141494
[versioning_js_comment_4]: https://dev.to/joelnet/a-pragmatic-solution-to-flatten-proposal-problem-smooshgate-javascript--jal
[hacker_news_its_by_design]: https://news.ycombinator.com/item?id=17143251
[smooshgate]: https://developers.google.com/web/updates/2018/03/smooshgate
[do_expression_proposal]: https://github.com/tc39/proposal-do-expressions
[tc39_process]: https://tc39.es/process-document/
[do_exp_june_2020_issue]: https://github.com/tc39/proposal-do-expressions/issues/49
[do_exp_labels]: https://github.com/tc39/proposal-do-expressions/issues/39#issuecomment-468645498
[bundling_http_1x]: https://stackoverflow.com/a/53690469/3839114
[pika_dev]: https://www.pika.dev/
[pika_dev_about]: https://www.pika.dev/about
[one_javascript]: https://exploringjs.com/es6/ch_one-javascript.html#sec_versioning
