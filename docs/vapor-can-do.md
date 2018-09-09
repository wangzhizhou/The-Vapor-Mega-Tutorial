如果你上一节中使用了`vapor --help`命令，你会发现一些有用的信息，其中就有告诉你Vapor能作哪些事情。

```bash
$ vapor --help
Usage: vapor command

Join our team chat if you have questions, need help,
or want to contribute: http://vapor.team

Commands:
       new Creates a new Vapor application from a template.
           Use --template=repo/template for github templates
           Use --template=full-url-here.git for non github templates
           Use --web to create a new web app
           Use --auth to create a new authenticated API app
           Use --api (default) to create a new API
     build Compiles the application.
       run Runs the compiled application.
     fetch Fetches the application's dependencies.
    update Updates your dependencies.
     clean Cleans temporary files--usually fixes
           a plethora of bizarre build errors.
      test Runs the application's tests.
     xcode Generates an Xcode project for development.
           Additionally links commonly used libraries.
   version Displays Vapor CLI version
     cloud Commands for interacting with Vapor Cloud.
    heroku Commands to help deploy to Heroku.
  provider Commands to help manage providers.

Use `vapor command --help` for more information on a command.
```

`Vapor`可以通过`new`命令，从模板创建工程，这种工程模板可以是自己定义的，也可以是Vapor默认提供的。自定义的工程模板可以放在本地，也可以放在github上。默认提供的模板有：`Web模板`、`身份认证模板`、`后端API开发模板`。其它的一些命令会在之后的实践中慢慢用到，我们可以先不用理会。