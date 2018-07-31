在Python代码中调用Jar
----
### 1、安装jpype
### 2、测试代码
```
from jpype import *
startJVM(getDefaultJVMPath(), "-ea")
java.lang.System.out.println("Hello World")
shutdownJVM()
```
### 3、直接调用该类
```
startJVM(getDefaultJVMPath())
JavaClass = JClass("JavaClass")
jc = JavaClass()
jc.setValue('111')
print(jc.getValue())
shutdownJVM()
```
### 4、引用jar包
#### 在com目录下新建文件Test.java
```
package com;
public class Test {
    public String run(String str){
        return str;
    }
}
```

#### 编译
```
javac Test.java
```

#### 打包(必须把整个目录（报名和目录名要对应）打包，否则无法访问类)
```
jar cvf test.jar com
```

#### 5、Python调用
```
jvmPath = jpype.getDefaultJVMPath()
jarpath = 'test.jar'
jvmArg = '-Djava.class.path=%s' % jarpath
jpype.startJVM(jvmPath, '-ea', jvmArg)
javaClass = jpype.JClass("com.Test")
javaInstance = javaClass()
print(javaInstance.run(str))
jpype.shutdownJVM()
```
