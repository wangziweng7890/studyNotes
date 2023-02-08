简介
说到前端开发，就一定离不开npm，作为前端包管理的老大，npm是我们必须知道的一个东西。

虽然每天都用npm安装包，但是你们对package.json和package-lock.json这两个文件又了解多少呢？今天笔者就来详细分析下这两个文件，希望能对大家有所帮助。

在说package.json和package-lock.json之前，我们先来说说npm安装包的方式和npm的安装流程。

npm 安装包的方式
npm 安装包的方式分为本地安装和全局安装。安装使用npm install或简写形式npm i。

本地安装
本地安装的包只能在当前目录下使用。

本地安装很简单，以element-ui为例

npm i element-ui
1
在实际项目开发过程中，本地安装我们都会先生成package.json文件。然后安装的时候包的版本会自动记录在该文件中。

本地安装的包又分为开发依赖(devDependencies)和生产依赖(dependencies)。

对于一些只会在开发环境中用到的包我们可以安装在开发依赖(devDependencies)中。比如webpack、vite等等。

对于一些会在生产环境中用到的包我们可以安装在生产依赖(dependencies)中。比如element-ui等等。

那么怎么让安装的包分别放到对应的依赖位置呢？

如果想要安装的包放在开发依赖(devDependencies)中，我们传递参数 --save-dev 或 -D 即可。

如果想要安装的包放在生产依赖(dependencies)中，我们传递参数 --save 或 -S 即可。

注意如果我们没传递依赖参数。会把包默认安装到生产依赖(dependencies)中

全局安装
全局安装的包能在当本机所有目录下使用。我们在安装包的时候传递参数-g或--global即可。

npm i -g element-ui

npm i --global element-ui
1
2
3
全局安装的包路径我们可以通过npm config get prefix查看。

[外链图片转存失败,源站可能有防盗链机制,建议将图片保存下来直接上传(img-v66HX789-1666599423262)(https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/44f1dc55f9e94b54b9c4c67c260b4667~tplv-k3u1fbpfcp-watermark.image?)]

npm 安装流程
前面我们只是介绍了 npm install 是用来安装依赖的，下面再讲一下它到底是怎么安装的以及一些具体的安装细节。

查找 npm 的配置信息
执行安装命令之后，npm 首先会去查找 npm 的配置信息。 其中，我们最熟悉的就是安装时候的源信息。npm 会在项目中查找是否有 .npmrc 文件，没有的话会再检查全局配置的 .npmrc ，还没有的话就会使用 npm 内置的 .npmrc 文件。

构建依赖树
获取完配置文件之后，就会构建依赖树。首先会检查下项目中是否有 package-lock.json 🔐文件：存在 lock 文件的话，会判断 lock 文件和 package.json 中使用的依赖版本是否一致，如果一致的话就使用 lock 中的信息，反之就会使用 npm 中的信息；那如果没有 lock 文件的话，就会直接使用 package.json 中的信息生成依赖树。

根据依赖树下载完整的依赖资源
在有了依赖树之后，就可以根据依赖树下载完整的依赖资源。在下载之前，会先检查下是否有缓存资源，如果存在缓存资源的话，那么直接将缓存资源解压到 node_modules 中。如果没有缓存资源，那么会先将 npm 远程仓库中的包下载至本地，然后会进行包的完整性校验，校验通过后将其添加的缓存中并解压到 node_modules 中。

生成 package-lock.json 文件
生成 package-lock.json 文件。 这个文件主要是用来锁定包的版本，这个笔者后面会详细说。

接下来我们说说 package.json和package-lock.json生成方式

package.json和package-lock.json生成方式
首先要明确一点，package.json不会自动生成，需要我们使用命令创建。package-lock.json是自动生成的，我们使用 npm install 安装包后就会自动生成。

下面我们介绍下怎么创建package.json。

npm 为我们提供了创建 package.json 文件的命令 npm init，执行该命令会问几个基本问题，如包名称、版本号、作者信息、入口文件、仓库地址、关键字、描述、许可协议等，多数问题已经提供了默认值，你可以在问题后敲回车接受默认值。

基本问题问完之后 npm 会在生成文件之前把 package.json 文件内容打出来供你确认，点击enter键就会生成package.json 文件。

[外链图片转存失败,源站可能有防盗链机制,建议将图片保存下来直接上传(img-R0458CB3-1666599423265)(https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/71ae77e77b6240c98c67ededf9de8cbc~tplv-k3u1fbpfcp-watermark.image?)]

