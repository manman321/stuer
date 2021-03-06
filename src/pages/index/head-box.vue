<template>
  <div class="header-main">
    <a href="/" class="stuer-logo" title="Stuer">STUER</a>
    <nav class="nav-main">
      <ul class="nav-list">
        <li class="main-nav-list">
          <ul class="link-list">
            <li class="link-item" v-for="(route, index) in routes" :key="index">
              <router-link
                :to="route.href"
                :class="activeIndex === index? 'active': ''"
                @click.native="activeIndex = index"
              >{{route.name}}</router-link>
            </li>
          </ul>
        </li>
        <li class="nav-item search">
          <el-input
            :placeholder="searchFocus ? '搜索帖子/用户': '搜索'"
            prefix-icon="el-icon-search"
            v-model="search"
            size="small"
            @focus="searchFocus = true"
            @blur="searchFocus = false"
            :class="{'search-input': true, 'search-input-focus': searchFocus}"
          ></el-input>
        </li>
        <li class="nav-item add">
          <el-button
            type="primary"
            size="small"
            icon="el-icon-edit-outline"
            @click="routeToEditor"
          >写帖子</el-button>
        </li>
        <li class="nav-item notification" v-if="$store.getters.isLogin">
          <router-link to="/notification" target="_blank">
            <i class="el-icon-message"></i>
            <span class="count">{{count}}</span>
          </router-link>
        </li>
        <li class="nav-item menu" v-if="$store.getters.isLogin">
          <div class="img-box" @click="showMenu=!showMenu">
            <img :src="avatar" class="avatar">
          </div>
          <ul v-if="showMenu" class="nav-menu" @click="showMenu=!showMenu">
            <div class="nav-menu-item-group">
              <li class="nav-menu-item">
                <router-link to="/editor">
                  <svg class="icon nav-menu-item-logo" aria-hidden="true">
                    <use xlink:href="#icon-edit"></use>
                  </svg>
                  <span>写帖子</span>
                </router-link>
              </li>
            </div>
            <div class="nav-menu-item-group">
              <li class="nav-menu-item">
                <router-link :to="`/profile/${$store.getters.user.id}`">
                  <svg class="icon nav-menu-item-logo" aria-hidden="true">
                    <use xlink:href="#icon-me"></use>
                  </svg>
                  <span>我的主页</span>
                </router-link>
              </li>
              <li class="nav-menu-item">
                <router-link :to="`/profile/${$store.getters.user.id}/likes`">
                  <svg class="icon nav-menu-item-logo" aria-hidden="true">
                    <use xlink:href="#icon-like"></use>
                  </svg>
                  <span>我的点赞</span>
                </router-link>
              </li>
              <li class="nav-menu-item">
                <router-link :to="`/profile/${$store.getters.user.id}/setting`">
                  <svg class="icon nav-menu-item-logo" aria-hidden="true">
                    <use xlink:href="#icon-setting1"></use>
                  </svg>
                  <span>个人设置</span>
                </router-link>
              </li>
            </div>
            <div class="nav-menu-item-group">
              <li class="nav-menu-item">
                <router-link to="/">
                  <svg class="icon nav-menu-item-logo" aria-hidden="true">
                    <use xlink:href="#icon-about"></use>
                  </svg>
                  <span>关于我们</span>
                </router-link>
              </li>
              <li class="nav-menu-item">
                <a @click="logout">
                  <svg class="icon nav-menu-item-logo" aria-hidden="true">
                    <use xlink:href="#icon-logout"></use>
                  </svg>
                  <span>登出</span>
                </a>
              </li>
            </div>
          </ul>
        </li>
        <li class="nav-item sso" v-else>
          <a :href="SsoHref()" class="loginSignup">
            <span>登陆/注册</span>
          </a>
        </li>
      </ul>
    </nav>
  </div>
</template>

