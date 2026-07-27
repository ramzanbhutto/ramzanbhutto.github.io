---
title: "Setting Up a Codeforces Problem on Polygon: A Practical Walkthrough"
date: 2026-07-26
tags: ["codeforces", "polygon", "competitive-programming", "testlib"]
author: "Muhammad Ramzan"
description: "Learn how to prepare a Codeforces problem on Polygon, from statement and validator to checker, generators, stress testing, and interactors."
draft: "true"
---

## Why Polygon?

To prepare a problem, you need a statement, a validator, a checker, a generator, tests, and a solution to prove it's all correct. You can hack this together with loose scripts, but it gets messy fast: how do you know a test is valid? How do you know your solution is right before you even have tests? What happens when the generator changes and everything needs to be redone?

Polygon is the tool Codeforces itself uses to solve this. It keeps everything in one place with version history built in.

This post uses [testlib.h](https://github.com/MikeMirzayanov/testlib) for the validator, checker, and generator, same as Codeforces problems do.

## How Polygon thinks

Before touching the UI, know two things about how Polygon works. It will make the rest of this post make sense instead of feeling random.

**Everything has a fixed folder.** Statement, checker, validator, generators, solutions, tests: each goes into its own predefined folder automatically. You never touch this folder directly, you work through the UI, but this structure is exactly what you get when you download a package at the end.

**Nothing is real until you commit.** Polygon has its own built-in version control, simpler than git, no need to learn git for this. You edit something, it stays "uncommitted," and only becomes permanent once you write a commit message and save. If you're working with someone else, uncommitted changes stay invisible to them.

Commit small, commit often. One giant commit at the end tells you nothing when something breaks later. Ten small commits with clear messages tell you exactly where to look.

## The example problem

We will use [1788B - Sum of Two Numbers](https://codeforces.com/problemset/problem/1788/B).

In plain words: you are given a number `n`. Split it into two non-negative numbers `x` and `y`, so `x + y = n`, and the digit sum of `x` and the digit sum of `y` differ by at most 1.

Good problem to learn Polygon with:

- The answer isn't unique, many `(x, y)` pairs work for the same `n`, so we need a **special checker**, not a plain diff.
- Input is small and simple, so the validator stays short.
- Still needs a real generator and real stress testing, so we don't skip anything.

## Creating the problem

Log into Polygon, click **New problem**, give it a name (internal only, participants never see it).

Then you'll see the list of problems and then open the problem that you just created by clicking on **Start** in the Working Copy column.

![open-problem](/open-problem.png)

You land on **General info**. Set:

- Input file: `standard input`, Output file: `standard output`
- Time limit: `1000` ms, Memory limit: `256` MB

Save. You can also add tags here (`constructive algorithms`, `greedy`, `math`, 'implementation', 'probabilities', '1100'), useful later when searching your own problems. **1100** is rating of this problem. Higher the rating, harder the problem.

![general-info](/general-info.png)

## Our first commit
Commit panel looks like this:
![commit-panel](/commit-panel.png)

Now open the commit panel at the bottom of this page. Every line says `ADDED`, since Polygon just generated the default files for a new problem. Click **Commit Changes**, message: `Initial commit`. This is the folder-and-commit idea from above, in action. 
And if you're making minor changes like this one, tick the box also.

![commit-page](/commit-page.png)

The top bar is your map from here on: Statement, Files, Checker, Validator, Tests, Stresses, Solution files, Invocations, Verification, Package, Manage access. We'll go through them one by one.

![top-bar](/top-bar.png)

## Writing the problem statement

Click **Statement**, select English and click Create (Polygon can hold multiple languages per problem). You get four boxes:

- **Legend**: the actual problem description
- **Input format**: what each line of input holds
- **Output format**: what to print
- **Notes**: explains the sample cases (optional but helpful)
- **Tutorial** (optional, hidden from participants): your own notes on the intended solution, useful if you're working with co-authors

![statement](/statement.png)

Use **Edit with preview** at the top, it splits your screen so you see the rendered result as you type.

### TeX basics

Statements are written in TeX. You don't need to know much:

- `$...$` for inline math, `$$...$$` for math on its own line
- `\bf{text}` for bold, `\begin{itemize}\item...\end{itemize}` for bullets
- `\begin{tabular}` inside `\begin{center}` for a table
- `%` at the start of a line is a comment
- Images go through the statement's resource files section, then get referenced in text

Polygon has a short built-in manual right on the page: [polygon-tex-manual](https://polygon.codeforces.com/docs/statements-tex-manual).

> Note the Time limit and Memory limit we set will appear in the problem statement as header, make sure it is right. You can confirm it in the General Info page and **Save** it.

### Writing it for our problem
**Name**
```tex
Sum of Two Numbers
```
**Legend**
```tex
The sum of digits of a non-negative integer $a$ is the result of summing up its digits together when written in the decimal system. For example, the sum of digits of $123$ is $6$ and the sum of digits of $10$ is $1$. Formally, if
$a = \sum_{i=0}^{\infty} a_i \cdot 10^i$, where $0 \le a_i \le 9$, then the
sum of digits of $a$ is defined as $\sum_{i=0}^{\infty} a_i$.

You are given an integer $n$. Find two non-negative integers $x$ and $y$
such that:

\begin{itemize}
\item $x + y = n$, and
\item the digit sum of $x$ and the digit sum of $y$ differ by at most $1$.
\end{itemize}

It can be shown that such $x$ and $y$ always exist.
```

**Input format**
```tex
The first line contain number of test cases $t$ ($1 \le t \le 10\,000$).

Each of the next $t$ lines has one integer $n$ ($1 \le n \le 10^9$).
```

**Output format**
```tex
For each test case, print two integers $x$ and $y$.

If there are multiple answers, print any one.
```

**Notes**
```tex
In the second test case, $n = 161$. One valid split is $x = 67$, $y = 94$.

\begin{center}
\begin{tabular}{|c|c|c|}
\hline
Value & Digits & Digit sum \\ \hline
$x = 67$ & $6, 7$ & $13$ \\ \hline
$y = 94$ & $9, 4$ & $13$ \\ \hline
\end{tabular}
\end{center}

Both sums are equal, so the difference $(13-13)$ is $0$.

In the third test case, $n = 67$. One valid split is $x = 60$, $y = 7$.

\begin{center}
\begin{tabular}{|c|c|c|}
\hline
Value & Digits & Digit sum \\ \hline
$x = 60$ & $6, 0$ & $6$ \\ \hline
$y = 7$ & $7$ & $7$ \\ \hline
\end{tabular}
\end{center}

The two sums differ by $1$ only (as |6-7|=1), which satisfies the condition.
```

![statement-preview1](/statement-preview1.png)
![statement-preview1](/statement-preview2.png)

### Sample Input
```tex
5
1
161
67
1206
19
```

### Sample Output
```tex
1 0
67 94
60 7
1138 68
14 5
```

You won't find a spot to paste the sample input/output table by hand. That comes from the **Tests** page later, where you mark a test as "used in statement." For now you can add just the sample under Tests as:
Open **Tests → Add test**. 
![Add-test](/Add-test.png)
Test type stays **Manual**. test number `1` if its the 1st test case. Paste the sample input in the **Data** box, check **Use in statements**, then click **click here** next to it to reveal the custom content boxes.
![Add-test-sample1](/Add-test-sample1.png)
Paste the matching sample output in **Output in statements**, and leave **Verify output for statements** unchecked for now, since there's no checker yet to verify against. Click **Create**.
![Add-test-sample2](/Add-test-sample2.png)



Use the **In HTML** / **In PDF** buttons on the Statement page to see the final result. Commit: `Add problem statement`.

## Test validation


A validator's job is simple: read an input file and reject it if it breaks the stated constraints. This catches broken tests before they ever reach a solution, whether the test came from your own generator or from a participant's hack attempt during a contest (Codeforces runs hack tests through your validator too).

Open **Validator** in the top bar. Polygon shows a short built-in guide plus example validators here, worth a quick read before writing your own. It is strongly recommended to write validators with testlib.h rather than plain C++, it gives you clean bound-checking and readable error messages for free.

```cpp
#include "testlib.h"
using namespace std;

const int max_t = 10000;
const int max_n = 1000000000;

int main(int argc, char** argv) {
    registerValidation(argc, argv);

    int t = inf.readInt(1, max_t, "t");
    inf.readEoln();

    for (int i = 1; i <= t; i++) {
        setTestCase(i);
        inf.readInt(1, max_n, "n");
        inf.readEoln();
    }

    inf.readEof();
    return 0;
}
```

Give every `read*` call a variable name, it shows up in error messages later. Check whitespace with `readEoln()`/`readEof()`, this matters because some languages (Python, Java) read by lines and break on a trailing space. This whitespace rule is called the **well-formed policy**, and Polygon can enforce it automatically later on the Tests page.

### Testing the validator

Add tests under **Add test**, separated by `===`. Rule of thumb: one test just below each bound, one just above.

```
0
===
10001
===
1
0
===
1
1000000001
===
1
1
===
2
1
10
```

First two test `t`, next two test `n`, last two are normal valid input. Run them, check each verdict.

> 📸 *screenshot: validator test results table*

Commit: `Add validator with validator tests`.

## The checker

The checker decides if the participant's output is correct. Since many `(x, y)` pairs work for the same `n`, we can't diff against a fixed answer, we need a **special checker**.

Polygon ships standard checkers for common cases (`wcmp.cpp` for token comparison, `ncmp.cpp` for numbers, checkers with tolerance for real numbers), worth checking the list before writing your own, but none fit for this problem.

```cpp
#include "testlib.h"
using namespace std;

long long digitSum(long long x){
    long long s= 0;
    while(x>0){ 
      s+= x%10; 
      x/= 10; 
    }
    return s;
}

int main(int argc, char* argv[]){
    registerTestlibCmd(argc, argv);

    int t = inf.readInt();
    for(int i=1; i<=t; i++){
        long long n= inf.readLong();
        long long x= ouf.readLong(0, n, "x");
        long long y= ouf.readLong(0, n, "y");

        if(x+y != n)    quitf(_wa, "on test case %d: x + y != n", i);

        long long diff= llabs(digitSum(x) - digitSum(y));

        if(diff > 1)    quitf(_wa, "on test case %d: digit sums differ by %lld", i, diff);
    }

    quitf(_ok, "all test cases correct");
    return 0;
}
```

`quitf(_wa, ...)` fails with a message, `quitf(_ok, ...)` accepts. `_pe` exists for formatting problems, `_fail` for a jury-side mistake.

One habit worth knowing: write a single function to read an answer, and call it on both `ouf` (participant) and `ans` (jury), instead of writing separate logic for each. This is called the **readAns pattern**. It matters because if your model solution has a bug, a checker that blindly trusts the jury's answer will never catch it, one that reads both sides through the same logic will.

### Testing the checker

Add a correct case, a case where `x + y != n`, a case where digit sums are too far apart, and a case with a missing or extra token. The Add test page has separate boxes for Input, Output, and Answer, plus a button to auto-fill Answer from your model solution once it exists.

> 📸 *screenshot: checker test page and results*

Commit: `Add checker with checker tests`.

## Model solution

Open **Solution files**, add the main solution:

```cpp
/*
* Author: Muhammad Ramzan
*/
#include<bits/stdc++.h>
using namespace std;
#define fastio ios_base::sync_with_stdio(false); cin.tie(NULL); cout.tie(NULL);
using ll=long long;
void solve(){
  ll num;
  cin>>num;
  if(num==1){
    cout<<"1 0\n";
    return;
  }
  std::function<int(ll)> digit_sum= [](ll n){
    int sum=0;
    while(n>0LL){
      sum+= static_cast<int>(n%10LL);
      n/=10LL;
    }
    return sum;
  };
  int low=0, high=static_cast<int>(num);
  while(low<=high){
    int x=low+(high-low)/2;
    int y= static_cast<int>(num)-x;
    int xs=digit_sum(x);
    int ys=digit_sum(y);
    int dif= abs(xs-ys);
    if(dif==0 || dif==1){
      cout<<x<<" "<<y<<'\n';
      return;
    }
    else if(xs>ys) high=x-1;
    else low=x+1;
  }
}
int main(){
  #ifdef TESTING
    freopen("input.txt", "r", stdin);
  #endif
  fastio
  int t;
  cin>>t;
  while(t--){
    solve();
  }
  return 0;
}
```

As `x` grows, `digitSum(x)` trends up and `digitSum(y) = digitSum(n - x)` trends down, so binary search finds where they land within 1 of each other.

Polygon marks the first solution you add as **Main correct solution** automatically, this is the one used to generate real test outputs. Add a couple of wrong solutions too (always print `n 0`, mark it Wrong Answer; a correct-but-slow one, mark it Time Limit Exceeded). These aren't wasted effort, they're what proves your test set actually catches mistakes, in the next steps.

> 📸 *screenshot: solutions page, Main / Correct / Wrong Answer types*

Commit: `Add main solution and two wrong solutions`.

## Stress testing

Run your solution against random small tests, compare its output to another solution written differently. If they disagree, you've found a real bug, in one of them, with proof.

Generators live under **Files**, not Tests. Two things to know about them: they take command-line options via `opt<T>("name")`, so one generator file makes many shapes of test just by changing arguments, and testlib's `rnd` is seeded from those exact arguments, same arguments always make the same test, which is what makes tests reproducible.

```cpp
#include "testlib.h"
using namespace std;

int main(int argc, char* argv[]) {
    registerGen(argc, argv, 1);

    int maxN = opt<int>("max-n");
    int t = opt<int>("t");

    cout << t << "\n";
    for (int i = 0; i < t; i++)
        cout << rnd.next(1, maxN) << "\n";

    return 0;
}
```

```
$ ./gen -max-n 100 -t 5
5
42
7
88
13
100
```

> 📸 *screenshot: files page with gen.cpp uploaded*

Go to **Stresses** → **Add stress test**:

- Script pattern: `gen -max-n 1000 -t [1..20]`, the `[1..20]` picks a random `t` each run
- Time/memory limit for this stress run (can differ from the real problem limits)
- Total time limit, say 60 seconds, how long Polygon keeps generating and comparing
- Which solutions to run against each other

Save and run. If something fails, Polygon shows the exact failing command, including a random string it quietly appends so the seed changes between runs even with the same range. Click into it to see the countertest, the actual input and each solution's output. Fix whatever's wrong, run again until it stays clean.

> 📸 *screenshot: stress test result, and a countertest view*

Commit: `Add generator and stress tests`.

## Building the full test set

A totally random generator is fine for stress testing but too weak for real tests. Make sure you cover: `n = 1`, `n` at the max `1000000000`, `t` at the max `10000`, and a real spread of small and large random values, not just one flavor.

Open **Tests**. Turn on the **well-formed policy** checkbox, so every test gets auto-checked for the whitespace rules from earlier. The **points/groups** checkboxes are for subtasks (IOI-style), skip them for a normal Codeforces problem.

Redo the sample test properly now: same input as before, **Use in statements** checked, but this time leave custom output off, so it's generated for real from your checker and solution.

The script box below is the real test list, one generator call per line, ending in `> $` (auto id) or `> 5` (specific id):

```
gen -max-n 10 -t 5 > $
gen -max-n 1000 -t 100 > $
gen -max-n 1000000000 -t 10000 > $
```

Typing every line by hand doesn't scale. The script box supports [FreeMarker](https://freemarker.apache.org/) loops (manual right next to the box):

```
<#assign sizes = [
  [10000, 1000000000],
  [100, 1000000000],
  [1, 1000000000],
  [10000, 10]
] >
<#list sizes as size>
  gen -max-n ${size[1]} -t ${size[0]} > $
</#list>
```

This makes 4 tests in one loop: max `t` with max `n`, small `t` with max `n`, the smallest case, and max `t` with tiny `n`. Nest two `<#list>` loops if you want every combination of a few `t` values against a few `n` ranges.

> 📸 *screenshot: script box with a FreeMarker loop*

Use **Preview tests** before committing, it shows the start of each test's input/output, and flags anything broken right there in red so you catch it before it becomes part of your real set.

> 📸 *screenshot: test preview, one clean test and one showing an error*

Commit: `Add full test set`.

## Interactors

Easy to skip if your first few problems are plain input/output, worth knowing before you need it. An **interactive problem** is one where the participant's program talks back and forth with a judge program while running, instead of reading one fixed file, think "guess the number" or query-based problems.

1788B isn't interactive, so here's a toy version to show the shape: the judge sends `n`, the participant sends back a final `x, y`.

```cpp
#include "testlib.h"

int main(int argc, char* argv[]) {
    registerInteraction(argc, argv);

    long long n = inf.readLong();
    println(n);          // sent to the participant

    long long x = ouf.readLong();
    long long y = ouf.readLong();

    if (x + y != n)
        quitf(_wa, "x + y != n");

    quitf(_ok, "correct");
    return 0;
}
```

`registerInteraction` instead of `registerTestlibCmd`. Whatever the interactor prints becomes the participant's stdin, live. **Flush after every message on both sides** (`cout.flush()` in C++), or the other side hangs waiting for buffered data. Judge and participant run as two processes connected by pipes, Polygon wires this up once you mark the problem interactive.

Set up: upload the interactor under **Files**, mark it as the interactor, set problem type to **Interactive** in settings. Test the same way as a checker, add an interactor test, run the model solution against it, confirm through Invocation.

> 📸 *screenshot: files page, interactor selected as problem type*

## Final checks and packaging

A few things worth glancing at before you ship:

- **Show warnings** (right panel): flags loose ends, missing variable names, missing tags. Clear them.
- **Review problem**: statement, validator, and checker shown side by side, good for a second pass before packaging.
- **Invocation**: pick solutions and tests, click **Run judgement**. Shows a pass/fail table with time and memory per test, color-coded if something's close to the limit. Private to you, other authors don't see your runs.
- **Verification**: reruns everything and checks each solution's real verdict against the type you tagged it with. If a "Wrong Answer" solution passes everything, verification fails and tells you exactly where, meaning your tests aren't strong enough yet. Fix and rerun.
- **Package**: a zip of the whole problem, same folder structure Polygon uses internally, this is what Codeforces imports directly. **Standard package** has everything needed to regenerate from scratch. **Full package** adds the already-generated tests, bigger but nothing needs to rerun. Must commit before you can package.

> 📸 *screenshot: invocation results, verification pass/fail, package page*

## Publishing to a Codeforces mashup

Grant access first: **Manage access** → **Add users** → type `codeforces`.

Copy the problem's Polygon link from the right-side panel. On Codeforces: **Gym → Mashup → Create new mashup**, paste the link, **Create Mashup Contest**. From there it's a normal Gym contest, participants can submit and you get real data on how your tests hold up.

> 📸 *screenshot: manage access with "codeforces" added, and the create mashup page*

## Wrapping up

Write the statement, lock the input down with a validator, decide what "correct" means with a checker, prove your solution with stress testing, then build a test set strong enough to actually break wrong solutions. Interactive problems just add a program that talks to the participant live instead of reading a file, the core tools stay the same.

- [testlib.h on GitHub](https://github.com/MikeMirzayanov/testlib)
- [Polygon](https://polygon.codeforces.com/)
- [1788B - Sum of Two Numbers](https://codeforces.com/problemset/problem/1788/B)
