---
layout: post
title: GitHub Actions Multi-line Output Variable
date: 2026-01-08
categories:
- devops
---

When building GitHub Actions it's common to want to output multi-line text from a composite action or step. GitHub Actions provides the `$GITHUB_OUTPUT` [environment file](https://docs.github.com/en/actions/how-tos/write-workflows/choose-what-workflows-do/pass-job-outputs) mechanism to pass outputs between steps. The syntax is simple, if you are familiar with a Linux shell: `echo "test=hello" >> "$GITHUB_OUTPUT"`. This will set a variable from that job-step called `test` with the value `hello`  and can be called from another step using the syntax `${{ steps.<jobid>.outputs.test }}`. This deals with single line outputs well, but what about multis-line outputs? That can be more complex.

Bash has the traditional `EOF` [delimiter](https://www.man7.org/linux/man-pages/man3/EOF.3const.html) for dealing with this, but it can be complex, especially when it's not your machine (i.e. a GitHub hosted runner). Usually the syntax is syntax is straightforward: you write the output name followed by `<<DELIMITER`, then the value, and finally then delimetirer again. For example:

```bash
    {
      echo "message<<<EOF"
      echo "Line 1"
      echo "Line 2"
      echo "Line 3"
      echo "EOF"
    } >> "$GITHUB_OUTPUT"
```

In this case the variable will have the name `message`. 

However, a common gotcha arises when the multi-line content comes from a file or command. GitHub's parser is very strict, but the community have found this to be not so well [documented](https://github.com/github/docs/issues/21529).  The closing delimiter must be on its own line, with no extra spaces, and must follow a newline that isn't swallowed by the file content. This means that if your file doesn't end with a newline, or the shell appends the closing delimiter immediately after the last line of content, GitHub will throw the error:

    Error: Unable to process file command 'output' successfully.
    Error: Invalid value. Matching delimiter not found 'EOF'

> **WARNING** This code will fail:

    {
      echo "commits<<EOF"
      cat some-file.txt
      echo "EOF"
    } >> "$GITHUB_OUTPUT"

The solution is simple and reliable: ensure a newline between the file content and closing delimeter. Using `printf '\n'` guarantees this:

    {
      echo "commits<<EOF"
      cat commits.txt
      printf '\n'
      echo "EOF"
    } >> "$GITHUB_OUTPUT"
