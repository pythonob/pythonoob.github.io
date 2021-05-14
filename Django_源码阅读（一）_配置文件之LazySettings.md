本打算从model读起，结果发现需要先了解其它类，看着看着看到了设置模块，依赖的内容比较少，也有值得学习记录的地方，就从这里开始吧。


### LazySettings
LazySettings 类继承自LazyObject类，是一个懒加载的类。可以通过LazySettings来懒加载(即需要时再获取)配置的属性。

-  _setUp()方法

  ```
  用于获取项目中settings.py中的配置，并将配置放在_wrapped属性中
  具体是从环境变量中获取 DJANGO_SETTINGS_MODULE 的值(一般为settings文件)，然后从这个模块中获取配置，赋值到_wrapped属性
  ```
  
- 魔术方法
```
__repr__（）， __getattr__() ,  __setattr__() , __delattr__()
  这几个方法都是python中的魔术方法。之所以要写这篇文档主要也是为了记录和复习一下魔术方法相关的内容。
```

    - __repr__() 方法类似__str__(), 返回一个字符串，用于表述类的大致信息。
  
    - __getattr__()  先判断_wrapped是否为None, 是则调用_setup()方法，将settings中的内容赋值到_wrapped属性。
    这个方法除self参数外，接受一个name参数，表示要获取的属性的属性名。
    从_wrapped属性中获取到要获取的属性内容后，将其缓存到类的__dict__属性中。
  
    - __setattr__() 此方法除self外，还接收name 和value 两个属性。
    如果name等于_wrapped, 则清空self.__dict__,不等于_wrapped, 则从self.__dict__去掉要设置的属性。
    最后调用父类即LazyObject类的__setattr__属性，设置要设置的属性
  
  - configure方法
 
    接收一个default_settings参数和options key word 参数。可以通过configure函数手动指定配置文件。 
    但如果_wrapped非空，或者说DJANGO_SETTINGS_MODULE已经指定了配置文件的位置，再通过configure函数被
    来设置settings, 则会产生一个runtimeerror().
    
  - configured 方法
 
     属性方法， 通过判断_wrapped是否为空来判断setings是否已经设置
     
     
### Settings()
 - 初始化时接收一个模块作为配置文件存放的位置。 django中有一个global_settings模块，作为全局默认配置。
 - settings类初始化时，先从global_settings获取默认属性，然后再从settings实例化时传入的模块获取配置，对默认配置进行增加或修改。
 - 最终关于设置会有两个属性，一个self.settings,保存了全部的设置，另一个 self._explicit_settings, 保存了自定义的设置的名称
 - 过程中会检查INSTALLED_APPDS, TMPLATE_DIRS, LOCALE_PATHS三个配置，必须为list或tuple类型。设置中必须包含sefl.SECRET_KEY。
 - 如果设置了TIME_ZONE, 会根据 /usr/share/zoneinfo 进行TIME_ZONE设置的检查，如果不存在/usr/share/zoneinfo则不会对该属性进行检查


### UserSettingsHolder
    用户自定义设置类
    这个类中用到的几个魔术类，前面基本已经提到了。值得注意的是is_overriden函数中
    ```def is_overridden(self, setting):
           deleted = (setting in self._deleted)
           set_locally = (setting in self.__dict__)
           set_on_default = getattr(self.default_settings, 'is_overridden', lambda s: False)(setting)
           return (deleted or set_locally or set_on_default)
   ```
   set_on_default这一行
   根据is_overriden这个类在test/utils.py中的调用情况来看，传入参数default_settings为LazySettings的_wrapped属性，也就是一个Settings()类
   这里调用了Settings类的is_overridden方法。
   个人感觉不太容易看懂，不过值得借鉴。
  ```
   
### 总结
  1. django中默认有一个global_settings文件，保存了项目中的默认配置
  2. django通过环境变量DJANGO_SETTINGS_MODULE指定用于获取用户自定义配置的模块
  3. settings初始化时，会先设置global_settings, 然后根据用户自定义的设置，对settings进行增加和更新。过程中会对其中的几个配置进行检查，如INSTALLED_APP, SECRET等
  4. 懒加载的方式值得考虑
  



  
