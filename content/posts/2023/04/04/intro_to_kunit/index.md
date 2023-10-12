---
title: High level introduction to KUnit
date: 2023-04-04
tags: ["linux", "kernel", "testing", "kunit", "learning"]
---

KUnit is a unit testing framework for the Linux kernel. It is inspired by other x-unit testing frameworks (JUnit, Python unittest). According to [Brendan Higgins (the author of KUnit) at LPC2019](https://www.youtube.com/watch?v=507n-t0sfcU), the need for yet another framework is because other frameworks have their own style and conventions, which does not match kernel conventions. Since the kernel community is huge and telling everyone to change their conventions is a laborious idea, KUnit was made specifically to cater to kernel standards and usage.

Further, quoting Brendan again:

>   **Tests are also documentation. Documentation gets stale. Tests don't. Tests show how code behaves.**
>
>   ~ Brendan Higgins in [an LF Mentorship session](https://www.youtube.com/watch?v=i0xrfn5PSsM).

So unit testing also provides a degree of documentation, by documenting how a unit behaves and what are its interfaces.

KUnit works by running tests when the kernel is run. The tests are compiled into the kernel binary, and the kernel basically self-tests itself on boot. The test results are reported in dmesg logs in a format known as Kernel Test Anything Protocol (KTAP).

While KTAP is sorta human readable, it is not pretty for our eyes. The `kunit_tool` Python script (`tools/testing/kunit/kunit.py`) can be used to parse KTAP outputs into a nice colour-formatted text on a terminal. `kunit_tool` can handle building the kernel, running the tests, and reporting the results all in one go automatically, so it is a very handy tool. Unfortunately, it does not support gcc-7 and higher, one has to use gcc-6 or llvm.

Tests in KUnit are written as a function known as test case. Each test case has several "expectations" and "assertions" on how the code should behave. These expectations are nothing but macros like `KUNIT_EXPECT_EQ`, `KUNIT_EXPECT_NULL`, `KUNIT_ASSERT_EQ`, etc. The difference between an `ASSERT` and an `EXPECT` is that the former will stop the test case when it fails, while the latter does not. The result will be a fail in both cases though. The [documentation](https://docs.kernel.org/dev-tools/kunit/usage.html) consists of various simple examples, such as below (comments are mine):

```c
/* 
 * Test basics of an add() function.
 * 
 * struct kunit represents instance of a test.
 * It is defined in /include/kunit/test.h.
 */
void add_test_basic(struct kunit *test)
{
	/*
	 * Testing 1 + 0 = 1.
	 * Expected output = 1.
	 * Function call to test = add(1, 0).
	 * 
	 * KUNIT_EXPECT_EQ checks if both are equal.
	 * Order does not matter here.
	 */
	KUNIT_EXPECT_EQ(test, 1, add(1, 0));
    
    /* Testing 1 + 1 = 2 similarly. */
	KUNIT_EXPECT_EQ(test, 2, add(1, 1));
}
```

The structure of KUnit's test collection is referred to as a test suite. It contains the following:

1.   `name`: The name of the test suite.
2.   `test_cases`: An array of `struct kunit_case`, which encapsulates the test cases.
3.   `init`: Setup/ctor function to run before every test case.
4.   `exit`: Teardown/dtor function to run after every test case.
5.   `suite_init`: Setup/ctor function for the test suite (gets executed first).
6.   `suite_exit`: Teardown/dtor function for the test suite (gets executed last).

The test files are meant to be housed alongside the code it tests, or in a folder named `tests`. The convention for the test files in DRM core seems to be `tests/<name>_test.c`.
