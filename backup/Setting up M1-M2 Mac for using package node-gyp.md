# Setting up M1/M2 Mac for using package node-gyp

If you're using an M1 Mac and trying to use Node.js for native module development, you may run into errors with node-gyp. Node-gyp is a tool that compiles native modules for Node.js, and it needs to be configured correctly to work on an M1 Mac. In this article, we'll explore how to set up an M1 Mac to use node-gyp without errors.

##  [https://dev.to/steckdev/setting-up-m1m2-mac-for-using-package-node-gyp-1d9e#step-1-install-command-line-tools](https://dev.to/steckdev/setting-up-m1m2-mac-for-using-package-node-gyp-1d9e#step-1-install-command-line-tools)Step 1: Install Command Line Tools

After installing Xcode, you need to install the Command Line Tools. Open a terminal window and enter the following command:  

```Plain Text
xcode-select --install

```
 
This command installs the Command Line Tools, which include the necessary tools for compiling software on a Mac.

##  [https://dev.to/steckdev/setting-up-m1m2-mac-for-using-package-node-gyp-1d9e#step-2-install-homebrew](https://dev.to/steckdev/setting-up-m1m2-mac-for-using-package-node-gyp-1d9e#step-2-install-homebrew)Step 2: Install Homebrew

Homebrew is a package manager for macOS that makes it easy to install and manage software packages. You can install Homebrew by entering the following command in a terminal window:  

```Plain Text
/bin/bash -c "$(curl -fsSL https://raw.githubusercontent.com/Homebrew/install/HEAD/install.sh)"

```
 
This command installs Homebrew and sets it up on your system.

##  [https://dev.to/steckdev/setting-up-m1m2-mac-for-using-package-node-gyp-1d9e#step-3-install-nodejs](https://dev.to/steckdev/setting-up-m1m2-mac-for-using-package-node-gyp-1d9e#step-3-install-nodejs)Step 3: Install Node.js

After installing Homebrew, you can use it to install Node.js. Enter the following command in a terminal window:  

```Plain Text
brew install node

```
 
This command installs the latest version of Node.js and its dependencies.

##  [https://dev.to/steckdev/setting-up-m1m2-mac-for-using-package-node-gyp-1d9e#step-4-configure-nodegyp](https://dev.to/steckdev/setting-up-m1m2-mac-for-using-package-node-gyp-1d9e#step-4-configure-nodegyp)Step 4: Configure node-gyp

Now that Node.js is installed, you need to configure node-gyp to work on an M1 Mac. Open a terminal window and enter the following command:  

```Plain Text
export CXXFLAGS="-stdlib=libc++"

```
 
This command sets the C++ standard library to libc++, which is required for node-gyp to work on an M1 Mac.

##  [https://dev.to/steckdev/setting-up-m1m2-mac-for-using-package-node-gyp-1d9e#step-5-test-nodegyp](https://dev.to/steckdev/setting-up-m1m2-mac-for-using-package-node-gyp-1d9e#step-5-test-nodegyp)Step 5: Test node-gyp

To test that node-gyp is configured correctly, you can install a native module that depends on it. Open a terminal window and enter the following command:  

```Plain Text
npm install -g node-gyp

```
 
This command installs the node-gyp package globally.

To test that node-gyp is working correctly, you can create a new Node.js project and install a native module that depends on it. For example, you can create a new project with the following command:bash  

```Plain Text
mkdir test-project &amp;&amp; cd test-project
npm init -y

```
 
Then, you can install a native module that depends on node-gyp with the following command:  

```Plain Text
npm install sharp

```
 
The sharp module is an image processing library that uses node-gyp to compile native code.

If node-gyp is configured correctly, you should be able to install the sharp module without errors.

##  [https://dev.to/steckdev/setting-up-m1m2-mac-for-using-package-node-gyp-1d9e#conclusion](https://dev.to/steckdev/setting-up-m1m2-mac-for-using-package-node-gyp-1d9e#conclusion)Conclusion

Setting up node-gyp on an M1 Mac can be challenging, but it's necessary for native module development in Node.js. By following these steps, you can configure node-gyp to work correctly on your M1 Mac and avoid errors when compiling native modules.



降低python版本<br>`brew install python@3.10`<br>`export PYTHON=$(brew --prefix python@3.10)/bin/python3.10`





 Node v14 doesn't have ARM64 support, which is why `nvm`  failed to find a supported binary, and attempted to build source code (which failed as well)

 [https://github.com/nodejs/node/issues/52306#issuecomment-2031073649](https://github.com/nodejs/node/issues/52306#issuecomment-2031073649)
