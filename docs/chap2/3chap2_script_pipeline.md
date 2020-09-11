# **第三节 脚本化Pipeline**

脚本化流水线, 与声明式一样的是, 是建立在底层流水线的子系统上的。与声明式不同的是, 脚本化流水线实际上是由 `Groovy`构建的通用 `DSL` 。 `Groovy` 语言提供的大部分功能都可以用于脚本化流水线的用户。这意味着它是一个非常有表现力和灵活的工具，可以通过它编写持续交付流水线。

## 1.流程控制

脚本化流水线从`Jenkinsfile`的顶部开始向下串行执行, 就像 `Groovy` 或其他语言中的大多数传统脚本一样。 因此，提供流控制取决于 `Groovy` 表达式, 比如 `if/else` 条件, 例如:

```
node {
    stage('Example') {
        if (env.BRANCH_NAME == 'master') {
            echo 'I only execute on the master branch'
        } else {
            echo 'I execute elsewhere'
        }
    }
}
```

另一种方法是使用`Groovy`的异常处理支持来管理脚本化流水线流控制。当 **步骤 失败** ，无论什么原因，它们都会抛出一个异常。处理错误的行为必须使用`Groovy`中的 `try/catch/finally` 块 , 例如:

```
node {
    stage('Example') {
        try {
            sh 'exit 1'
        }
        catch (exc) {
            echo 'Something failed, I should sound the klaxons!'
            throw
        }
    }
}
```