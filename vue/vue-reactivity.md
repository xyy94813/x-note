# VUE 响应式系统

> 在 JavaScript 中有两种劫持 property 访问的方式：getter / setters 和 Proxies。
> Vue 2 使用 getter / setters 完全是出于支持旧版本浏览器的限制。
> 而在 Vue 3 中则使用了 Proxy 来创建响应式对象，仅将 getter / setter 用于 ref

```js
function reactive(obj) {
  return new Proxy(obj, {
    get(target, key) {
      track(target, key);
      return target[key];
    },
    set(target, key, value) {
      target[key] = value;
      trigger(target, key);
    },
  });
}

function ref(value) {
  const refObject = {
    get value() {
      track(refObject, 'value');
      return value;
    },
    set value(newValue) {
      value = newValue;
      trigger(refObject, 'value');
    },
  };
  return refObject;
}

function track(target, key) {
  if (activeEffect) {
    const effects = getSubscribersForProperty(target, key);
    effects.add(activeEffect);
  }
}

function trigger(target, key) {
  const effects = getSubscribersForProperty(target, key);
  effects.forEach((effect) => effect());
}

function whenDepsChange(update) {
  const effect = () => {
    activeEffect = effect;
    update();
    activeEffect = null;
  };
  effect();
}
```

> Vue 的响应式系统基本是基于运行时的。
> 追踪和触发都是在浏览器中运行时进行的。

### v-model

与 Angular 类似。但是实际是基于响应式数据

Vue v3.4 后的使用方式推荐使用 defineModel。
defineModel 由编译器编译后，实际上是类似于 React `onXXX=()=>updateValue()` 的机制

```
<script setup>
const props = defineProps(['modelValue'])
const emit = defineEmits(['update:modelValue'])
</script>

<template>
  <input
    :value="props.modelValue"
    @input="emit('update:modelValue', $event.target.value)"
  />
</template>
```