如果觉得这样一步一步太慢的话我们还可以一键生成。使用npm init -y 或者 npm init -f就可以一键生成package.json 文件啦。(npm init -y 和 npm init -f是npm init --yes 和 npm init --force的简写形式)。

[外链图片转存失败,源站可能有防盗链机制,建议将图片保存下来直接上传(img-LCc1jvJw-1666599423272)(https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/f02d7e379e4a45e684d04606ff76934d~tplv-k3u1fbpfcp-watermark.image?)]

到这里，小伙伴们肯定有疑问了，一键生成的package.json里面的内容是在哪里控制的呢？比如我想一键生成出来的package.json里面的author是我自己需要怎么做呢？

我们来看看npm的初始配置吧，使用npm config ls -l命令查看npm的所有配置。

[外链图片转存失败,源站可能有防盗链机制,建议将图片保存下来直接上传(img-quRbA2tK-1666599423274)(https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8536165234df48489d2f0c18f23cafce~tplv-k3u1fbpfcp-watermark.image?)]

可以发现我们的init-author-name的配置是空，所以我们来设置下。

# 设置配置
npm config set init-author-name 'randy'

# 查看配置
npm config get init-author-name
1
2
3
4
5
我们再来生成一遍package.json

[外链图片转存失败,源站可能有防盗链机制,建议将图片保存下来直接上传(img-JocZBD9z-1666599423282)(https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/97ce5451f9324cea8b5456db582c11fb~tplv-k3u1fbpfcp-watermark.image?)]

可以发现，我们的author就有默认值啦，其他的默认配置都可以设置，这里笔者就不再举例了。

package.json详解
package.json文件，它是项目的配置文件，常见的配置有配置项目启动、打包命令，声明依赖包等。package.json文件是一个JSON对象，该对象的每一个成员就是当前项目的一项设置。

[外链图片转存失败,源站可能有防盗链机制,建议将图片保存下来直接上传(img-CBfU93TR-1666599423283)(https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4533e8caf65d43bd82e507f0aa02c8d7~tplv-k3u1fbpfcp-watermark.image?)]

下面笔者分别介绍各个配置代表的意义。

name
name很容易理解，就是项目的名称。

version
version字段表示该项目包的版本号，它是一个字符串。在每次项目改动后，即将发布时，都要同步的去更改项目的版本号。

npm依赖包版本号
npm默认所有的Node包都使用语义化版本号，这是一套指导开发人员如何增长版本号的规则。

每个版本号都形如：1.2.3，有三部分组成，依次叫主版本号、次版本号、修订号；

主版本号
当新版本无法兼容基于前一版本的代码时，则提高主版本号；

次版本号
当新版本新增了功能和特性，但仍兼容前一版本的代码时，则提高次版本号；

修订号
当新版本仅仅修正了漏洞或者增强了效率，仍然兼容前一版本代码，则提高修订号；

最优版本
默认情况下，npm install xxx --save下载的都是最新版本，并且会在package.json文件里登记一个最优版本号。

举个例子，我们安装element-ui

npm i element-ui -S
1
在安装的时候它会选择目前的一个最新的版本进行安装，我们查看github，看到element-ui的最新版本是2.15.9。

[外链图片转存失败,源站可能有防盗链机制,建议将图片保存下来直接上传(img-jBddkJDA-1666599423286)(https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/8fabec181432479ea4c07c479d7a9a29~tplv-k3u1fbpfcp-watermark.image?)]

这里我们自动安装的也就是2.15.9版本，并且版本号并不是固定的而是一个最优版本。

[外链图片转存失败,源站可能有防盗链机制,建议将图片保存下来直接上传(img-WB2esfRS-1666599423291)(https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/fcb83c8b8a66454ba717dbf801666d2f~tplv-k3u1fbpfcp-watermark.image?)]

那什么是最优版本呢？

最优版本会在版本前多了一个^符号，这个符号其实是有特殊意义的。同理这个符号还有可能是~符号。

那这两个符号有什么区别呢？

^ 兼容某个大版本
^意味着下载的包可能会是更高的次版本号或者修订版本号。(也就是主版本不能变，次版本、修订版本可以随意变)。

兼容某个大版本

如：^1.1.2 ，表示>=1.1.2 <2.0.0，可以是1.1.2，1.1.3，.....，1.1.n，1.2.n，.....，1.n.n 
1
2
3
~ 兼容某个次版本
而~意味着有可能会有更高的修订版本号。(也就是主版本、次版本不能变，修订版本可以随意变)。

