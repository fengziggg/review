### cocos2dx自行导出为py，可以在py的环境下进行cocos开发
1. 自行编译python解释器
   - 到时候是引擎+py两个运行环境，而且还要导出引擎的东西到py，自行编译可以将导出内容内置到自定义py
2. CAPI或者其他形式，导出py模块
   - 目的是为了将cocos2dx的内容导出为py节点
     
   - 两种方式✅：一个是直接将封装导出代码内置到python的解释器，CAPI的方式是直接在在config.c的InitTable里面注册对应的方法
     - ![image](https://github.com/user-attachments/assets/e327294d-61b3-48c6-97ae-5f9eddd7ff1b)

   - 🔻另一个是pyd思路：直接将CAPI文件用工具打包成pyd，这里是用的setuptool.py
     - 自己编译的python一开始setuptool库，然后因为没有配置好ensuerpip会导致没有pip
     - python -m ensurepip --upgrade --default-pip安装pip
     - setuptool使用简单，但是有几个要注意的，不然会编译失败：
     - ![image](https://github.com/user-attachments/assets/5cec4c2e-cd91-4be7-bc5c-8b9816770277)
     - pyd放到解释器目录下就可以直接import，但这种需要编译出很多pyd(按目标是要吧整个2dx编译为pyd，相当与饶了一圈那还不如直接编译在解释器内置里面)
