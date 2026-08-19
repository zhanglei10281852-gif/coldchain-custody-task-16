# Bug Reproduction

## 包的性质

当前 test_model_fix 保存的是被测模型修复后的结果源码，不是初始含 Bug 源码。要复现原始缺陷，必须检出下面固定的 parent SHA；不要在当前修复结果源码上期待重新出现修复前失败。生成系统使用的可信验证补丁和完整验证日志仅在本地留存，不提交到结果分支。

## 问题现象

客户端在样本状态更新前取消请求后，接口调用虽然结束，但样本仍被后台写成就绪状态。请修复 context 传播，取消必须阻止事务继续写入。 请只修改必要的生产代码，不得新增、删除或修改测试文件，不得跳过测试或放宽断言。

## 含 Bug 版本

- 仓库：zhanglei10281852-gif/coldchain-custody-task-16
- 仓库地址：https://github.com/zhanglei10281852-gif/coldchain-custody-task-16.git
- parent SHA：7f7d6d00bbe621159caf35d4f31c46cbbb4de658

## 复现步骤

```bash
git clone -- https://github.com/zhanglei10281852-gif/coldchain-custody-task-16.git bug-repro
cd bug-repro
git checkout --detach 7f7d6d00bbe621159caf35d4f31c46cbbb4de658
go test ./internal/service -run "^TestCancelledRegistrationDoesNotAdvance$" -count=1
```

## 双架构完整错误信息

### linux/amd64

- 容器内复现预期退出码：1
- 容器内复现实际退出码：1

stdout：

```text
$ go test ./internal/service -run "^TestCancelledRegistrationDoesNotAdvance$" -count=1
--- FAIL: TestCancelledRegistrationDoesNotAdvance (0.50s)
    annotation_behavior_test.go:237: cancelled state change succeeded
FAIL
FAIL	github.com/zhanglei10281852-gif/coldchain-custody-base/internal/service	0.501s
FAIL

```

stderr：

```text
warning: internal/service/annotation_behavior_test.go has type 100755, expected 100644
warning: internal/service/service_test.go has type 100755, expected 100644
warning: internal/service/annotation_behavior_test.go has type 100755, expected 100644
warning: internal/service/service_test.go has type 100755, expected 100644

```

### linux/arm64

- 容器内复现预期退出码：1
- 容器内复现实际退出码：1

stdout：

```text
$ go test ./internal/service -run "^TestCancelledRegistrationDoesNotAdvance$" -count=1
--- FAIL: TestCancelledRegistrationDoesNotAdvance (1.22s)
    annotation_behavior_test.go:237: cancelled state change succeeded
FAIL
FAIL	github.com/zhanglei10281852-gif/coldchain-custody-base/internal/service	1.400s
FAIL

```

stderr：

```text
warning: internal/service/annotation_behavior_test.go has type 100755, expected 100644
warning: internal/service/service_test.go has type 100755, expected 100644
warning: internal/service/annotation_behavior_test.go has type 100755, expected 100644
warning: internal/service/service_test.go has type 100755, expected 100644

```

## 通过条件

定向公开行为验证通过，相关包和全量测试通过，go vet 及 linux/amd64 构建通过。 定向命令 go test ./internal/service -run ^TestCancelledRegistrationDoesNotAdvance$ -count=1 必须由修复前失败变为修复后通过；相关包与 go test ./... -count=1 全量回归通过，回退 gold 关键修改后定向命令重新失败。
