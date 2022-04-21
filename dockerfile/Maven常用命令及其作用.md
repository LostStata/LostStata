#### 一、Maven常用命令及其作用
##### 命令	描述

|    命令     |                             描述                             |
| :---------: | :----------------------------------------------------------: |
|  mvn clean  |          对项目进行清理，删除target目录下编译的内容          |
| mvn compile |                        编译项目源代码                        |
|  mvn test   |                      对项目进行运行测试                      |
| mvn package | 打包文件并存放到项目的target目录下，打包好的文件通常都是编译后的class文件 |
| mvn install | 在本地仓库生成仓库的安装包，可供其他项目引用，同时打包后的文件放到项目的target目录下 |


#### 二、常用命令使用场景举例

##### 1、mvn clean package
依次执行了clean、resources、compile、testResources、testCompile、test、jar(打包)等７个阶段
package命令完成了项目编译、单元测试、打包功能，但没有把打好的可执行jar包（war包或其它形式的包）布署到本地maven仓库和远程maven私服仓库

##### 2、mvn clean install
依次执行了clean、resources、compile、testResources、testCompile、test、jar(打包)、install等8个阶段
install命令完成了项目编译、单元测试、打包功能，同时把打好的可执行jar包（war包或其它形式的包）布署到本地maven仓库，但没有布署到远程maven私服仓库

##### 3、mvn clean deploy
依次执行了clean、resources、compile、testResources、testCompile、test、jar(打包)、install、deploy等９个阶段，deploy命令完成了项目编译、单元测试、打包功能，同时把打好的可执行jar包（war包或其它形式的包）布署到本地maven仓库和远程maven私服仓库

#### 三、常见问题

##### 1、mvn clean install 和 mvn install 的区别
根据maven在执行一个生命周期命令时，理论上讲，不做mvn install 得到的jar包应该是最新的，除非使用其他方式修改jar包的内容，但没有修改源代码
平时可以使用mvn install ，不使用clean会节省时间，但是最保险的方式还是mvn clean install，这样可以生成最新的jar包或者其他包

##### 2、maven两种跳过单元测试方法的区别
mvn package -Dmaven.test.skip=true
不但跳过了单元测试的运行，同时也跳过了测试代码的编译mvn package -DskipTests
跳过单元测试，但是会继续编译。如果没时间修改单元测试的bug，或者单元测试编译错误，则使用第一种，不要使用第二种