兼容某个次版本 

如：~1.1.2，表示>=1.1.2 <1.2.0，可以是1.1.2，1.1.3，1.1.4，.....，1.1.n 
1
2
3
看了上面版本号的指定后，我们可以知道，当我们使用了 ^ 或者 ~ 来控制依赖包版本号的时候 ，多人开发，就有可能存在大家安装的依赖包版本不一样的情况，就会存在项目运行的结果不一样。

举个例子

假设我们中安装了 vue, 当我们运行安装 npm install vue -save 的时候，在项目中的package.json 的 vue 版本是 vue: ^3.0.0, 我们电脑安装的vue版本就是 3.0.0 版本，我们把项目代码提交后，过了一段时间，vue 发布了新版本 3.1.0，这时新来一个同事，从新 git clone 克隆项目，执行 npm install安装的时候，在他电脑的vue版本就是 3.1.0了，因为^只是锁了主要版本，这样我们电脑中的vue版本就会不一样。从理论上讲（大家都遵循语义版本控制的话） ，它们应该仍然是兼容的(因为次版本号的修改只是新功能，并不是不兼容旧版本)，这里假设 vue 次版本的迭代会影响我们正在使用的功能，那么这就会导致严重的问题，我们同一个项目会产生不同的运行结果。

这时也许有同学想到，那么我们在package.json上面锁死依赖包的版本号不就可以了? 直接写 vue: 3.0.0锁死，这样大家安装vue的版本都是3.0.0版本了。

这个想法固然是不错的，但是你只能控制你自己的项目锁死版本号，那你项目中依赖包的依赖包呢？你怎么控制限制别人锁死版本号呢？

description
description字段用来描述这个项目包，它是一个字符串，可以让其他开发者在 npm 的搜索中发现我们的项目包。

author
author顾名思义就是作者，表示该项目包的作者。

contributors
contributors表示该项目包的贡献者，和author不同的是，该字段是一个数组，包含所有的贡献者

homepage
homepage就是项目的主页地址了，它是一个字符串。

repository
repository表示代码的存放仓库地址

bugs
bugs表示项目提交问题的地址

依赖类型
前面说到有开发依赖和生产依赖，其实npm还有同版本的依赖、捆绑依赖、可选依赖。

dependencies 生产依赖
devDependencies 开发依赖
peerDependencies 同版本的依赖
bundledDependencies 捆绑依赖
optionalDependencies 可选依赖
dependencies 生产依赖
dependencies 表示项目依赖，这些依赖都会成为你的线上生产环境中的代码组成的部分。当它关联到 npm 包被下载的时候，dependencies下的模块也会作为依赖，一起被下载。

devDependencies 开发依赖
devDependencies表示开发依赖, 不会被自动下载的。因为 devDependencies 一般是用于开发阶段起作用或是只能用于开发环境中被用到的。 比如说我们用到的 Webpack，预处理器 babel-loader、scss-loader，测试工具E2E等， 这些都相当于是辅助的工具包, 无需在生产环境被使用到的。

这里笔者啰嗦一句，当我们只开发应用，不对外开源的话，包随意放在dependencies或devDependencies是不影响的，因为被用到的模块不管你再哪个依赖里面都会被打包。但是如果开发的是库文件，是开源的，已经上传到npm仓库的，这个你就得严格区分dependencies和devDependencies依赖了。因为当你在安装第三方包的时候，只会同步下载第三方包dependencies里面的依赖，所以，当你第三方包的某依赖写到devDependencies中，被别人下载后没下载到该依赖是会报错运行不了的。

peerDependencies 同版本的依赖
peerDependencies 表示同版本的依赖，很多小伙伴肯定难以理解，下面笔者举个例子就明白了。

vuex我们都使用过，vue的状态管理器，它肯定是依赖vue的，但是我们查看它的package.json文件，会发现dependencies并没有依赖vue。

[外链图片转存失败,源站可能有防盗链机制,建议将图片保存下来直接上传(img-voQ2AW1I-1666599423292)(https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/fb59a80992a44b978d60158c27b43ce6~tplv-k3u1fbpfcp-watermark.image?)]

啊！这是怎么回事呢？其实他就是用到了peerDependencies 同版本的依赖。

[外链图片转存失败,源站可能有防盗链机制,建议将图片保存下来直接上传(img-ejTfYLaP-1666599423296)(https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/3c3791bca45d4279851709ce72ab7026~tplv-k3u1fbpfcp-watermark.image?)]

