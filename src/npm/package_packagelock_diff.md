## package.json 和 package-lock.json 的区别
既然在管理第三方库中，我们已经拥有了 `package.json` 那么为什么还需要 `packge-lock.json` 呢？

首先在package.json中包含了我们在开发web或node应用时所需要的 `dependencies 与 devDependencies`, 在dependencies这个KV对象中，通常又包含这样的版本号：`"classnames": "^2.3.2"` 前面会有一个 `^` 这个字符号表示了较小的版本和补丁是可以的接受的，以 `"classnames": "^2.3.2"` 举例，npm将安装大于或等于 `2.3.2` 但小于 `3.0.0` 的最新版本，例如 `2.4.0/2.3.3` 都是可能被接受的。这种表示法是 `semantic versioning`， 在这里就不赘述了。而通过对 `^` 符号的解释，我们可以发现这个字符代表了一个范围而已，所以如果只有 `package.json` 可能会造成新的安装结果不一致。而 `package-lock.json` 可以帮助我们锁住第三方库的版本，在多个研发同学在开发时，第三方库保证是统一的。其实在安装依赖时，可能会影响安装结果的因素有： npm版本，依赖最新版本，操作系统等其他因素。所以维护好一个 `package-lock` 的干净与整洁还是有必要的，可以帮助我们在多人开发一个项目过程中，依赖与依赖的依赖的版本都是一致的。

