打开 Intellij IDEA, 选择创建新项目：

![[Notes/j-工程管理/开发构建/attachments/b8ff73f568e1cac0ef0feb53c81d4069_MD5.jpg]]

> 也可以选择从菜单栏 File -> New -> Project… 创建项目。

选择 Gradle => Java：

![[Notes/j-工程管理/开发构建/attachments/e4c71c28e01c26b909e7ded1a6f2d606_MD5.jpg]]

点击 Next 进入下一步，填写 GroupId、ArtifactId：

![[Notes/j-工程管理/开发构建/attachments/cab6a4f35fb94e8e9833da58874117fd_MD5.jpg]]

点击 Next 进入下一步，指定本地gradle的位置：

![[Notes/j-工程管理/开发构建/attachments/4bcbc7641f98a3d13fba44e69cb7a65f_MD5.jpg]]

点击 Next 进入下一步，指定项目位置，我们将其放在 ~/Desktop/demo 目录中：

![[Notes/j-工程管理/开发构建/attachments/a70c51dd7788a5c64ceefff0a3ed99f7_MD5.jpg]]

点击 Finish 。完成项目创建，IDE 进入项目编辑界面：

![[Notes/j-工程管理/开发构建/attachments/0cf052d54995d0d7e918d493aba523a3_MD5.jpg]]

在 `src/main/java` 目录下创建Main类，将其内容修改如下：

```java
public class Main {

    public static void main(String[] args) {
        System.out.println("Hello World");
    }
}
```

Copy

右击，选择'Run Main.main()' ，运行结果如下：

![[Notes/j-工程管理/开发构建/attachments/d001726fb43ebd6910d7bbe859ce4b5b_MD5.jpg]]

# 打开基于 Gradle 的 Java 项目

创建项目后，该项目会自动放入 IDE 的历史记录中，如果当前未打开该项目，可以从 File -> Open Recent 找到它，点击打开即可。但是历史记录是有数量限制的，如果不曾打开过一个项目，或者历史记录中已经没有该项目了，便需要用最基础的方法：File -> Open -> 选择build.gralde文件 -> Open As Project 即可。