这下知道什么意思了吧，就是这个包它非常清楚的知道，你使用我的时候必定是在vue环境下使用，所以你一定会安装vue，所以我就没必要再在dependencies下申明vue依赖了，只需要在peerDependencies申明依赖就行了，和主项目共用同一个vue依赖，避免了重复安装。

这样的好处就是node_modules里面不会安装两个vue。减少了node_modules体积的同时大大提高了安装速度。

bundledDependencies 捆绑依赖
bundledDependencies和前面的依赖不同它是一个数组，里面定义了捆绑依赖。

"name": "test",
"version": "1.0.0",
"bundledDependencies": [ "bundleD1", "bundleD2" ]
1
2
3
当我们此时执行 npm pack的时候, 就会生成一个 test-1.0.0.tgz的压缩包, 在该压缩包中还包含了 bundleD1和 bundleD2 两个安装包。

实际使用到这个压缩包的时候，npm install test-1.0.0.tgz 的命令时, bundleD1和 bundleD2 也会被安装的。

这样做的意义是什么呢？

如果bundleD1和bundleD2包在npm上有，能下载到固然没意义。但是当bundleD1和bundleD2是自己临时开发的，并且当前项目又依赖这两个包，并且这两个包在npm上没有，下载不到，这下就有意义了。相当于跟主包一起捆绑发布。

这里需要注意的是: 在 bundledDependencies 中指定的依赖包, 必须先在dependencies 和 devDependencies 声明过, 否则 npm pack 阶段是会报错的。

optionalDependencies 可选依赖
optionalDependencies表示可选依赖，就是说当你安装对应的依赖项安装失败了, 也不会对整个安装过程有影响的。一般我们很少会用到它, 这里我是不建议大家去使用, 可能会增加项目的不确定性和复杂性。 了解即可。

engines
当我们维护一些旧项目时，可能对npm包的版本或者Node版本有特殊要求，如果不满足条件就可能无法将项目跑起来。为了让项目开箱即用，可以在engines字段中说明具体的版本号：

"engines": {
  "node": ">=8.10.3 <12.13.0",
  "npm": ">=6.9.0"
}
1
2
3
4
需要注意，engines只是起一个说明的作用，即使用户安装的版本不符合要求，也不影响依赖包的安装。

scripts
关于scripts可以看看笔者前面写的 npm script的那些骚操作，感兴趣的小伙伴可以自行查看。

config
config字段用来配置scripts运行时的配置参数。

main
main 字段用来指定加载的入口文件，在 browser 和 Node 环境中都可以使用。如果我们将项目发布为npm包，那么当使用 require 导入npm包时，返回的就是main字段所列出的文件的module.exports 属性。如果不指定该字段，默认是项目根目录下的index.js。如果没找到，就会报错。

browser
browser字段可以定义 npm 包在 browser 环境下的入口文件。如果 npm 包只在 web 端使用，并且严禁在 server 端使用，使用 browser 来定义入口文件。

module
module字段可以定义 npm 包的 ESM 规范的入口文件，browser 环境和 node 环境均可使用。如果 npm 包导出的是 ESM 规范的包，使用 module 来定义入口文件。

需要注意，*.js 文件是使用 commonJS 规范的语法(require(‘xxx’))，* .mjs 是用 ESM 规范的语法(import ‘xxx’)。

上面三个的入口入口文件相关的配置是有差别的，特别是在不同的使用场景下。在Web环境中，如果使用loader加载ESM（ES module），那么这三个配置的加载顺序是browser→module→main，如果使用require加载CommonJS模块，则加载的顺序为main→module→browser。

Webpack在进行项目构建时，有一个target选项，默认为Web，即构建Web应用。如果需要编译一些同构项目，如node项目，则只需将webpack.config.js的target选项设置为node进行构建即可。如果在Node环境中加载CommonJS模块，或者ESM，则只有main字段有效。

bin
bin字段用来指定各个内部命令对应的可执行文件的位置

"bin": { "someTool": "./bin/someTool.js" },
"scripts": {
  "dev": "someTool build"
}
1
2
3
4
上面的例子相当于定义了一个命令someTool，它对应的可执行文件的位置是./bin/someTool.js，这样我们就能在scripts里面直接使用该命令了someTool。

files
files配置是一个数组，用来描述当把npm包作为依赖包安装时需要说明的文件列表。当npm包发布时，files指定的文件会被推送到npm服务器中，如果指定的是文件夹，那么该文件夹下面所有的文件都会被提交。

