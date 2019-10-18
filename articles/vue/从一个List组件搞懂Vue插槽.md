# 需求
先假设我们需要实现如下效果，并将其封装成一个组件，当然目的并不是要真的封装组件，而是在这个过程中学习 **插槽** 的使用以及优化的思路哈：

![alt](/assets/img/list_01.png)

一看到这个效果，有的小伙伴可能就要说了：这还不简单，反手就是一段代码甩上来

```html
<template>
  <div>
    <div>{{ title }}</div>
    <hr>
    <div>
      <div v-for="(item, i) in List" :key="i">{{ item }}</div>
    </div>
  </div>
</template>
```

然后在父组件中这样调用就 `ok` 了:

```html
<template>
  <custom-list :title="title" :List="books" />
</template>

<script>
import CustomList from './custom-list.js';

export default {
  components: { CustomList },
  data() {
    return {
      title: '书籍列表',
      books: ['Dom编程艺术', '你不知道的Javascript', 'CSS世界', 'HTML入门教程'],
    };
  },
};
</script>
```

Emmm。。乍一看似乎没有任何毛病，甚至数据变化了也是可以满足要求的。

然而，这样就够了吗？需求总是善变的，比如说，有一天又要满足如下效果呢？

![alt](/assets/img/list_02.png)

此时，这个组件就有点不够用了，无法应对各种需求的组件不是好组件。既然内容是多变的，那就不能固定,而应该是灵活的，最灵活的方式应该是将决定权交给使用者（父组件），使用者是最清楚内容应该是什么的，那具体怎么做呢？
# 插槽
这个时候就到我们的 **插槽** 登场了。插槽允许我们将子组件中的内容分发到父组件，由父组件决定子组件要渲染的内容。
## 普通插槽
还是以上述组件为例，我们需要将子组件中 `title`的内容分发出去，由父组件将内容传进来。此时我们可以这样修改子组件：

```html
<template>
  <div>
    <div>
      <slot></slot>
    </div>
    <hr>
    <div>
      <div v-for="(item, i) in List" :key="i">{{ item }}</div>
    </div>
  </div>
</template>
```

然后在父组件中调用：

```html
<template>
  <custom-list :List="books">
    <span><icon type="waves"> 书籍列表</span>
  </custom-list>
</template>

<script>
import CustomList from './custom-list.js';

export default {
  components: { CustomList },
  data() {
    return {
      title: '书籍列表',
      books: ['Dom编程艺术', '你不知道的Javascript', 'CSS世界', 'HTML入门教程'],
    };
  },
};
</script>
```

这样的话，父组件中包裹的内容就会替代 `<slot>` 标签，所以最终的渲染结果就是这样的：

```html
<template>
  <div>
    <div>
      <span><icon type="waves"> 书籍列表</span>
    </div>
    <hr>
    <div>
      <div v-for="(item, i) in List" :key="i">{{ item }}</div>
    </div>
  </div>
</template>
```

顺便提一嘴：在 `<slot>` 标签中也是可以放置内容的，如：`<slot> 标题 </slot>`, 这表示插槽的默认内容，只有当父组件没有通过插槽传递内容的时候，才会显示该默认内容。

同理我们的 `content` 的内容也是需要分发出去的，于是我们再修改：

```html
<template>
  <div>
    <div>
      <slot></slot>
    </div>
    <hr>
    <slot></slot>
  </div>
</template>
```

这样一来又有问题了：我们定义了两个 <slot> 怎么去区分它们的内容呢，总不能靠书写顺序吧？这显然是不靠谱的。于是，就有了 **具名插槽**。

## 具名插槽
顾名思义就是带名字的插槽，便于多个插槽之间的区分。然后我们继续修改：

```html
<template>
  <div>
    <div>
      <slot name="title"></slot>
    </div>
    <hr>
    <slot name="content"></slot>
  </div>
</template>
```

给每个插槽定义一个 `name` 属性，然后我们在父组件传值的时候带上对应插槽的 `name` 就行了。

```html
<template>
  <custom-list>
    <template v-slot:title>
      <span><icon type="waves"> 书籍列表</span>
    <template>
    <template v-slot:content>
      <div>
        <div v-for="(item, i) in book" :key="i">
          <icon :type="item.icon">{{ item.name }}
        </div>
      </div>
    </template>
  </custom-list>
</template>

<script>
import CustomList from './custom-list.js';

export default {
  components: { CustomList },
  data() {
    return {
      title: '书籍列表',
      books: [
        { name: 'Dom编程艺术',icon: 'face' },
        { name: '你不知道的Javascript', icon: 'favorite' },
        { name: 'CSS世界', icon: 'av_timer'},
        { name: 'HTML入门教程', icon: 'star_half' },
      ],
    };
  },
};
</script>
```

> 在向具名插槽提供内容的时候，我们可以在一个 `<template>` 元素上使用 `v-slot` 指令，并以 `v-slot` 的参数的形式提供其名称。

这下可好了，连数据都不用传了，`List` 在子组件根本用不到，数据直接在父组件中就渲染了，而且每次使用这个组件的时候都要写一遍 `v-for`，那为什么不把 `v-for` 写在子组件中呢？这样就只需要写一次了。
那我们再来修改子组件：

```html
<template>
  <div>
    <div>
      <slot name="title"></slot>
    </div>
    <hr>
    <div>
      <div v-for="(item, i) in List">
        <slot name="item"></slot>
      </div>
    </div>
  </div>
</template>
```

这样又有问题了：我要显示 `item` 但是 `item` 是子组件中的数据，当父组件通过 `v-slot` 传递内容进来的时候，`<slot>{{ item }}</slot>` 的内容就会被传进来的内容替换了。这咋搞呢？思考一下，如果我们将数据给父组件，在父组件中显示不就解决了么？那么这个时候就要轮到 `作用域插槽` 登场了。
## 作用域插槽
作用域插槽允许我们将子组件的数据传给父组件，这不正好解决了上面的问题么？下面我们再调整子组件

```html
<template>
  <div>
    <div>
      <slot name="title"></slot>
    </div>
    <hr>
    <div>
      <div v-for="(item, i) in List">
        <slot name="item" :row="item"></slot>
      </div>
    </div>
  </div>
</template>
```

此时我们每个将 `item` 的值通过 `row` 属性传给了 `<slot>`, 然后我们在父组件中取出这个值：

```html
<template>
  <custom-list>
    <template v-slot:title>
      <span><icon type="waves"> 书籍列表</span>
    <template>
    <!--实际传的数据格式为： { row：{ name: xx, icon: xx } },所以这里可以使用结构赋值-->
    <template v-slot:item="{ row }">
      <icon :type="row.icon">{{ row.name }}
    </template>
  </custom-list>
</template>
```

这样我们的组件就完成了，不仅灵活性更高，代码健壮性也更好。
总得来说，插槽还是很好用的，特别是在封装组件的时候。