<script>
import { mapGetters } from 'vuex';
import { deleteCookie } from '@/utils';
import { logout } from '@/api';
import defaultAvatar from '@/assets/default-avatar.png';
export default {
  data() {
    return {
      search: '',
      searchFocus: false,
      routes: [
        {
          name: '首页',
          href: '/'
        },
        {
          name: '讨论区',
          href: '/discuss'
        },
        {
          name: '树洞',
          href: '/tree-hole'
        },
        {
          name: '找对象',
          href: '/find-love'
        },
        {
          name: '求职区',
          href: '/job'
        },
        {
          name: 'OA',
          href: '/oa'
        }
      ],
      activeIndex: -1,
      count: 6,
      showMenu: false
    };
  },
  computed: {
    ...mapGetters(['user']),
    avatar() {
      if (this.user && this.user.avatar) {
        return this.user.avatar;
      }
      return defaultAvatar;
    }
  },
  watch: {
    $route(to, from) {
      // 导航栏高亮设置
      if (to.path === '/') {
        // 首页
        this.activeIndex = 0;
        return;
      }
      for (let index in this.routes) {
        // 导航栏其他页
        if (
          this.routes[index].href.includes(to.path) &&
          this.routes[index].href !== '/' &&
          to.path !== '/'
        ) {
          this.activeIndex = parseInt(index, 10);
          return;
        }
      }
      this.activeIndex = -1; // 非导航栏的其他页面，导航栏不高亮
    }
  },
  methods: {
    SsoHref() {
      return `/sso?redirect=${window.location.href}`;
    },
    async logout() {
      const result = await logout();
      if (result.code === 0) {
        this.$store.commit('setUser', null);
        this.$store.commit('loginStatus', false);
        localStorage.removeItem('isLogin');
        deleteCookie('auth_token', process.env.VUE_APP_DOMAIN);
        window.location.href = '/';
      } else {
        this.$message.error('退出登录失败');
      }
    },
    routeToEditor() {
      this.$router.push('/editor');
    }
  }
};
</script>

<style scoped lang="scss">
@import '~@/style/variables.scss';
.header-main {
  display: flex;
  justify-content: flex-start;
  align-items: center;
  max-width: 1200px;
  height: 5rem;
  margin: 0 auto;
  a {
    text-decoration: none;
  }
}
.stuer-logo {
  display: block;
  width: 12rem;
  height: 5rem;
  line-height: 5rem;
  font-size: 3.5rem;
  margin-right: 2rem;
  color: $primary-color;
}
.nav-main {
  height: 100%;
  flex: 1 0 auto;
}
.nav-list {
  display: flex;
  justify-content: flex-end;
  height: 5rem;
}
.nav-item {
  display: flex;
  justify-content: center;
  align-items: center;
  height: 5rem;
  font-size: 1.33rem;
  color: $link-color;
  padding: 0 1rem;
  &.search {
    flex: 1 1 auto;
    justify-content: flex-end;
    cursor: auto;
  }
}
.sso {
  .loginSignup {
    height: 2.67rem;
    line-height: 2.67rem;
    border-radius: 3px;
    padding: 0 0.5rem;
    font-size: 1.2rem;
    color: $primary-color;
    border: 1px solid $primary-color;
  }
}

.main-nav-list {
  display: flex;
}
.link-list {
  display: flex;
}
.link-item {
  display: flex;
  justify-content: center;
  align-items: center;
  height: 5rem;
  a {
    display: block;
    height: 5rem;
    line-height: 5rem;
    font-size: 1.33rem;
    font-weight: bold;
    color: $link-color;
    padding: 0 1rem;
    &:hover {
      color: $primary-color;
    }
    &.active {
      color: $primary-color;
    }
  }
}
.search-input {
  width: 8rem;
  &-focus {
    width: 16rem;
  }
}
.notification {
  position: relative;
  font-size: 2rem;
  a {
    color: $primary-color;
  }
  .count {
    position: absolute;
    left: 50%;
    bottom: 50%;
    padding: 0.3rem 0.5rem;
    font-size: 1rem;
    line-height: 1;
    text-align: center;
    color: #fff;
    background-color: $primary-color;
    border-radius: 3rem;
    border: 2px solid #fff;
    word-break: normal;
  }
}
.menu {
  position: relative;
}
.img-box {
  display: flex;
  justify-content: center;
  align-items: center;
  height: 3rem;
  width: 3rem;
  border-radius: 50%;
  overflow: hidden;
  background: #eee;
  cursor: pointer;
  .avatar {
    height: 3rem;
    width: 3rem;
  }
}
.nav-menu {
  position: absolute;
  width: 11rem;
  top: 100%;
  right: 0;
  transform: translateX(0);
  background-color: #fff;
  box-shadow: 0 1px 2px 0 rgba(0, 0, 0, 0.1);
  border: 1px solid rgba(177, 180, 185, 0.45);
  border-radius: 4px;
  &-item-group {
    border-bottom: 1px solid rgba(0, 0, 0, 0.04);
    padding: 1rem 0;
  }
  &-item {
    font-size: 1.3rem;
    cursor: pointer;
    &:hover {
      background-color: hsla(0, 0%, 94.9%, 0.5);
    }
    a {
      display: flex;
      align-items: center;
      padding: 0.5rem 1rem;
    }
    &-logo {
      margin-right: 0.8rem;
    }
  }
}
</style>