如果有不想提交的文件，可以在项目根目录中新建一个.npmignore文件，并在其中说明不需要提交的文件，防止垃圾文件推送到npm上。这个文件的形式和.gitignore类似。写在这个文件中的文件即便被写在files属性里也会被排除在外。比如可以在该文件中这样写：

node_modules

.vscode

build

.DS_Store
1
2
3
4
5
6
7
man
man 命令是 Linux 中的帮助指令，通过该指令可以查看 Linux 中的指令帮助、配置文件帮助和编程帮助等信息。如果 node.js 模块是一个全局的命令行工具，在 package.json 通过 man 属性可以指定 man 命令查找的文档地址。

man 字段可以指定一个或多个文件, 当执行man {包名}时, 会展现给用户文档内容。

需要注意：

man文件必须以数字结尾，如果经过压缩，还可以使用.gz后缀。这个数字表示文件安装到哪个 man 节中；
如果 man 文件名称不是以模块名称开头的，安装的时候会加上模块名称前缀。
对于上面的配置，可以使用以下命令来执行查看文档：

man npm-access
man npm-audit
1
2
directories
directories字段用来规范项目的目录。node.js 模块是基于 CommonJS 模块化规范实现的，需要严格遵循 CommonJS 规范。模块目录下除了必须包含包项目描述文件 package.json 以外，还需要包含以下目录：

bin ：存放可执行二进制文件的目录
lib ：存放js代码的目录
doc ：存放文档的目录
test ：存放单元测试用例代码的目录
…
在实际的项目目录中，我们可能没有按照这个规范进行命名，那么就可以在directories字段指定每个目录对应的文件路径：

"directories": {
    "bin": "./bin",
    "lib": "./lib",
    "doc": "./doc",
    "test" "./test",
    "man": "./man"
},
1
2
3
4
5
6
7
这个属性实际上没有什么实际的作用，当然不排除未来会有什么比较有意义的用处。

private
private字段可以防止我们意外地将私有库发布到npm服务器。只需要将该字段设置为true。

preferGlobal
preferGlobal字段表示用户不把该模块安装为全局模块。如果设置为true并把该模块安装为全局模块就会显示警告。

它并不会真正的防止用户进行局部的安装，只是对用户进行提示，防止产生误解。

publishConfig
publishConfig配置会在模块发布时生效，用于设置发布时一些配置项的集合。如果不想模块被默认标记为最新，或者不想发布到公共仓库，可以在这里配置tag或仓库地址。

os
os字段可以让我们设置该npm包可以在什么操作系统使用，不能再什么操作系统使用。如果我们希望开发的npm包只运行在linux，为了避免出现不必要的异常，建议使用Windows系统的用户不要安装它，这时就可以使用os配置：

"os" ["linux"]   // 适用的操作系统
"os" ["!win32"]  // 禁用的操作系统
1
2
cpu
该配置和OS配置类似，用CPU可以更准确的限制用户的安装环境：

"cpu" ["x64", "AMD64"]   // 适用的cpu
"cpu" ["!arm", "!mips"]  // 禁用的cpu
1
2
可以看到，黑名单和白名单的区别就是，黑名单在前面加了一个!。

license
license 字段用于指定软件的开源协议，开源协议表述了其他人获得代码后拥有的权利，可以对代码进行何种操作，何种操作又是被禁止的。常见的协议如下：

MIT ：只要用户在项目副本中包含了版权声明和许可声明，他们就可以拿你的代码做任何想做的事情，你也无需承担任何责任。
Apache ：类似于 MIT ，同时还包含了贡献者向用户提供专利授权相关的条款。
GPL ：修改项目代码的用户再次分发源码或二进制代码时，必须公布他的相关修改。
typings或者types
typings或者types字段用来指定TypeScript的入口文件

eslintConfig
eslint的配置可以写在单独的配置文件.eslintrc.json 中，也可以写在package.json文件的eslintConfig配置项中。

babel
babel的配置可以写在单独的配置文件babel.config.json 中，也可以写在package.json文件的babel配置项中。

unpkg
使用该字段可以让 npm 上所有的文件都开启 cdn 服务，该CND服务由unpkg提供。

lint-staged
lint-staged是一个在Git暂存文件上运行linters的工具，配置后每次修改一个文件即可给所有文件执行一次lint检查，通常配合gitHooks一起使用。

使用lint-staged时，每次提交代码只会检查当前改动的文件。

