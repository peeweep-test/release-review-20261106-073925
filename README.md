# DDE 发布流程实测

2026-11-06，在 peeweep-test 的独立测试仓库中使用真实 GitHub Actions、PR 和分支保护复现。三个问题均确认存在。

测试包含一个协调仓库、两个 G2 成员和一个 G1 成员。待审的发布、恢复、版本比较和状态写入逻辑保持原样；环境配置将仓库地址指向这些测试仓库，关闭定时触发。G2 的成员 B 通过调用入口中的 REVIEW_FAIL_BEFORE_RELEASE 变量模拟首次失败。

工作流源码见 [.github/workflows](./.github/workflows)，原始源码摘要见 [source-sha256.json](./source-sha256.json)，关键 API 响应见 [proof.json](./proof.json)。测试结束时重新比对过源码，工作区原文件与测试开始时一致。

## 1. 同一提交的检查状态互相覆盖

[合法 PR](https://github.com/peeweep-test/release-review-go-a-20261106-073925/pull/1) 指向 review/allowed；[错误目标 PR](https://github.com/peeweep-test/release-review-go-a-20261106-073925/pull/2) 指向 review/forbidden，两者使用同一提交 `b995756b1b30260bc4b8b1c3e47b0c0787c3ad8c`。

两个目标分支从同一提交创建。分支保护要求 group-version-guard 由 GitHub Actions 提供，开启严格的最新分支要求，并对管理员执行保护。

1. 错误 PR 的[校验运行](https://github.com/peeweep-test/release-review-go-a-20261106-073925/actions/runs/42183780719)失败，日志显示该 PR 不属于本轮发布。
2. 尝试合并错误 PR，GitHub 返回 HTTP 405：`Required status check "group-version-guard" is failing.`
3. 重跑合法 PR 的[检查](https://github.com/peeweep-test/release-review-go-a-20261106-073925/actions/runs/42183649360/attempts/2)，结果为 success。
4. 再次合并同一个错误 PR，GitHub 返回 `merged: true`。两次尝试之间没有更改分支保护或 PR 的提交。

错误 PR 已实际合入 review/forbidden，合并提交为 `7a3823c511547b1997938ca1a4cea1b9e0008aa7`。

## 2. 恢复发布混用了发行套件

[首次发布](https://github.com/peeweep-test/release-review-20261106-073925/actions/runs/42183505658)指定 G2 / 6.0.50 / stable / review/allowed。成员 A 创建 PR 并合并；成员 B 在创建 PR 前按测试设置失败。

关闭成员 B 的故障开关后，[重新运行 Release group](https://github.com/peeweep-test/release-review-20261106-073925/actions/runs/42184405434)，保持组和版本相同，省略 distribution 和 base_branch，使用表单默认值。协调器只重新派发了成员 B。

| 成员 | 版本 | 最终 changelog 发行套件 |
|---|---|---|
| [成员 A](https://github.com/peeweep-test/release-review-go-a-20261106-073925/pull/1) | 6.0.50 | stable |
| [成员 B](https://github.com/peeweep-test/release-review-go-b-20261106-073925/pull/1) | 6.0.50 | unstable |

两个成员的版本检查均通过。成员 PR 合并后，Sync baseline 将 group-release-ready 置为 success，[G2 基线 PR](https://github.com/peeweep-test/release-review-20261106-073925/pull/2)也已合并。

## 3. RC 发布后无法发布对应正式版

[Release group](https://github.com/peeweep-test/release-review-20261106-073925/actions/runs/42183443217)接受 G1 / 6.7.53-rc.1，真实的 Auto Release 生成了[成员 PR](https://github.com/peeweep-test/release-review-dtk-20261106-073925/pull/1)。成员 PR 和 [G1 基线 PR](https://github.com/peeweep-test/release-review-20261106-073925/pull/1)均已通过检查并合并。

随后[发布正式版 6.7.53](https://github.com/peeweep-test/release-review-20261106-073925/actions/runs/42184793295)，协调器失败，实际日志为：

```text
RuntimeError: 6.7.53 必须大于 G1 基线 6.7.53-rc.1
```

这次失败发生在创建下一轮发布记录和派发成员工作流之前。

## 测试范围

本次使用当前工作流和三个最小成员运行真实发布，没有重跑现有 35 个成员。原 peeweep-test/dde 和其他已有成员仓库未被修改。工作流与复现记录保留在这些测试仓库中，三个问题尚未修复。
