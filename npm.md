### package-lock.json的表现
----
#### npm install:
1. 当不存在package-lock.json文件时，会根据package.json文件生成新的package-lock.json文件
2. 当存在package-lock.json文件时，但其依赖版本与package.json文件不兼容的情况下，会根据package.json下载依赖，并且更新package-lock.json文件
3. 当存在package-lock.json文件时，且依赖班恩与package.json文件兼容的情况下，会直接根据package-lock.json文件来下载依赖
#### npm ci：
**应用场景**：主要用于自动化构建的时候安装依赖，由于npm install并不能保证线上构建时的依赖版本与本地开发时的一致，npm ci主要用来解决这个问题
1. 使用npm ci安装依赖时，要求项目必须存在package-lock.json文件或npm-shrinkwrap.json，否则无法执行
2. package-lock.json文件或npm-shrinkwrap.json与package.json中依赖不一致时，npm ci会报错并退出，而不是更新package-lock.json文件，所以当报错时，开发者需要去本地执行npm install重新生成package-lock.json文件，并把该文件上传到git更新，再进行自动化构建。

基于以上两种特性，npm ci能够有效防止线上构建的依赖与开发者本地不一致的情况。
npm ci不能用来单独安装单个依赖，只能用于安装整个项目的依赖，如果存在node_modules文件，则先删除再进行安装操作。

#### 优化：
问题：由于npm ci每次都需要删除node_modules开头的目录来清除所有的本地宝，重新安装所有依赖，这也是导致每次构建时间都较长的原因。
**npm ci --cache .npm**
启用缓存，默认目录下，这些文件夹都不会被缓存。可以通过修改npm缓存目录和位置，设置本地缓存目录，--cache .npm。
**npm ci --cache .npm --prefer-offline**
通过配置选项--prefer-offline，告诉 NPM 忽略缓存最短时间并立即使用本地缓存的包，而不是根据注册表验证它们。
**npm ci --cache .npm --prefer-offline --only=production**
#### 总结：
通过配置选项--only=production，可以让npm忽略开发依赖的选项。
>npm ci 命令会根据 lock 文件（比如 package-lock.json）去下载node_modules。它比npm install命令快2至10倍，因为npm ci安装包之前，会删除掉node_modules文件夹，因此他不需要去校验已下载文件版本与控制版本的关系，也不用校验是否存在最新版本的库，所以下载的速度更快。