gitHooks
gitHooks用来定义一个钩子，在提交（commit）之前执行ESlint检查。在执行lint命令后，会自动修复暂存区的文件。修复之后的文件并不会存储在暂存区，所以需要用git add命令将修复后的文件重新加入暂存区。在执行pre-commit命令之后，如果没有错误，就会执行git commit命令

"gitHooks": {
  "pre-commit": "lint-staged"
}
1
2
3
这里就是配合lint-staged来进行代码的检查操作。每次commit前运行lint-staged。

browserslist
browserslist字段用来告知支持哪些浏览器及版本。Babel、postcss、eslint等工具都会用到它。

当然你也可以单独建立.browserslistrc文件来配置。如需了解更多可以查看browserslist 官方文档。

package-lock.json 详解
前面说了package.json里面的包版本不是一个具体的版本，而是一个最优版本。而package-lock.json里面定义的是某个包的具体版本，以及包之间的层叠关系。

一个 package-lock.json 里面的每个依赖主要是有以下的几部分组成的:

Version: 依赖包的版本号
Resolved: 依赖包的安装源(其实就是可以理解为下载地址)
Intergrity: 表明完整性的 Hash 值
Dev: 表示该模块是否为顶级模块的开发依赖或者是一个的传递依赖关系
requires: 依赖包所需要的所有依赖项,对应依赖包 package.json 里 dependencices 中的依赖项
dependencices: 依赖包 node_modeles 中依赖的包(特殊情况下才存在)。并不是所有的子依赖都有 dependencies 属性,只有子依赖的依赖和当前已安装在根目录的 node_modules 中的依赖冲突之后, 才会有这个属性。 这可能涉及嵌套情况的依赖管理。
假设我们的项目依赖A、B、C三个包，并且B包又依赖特定版本的C包，所以B包就会有dependencices，那么我们项目的整个node_modules的目录结构如下

node_modules
  A.package
  B.package
    node_modules
      C.package
  C.package
1
2
3
4
5
6
那什么情况下package.json和package-lock.json里面的版本号一致呢？
当package.json里面不再使用最优版本，而是一个特定有效版本，也就是版本号前不带修饰符，这样package.json和package-lock.json里面的版本号才是一致的。

或者我们npm install xxx@1.1.2 带上版本号，安装特定版本的包package.json和package-lock.json里面的版本号也是一致的。

或者我们package.json里面定义的包版本就是最新的，我们安装的时候package.json和package-lock.json里面的版本号也会是一致的。

package-lock.json什么时候会变
package-lock.json在npm install的时候会自动生成。

当我们修改依赖位置，比如将部分包的位置从 dependencies 移动到 devDependencies 这种操作，虽然包未变，但是也会影响 package-lock.json，会将部分包的 dev 字段设置为 true。

还有如果我们安装源 registry 不同，执行 npm install 时也会修改 package-lock.json。

因为他是会记录我们的依赖包地址的。

[外链图片转存失败,源站可能有防盗链机制,建议将图片保存下来直接上传(img-rHFrTqdy-1666599423299)(https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/9ebf8c6a0b2348bebf47d03975ea745a~tplv-k3u1fbpfcp-watermark.image?)]

当我们使用npm install添加或npm uninstall移除包的时候，同时也会修改 package-lock.json。

当我们更新某个包的版本的时候，同时也会修改 package-lock.json。

怎么安装指定版本的包
我们知道，使用npm install xxx会安装某最新版本的包。

如果在根目录下直接运行npm install，那么npm会根据package.json下载所有的最新版本的包。

那么怎么安装指定版本的包呢？

我们只需要安装的时候带上版本号就可以啦。比如笔者上面element-ui默认安装的最新版本是2.15.9。我们来安装一个2.15.7的特定版本。

npm install element-ui@2.15.7
1
我们最新版本的2.15.9的版本包就会被替换。并且它会同步修改我们的package.json和package-lock.json。

[外链图片转存失败,源站可能有防盗链机制,建议将图片保存下来直接上传(img-dAFaEpcE-1666599423301)(https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/04a9e2f4eded47d3b4e5bfa8eb954370~tplv-k3u1fbpfcp-watermark.image?)]

[外链图片转存失败,源站可能有防盗链机制,建议将图片保存下来直接上传(img-3bYU5mzR-1666599423304)(https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2c0cc80e55a848cba69847e2c7aab904~tplv-k3u1fbpfcp-watermark.image?)]

我们知道，安装完生成package-lock.json之后包的版本就会被固定下来，后续不管你怎么npm install都不会再变化。

如果我想更新到最新版本的包该怎么办呢？

