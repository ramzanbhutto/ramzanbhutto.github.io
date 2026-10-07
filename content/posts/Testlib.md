---
title: "Learning testlib.h: Validators, Generators, Checkers, and Interactors"
date: 2026-10-08
tags: ["testlib", "competitive-programming", "codeforces", "polygon", "cpp"]
author: "Muhammad Ramzan"
description: "Learn testlib.h step by step, then prepare Sum of Two Numbers and a small interactive problem with complete C++ examples."
draft: true
---

When you prepare a programming problem, you need more than a solution. You also need to create inputs, check their constraints, and decide whether a submitted answer is correct.

[testlib.h](https://github.com/MikeMirzayanov/testlib) is a C++ library that helps with these jobs. It also supports interactive problems, where the judge and solution exchange messages while the solution runs.

This post assumes you know basic C++: variables, loops, functions, and input/output. First, we will learn the parts of testlib that we need. Then we will use them in two problems:

1. **Sum of Two Numbers:** a normal problem with more than one valid answer.
2. **Find the Hidden Number:** an interactive problem defined for this tutorial.

## What each program does

| Program | What it does | What it does not do |
|---|---|---|
| Generator | Creates input files. | Prove that a solution is correct. |
| Validator | Checks input format and constraints. | Solve the problem. |
| Checker | Checks the participant's answer. | Require one particular answer when several are valid. |
| Interactor | Talks to the running solution and handles its questions. | Give the solution the private test data. |

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

## Learn the basic functions

### Tell testlib which program you are writing

Each program starts by registering its role:

| Program | First call inside `main` |
|---|---|
| Validator | `registerValidation(argc, argv);` |
| Generator | `registerGen(argc, argv, 1);` |
| Checker | `registerTestlibCmd(argc, argv);` |
| Interactor | `registerInteraction(argc, argv);` |

Use `int main(int argc, char* argv[])` so testlib can read the command-line arguments. These calls set up the streams or random generator needed by that program. The last `1` in `registerGen` selects the random generator version; it is not a seed. [Registration functions](https://github.com/MikeMirzayanov/testlib/blob/master/testlib.h).

### Read values and check their ranges

In a validator, use `inf` to read the input:

```cpp
int n = inf.readInt(1, 100, "n");
```

This reads an integer between `1` and `100`, including both limits. The name `"n"` helps identify it in an error message. A value such as `101`, or a token such as `hello`, fails validation.

For larger integers, use `readLong`:

```cpp
long long x = inf.readLong(0LL, 1000000000000000000LL, "x");
```

The `LL` suffix makes these constants `long long` values. [Integer validator example](https://github.com/MikeMirzayanov/testlib/blob/master/validators/nval.cpp).

To read a string with a restricted alphabet and length:

```cpp
string s = inf.readToken("[a-z]{1,100}", "s");
```

This accepts one token containing between `1` and `100` lowercase English letters. It does not accept spaces inside the string. [String validator example](https://github.com/MikeMirzayanov/testlib/blob/master/validators/sval.cpp).

### Check spaces, line endings, and extra input

A validator should follow the input layout stated in the problem:

```cpp
int n = inf.readInt(1, 100, "n");
inf.readSpace();
int m = inf.readInt(1, 100, "m");
inf.readEoln();
inf.readEof();
```

This expects two integers separated by one space, followed by a line ending and then the end of the file.

- `readSpace()` requires a space.
- `readEoln()` requires a line ending.
- `readEof()` requires the file to end at that point.

Unlike normal `cin >>`, validator reads are strict about whitespace. A file may contain valid numbers but still have the wrong layout. [Small validator example](https://github.com/MikeMirzayanov/testlib/blob/master/validators/ival.cpp).

### Check conditions that a range cannot express

Suppose the input contains `l` and `r`, and the statement requires `l <= r`:

```cpp
ensuref(l <= r, "l must not exceed r");
```

`ensuref` stops the program if its condition is false and reports the message. It is useful for relationships between values and limits such as the total array length across all test cases.

Checking each array's length separately does not check a limit on their combined length. You must keep a total and check that too.

### Generate random values and read arguments

A generator uses `rnd`:

```cpp
int n = rnd.next(1, 100);
```

Both limits are included. After `registerGen`, you can read command-line parameters with `opt`:

```cpp
int t = opt<int>(1);
```

For a command such as `./generator 10 100 42`, the first argument is `10`. Your program decides what each argument means. testlib does not automatically treat the first argument as a seed.

The random seed is calculated from the command-line arguments. Keeping the same arguments, program, and testlib version lets you reproduce a test. Use a different trailing seed argument to generate another test with the same limits. [Generator setup](https://github.com/MikeMirzayanov/testlib/blob/master/README.md).

Named parameters are also available, such as `opt<int>("test-count")` for `-test-count 10`. The examples below use positional arguments to keep the commands short. [Named-argument example](https://github.com/MikeMirzayanov/testlib/blob/master/generators/gen-array-with-opt.cpp).

### Return a verdict

A checker or interactor ends with a verdict:

```cpp
quitf(_wa, "The answer is incorrect");
```

`quitf` prints a diagnostic and terminates the program.

| Verdict | Meaning |
|---|---|
| `_ok` | The answer is correct. |
| `_wa` | The answer is wrong. |
| `_pe` | The output has a format error. |
| `_fail` | There is a problem in the judging setup, such as invalid private input. |

Use `_fail` for judge-side errors, not an ordinary wrong submission. Parsing a participant's output can also produce a verdict automatically when a required value is missing or malformed. [Verdict handling](https://github.com/MikeMirzayanov/testlib/blob/master/testlib.h).

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

if len(sys.argv) != 2:
    sys.exit("Usage: python3 run-interaction.py private-input.txt")

judge = subprocess.Popen(
    ["./interactor", sys.argv[1], "transcript.txt"],
    stdin=subprocess.PIPE,
    stdout=subprocess.PIPE,
)
try:
    solution = subprocess.Popen(
        ["./hidden-solution"],
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

## What to take to Polygon

For Sum of Two Numbers, you now have a validator, generator, custom checker, and solution. For Find the Hidden Number, you have a validator for private tests, an interactor, and an interactive solution.

Run them locally before uploading. Then use Polygon's tests and Invocations to check submissions under the problem's limits. The Polygon walkthrough covers uploading these files, running Invocations, and downloading a problem package.
