---
title: "Learning testlib.h: Validators, Generators, Checkers, and Interactors"
date: 2026-10-08
tags: ["testlib", "competitive-programming", "codeforces", "polygon", "cpp"]
author: "Muhammad Ramzan"
description: "Learn testlib generators, validators, checkers, and interactors, then run complete examples and batch test scripts locally."
draft: true
---

When you prepare a programming problem, you need more than a solution. You also need to create inputs, check their constraints, and decide whether a submitted answer is correct.

[testlib.h](https://github.com/MikeMirzayanov/testlib) is a C++ library that helps with these jobs. It also supports interactive problems, where the judge and solution exchange messages while the solution runs.

This post assumes you know basic C++: variables, loops, functions, and input/output. First, we will learn the parts of testlib that we need. Then we will use them in two problems:

1. **Sum of Two Numbers:** a normal problem with more than one valid answer.
2. **Find the Hidden Number:** an interactive problem defined for this tutorial.

## What each program does

| Program | Job |
|---|---|
| Generator | Create input files. |
| Validator | Check input format and constraints. |
| Checker | Judge the participant's answer. |
| Interactor | Exchange messages with the running solution. |

For a normal problem, the process looks like this:

```text
generator → input → validator
               ↓
            solution → output → checker
```

The solution is an ordinary program. It usually does not need testlib.

For an interactive problem, the solution sends a question, the interactor replies, and the conversation continues until the solution gives its final answer. The interactor can judge that answer itself, or write information for a separate checker. [Official examples](https://github.com/MikeMirzayanov/testlib/blob/master/README.md).

## Set up testlib

Create a working folder and download the header into it:

```bash
mkdir testlib-demo
cd testlib-demo
curl -fsSL https://raw.githubusercontent.com/MikeMirzayanov/testlib/master/testlib.h -o testlib.h
```

Keep `testlib.h` beside the source files. Programs using it start with:

```cpp
#include "testlib.h"
using namespace std;
```

Include it before any other headers. No separate library build is needed for these examples; the code is in the header. We will compile with `g++ -std=c++14`. Keep the downloaded header if you want to reproduce the same tests later. [Header source](https://github.com/MikeMirzayanov/testlib/blob/master/testlib.h).

## 1. Generator: build the input

A generator is a program whose output will become a test input. It decides the sizes, values, and patterns used in that input. It should print only input data to standard output.

For example, if a problem reads `n` followed by `n` integers, printing ten integers after `n = 5` creates a broken test. The generator must follow the same format as the statement.

### Start the random generator

Inside `main`, call:

```cpp
registerGen(argc, argv, 1);
```

This prepares testlib's random object, `rnd`, using the command-line arguments. The final `1` selects the random generator version. It is not your seed. Use `rnd` rather than mixing in `rand()` or another random engine if you want the whole test controlled by testlib's seed. [Generator setup](https://github.com/MikeMirzayanov/testlib/blob/master/README.md).

### Understand command-line arguments

Suppose we choose this interface for our program:

```text
./generator t max_n seed
```

Then this command:

```bash
./generator 10 100 42
```

means: generate `10` cases, use an upper limit of `100`, and use `42` as an extra seed argument. These meanings come from our code, not from testlib guessing what the numbers represent.

In `int main(int argc, char* argv[])`:

| Item | Value in this command |
|---|---|
| `argc` | `4`, including the executable name |
| `argv[0]` | `./generator` |
| `argv[1]` | `10` |
| `argv[2]` | `100` |
| `argv[3]` | `42` |

Use `opt` to convert an argument to the required type:

```cpp
int t = opt<int>(1);
int max_n = opt<int>(2);
```

The first line reads `10`; the second reads `100`. `registerGen` already uses the arguments to seed `rnd`, so we do not have to read `42` into a variable to make it affect the seed.

The same executable, arguments, and testlib version reproduce the same generated input. Changing only `42` to `43` gives another random sequence. This is useful when you need to reproduce a failing test.

Named options are another way to design the interface:

```cpp
int t = opt<int>("t");
int max_n = opt<int>("max-n", 100);
```

For this version, use `./generator -t 10 -max-n 500`. If `-max-n` is omitted, its default is `100`. Do not paste these reads into a positional generator without changing its command format. [Named-option example](https://github.com/MikeMirzayanov/testlib/blob/master/generators/gen-array-with-opt.cpp).

### Choose random values with `rnd.next`

These integer calls have different ranges:

| Call | Possible values |
|---|---|
| `rnd.next(5)` | `0, 1, 2, 3, 4` |
| `rnd.next(1, 5)` | `1, 2, 3, 4, 5` |
| `rnd.next(0LL, 1000000000000LL)` | A `long long` from `0` through `10^12` |

With one integer argument, the upper bound is excluded. With two integer arguments, both bounds are included. Use `LL` bounds when the values need `long long`.

`rnd.next(0)` is invalid because its range would be empty. Likewise, the lower bound must not exceed the upper bound in a two-argument call. Check calculated ranges before using them.

You can generate a string from a pattern too:

```cpp
string s = rnd.next("[a-z]{5,10}");
```

Here, every character is lowercase, and the length is between `5` and `10`. testlib has its own pattern syntax; do not assume every feature of a full regular-expression library is supported.

### Generate useful structures

You do not need to write a loop for every kind of random data:

```cpp
vector<int> order = rnd.perm(5, 1);
vector<int> chosen = rnd.distinct(3, 1, 10);
vector<int> lengths = rnd.partition(3, 12);
```

- `perm(5, 1)` returns the numbers `1` through `5` in random order, each used once. `perm(5)` instead uses `0` through `4`.
- `distinct(3, 1, 10)` chooses three different values from `[1, 10]`. They are not necessarily sorted. Asking for eleven different values from this range is impossible.
- `partition(3, 12)` returns three positive integers whose sum is exactly `12`. For example, `2 7 3` satisfies those conditions, but that is not a promised output. `partition(3, 12, 0)` allows zero parts instead.

`partition` is useful when a statement limits the total length across cases. It controls the total, but it does not automatically enforce an upper bound on each individual part. Choose parameters or a construction that satisfies both constraints.

To randomize an existing vector:

```cpp
shuffle(order.begin(), order.end());
```

This is testlib's `shuffle`, using its random generator. These helpers construct values; print them in the problem's input format yourself. [Random helper implementations](https://github.com/MikeMirzayanov/testlib/blob/master/testlib.h).

### Control the distribution with `rnd.wnext`

Uniform random numbers can miss the kinds of sizes you want to emphasize:

```cpp
int large = rnd.wnext(1, 100, 3);
int small = rnd.wnext(1, 100, -3);
```

A positive final argument favors larger values; a negative one favors smaller values. Zero gives the ordinary uniform behavior. All these calls still stay in `[1, 100]`. [Bias example](https://github.com/MikeMirzayanov/testlib/blob/master/generators/gen-array-with-opt.cpp).

Bias does not guarantee a boundary. If you need a test with `n = 100`, set `n = 100`. A useful test set combines minimum values, maximum values, hand-chosen patterns, and random cases.

## 2. Validator: read the input and enforce the statement

A validator answers one question: **does this file satisfy the input specification?** It reads through `inf` after:

```cpp
registerValidation(argc, argv);
```

A normal solution often assumes that the input is valid. A validator has to check that assumption, including the format.

### Read integers with bounds

```cpp
int n = inf.readInt(1, 100, "n");
```

Read this as: “consume the next integer, require it to be in `[1, 100]`, and call it `n` in diagnostics.”

The function returns the value, so later checks can use it. `0`, `101`, or a token such as `hello` fails validation. For larger values, use:

```cpp
long long cost = inf.readLong(0LL, 1000000000000000000LL, "cost");
```

The bounds are inclusive. The name is for error messages; it does not make testlib infer any other rules about the variable. [Integer-reading example](https://github.com/MikeMirzayanov/testlib/blob/master/validators/nval.cpp).

### Follow the current position in the file

Imagine the input is:

```text
3 5
```

After reading `3`, the validator is positioned before the space. After reading the space, it is before `5`. After reading `5`, it is before the line ending.

Write the reads in that same order:

```cpp
int n = inf.readInt(1, 100, "n");
inf.readSpace();
int m = inf.readInt(1, 100, "m");
inf.readEoln();
inf.readEof();
```

| Function | What it requires at the current position |
|---|---|
| `readSpace()` | One space character |
| `readEoln()` | A line ending |
| `readEof()` | The end of the file, without skipping leftover data |

This format rejects `3  5` because it contains two spaces, and rejects `3 5 7` because another value appears where the line should end. Unlike ordinary `cin >>`, validator reads do not automatically skip arbitrary whitespace. [Strict validator example](https://github.com/MikeMirzayanov/testlib/blob/master/validators/ival.cpp).

The order matters. Calling `readEof()` immediately after the last integer would reject its still-unread line ending. Consume the required line ending first, then check EOF.

### Read arrays

Suppose the first line contains `n`, and the second contains `n` integers separated by spaces:

```cpp
int n = inf.readInt(1, 100, "n");
inf.readEoln();
vector<int> a = inf.readInts(n, -1000, 1000, "a");
inf.readEoln();
inf.readEof();
```

`readInts` reads exactly `n` integers and applies the range to every element. In validation mode, it expects single spaces between them. It does not consume the line ending after the last element; that is why `readEoln()` follows it. There is also `readLongs` for arrays of `long long`. [Array validator](https://github.com/MikeMirzayanov/testlib/blob/master/validators/nval.cpp).

### Read tokens and lines

A token stops at whitespace:

```cpp
string s = inf.readToken("[a-zA-Z0-9]{1,100}", "s");
```

This requires `1` to `100` alphanumeric characters. To require a separately supplied length `n`, check it yourself:

```cpp
ensuref((int)s.size() == n, "String length must equal n");
```

A whole line is different:

```cpp
string text = inf.readLine();
```

`readLine()` reads the remainder of the current line and consumes its ending. Use it when spaces belong to the data. If the previous read was an integer on a separate line, call `readEoln()` before starting to read the text line; otherwise, you are still on the integer's line. [Token validator](https://github.com/MikeMirzayanov/testlib/blob/master/validators/sval.cpp).

### Check relationships and totals with `ensuref`

Bounds check individual values. They do not check all the relationships in a statement.

For `l <= r`, write:

```cpp
ensuref(l <= r, "l must not exceed r");
```

For a different problem with several arrays and a total length limit of `200000`, keep a running total. This fragment belongs inside a registered validator:

```cpp
int t = inf.readInt(1, 10000, "t");
inf.readEoln();
long long total_n = 0;
for (int tc = 1; tc <= t; tc++) {
    int n = inf.readInt(1, 200000, "n");
    inf.readEoln();
    total_n += n;
    ensuref(total_n <= 200000, "Total length exceeds 200000 at case %d", tc);
    inf.readInts(n, 1, 1000000000, "a");
    inf.readEoln();
}
inf.readEof();
```

Two arrays of length `150000` each satisfy the individual bound but violate the total. Only the running check catches that.

`ensuref` stops with a failure message when its condition is false. In a validator, that means the test input must be fixed. A successful validator reaches `readEof()` and exits with code `0`.

### Test the validator itself

Do not test only inputs that should pass. Change a valid file in one way at a time: exceed a bound, remove a value, insert an extra space, add an extra token, or use a wrong character. Each file should fail for the reason you intended.

## 3. Checker: judge the participant's output

A checker runs after a normal solution finishes. It receives the input, the participant's output, and the jury answer:

```text
./checker input.txt participant.txt answer.txt
```

Call `registerTestlibCmd(argc, argv)` before reading them.

| Stream | What it reads |
|---|---|
| `inf` | The test input, such as `n` |
| `ouf` | The participant's output |
| `ans` | The jury's answer |

The jury answer is the expected output prepared by the problem setter. Keep the argument order correct: swapping `ouf` and `ans` changes which file you are judging.

### Decide what “correct” means before writing code

There are two common situations:

- **Fixed answer:** a problem asks for one integer, and that integer is determined by the input. Compare the participant's value with the jury's value.
- **Several valid answers:** a problem asks for any arrangement, path, or pair satisfying conditions. Check those conditions. The participant need not reproduce the jury's choice.

For a problem with exactly one integer answer, the core comparison can be:

```cpp
long long expected = ans.readLong();
long long actual = ouf.readLong();
if (actual != expected)
    quitf(_wa, "Expected %lld, found %lld", expected, actual);
```

For a fixed sequence of integers, the standard [`ncmp.cpp`](https://github.com/MikeMirzayanov/testlib/blob/master/checkers/ncmp.cpp) already compares the values and their count while ignoring whitespace. A custom checker is needed when correctness cannot be expressed by that comparison.

### Parse first, then check meaning

Suppose the participant must print a vertex numbered from `1` through `n`:

```cpp
int vertex = ouf.readInt(1, n, "vertex");
```

This reads an integer and checks its range. It does not prove that the vertex is the correct answer; your next checks must do that.

The same `readInt`, `readLong`, and `readToken` methods work on these streams. However, checker token reads skip whitespace. You normally accept spaces or newlines between answer values instead of enforcing the validator's exact layout.

A malformed or missing value is detected during parsing. A valid integer outside the requested range is also rejected. You should never use a value as an array index before checking its range.

### Report a verdict with `quitf`

```cpp
quitf(_wa, "Case %d: the sum is incorrect", tc);
```

`quitf` terminates the program. Its message uses `printf`-style formatting: `%d` for an `int`, `%lld` for a `long long`, and `%s` for a C string.

| Verdict | Meaning |
|---|---|
| `_ok` | Correct answer |
| `_wa` | Wrong answer |
| `_pe` | Output format error |
| `_fail` | Judge-side failure |

Use `_wa` when the submission breaks a problem rule. Use `_fail` when the judging setup is broken. Their difference matters: an invalid test or bad jury answer is not the participant's fault.

Global `ensuref` signals a judge-side failure. For a condition about participant output, use an explicit `_wa` verdict, or the stream method:

```cpp
ouf.ensuref(x + y == n, "The printed numbers must add to n");
```

A failed `ouf.ensuref` gives `_wa`. Applying `ans.ensuref` to an invalid jury answer instead gives `_fail`. This makes a difference when you write a helper that checks both streams. [Stream checks and verdicts](https://github.com/MikeMirzayanov/testlib/blob/master/testlib.h).

### Check that the answer is complete

After reading the required answer, you can write:

```cpp
if (!ouf.seekEof()) quitf(_pe, "Extra output after the answer");
quitf(_ok, "Valid answer");
```

`seekEof()` skips whitespace and reports whether the stream has ended. It does not require the end to be immediately after the last digit. `readEof()` is stricter and does not skip that whitespace.

For example, `12` followed by a newline has no extra answer token, while `12 99` does. In a normal checker, testlib also checks for extra output on `_ok`; the explicit check above makes the intended behavior visible.

Test the checker with correct alternatives, wrong values, missing values, and extra values. A solution passing its own checker is useful evidence, but does not prove that the checker enforces every rule.

## 4. Interactor: have a conversation with the solution

In an interactive problem, the solution does not receive the complete input at once. It asks questions and learns from the replies. The interactor is the program on the other side of that conversation.

Start with `registerInteraction(argc, argv)`. A local invocation uses:

```text
./interactor private-input.txt record.txt
```

The private input can include a hidden number or another secret state. The second file receives what you choose to write to `tout`. An optional third file argument opens `ans` for a jury answer. [Interactor invocation](https://github.com/MikeMirzayanov/testlib/blob/master/testlib.h).

### Know where each message goes

| Name | Interactor use |
|---|---|
| `inf` | Read private test data from the first file argument |
| `ouf` | Read the solution's messages from the interactor's stdin |
| `cout` | Send messages to the solution through stdout |
| `tout` | Write to the second file argument, for recording or later checking |

`cout` and `tout` have different destinations. Writing a reply to `tout` only saves it in a file; the waiting solution does not receive it. [Official interactor example](https://github.com/MikeMirzayanov/testlib/blob/master/interactors/interactor-a-plus-b.cpp).

For a question like `? 5`, the interactor can read:

```cpp
string command = ouf.readToken();
int x = ouf.readInt();
```

These reads wait for the solution's output when no message is available yet. Check the command, value, and protocol rules before replying. For example, you may need to count questions or reject asking about an index that does not exist.

### Flush before waiting for the other program

Printing text does not always send it immediately. It may stay in a buffer. Use:

```cpp
cout << reply << endl;
```

`endl` writes a newline and flushes the stream. This also works:

```cpp
cout << reply << '\n' << flush;
```

The solution must flush its questions too. Otherwise, it can wait for a reply while the interactor is still waiting for the question. Debug messages belong on `cerr`; printing them to `cout` changes the conversation.

### Decide when the conversation ends

Write a protocol that distinguishes questions from the final answer. State the question limit, valid values, reply meanings, and what the solution must do after an error.

An interactor may return `_ok` or `_wa` itself. Another design is to record data in `tout` and let a separate checker judge it. `_ok` in an interactor that only records messages does not by itself prove the final answer correct; that check belongs wherever the protocol puts it.

Do not add a normal checker's `ouf.seekEof()` after a final answer without thinking about the protocol. The solution may still be running, so waiting for EOF can block. Our example finishes judging as soon as it reads the final answer. A runner will enforce the overall timeout and check that the solution also exits.

We will put these parts together in the Hidden Number example after the ordinary problem.

## Example 1: Sum of Two Numbers

In [Codeforces 1788B](https://codeforces.com/problemset/problem/1788/B), you are given `n` and must print two non-negative integers `x` and `y` such that:

- `x + y = n`;
- their digit sums differ by at most `1`.

The input contains `t` cases, with `1 <= t <= 10000` and `1 <= n <= 10^9`.

For `n = 161`, both pairs below work:

| Answer | Digit sums |
|---|---|
| `67 94` | `13` and `13` |
| `80 81` | `8` and `9` |

This is why we need a custom checker. A fixed answer file cannot list every valid pair.

### Write the validator

Save this as `validator.cpp`:

```cpp
#include "testlib.h"

int main(int argc, char* argv[]) {
    registerValidation(argc, argv);

    int t = inf.readInt(1, 10000, "t");
    inf.readEoln();
    for (int i = 0; i < t; i++) {
        inf.readInt(1, 1000000000, "n");
        inf.readEoln();
    }
    inf.readEof();
}
```

The first read checks `t`. The loop reads exactly `t` values of `n`, one per line. The final `readEof()` rejects anything left after those cases.

### Write the generator

Save this as `generator.cpp`:

```cpp
#include "testlib.h"
using namespace std;

// Usage: ./generator t max_n [seed]
int main(int argc, char* argv[]) {
    registerGen(argc, argv, 1);
    ensuref(argc == 3 || argc == 4, "Expected t max_n [seed]");

    int t = opt<int>(1);
    int max_n = opt<int>(2);
    ensuref(1 <= t && t <= 10000, "Invalid t");
    ensuref(1 <= max_n && max_n <= 1000000000, "Invalid max_n");

    cout << t << '\n';
    for (int i = 0; i < t; i++) {
        int n = rnd.next(1, max_n);
        if (i == 0) n = 1;
        if (i == 1) n = max_n;
        cout << n << '\n';
    }
}
```

Here, `argc` includes the executable name, so three arguments after it give `argc == 4`.

The generator prints the minimum value first. If there is a second case, it prints `max_n`. The other cases are random. With `max_n = 10^9`, this deliberately reaches both statement limits.

Random inputs are useful, but they may miss particular patterns. Add cases such as `9`, `99`, `999`, and numbers around powers of ten to exercise the digit splitting.

### Write the checker

The checker gets three streams:

| Stream | File |
|---|---|
| `inf` | Test input |
| `ouf` | Participant output |
| `ans` | Jury answer |

For this problem, we can judge the answer directly from `n`, `x`, and `y`. We do not need to compare with `ans`.

Save this as `checker.cpp`:

```cpp
#include "testlib.h"
using namespace std;

long long digitSum(long long x) {
    long long sum = 0;
    while (x > 0) {
        sum += x % 10;
        x /= 10;
    }
    return sum;
}

int main(int argc, char* argv[]) {
    registerTestlibCmd(argc, argv);

    int t = inf.readInt();
    for (int i = 0; i < t; i++) {
        long long n = inf.readLong();
        long long x = ouf.readLong(0, n, "x");
        long long y = ouf.readLong(0, n, "y");

        if (x + y != n)
            quitf(_wa, "Case %d: x + y must equal n", i + 1);
        if (llabs(digitSum(x) - digitSum(y)) > 1)
            quitf(_wa, "Case %d: digit sums differ by more than 1", i + 1);
    }
    if (!ouf.seekEof()) quitf(_pe, "Extra output");
    quitf(_ok, "Valid answer");
}
```

Because the numbers are non-negative and add to `n`, neither can exceed `n`. The two `readLong` calls enforce that range. Then we check the sum and digit sums.

`seekEof()` skips whitespace and checks whether anything else remains. Blank space after the answer is fine; an extra number is not. A checker's token reads allow whitespace between values, so this checker does not demand one specific arrangement of spaces and lines. [Checker input and output handling](https://github.com/MikeMirzayanov/testlib/blob/master/testlib.h).

If a problem requires a fixed sequence of integer answers, the standard [`ncmp.cpp`](https://github.com/MikeMirzayanov/testlib/blob/master/checkers/ncmp.cpp) compares those integers while ignoring whitespace. It is not suitable for this problem, because different pairs can be correct.

### Add a solution to run locally

Save this as `solution.cpp`. It is ordinary C++, with no testlib include:

```cpp
#include <bits/stdc++.h>
using namespace std;

int main() {
    ios::sync_with_stdio(false);
    cin.tie(nullptr);

    int t;
    cin >> t;
    while (t--) {
        long long n;
        cin >> n;
        long long x = 0, y = 0, place = 1;
        bool giveExtraToX = true;

        while (n > 0) {
            int digit = n % 10;
            int a = digit / 2;
            int b = digit / 2;
            if (digit % 2) {
                if (giveExtraToX) a++;
                else b++;
                giveExtraToX = !giveExtraToX;
            }
            x += a * place;
            y += b * place;
            place *= 10;
            n /= 10;
        }
        cout << x << ' ' << y << '\n';
    }
}
```

Split each digit equally. When it is odd, alternate which number receives the extra `1`. Each pair of digits adds to the original digit without a carry, so `x + y` is correct. Alternating the extras keeps the total digit sums within `1` of each other.

### Compile and run the complete example

```bash
g++ -std=c++14 -O2 generator.cpp -o generator
g++ -std=c++14 -O2 validator.cpp -o validator
g++ -std=c++14 -O2 checker.cpp -o checker
g++ -std=c++14 -O2 solution.cpp -o solution
```

Run these steps from a Bash script, or start Bash before pasting them:

```bash
set -e
./generator 100 1000000000 42 > input.txt
./validator < input.txt
./solution < input.txt > answer.txt
./checker input.txt answer.txt answer.txt
```

`set -e` stops this sequence if a command fails. A failed generator or crashed solution must not be treated as a successful test.

The checker arguments are **input, participant output, jury answer**. Here we pass `answer.txt` twice because we are checking the supplied solution's own output and the checker does not read the jury answer. This tests the output's conditions; it is not a comparison between two independent solutions.

To check a participant program, run it separately:

```bash
./candidate < input.txt > participant.txt
./checker input.txt participant.txt answer.txt
```

Try giving the checker `0 161` for a case with `n = 161`. The sum is correct, but the digit sums are `0` and `8`, so it must reject the answer. Also try an empty output file and an extra number. A checker needs tests too.

## Test many files with `script.sh` and `compare.sh`

So far, the commands checked one file. In practice, keep separate files for small cases, boundary cases, and larger random cases.

These two scripts belong in the **Sum of Two Numbers** folder. They use the four source files from Example 1:

```text
sum-of-two-numbers/
├── testlib.h
├── generator.cpp
├── validator.cpp
├── checker.cpp
├── solution.cpp
├── script.sh
├── compare.sh
├── Inputs/
│   ├── 1.in
│   └── ...
└── Outputs/
    ├── 1.ans
    └── ...
```

The scripts below use Bash and GNU `timeout`, as available on a typical Linux setup. `Inputs/17.in` and `Outputs/17.ans` form one test file and its jury answer. A file can contain many cases: `t = 1000` inside one file is different from having `1000` files.

### `script.sh`: compile, generate, validate, and prepare answers

Save this as `script.sh`:

```bash
#!/usr/bin/env bash
set -euo pipefail
cd -- "$(dirname -- "${BASH_SOURCE[0]}")"

g++ -std=c++14 -O2 -Wall -Wextra generator.cpp -o gen
g++ -std=c++14 -O2 -Wall -Wextra validator.cpp -o val
g++ -std=c++14 -O2 -Wall -Wextra checker.cpp -o checker
g++ -std=c++14 -O2 -Wall -Wextra solution.cpp -o sol
mkdir -p Inputs Outputs

# File 1: the original statement sample.
cat > Inputs/1.in <<'INPUT'
5
1
161
67
1206
19
INPUT

# File 2: minimum case count and minimum n.
printf '1\n1\n' > Inputs/2.in

# File 3: values around decimal boundaries and the maximum n.
cat > Inputs/3.in <<'INPUT'
12
1
9
10
11
99
100
101
999
1000
1001
999999999
1000000000
INPUT

# Files 4..100: raise t and max_n in groups.
CAPS=(10 100 1000 10000 100000 1000000 10000000 100000000 500000000 1000000000)
for id in {4..100}; do
    group=$(((id - 4) / 10))
    t=$(((group + 1) * 1000))
    ./gen "$t" "${CAPS[$group]}" "$id" > "Inputs/$id.in"
done

# Check input before solving it; check the answer before saving it.
work=$(mktemp -d)
trap 'rm -rf -- "$work"' EXIT
for id in {1..100}; do
    ./val < "Inputs/$id.in"
    timeout 5s ./sol < "Inputs/$id.in" > "$work/answer.txt"
    ./checker "Inputs/$id.in" "$work/answer.txt" "$work/answer.txt"
    mv -- "$work/answer.txt" "Outputs/$id.ans"
    printf 'Prepared file %s\n' "$id"
done
printf 'Prepared 100 validated test files and answers.\n'
```

Read it in four parts:

1. **Compile:** `g++` builds the generator, validator, checker, and solution. A compilation error stops the script.
2. **Generate:** the first three files are deliberate cases. The rest use increasing parameter caps. The file number is also the seed argument, so rerunning the same sources reproduces these files.
3. **Validate:** `./val` checks each input before the solution reads it.
4. **Prepare the answer:** the solution writes to a temporary file. The checker verifies its conditions, then `mv` saves it under the matching number.

`set -e` stops on a failed command in this straight-line workflow. `-u` catches an unset variable, and `pipefail` makes a pipeline fail if one of its commands fails. `cd` makes paths relative to the script's folder. `mkdir -p` creates missing directories and accepts existing ones.

`mktemp -d` creates a private temporary directory. The `trap` cleans it when the script exits. This prevents a failed solution from replacing an answer file with partial output. The script is ready for comparison only after it completes successfully.

The checker receives the temporary answer as both output and jury answer because this particular checker judges the conditions without using `ans`. A checker for a fixed-answer problem would need a separately trusted jury answer instead. Generating an answer and comparing it with itself does not verify a solution.

The five-second `timeout` is a local guard against a hanging program, not the problem's official time limit or a measurement of its memory use.

### `compare.sh`: judge another solution on all files

Prepare the answers once with `script.sh`. Then use them to check candidate solutions without regenerating the jury answers.

Save this as `compare.sh`:

```bash
#!/usr/bin/env bash
set -euo pipefail
cd -- "$(dirname -- "${BASH_SOURCE[0]}")"
candidate=${1:-./sol}

[[ -x "$candidate" ]] || { echo "Missing executable: $candidate" >&2; exit 1; }
[[ -x ./val && -x ./checker ]] || { echo 'Run script.sh first.' >&2; exit 1; }

work=$(mktemp -d)
trap 'rm -rf -- "$work"' EXIT
failed=0
fail() {
    printf 'File %s: FAIL (%s)\n' "$id" "$1"
    cat "$work/message.txt"
    failed=$((failed + 1))
}

for id in {1..100}; do
    input="Inputs/$id.in"
    answer="Outputs/$id.ans"
    : > "$work/message.txt"

    if [[ ! -s "$input" || ! -s "$answer" ]]; then
        fail 'missing or empty input/answer'
        continue
    fi
    if ! ./val < "$input" 2> "$work/message.txt"; then
        fail 'invalid input'
        continue
    fi
    if ! timeout 5s "$candidate" < "$input" > "$work/got.txt" 2> "$work/message.txt"; then
        fail 'solution failed or timed out'
        continue
    fi
    if ./checker "$input" "$work/got.txt" "$answer" > /dev/null 2> "$work/message.txt"; then
        printf 'File %s: OK\n' "$id"
    else
        fail 'checker rejected the output'
    fi
done

printf 'Checked 100 files; %s failed.\n' "$failed"
[[ "$failed" -eq 0 ]]
```

The script checks all **100 expected file numbers**. If you change that count in `script.sh`, change it here too. A missing file is a failure, rather than a silently skipped test.

For each file, it checks that the input and jury answer exist, validates the input, runs the candidate, and calls the checker. The `if` statements handle expected failures explicitly, allowing the script to continue and report other failing files even with `set -e` enabled.

`>` sends stdout to a file; `2>` sends stderr there. The candidate's answer goes to `got.txt`, while its diagnostic or the checker's verdict goes to `message.txt`. `: > file` empties a diagnostic file so an old message cannot appear under a new failure.

Use the **checker**, not `diff`, for this problem. Different valid pairs are allowed. An empty output is rejected when the checker tries to read the required integers. A crashed candidate is caught before the checker runs, regardless of what it wrote.

The last command makes the script return success only when every file passes. The optional executable path is interpreted relative to this folder unless you provide an absolute path.

### Run the scripts

```bash
chmod +x script.sh compare.sh
./script.sh
./compare.sh
```

To check another program, compile it under another name:

```bash
g++ -std=c++14 -O2 candidate.cpp -o candidate
./compare.sh ./candidate
```

`./compare.sh` defaults to `./sol`, which is useful for checking the prepared workflow. To test another algorithm, pass that algorithm's executable explicitly. Do not replace the jury answer files with the candidate's output.

Passing these tests does not cover every possible input. For small values, you can also compare with a simple exhaustive solution; for larger values, add patterns suggested by likely mistakes.

## Example 2: Find the Hidden Number

Now we will define an interactive problem. The important change is that the solution cannot read all the information at the start. It must ask questions to learn a hidden value.

### Problem statement

The judge chooses an integer `h` in the range `[1, n]`, where `1 <= n <= 10^9`. You are given `n`, but not `h`.

Find `h` using at most **30 questions**.

**Interaction:**

1. Read `n` from standard input.
2. To ask a question, print `? x`, where `1 <= x <= n`. Flush the output.
3. Read the judge's reply: `1` if `h <= x`, or `0` otherwise.
4. When you know `h`, print `! h`, flush the output, and terminate.

The final answer does not count as a question. There is exactly one hidden number per run. For `n = 1`, you can answer immediately without asking a question.

An unknown command, an out-of-range value, more than 30 questions, or a wrong final answer causes rejection. For those errors, the interactor sends `-1` and ends the interaction. If the solution reads `-1`, it must terminate immediately. Missing or malformed messages may end the interaction without a reply. Send each message on its own line, with no explanatory text on standard output.

### A sample conversation

Suppose `n = 10` and the hidden number is `7`:

| Sender | Message | Meaning |
|---|---|---|
| Judge | `10` | The number is between `1` and `10`. |
| Solution | `? 5` | Is the number at most `5`? |
| Judge | `0` | No: it is between `6` and `10`. |
| Solution | `? 8` | Is it at most `8`? |
| Judge | `1` | Yes: it is between `6` and `8`. |
| Solution | `? 7` | Is it at most `7`? |
| Judge | `1` | Yes: it is either `6` or `7`. |
| Solution | `? 6` | Is it at most `6`? |
| Judge | `0` | No: it must be `7`. |
| Solution | `! 7` | Final answer. |

These messages happen in order. This table is a conversation, not one file that the solution reads at the start.

### Separate private input from public messages

For local testing, the private test file contains `n h` on one line:

```text
10 7
```

The interactor reads both numbers. It sends only `n` to the solution. If the solution received `h` too, there would be nothing left to find.

Save this validator as `hidden-validator.cpp`:

```cpp
#include "testlib.h"

int main(int argc, char* argv[]) {
    registerValidation(argc, argv);
    int n = inf.readInt(1, 1000000000, "n");
    inf.readSpace();
    inf.readInt(1, n, "h");
    inf.readEoln();
    inf.readEof();
}
```

It checks the private test data, not the messages sent during the conversation.

### Write the interactor

An interactor uses its streams differently from a checker:

| Stream | Use in an interactor |
|---|---|
| `inf` | Read the private test file. |
| `cout` | Send messages to the solution. |
| `ouf` | Read messages from the solution. |
| `tout` | Write a record to a file. It is not the communication channel. |

The optional `ans` stream can read a separate jury answer file, but this example does not need one. The private input already contains the hidden number. [Official interactor example](https://github.com/MikeMirzayanov/testlib/blob/master/interactors/interactor-a-plus-b.cpp).

Save this as `interactor.cpp`:

```cpp
#include "testlib.h"
using namespace std;

void reject(const char* reason) {
    cout << -1 << endl;
    quitf(_wa, "%s", reason);
}

int main(int argc, char* argv[]) {
    registerInteraction(argc, argv);
    int n = inf.readInt(1, 1000000000, "n");
    int hidden = inf.readInt(1, n, "h");
    const int LIMIT = 30;
    int queries = 0;

    cout << n << endl;
    while (true) {
        string command = ouf.readToken();
        if (command != "?" && command != "!")
            reject("Expected ? or !");

        long long x = ouf.readLong();
        if (x < 1 || x > n)
            reject("Value is outside [1, n]");

        if (command == "!") {
            if (x != hidden) reject("Wrong hidden number");
            tout << "! " << x << '\n';
            quitf(_ok, "Found the number using %d questions", queries);
        }

        if (queries == LIMIT) reject("Too many questions");
        queries++;
        int reply = hidden <= x ? 1 : 0;
        tout << "? " << x << " -> " << reply << '\n';
        cout << reply << endl;
    }
}
```

Read the loop in this order:

1. Read the command and reject anything except `?` or `!`.
2. Read its number and check that it is in range.
3. For `!`, compare the final answer with the hidden number and finish.
4. For `?`, check the limit, count the question, and send the reply.

Handling `!` before the limit check lets a solution submit its final answer after using all 30 questions. A 31st question is rejected.

The interactor reads messages as whitespace-separated tokens. The final answer ends the conversation; it does not wait for further output. It judges the answer itself, and `tout` records the questions and final answer for inspection.

### Why flushing matters

Output may stay in a buffer instead of reaching the other program immediately. If the solution prints a question without flushing and then waits for a reply, the interactor may still be waiting to receive that question.

`endl` prints a newline and flushes. Another way is:

```cpp
cout << "? " << x << '\n' << flush;
```

Flush both questions and replies. Do not write debug messages to standard output; they become part of the protocol. Use `cerr` for debugging.

### Write a solution

Keep the possible interval `[l, r]`. Ask about its middle value:

- A reply of `1` means the number is in `[l, mid]`.
- A reply of `0` means it is in `[mid + 1, r]`.

This is binary search. Each question reduces the interval to at most half its previous size, rounded up. Since `2^30 > 10^9`, 30 questions are enough. When `l == r`, only one possible number remains.

Save this as `hidden-solution.cpp`:

```cpp
#include <bits/stdc++.h>
using namespace std;

int main() {
    int n;
    if (!(cin >> n)) return 0;
    int l = 1, r = n;

    while (l < r) {
        int mid = l + (r - l) / 2;
        cout << "? " << mid << endl;

        int reply;
        if (!(cin >> reply) || reply == -1) return 0;
        if (reply == 1) r = mid;
        else if (reply == 0) l = mid + 1;
        else return 0;
    }
    cout << "! " << l << endl;
}
```

The solution does not use testlib. It only follows the problem's communication rules.

### Run an interactive test locally

An interactive run needs two connections:

```text
solution stdout   → interactor stdin
interactor stdout → solution stdin
```

A single shell pipe gives only one direction. This small Python runner makes both connections. Save it as `run-interaction.py`:

```python
import subprocess
import sys

if not 2 <= len(sys.argv) <= 4:
    sys.exit("Usage: python3 run-interaction.py input [solution] [record]")

solution_path = sys.argv[2] if len(sys.argv) >= 3 else "./hidden-solution"
record_path = sys.argv[3] if len(sys.argv) >= 4 else "transcript.txt"

judge = subprocess.Popen(
    ["./interactor", sys.argv[1], record_path],
    stdin=subprocess.PIPE,
    stdout=subprocess.PIPE,
)
try:
    solution = subprocess.Popen(
        [solution_path],
        stdin=judge.stdout,
        stdout=judge.stdin,
    )
except Exception:
    judge.kill()
    judge.wait()
    raise

# The children now own the pipe connections.
judge.stdout.close()
judge.stdin.close()

try:
    judge_code = judge.wait(timeout=5)
    solution_code = solution.wait(timeout=5)
except subprocess.TimeoutExpired:
    judge.kill()
    solution.kill()
    judge.wait()
    solution.wait()
    sys.exit("Interaction timed out")

print(f"Interactor exit code: {judge_code}")
print(f"Solution exit code: {solution_code}")
sys.exit(0 if judge_code == 0 and solution_code == 0 else 1)
```

The runner leaves diagnostics on the terminal and returns failure if either program fails or the conversation times out. Its timeout is a local testing convenience, not a complete contest judging system.

Compile the programs and create the private input:

```bash
g++ -std=c++14 -O2 hidden-validator.cpp -o hidden-validator
g++ -std=c++14 -O2 interactor.cpp -o interactor
g++ -std=c++14 -O2 hidden-solution.cpp -o hidden-solution
printf '10 7\n' > private-input.txt
./hidden-validator < private-input.txt
python3 run-interaction.py private-input.txt
cat transcript.txt
```

The interactor's two file arguments are the private input and the output record. Neither is the live communication channel; the runner supplies those pipe connections.

Test `n = 1`, a hidden value at either end of the range, and `n = 10^9`. Also test a solution that gives a wrong final answer, asks a 31st question, or forgets to flush. Those checks tell you whether the interactor and runner handle mistakes as intended.

### Check several interactive files

Keep this interactive example in a separate folder from Sum of Two Numbers. Its inputs are private `n h` pairs, so they cannot use the ordinary problem's generator or checker.

Here is a small batch with four private tests. Run it in Bash after compiling the interactive example:

```bash
set -euo pipefail
mkdir -p Inputs Records
printf '1 1\n' > Inputs/1.in
printf '10 7\n' > Inputs/2.in
printf '1000000000 1\n' > Inputs/3.in
printf '1000000000 1000000000\n' > Inputs/4.in
for id in {1..4}; do
    ./hidden-validator < "Inputs/$id.in"
    python3 run-interaction.py "Inputs/$id.in" ./hidden-solution "Records/$id.txt"
done
```

The runner accepts an optional solution path and record path. Replace `./hidden-solution` with another compiled interactive solution to test it on the same files. Each run has its own record, and a failed run stops this batch.

There is no expected conversation to compare with `diff`: two correct solutions can ask different questions. The interactor judges the answers and query limit during each run.

## What to take to Polygon

For Sum of Two Numbers, you now have a validator, generator, custom checker, and solution. For Find the Hidden Number, you have a validator for private tests, an interactor, and an interactive solution.

Run them locally before uploading. Then use Polygon's tests and Invocations to check submissions under the problem's limits. The Polygon walkthrough covers uploading these files, running Invocations, and downloading a problem package.
