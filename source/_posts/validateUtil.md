---
title: Validate & CommonUtils
date: 2018-01-09 15:06:20
tags:
- validate
categories:
- 工具
---

*常有校验参数的需求，却又没有合适的工具可以使用*
*因此仿造spring web的校验机制，结合hibernate的校验功能，实现了这样一个小工具*
*其实还有一些有趣的小工具，虽然可能有各种的瑕疵或者各种原因没能竣工，不过还是蛮有趣的*
*[欢迎去看看 to github line](https://github.com/ding-guohang/common_util)*
<!--more-->

# Code

```java
package com.ca.util.utils;

import com.google.common.base.Stopwatch;
import org.apache.commons.collections.CollectionUtils;
import org.apache.commons.lang3.ArrayUtils;
import org.hibernate.validator.constraints.NotBlank;
import org.hibernate.validator.internal.metadata.core.ConstraintHelper;
import org.slf4j.Logger;
import org.slf4j.LoggerFactory;

import javax.validation.Constraint;
import javax.validation.ConstraintValidator;
import javax.validation.ConstraintValidatorContext;
import java.lang.annotation.Annotation;
import java.lang.reflect.Field;
import java.util.concurrent.ConcurrentHashMap;
import java.util.concurrent.TimeUnit;

import static com.google.common.base.Preconditions.checkArgument;

/**
 * 参数校验工具
 * Spring的参数校验好像就是基于Hibernate的Validator实现类
 * 在寻找一个直接调用一下，就可以触发参数校验的方式……但是没找到
 * 自己写了一个依赖于Hibernate的，很麻烦的参数校验
 * <p>
 * updated at 17-01-13
 * 修改了一下ValidateUtil， 解决了用map的单例问题，自己实现了NotBlank的校验器，不然每次都要校验notnull和notBlank，太麻烦了
 *
 * @author guohang.ding on 16-11-8
 */
public class ValidateUtil {

    private static final ConstraintHelper helper = new ConstraintHelper();
    private static final Logger LOG = LoggerFactory.getLogger(ValidateUtil.class);
    private static final String PARAMS_INVALID = "Params_Invalid";
    public static final String SYSTEM_ERROR = "System_Error";

    private static final ConcurrentHashMap<Class<? extends ConstraintValidator<? extends Annotation, ?>>,
            ConstraintValidator> cache = new ConcurrentHashMap<>();

    static {
        NotBlankValidator validator = new NotBlankValidator();
        cache.put(org.hibernate.validator.internal.constraintvalidators.NotBlankValidator.class, validator);
    }


    @SuppressWarnings("unchecked")
    public static void checkRequest(Object request) {
        checkArgument(request != null, PARAMS_INVALID);
        Stopwatch watch = Stopwatch.createStarted();

        try {
            boolean valid = true;
            LOG.info("validate request: {}", request);

            Field[] fields = request.getClass().getDeclaredFields();
            if (ArrayUtils.isEmpty(fields)) {
                LOG.info("empty fields for request: {}", request.getClass());
                return;
            }

            Annotation[] annotations;
            for (Field field : fields) {
                annotations = field.getDeclaredAnnotations();
                if (ArrayUtils.isEmpty(annotations)) {
                    continue;
                }
                for (Annotation annotation : annotations) {
                    if (annotation.annotationType().getAnnotation(Constraint.class) == null) {
                        continue;
                    }
                    if (CollectionUtils.isEmpty(helper.getAllValidatorClasses(annotation.annotationType()))) {
                        continue;
                    }
                    for (Class<? extends ConstraintValidator<? extends Annotation, ?>> clazz
                            : helper.getAllValidatorClasses(annotation.annotationType())) {
                        try {
                            ConstraintValidator validator = getValidator(clazz);
                            validator.initialize(annotation);

                            field.setAccessible(true);
                            Object req = field.get(request);
                            valid = validator.isValid(req, null);
                            if (!valid) {
                                LOG.error("invalid params, req-{}, val-{}, annotation-{}", req, field.getName(), annotation);
                                break;
                            }
                        } catch (Exception e) {
                            LOG.error("validator validate error", e);
                            throw new IllegalArgumentException(SYSTEM_ERROR);
                        }
                    }
                    checkArgument(valid, PARAMS_INVALID);
                }
            }
        } finally {
            LOG.info("valid cost {}", watch.elapsed(TimeUnit.MILLISECONDS));
        }
    }

    private static ConstraintValidator getValidator(Class<? extends ConstraintValidator<? extends Annotation, ?>> clazz)
            throws IllegalAccessException, InstantiationException {
        ConstraintValidator validator = cache.get(clazz);
        if (validator == null) {
            LOG.info("init validator: {}", clazz);
            validator = clazz.newInstance();
            ConstraintValidator value = cache.putIfAbsent(clazz, validator);
            if (value != null) {
                validator = value;
            }
        }

        return validator;
    }

    private static class NotBlankValidator implements ConstraintValidator<NotBlank, CharSequence> {

        public void initialize(NotBlank annotation) {
        }

        public boolean isValid(CharSequence charSequence, ConstraintValidatorContext constraintValidatorContext) {
            return charSequence != null && charSequence.toString().trim().length() > 0;
        }
    }
}
```
