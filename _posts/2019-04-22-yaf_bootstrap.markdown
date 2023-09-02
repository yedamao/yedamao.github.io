---
layout: post
title: "yaf框架的Bootstrap机制"
---

# yaf框架的Bootstrap机制

Bootstrap是用来在Application运行(run)之前做一些初始化工作的机制.

## 1. Yaf_Bootstrap_Abstract

```c
zend_class_entry *yaf_bootstrap_ce;

zend_function_entry yaf_bootstrap_methods[] = {
    {NULL, NULL, NULL}
};

YAF_STARTUP_FUNCTION(bootstrap) {
    zend_class_entry ce;

    YAF_INIT_CLASS_ENTRY(ce, "Yaf_Bootstrap_Abstract",  "Yaf\\Bootstrap_Abstract", yaf_bootstrap_methods);
    yaf_bootstrap_ce = zend_register_internal_class_ex(&ce, NULL);
    yaf_bootstrap_ce->ce_flags |= ZEND_ACC_EXPLICIT_ABSTRACT_CLASS;

    return SUCCESS;
}
```

Yaf_Bootstrap_Abstract在扩展代码中被定义为一个没有任何属性和方法的空抽象类。

## 2. Yaf_Application::bootstrap
Yaf_Application实例化之后,可以通过调用自身的boostrap方法，实现对用户自定义初始化代码的执行。下面看一下扩展内部bootstrap方法的逻辑.


### 导入(load)Boostrap类
```c
    if (!(ce = zend_hash_str_find_ptr(EG(class_table),
                    YAF_DEFAULT_BOOTSTRAP_LOWER, sizeof(YAF_DEFAULT_BOOTSTRAP_LOWER) - 1))) {
        if (YAF_G(bootstrap)) {
            bootstrap_path = zend_string_copy(YAF_G(bootstrap));
        } else {
            bootstrap_path = strpprintf(0, "%s%c%s.%s",
                    ZSTR_VAL(YAF_G(directory)), DEFAULT_SLASH, YAF_DEFAULT_BOOTSTRAP, ZSTR_VAL(YAF_G(ext)));
        }
        if (!yaf_loader_import(bootstrap_path, 0)) {
            php_error_docref(NULL, E_WARNING, "Couldn't find bootstrap file %s", ZSTR_VAL(bootstrap_path));
            retval = 0;
        } else if (UNEXPECTED((ce = zend_hash_str_find_ptr(EG(class_table),
                        YAF_DEFAULT_BOOTSTRAP_LOWER, sizeof(YAF_DEFAULT_BOOTSTRAP_LOWER) - 1)) == NULL)) {
            php_error_docref(NULL, E_WARNING, "Couldn't find class %s in %s", YAF_DEFAULT_BOOTSTRAP, ZSTR_VAL(bootstrap_path));
            retval = 0;
        } else if (UNEXPECTED(!instanceof_function(ce, yaf_bootstrap_ce))) {
            php_error_docref(NULL, E_WARNING,
                    "Expect a %s instance, %s give", ZSTR_VAL(yaf_bootstrap_ce->name), ZSTR_VAL(ce->name));
            retval = 0;
        }
        zend_string_release(bootstrap_path);
    }
```
首先，在class_table中查找用户自定义的Bootstrap类(注册类名不分大小写). 如果查找失败执行引入流程，从bootstrap_path导入Bootstrap类，检查是否是Yaf_Bootstrap_Abstract抽象类.

### 执行所有_init开头方法

```c
        object_init_ex(&bootstrap, ce);
        dispatcher = zend_read_property(yaf_application_ce,
                self, ZEND_STRL(YAF_APPLICATION_PROPERTY_NAME_DISPATCHER), 1, NULL);

        ZEND_HASH_FOREACH_STR_KEY(&(ce->function_table), func) {
            /* cann't use ZEND_STRL in strncasecmp, it cause a compile failed in VS2009 */
            if (strncasecmp(ZSTR_VAL(func), YAF_BOOTSTRAP_INITFUNC_PREFIX, sizeof(YAF_BOOTSTRAP_INITFUNC_PREFIX)-1)) {
                continue;
            }
            zend_call_method(&bootstrap, ce, NULL, ZSTR_VAL(func), ZSTR_LEN(func), NULL, 1, dispatcher, NULL);
            /** an uncaught exception threw in function call */
            if (UNEXPECTED(EG(exception))) {
                zval_ptr_dtor(&bootstrap);
                RETURN_FALSE;
            }
        } ZEND_HASH_FOREACH_END();
        zval_ptr_dtor(&bootstrap);
```
实例化Boostrap. 获取dispatcher实例，需要作为_init方法的参数.遍历执行ce(Bootstrap的实例)的所有以_init为前缀方法,完成用户自定义初始逻辑执行.


## 3. 动动手
删除Yaf_Application::bootstrap导入逻辑中的Boostrap实例检查,Boostrap类就不需要从Yaf_Bootstrap_Abstract继承啦. 只要你的自定义类名为Boostrap其中方法以_init开头.
