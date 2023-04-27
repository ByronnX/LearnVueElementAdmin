<template>
  <component :is="type" v-bind="linkProps(to)">
    <slot />
  </component>
</template>

<script>
// <app-lin><app-link/>是通过动态组件component:is=‘type’ 来动态显示相应内容,判断type来生成a或者router-link
import { isExternal } from '@/utils/validate'

export default {
  props: {
    to: {
      type: String,
      required: true
    }
  },
  computed: {
    isExternal() {
      //isExternal作用：判断是不是http请求
      return isExternal(this.to)
    },
    type() {
      // 如果是链接的话，就会变成a标签
      // 如果不是链接的话，就会变成router-link
      if (this.isExternal) {
        return 'a'
      }
      return 'router-link'
    }
  },
  methods: {
    linkProps(to) {// 判断路由是不是external
      if (this.isExternal) {
        return {
          // 如果是的话就把url直接传进来（外链）
          // 就是说如果把/book/create写成http形式的时候，就会变成一个a标签
          // 并且打开一个链接
          href: to,
          target: '_blank',
          rel: 'noopener'
        }
      }
      return {
        // 如果不是链接的话，就会变成router-link
        to: to
      }
    }
  }
}
</script>
