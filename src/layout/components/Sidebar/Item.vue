<script>
export default {
  //<Item/>组件 是用来显示对应的Icon图标和Title，通过 :Icon = item.meta.icon ;  :title = item.meta.title;
  /**
   * 在Item组件中可以看到 icon图标和Title 都是接受props传来的变量，展示对应的模块
      并且 这里使用到了render（）函数，创建 i 标签和 svg标签。
      所以在侧边栏中，是可以使用Icon图标和svg图标，然后就是创建了span标签展示对应的title
   * 
   */
  name: 'MenuItem',
  functional: true,
  props: {
    icon: {
      type: String,
      default: ''
    },
    title: {
      type: String,
      default: ''
    }
  },
  render(h, context) {
    const { icon, title } = context.props
    const vnodes = []

    if (icon) {
      if (icon.includes('el-icon')) {
        vnodes.push(<i class={[icon, 'sub-el-icon']} />)
      } else {
        vnodes.push(<svg-icon icon-class={icon} />)
      }
    }

    if (title) {
      vnodes.push(<span slot='title'>{(title)}</span>)
    }
    return vnodes
  }
}
</script>

<style scoped>
.sub-el-icon {
  color: currentColor;
  width: 1em;
  height: 1em;
}
</style>
