<template>
  <div v-if="!item.hidden">
    <template
      v-if="hasOneShowingChild(item.children, item) && (!onlyOneChild.children || onlyOneChild.noShowingChildren) && !item.alwaysShow">
      <app-link v-if="onlyOneChild.meta" :to="resolvePath(onlyOneChild.path)">
        <el-menu-item :index="resolvePath(onlyOneChild.path)" :class="{ 'submenu-title-noDropdown': !isNest }">
          <item :icon="onlyOneChild.meta.icon || (item.meta && item.meta.icon)" :title="onlyOneChild.meta.title" />
        </el-menu-item>
      </app-link>
    </template>
    <!-- 
        v-if="hasOneShowingChild(item.children, item) && (!onlyOneChild.children || onlyOneChild.noShowingChildren) && !item.alwaysShow" 
        =======> 来判断是该路由是否是只有一层级菜单
        
        hasOneShowingChild：判断是否只有一个子路由需要被展示 

        !onlyOneChild.children||onlyOneChild.noShowingChildren：onlyOneChilde是否包含children

        !item.alwaysShow：有没有alwaysshow这个属性 如果填入了这个属性，哪怕是hidden 也会被展示  

        这些条件全部命中的话就会展示template而不是submenu
        实际展示的是最里面的这个item
        而这个item是applink: meta存在的时候进行展示
        item:如果子路由没有icon的话他会取父路由的 但是title就只会取当前路由的title

        当前路由Item的hidden属性为true时，整个菜单栏是隐藏不展示的，当
        hasOneShowingChild( )返回true时执行el-menu-item，返回为false时则执行el-submenu
        然后再判断hasOneShowingChild( )的返回值，一直遍历下去。
    -->
    <el-submenu v-else ref="subMenu" :index="resolvePath(item.path)" popper-append-to-body>
      <template slot="title">
        <item v-if="item.meta" :icon="item.meta && item.meta.icon" :title="item.meta.title" />
      </template>
      <sidebar-item v-for="child in item.children" :key="child.path" :is-nest="true" :item="child"
        :base-path="resolvePath(child.path)" class="nest-menu" />
    </el-submenu>
    <!-- 
      如果该路由是复合菜单，则执行<el-submenu><el-submenu/>，
      在el-submenu里面将会再次执行sidebar-item组件，遍历直到路由只有一层菜单 
        
    -->
  </div>
</template>

<script>
import path from 'path'
import { isExternal } from '@/utils/validate'
import Item from './Item'
import AppLink from './Link'
import FixiOSBug from './FixiOSBug'

export default {
  name: 'SidebarItem',
  components: { Item, AppLink },
  mixins: [FixiOSBug],
  props: {
    // route object
    item: {
      type: Object,
      required: true
    },
    isNest: {
      type: Boolean,
      default: false
    },
    basePath: {
      type: String,
      default: ''
    }
  },
  data() {
    // To fix https://github.com/PanJiaChen/vue-admin-template/issues/237
    // TODO: refactor with render function
    this.onlyOneChild = null
    return {}
  },
  methods: {
    /**
     * hasOneShowingChild（）这个函数

        1.先定义数组 showingChildren 通过遍历item.children满足每一项hidden属性为true的项

        不加入showingChildren中。

        2. 完成遍历后判断showingChildren数组的长度，若长度为1则 hasOneShowingChild（）函数返回true, 执行el-menu-item; 若长度为0则返回true并且this.onlyOneChild 就等于父亲parent（当前的Item）就展示父亲菜单项; 当长度为其他时，都返回false，将执行el-submenu 一直遍历下去，直到当前项只有一个或没有children。

        3. this.onlyOneChild = item 只有hasOneShowingChild()函数成立，等式才有意义。

        4.在<app-lin><app-link/>组件中isExternal（）是校验函数，当校验地址path为‘http//...’返回true，则动态切换组件为a链接；否则返回的是<router-link>，并执行 :to传过来的path
     */
    // 参数对应(item.children,item)
    hasOneShowingChild(children = [], parent) {
      const showingChildren = children.filter(item => {
        /**
         * 1、对item.children进行遍历，判断每一项的hidden属性，若为true则返回false不显示菜单栏，
         * 否则返回true，就添加到定义的showingChildren数组中。
         *  */
        if (item.hidden) {
          return false
        } else {
          // Temp set(will be used if only has one showing child)
          this.onlyOneChild = item
          return true
        }
      })

      // When there is only one child router, the child router is displayed by default
      /**
       * 2、遍历完成后判断数组showingChildren的长度如果等于1，则返回true也就是this.onlyOneChild = item里面正好只存了一个item，也就是唯一的child，执行el-menu
       */
      if (showingChildren.length === 1) {
        return true
      }

      // Show parent if there are no child router to display
      /**
       * 3、遍历完后判断数组showingChildren的长度如果等于0，就是所有的item.children子路由的hidden属性都是true
       * 那么hasOneShowingChildren()函数返回true，并且this.onlyOneChildren就等于参数里的父亲parent，就展示item项
       */
      if (showingChildren.length === 0) {
        /** parent展开 全部赋值给onlyOneChild path置空, noShowingChildren: 不展示children 只展示parent */
        this.onlyOneChild = { ...parent, path: '', noShowingChildren: true }
        return true
      }
      // 4、除了showingChildren数组的长度等于0或1外，hasOneShowingChildren()返回的都是false，执行el-submenu
      return false
    },
    resolvePath(routePath) {
      if (isExternal(routePath)) {
        return routePath
      }
      if (isExternal(this.basePath)) {
        return this.basePath
      }
      return path.resolve(this.basePath, routePath)
    }
  }
}
</script>