怎么更新包的版本
更新包的版本有很多种方式。

npm install指定版本
我们知道最新包的版本，直接按上面的安装指定包并带上版本号即可。原理和安装特定版本的包一致。

npm install element-ui@2.15.9
1
它会同步修改我们的package.json和package-lock.json。

npm update
不知道最新包的版本，我们运行npm update xxx 即可。

npm update element-ui
1
它会将指定包更新成最新版本，并且同步修改我们的package.json和package-lock.json。

如果想更新项目下所有的包，直接运行 npm update 即可。

请注意，更新包它是会遵循我们的最优版本号控制的。也就是说当我们的版本是被^修饰时，再怎么更新它只会更新次版本和修订版本，并不会更新主版本。

比如笔者安装的element-ui包的版本是1.0.0

[外链图片转存失败,源站可能有防盗链机制,建议将图片保存下来直接上传(img-kyNArytr-1666599423307)(https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/4c3f331cf3f84143b335509837783b49~tplv-k3u1fbpfcp-watermark.image?)]

package.json里面版本号如下

[外链图片转存失败,源站可能有防盗链机制,建议将图片保存下来直接上传(img-w59A9PYu-1666599423312)(https://p3-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a91ab3d1ef1c4f0584e78c62bf61b98b~tplv-k3u1fbpfcp-watermark.image?)]

当我们npm update element-ui更新包时，它只会更新成最新的次版本和修订版本，并不会更新主版本。

[外链图片转存失败,源站可能有防盗链机制,建议将图片保存下来直接上传(img-zrn8lOrA-1666599423314)(https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/a3583b8c48bd41b18557413227cab298~tplv-k3u1fbpfcp-watermark.image?)]

package.json里面版本号如下

[外链图片转存失败,源站可能有防盗链机制,建议将图片保存下来直接上传(img-eWHWniGj-1666599423319)(https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/bde739ca245b4eefb7e7ce6851967116~tplv-k3u1fbpfcp-watermark.image?)]

删除 package-lock.json
如果想更新项目下所有的包我们还可以将package-lock.json删除，然后运行npm install，它也会安装所有的最新版本的包，并且会重新生成package-lock.json。

package-lock.json 需要提交到仓库吗
至于我们要不要提交 lockfiles 到仓库中？这个就需要看我们具体的项目的定位了。

如果是开发一个应用, 我的理解是 package-lock.json文件提交到代码版本仓库.这样可以保证项目中成员、运维部署成员或者是 CI 系统, 在执行 npm install后, 保证在不同的节点能得到完全一致的依赖安装的内容，减少bug的出现。

如果你的目标是开发一个给外部环境用的库，那么就需要认真考虑一下了, 因为库文件一般都是被其他项目依赖的，在不使用 package-lock.json的情况下，就可以复用主项目已经加载过的包，减少依赖重复和体积

如果说我们开发的库依赖了一个精确版本号的模块，那么在我们去提交 lockfiles 到仓库中可能就会出现, 同一个依赖被不同版本都被下载的情况，这样加大了node_modules的体积。如果我们作为一个库的开发者， 其实如果真的使用到某个特定的版本依赖的需求, 那么定义peerDependencies 是一个更好的选择。

所以, 我个人比较推荐的一个做法是:把 package-lock.json一起提交到仓库中去, 不需要 ignore。但是在执行 npm publish 命令的时候,也就是发布一个库的时候, 它其实应该是被忽略的不应该被发布出去的。

可以不使用package-lock.json吗
在项目中如果没有锁版本的必要，可以不使用package-lock.json，在安装模块时指定--no-lock即可。

依赖包版本精准控制
我们都知道package-lock.json主要是用来锁定包版本的。

其实在npm的发展历史中，控制包的版本有package-lock.json和npm-shrinkwrap.json两种方式

npm-shrinkwrap.json
在npm5之前（不包括5），主要是用npm-shrinkwrap.json来精准控制安装指定版本的npm包。

那怎么生成npm-shrinkwrap.json呢？其实一条命令npm shrinkwrap就能搞定。

它会生成一个npm-shrinkwrap.json文件，记录目前所有依赖包（及更底层依赖包）的版本信息。这样当以后其他人运行npm install命令时，npm首先会找到npm-shrinkwrap.json文件，依照其中的信息准确的安装每一个依赖包，只有当这个文件不存在时，npm才会使用package.json。

因为这是npm5之前的方案，所以现在基本上看不到了。现在都是package-lock.json方案。

package-lock.json
相较npm-shrinkwrap.json，package-lock.json的优点是能自动生成。

官方文档是这样解释的：package-lock.json 它会在 npm 更改 node_modules 目录树或者 package.json 时自动生成的 ，它准确的描述了当前项目npm包的依赖树，并且在随后的安装中会根据 package-lock.json 来安装，保证是相同的一个依赖树，不考虑这个过程中是否有某个依赖有小版本的更新。

当我们在一个项目中npm install时候，会自动生成一个package-lock.json文件，和package.json在同一级目录下。package-lock.json记录了项目的一些信息和所依赖的模块。这样在每次安装都会出现相同的结果，不管你在什么机器上面或什么时候安装。

可以看到package-lock.json里面包的版本不再是最优版本啦，而是一个固定版本，这也是它为什么锁定版本的原因。

[外链图片转存失败,源站可能有防盗链机制,建议将图片保存下来直接上传(img-VSqmlky2-1666599423321)(https://p6-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/2d4b3cd660cd4a59bf069d83a1217850~tplv-k3u1fbpfcp-watermark.image?)]

并且我们依赖的依赖它也会安装一个特定的版本，并锁定起来。比如element-ui所依赖的throttle-debounce

[外链图片转存失败,源站可能有防盗链机制,建议将图片保存下来直接上传(img-2TTmX0MQ-1666599423325)(https://p1-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/34c1191769cc4220845ed38cf0a9f328~tplv-k3u1fbpfcp-watermark.image?)]

所以当我们下次再npm install时候，npm 发现如果项目中有 package-lock.json 文件，则会根据 package-lock.json 里的内容来处理和安装依赖而不再根据 package.json。

可见package-lock.json原理就是将项目中用到的所有依赖都指定一个固定的版本，后续安装直接读取该文件来安装依赖。

在这里需要注意，如果用的是cnpm是不会自动生成package-lock.json文件的。我们平时开发最好的实践就是使用npm然后将镜像设置为淘宝源就可以了。

// 设置淘宝镜像
npm config set registry https://registry.npm.taobao.org

// 查看镜像源
npm config get registry
1
2
3
4
5
或许是package-lock.json最佳的实操建议
当我们的项目第一次去搭建的时候, 使用 npm install 安装依赖包, 并去提交 package.json、package-lock.json, 至于node_moduled目录是不用提交的。

当我们作为项目的新成员的时候, checkout/clone项目的时候, 执行一次 npm install 去安装依赖包，它会根据package-lock.json下载对应固定版本的包。

当我们出现了需要升级依赖的需求的时候:

升级小版本的时候, 依靠 npm update
升级大版本的时候, 依靠 npm install xxx
当然我们也有一种方法, 直接去修改 package.json 中的版本号, 并去执行 npm install 去升级版本
当我们本地升级新版本后确认没有问题之后, 去提交新的 package.json 和 package-lock.json文件。
当我们出现了需要降级依赖的需求的时候:

我们去执行npm install @x.x.x 命令后，验证没有问题之后, 是需要提交新的 package.json 和 package-lock.json 文件。
当我们出现了需要删除依赖的需求的时候:

当我们执行 npm uninstall 命令后，需要去验证，提交新的 package.json 和 package-lock.json 文件。
或者是更加暴力一点, 直接操作 package.json, 删除对应的依赖, 执行 npm install 命令, 需要去验证，提交新的package.json 和 package-lock.json 文件。
当你把更新后的package.json 和 package-lock.json提交到代码仓库的时候, 需要通知你的团队成员, 保证其他的团队成员拉取代码之后，更新依赖可以有一个更友好的开发环境保障持续性的开发工作。

如果你的 package-lock.json 出现冲突或问题, 我的建议是将本地的 package-lock.json文件删掉, 然后去找远端没有冲突的 package.json 和 package-lock.json, 再去执行 npm install 命令。

好了，感谢小伙伴们的耐心观看，关于详解package.json和package-lock.json今天就讲到这里。

[外链图片转存失败,源站可能有防盗链机制,建议将图片保存下来直接上传(img-uyfjOQRH-1666599423328)(https://p9-juejin.byteimg.com/tos-cn-i-k3u1fbpfcp/bb40dc223b5e47bfbc1c2a4d4ef0500f~tplv-k3u1fbpfcp-watermark.image?)]

系列文章
详解package.json和package-lock.json

你可能没见过的npm script操作
————————————————
版权声明：本文为CSDN博主「苏小苏95」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
原文链接：https://blog.csdn.net/weixin_38664300/article/details/127495039