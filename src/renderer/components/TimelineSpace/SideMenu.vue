<template>
  <div id="side_menu">
    <div :class="collapse ? 'profile-wrapper narrow-menu':'profile-wrapper'" style="-webkit-app-region: drag;">
      <div class="profile-narrow" v-if="collapse">
        <img :src="account.avatar" class="avatar" />
      </div>
      <div class="profile-wide" v-else>
        <div>@{{ account.username }}
          <el-dropdown trigger="click" @command="handleProfile" :title="$t('side_menu.profile')">
            <span class="el-dropdown-link">
              <i class="el-icon-arrow-down el-icon--right"></i>
            </span>
            <el-dropdown-menu slot="dropdown">
              <el-dropdown-item command="show">{{ $t("side_menu.show_profile") }}</el-dropdown-item>
              <el-dropdown-item command="edit">{{ $t("side_menu.edit_profile") }}</el-dropdown-item>
            </el-dropdown-menu>
          </el-dropdown>
        </div>
        <span>{{ account.domain }}</span>
      </div>
      <div class="collapse">
        <el-button type="text" class="release-collapse" @click="releaseCollapse" v-if="collapse" :title="$t('side_menu.expand')">
          <i class="el-icon-arrow-right"></i>
        </el-button>
        <el-button type="text" class="do-collapse" @click="doCollapse" v-else :title="$t('side_menu.collapse')">
          <i class="el-icon-arrow-left"></i>
        </el-button>
      </div>
    </div>
    <el-menu
      :default-active="activeRoute()"
      :background-color="themeColor"
      text-color="#909399"
      :collapse="collapse"
      active-text-color="#ffffff"
      :router="true"
      :class="collapse ? 'el-menu-vertical timeline-menu narrow-menu':'el-menu-vertical timeline-menu'">
      <el-menu-item :index="`/${id()}/home`">
        <icon name="home"></icon>
        <span>{{ $t("side_menu.home") }}</span>
        <el-badge is-dot :hidden="!unreadHomeTimeline">
        </el-badge>
      </el-menu-item>
      <el-menu-item :index="`/${id()}/notifications`">
        <icon name="bell"></icon>
        <span>{{ $t("side_menu.notification") }}</span>
        <el-badge is-dot :hidden="!unreadNotifications">
        </el-badge>
      </el-menu-item>
      <el-menu-item :index="`/${id()}/favourites`">
        <icon name="star"></icon>
        <span>{{ $t("side_menu.favourite") }}</span>
      </el-menu-item>
      <el-menu-item :index="`/${id()}/local`">
        <icon name="users"></icon>
        <span>{{ $t("side_menu.local") }}</span>
        <el-badge is-dot :hidden="!unreadLocalTimeline">
        </el-badge>
      </el-menu-item>
      <el-menu-item :index="`/${id()}/public`">
        <icon name="globe"></icon>
        <span>{{ $t("side_menu.public") }}</span>
      </el-menu-item>
      <el-menu-item :index="`/${id()}/hashtag`">
        <icon name="hashtag"></icon>
        <span>{{ $t("side_menu.hashtag") }}</span>
      </el-menu-item>
      <el-menu-item :index="`/${id()}/search`">
        <icon name="search"></icon>
        <span>{{ $t("side_menu.search") }}</span>
      </el-menu-item>
      <el-menu-item :index="`/${id()}/lists`">
        <icon name="list-ul"></icon>
        <span>{{ $t("side_menu.lists") }}</span>
      </el-menu-item>
      <template v-for="list in lists">
        <el-menu-item :index="`/${id()}/lists/${list.id}`" class="sub-menu" v-bind:key="list.id">
          <span>#{{ list.title }}</span>
        </el-menu-item>
      </template>
    </el-menu>
  </div>
</template>

<script>
import { mapState } from 'vuex'
import { shell } from 'electron'

export default {
  name: 'side-menu',
  computed: {
    ...mapState({
      account: state => state.TimelineSpace.account,
      unreadHomeTimeline: state => state.TimelineSpace.SideMenu.unreadHomeTimeline,
      unreadNotifications: state => state.TimelineSpace.SideMenu.unreadNotifications,
      unreadLocalTimeline: state => state.TimelineSpace.SideMenu.unreadLocalTimeline,
      lists: state => state.TimelineSpace.SideMenu.lists,
      themeColor: state => state.App.theme.side_menu_color,
      overrideActivePath: state => state.TimelineSpace.SideMenu.overrideActivePath,
      collapse: state => state.TimelineSpace.SideMenu.collapse
    })
  },
  created () {
    this.$store.dispatch('TimelineSpace/SideMenu/readCollapse')
  },
  methods: {
    activeRoute () {
      if (this.overrideActivePath === null) {
        return this.$route.path
      } else {
        return this.overrideActivePath
      }
    },
    id () {
      return this.$route.params.id
    },
    handleProfile (command) {
      switch (command) {
        case 'show':
          const accountURL = `${this.account.baseURL}/@${this.account.username}`
          this.$store.dispatch('TimelineSpace/Contents/SideBar/AccountProfile/searchAccount', accountURL)
            .then((account) => {
              this.$store.dispatch('TimelineSpace/Contents/SideBar/AccountProfile/changeAccount', account)
              this.$store.commit('TimelineSpace/Contents/SideBar/changeOpenSideBar', true)
            })

          this.$store.dispatch('TimelineSpace/Contents/SideBar/openAccountComponent')
          break
        case 'edit':
          shell.openExternal(this.account.baseURL + '/settings/profile')
          break
      }
    },
    doCollapse () {
      this.$store.dispatch('TimelineSpace/SideMenu/changeCollapse', true)
    },
    releaseCollapse () {
      this.$store.dispatch('TimelineSpace/SideMenu/changeCollapse', false)
    }
  }
}
</script>

<style lang="scss" scoped>
#side_menu {
  .profile-wrapper {
    background-color: var(--theme-side-menu-color);
    position: fixed;
    top: 0;
    left: 65px;
    width: 180px;
    height: 70px;
    font-size: 16px;
    display: flex;

    .profile-wide {
      color: #ffffff;
      font-weight: bold;
      padding: 20px 8px 10px 20px;
      box-sizing: border-box;
      overflow: hidden;
      text-overflow: ellipsis;

      .el-dropdown-link {
        cursor: pointer;
      }
    }

    .profile-narrow {
      float: left;

      .avatar {
        width: 36px;
        height: 36px;
        border-radius: 50%;
        margin: 18px 12px;
      }
    }

    .collapse {
      margin-top: 24px;

      .do-collapse {
        color: #dcdfe6;
        padding: 0;

        &:hover {
          color: #409eff;
        }
      }

      .release-collapse {
        color: #dcdfe6;
        padding: 0;

        &:hover {
          color: #409eff;
        }
      }
    }
  }

  .timeline-menu /deep/ {
    position: fixed;
    top: 70px;
    left: 65px;
    height: calc(100% - 70px);
    width: 180px;
    border: none;
    overflow-y: auto;

    .el-badge__content {
      background-color: #409eff;
      border: none;
      margin-left: 4px;
    }

    .menu-item-title {
      color: rgb(144, 147, 153);
      cursor: default;
    }

    .menu-item-title:hover {
      background-color: inherit;
    }

    .sub-menu {
      padding-left: 45px !important;
      height: 32px;
      line-height: 32px;
      font-size: 14px;
    }
  }

  .narrow-menu {
    width: 76px;
  }
}
</style>
