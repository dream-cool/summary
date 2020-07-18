C2系统管理组件

注入UserService的bean实例时由于存在三个模块的UserService的bean且这三个bean依次是集成关系，所以在一开始注入bean时选择了父类的bean实例来Autowired，根据spring扫描bean机制，先寻找该类名首字母小写的bean实例，发现找到了3个，然后再去根据类型判断，由于注入的是父类，所以通过instanceof判断类型时判断其子类时依然会将其认为时需要注入的bean实例。所以，由于这三个类是继承关系且同名，所以不能通Autowired来注入父类，否则将找到3个bean出现异常，只能通过Autowired注入超类。

```java
com.chinacreator.asp.comp.sys.core.user.service.UserService
					      ↑
com.chinacreator.asp.comp.sys.basic.user.service.UserService
						  ↑
com.chinacreator.asp.comp.sys.advanced.user.serviceUserService
```