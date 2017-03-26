# `npm` vs `yarn` script execution order

Based on discussion in https://github.com/yarnpkg/yarn/issues/2853

```shell
yarn --version
> 0.21.3
npm --version
> 4.3.0
```

_**tl;dr** `npm` scripts !== `yarn` scripts
(they sometimes execute different sets of scripts depending on if it's a dep of a dep,
or if it's a local file, etc)_

## Local modules

### From root of project (no deps)

<details><summary>Click to expand</summary><p>

See [`package.json@v2.0.0`](https://github.com/jesstelford/npm-scripts-test/blob/v2.0.0/package.json)

#### `npm`

```shell
git clone --branch v2.0.0 https://github.com/jesstelford/npm-scripts-test.git
cd npm-scripts-test
npm install
```

#### `yarn`

```shell
git clone --branch v2.0.0 https://github.com/jesstelford/npm-scripts-test.git
cd npm-scripts-test
yarn
```

#### Output

```shell
cat node_modules/npm-scripts-test/scripts
```

##### `npm`

```plain
npm-scripts-test.preinstall
npm-scripts-test.install
npm-scripts-test.postinstall
npm-scripts-test.prepublish
npm-scripts-test.prepare
```

##### `yarn`

```plain
npm-scripts-test.preinstall
npm-scripts-test.install
npm-scripts-test.postinstall
npm-scripts-test.prepublish
```

##### The diff

```diff
--- a/yarn-scripts
+++ b/npm-scripts
@@ -2,4 +2,3 @@ npm-scripts-test.preinstall
 npm-scripts-test.install
 npm-scripts-test.postinstall
 npm-scripts-test.prepublish
+npm-scripts-test.prepare
```

</p></details>

### Single level dep

<details><summary>Click to expand</summary><p>

See [`package.json@v2.0.0`](https://github.com/jesstelford/npm-scripts-test/blob/v2.0.0/package.json)

#### `npm`

```shell
git clone --branch v2.0.0 https://github.com/jesstelford/npm-scripts-test.git
mkdir test-single-level-npm
cd test-single-level-npm
npm init -y .
npm install --save ../npm-scripts-test
```

#### `yarn`

```shell
git clone --branch v2.0.0 https://github.com/jesstelford/npm-scripts-test.git
mkdir test-single-level-yarn
cd test-single-level-yarn
yarn init -y .
yarn add file:../npm-scripts-test
```

#### Output

```shell
cat node_modules/npm-scripts-test/scripts
```

##### `npm`

```plain
npm-scripts-test.preinstall
npm-scripts-test.install
npm-scripts-test.postinstall
```

##### `yarn`

```plain
npm-scripts-test.preinstall
npm-scripts-test.install
npm-scripts-test.postinstall
npm-scripts-test.prepublish
npm-scripts-test.prepublish
npm-scripts-test.prepare
npm-scripts-test.preinstall
npm-scripts-test.install
npm-scripts-test.postinstall
```

##### The diff

```diff
--- a/npm-scripts
+++ b/yarn-scripts
@@ -1,9 +1,3 @@
 npm-scripts-test.preinstall
 npm-scripts-test.install
 npm-scripts-test.postinstall
-npm-scripts-test.prepublish
-npm-scripts-test.prepublish
-npm-scripts-test.prepare
-npm-scripts-test.preinstall
-npm-scripts-test.install
-npm-scripts-test.postinstall
```

</p></details>

### Dep local, but dep of a dep from registry

<details><summary>Click to expand</summary><p>

See [`npm-scripts-test-parent/package.json@v1.0.0`](https://github.com/jesstelford/npm-scripts-test-parent/blob/v1.0.0/package.json)

#### `npm`

```shell
git clone --branch v1.0.0 https://github.com/jesstelford/npm-scripts-test-parent.git
mkdir test-dep-of-dep-npm
cd test-dep-of-dep-npm
npm init -y .
npm install --save ../npm-scripts-test-parent
```

#### `yarn`

```shell
git clone --branch v1.0.0 https://github.com/jesstelford/npm-scripts-test-parent.git
mkdir test-dep-of-dep-yarn
cd test-dep-of-dep-yarn
yarn init -y .
yarn add file:../npm-scripts-test-parent
```

#### Output

_(output is the same for both `npm` & `yarn`)_

```shell
cat node_modules/npm-scripts-test/scripts
```

```plain
npm-scripts-test.preinstall
npm-scripts-test.install
npm-scripts-test.postinstall
npm-scripts-test.prepublish
npm-scripts-test.prepare
npm-scripts-test.prepublish
npm-scripts-test.prepare
npm-scripts-test.preinstall
npm-scripts-test.install
npm-scripts-test.postinstall
```

</p></details>

### Non-linear Dep local, but dep of a dep from registry

<details><summary>Click to expand</summary><p>

See [`npm-scripts-test-parent/package.json@v1.0.0`](https://github.com/jesstelford/npm-scripts-test-parent/blob/v1.0.0/package.json)

#### `npm`

```shell
git clone --branch v1.0.0 https://github.com/jesstelford/npm-scripts-test-parent.git
mkdir test-dep-of-dep-npm
cd test-dep-of-dep-npm
npm init -y .
npm install --save ../npm-scripts-test-parent
npm install --save ../npm-scripts-test
rm -rf node_modules
npm install
```

#### `yarn`

```shell
git clone --branch v1.0.0 https://github.com/jesstelford/npm-scripts-test-parent.git
mkdir test-dep-of-dep-yarn
cd test-dep-of-dep-yarn
yarn init -y .
yarn add file:../npm-scripts-test-parent
yarn add file:../npm-scripts-test
rm -rf node_modules yarn.lock
yarn
```

#### Output

```shell
cat node_modules/npm-scripts-test/scripts
```

##### `npm`

```plain
npm-scripts-test.preinstall
npm-scripts-test.install
npm-scripts-test.postinstall
```

##### `yarn`

```plain
npm-scripts-test.preinstall
npm-scripts-test.install
npm-scripts-test.postinstall
npm-scripts-test.prepublish
npm-scripts-test.prepublish
npm-scripts-test.prepare
npm-scripts-test.preinstall
npm-scripts-test.install
npm-scripts-test.postinstall
```

##### The diff

```diff
--- a/npm-scripts
+++ b/yarn-scripts
@@ -1,9 +1,3 @@
 npm-scripts-test.preinstall
 npm-scripts-test.install
 npm-scripts-test.postinstall
-npm-scripts-test.prepublish
-npm-scripts-test.prepublish
-npm-scripts-test.prepare
-npm-scripts-test.preinstall
-npm-scripts-test.install
-npm-scripts-test.postinstall
```

</p></details>

## From registry

### As a dependency

<details><summary>Click to expand</summary><p>

See [`package.json@v2.0.0`](https://github.com/jesstelford/npm-scripts-test/blob/v2.0.0/package.json)

#### `npm`

```shell
npm init -y .
npm install --save npm-scripts-test@2.0.0
```

#### `yarn`

```shell
yarn init -y .
yarn add npm-scripts-test@2.0.0
```

#### Output

_(output is the same for both `npm` & `yarn`)_

```shell
cat node_modules/npm-scripts-test/scripts
```

```plain
npm-scripts-test.preinstall
npm-scripts-test.install
npm-scripts-test.postinstall
npm-scripts-test.prepublish
npm-scripts-test.prepare
npm-scripts-test.prepublish
npm-scripts-test.prepare
npm-scripts-test.preinstall
npm-scripts-test.install
npm-scripts-test.postinstall
```

</p></details>

### As a dependency of a dependency

<details><summary>Click to expand</summary><p>

See [`npm-scripts-test-parent/package.json@v1.0.0`](https://github.com/jesstelford/npm-scripts-test-parent/blob/v1.0.0/package.json)

#### `npm`

```shell
npm init -y .
npm install --save npm-scripts-test-parent@1.0.0
```

#### `yarn`

```shell
yarn init -y .
yarn add npm-scripts-test-parent@1.0.0
```

#### Output

_(output is the same for both `npm` & `yarn`)_

```shell
cat node_modules/npm-scripts-test/scripts
```

```plain
npm-scripts-test.preinstall
npm-scripts-test.install
npm-scripts-test.postinstall
npm-scripts-test.prepublish
npm-scripts-test.prepare
npm-scripts-test.prepublish
npm-scripts-test.prepare
npm-scripts-test.preinstall
npm-scripts-test.install
npm-scripts-test.postinstall
```

</p></details>

### Non-linear dep of dep

<details><summary>Click to expand</summary><p>

See [`npm-scripts-test-parent/package.json@v1.0.0`](https://github.com/jesstelford/npm-scripts-test-parent/blob/v1.0.0/package.json)

#### `npm`

```shell
npm init -y .
npm install --save npm-scripts-test-parent@1.0.0
npm install --save npm-scripts-test@2.0.0
rm -rf node_modules
npm install
```

#### `yarn`

```shell
yarn init -y .
yarn add npm-scripts-test-parent@1.0.0
yarn add npm-scripts-test@2.0.0
rm -rf node_modules yarn.lock
yarn
```

#### Output

_(output is the same for both `npm` & `yarn`)_

```shell
cat node_modules/npm-scripts-test/scripts
```

```plain
npm-scripts-test.preinstall
npm-scripts-test.install
npm-scripts-test.postinstall
npm-scripts-test.prepublish
npm-scripts-test.prepare
npm-scripts-test.prepublish
npm-scripts-test.prepare
npm-scripts-test.preinstall
npm-scripts-test.install
npm-scripts-test.postinstall
```

</p></details>
