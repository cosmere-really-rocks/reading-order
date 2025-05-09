<template>
<div v-show="!hideCompletely" class="legend">
  <div
    :class="['legend__intro', {'legend__intro--open': introToggled}]"
    v-closable="{ handler: handleIntroOutsideClick }"
  >
    <div class="legend__intro-toggle" @click="introToggled = !introToggled">
      关于
    </div>

    <div
      :class="[
        'legend__intro-content',
        { 'legend__intro-content-collapsed': introContentCollapsed }
      ]"
    >
      <h1>三界宙漫游指南</h1>
      <!--eslint-disable-next-line max-len-->
      <p>欢迎来到美国幻想作家布兰登·桑德森创造的宇宙！作品太多不知道该如何入坑？不要惊慌，本指南为您提供如下服务：1）三界宙架构一览；2）阅读顺序推荐；3）穿越及彩蛋查询；4）未出版作品查询。</p>
      <span
        class="legend__intro-content-toggle"
        @click="introMoreInfoToggleHandler"
        v-html="introContentCollapsed ? '展开' : '收起'"
      ></span>
      <div class="legend__intro-content-more">
        <p>1）作品默认按系列、世界和星系分组，并按顺时针方向排列。</p>
        <!--eslint-disable-next-line max-len-->
        <p>2）三界宙系列没有绝对正确的阅读顺序。本指南致力于平衡出版顺序，同时将系列作品分组，可灵活进行调整。推荐先阅读任一阶段的重要作品，再进入下一阶段。属于同一系列的作品应当以本指南所列的顺序阅读。次要作品可在所属阶段阅读，也可先跳过，日后再补。</p>
        <!--eslint-disable-next-line max-len-->
        <p>3）箭头表示作品间的关联，箭头所指的作品含有触发箭头的作品的内容。箭头的式样表示该穿越的重要程度，也可作为阅读顺序的参考。单击作品以查看信息，出现相关的箭头也会同时出现。单击箭头可以查看详情。</p>
        <p>4）未出版作品最终未必会得到出版。</p>
        <div class="legend-feedback">
          <h2>反馈</h2>
          <ul class="feedback">
            <li><a href="mailto:joshua@17thshard.com">Email</a></li>
            <li><a href="https://github.com/17thshard/reading-order/issues/new">Github</a></li>
            <li><a href="https://www.17thshard.com/forum/profile/18320-jofwu/">17th Shard</a></li>
            <li><a href="https://www.reddit.com/message/compose/?to=jofwu">Reddit</a></li>
          </ul>
        </div>
      </div>
    </div>

    <div class="legend__intro-close" @click="introToggled = false">
      x
    </div>
  </div>
  <div
    :class="['legend__keys', {'legend__keys--open': keysToggled}]"
    v-closable="{ handler: handleKeysOutsideClick }"
  >
    <div class="legend__keys-toggle" @click="legendKeysToggleHandler">
      图例
    </div>

    <div class="legend__keys-content">
      <div class="legend-options">
        <h2>选项</h2>
        <span class="legend-options-item">
          <label for="sort">排序</label>
          <select id="sort" @change="changeSort">
            <option :value="null" :selected="selectedOrder === null">系列</option>
            <option
              :value="sort.id" :selected="selectedOrder === sort"
              :key="sort.id"
              v-for="sort in sortedBooks"
            >
              {{sort.description}}
            </option>
          </select>
        </span>
        <span class="legend-options-item">
          <input id="show-connection-explanations" type="checkbox"
                 :checked="showSpoilers"
                 @input="$store.commit('toggleExplanations', $event.target.checked)">
          <label for="show-connection-explanations" title="Explain connection & appearance details">
            显示剧透
          </label>
        </span>
        <span class="legend-options-item">
          <input id="highlight-series" type="checkbox"
                 :checked="highlightSeries"
                 @input="$store.commit('toggleSeriesHighlight', $event.target.checked)">
          <label for="highlight-series" title="Activate arches around diagram">
            点亮系列和星球
          </label>
        </span>
      </div>

      <div class="legend-categories">
        <h2>类别</h2>
        <Layer
          :layer="layer"
          :key="layer.name"
          @update-category-route="updateRoute('categories', flatCategories, $event)"
          v-for="layer in layers"
        ></Layer>
      </div>
      <div class="legend-connections">
        <h2>关联</h2>
        <ConnectionPreview
          :type="type" :key="type.id"
          @update-route="updateRoute('connections', connectionTypes, $event)"
          v-for="type in connectionTypes"
        >
        </ConnectionPreview>
      </div>
      <div class="legend-appearances">
        <h2>出场</h2>
        <AppearancePreview
          :appearance="appearance" :key="appearance.id"
          @update-route="updateRoute('appearances', appearances, $event)"
          v-for="appearance in appearances"
        >
        </AppearancePreview>
      </div>
    </div>
  </div>
</div>
</template>

<script>
import { mapState } from 'vuex';
import ConnectionPreview from '@/components/ConnectionPreview.vue';
import Layer from '@/components/Layer.vue';
import AppearancePreview from '@/components/AppearancePreview.vue';

export default {
  name: 'Legend',
  components: { AppearancePreview, Layer, ConnectionPreview },
  props: {
    connectionTypes: Array,
    layers: Array,
    appearances: Array,
    sortedBooks: Array,
  },
  data() {
    return {
      introToggled: !this.isInfoHiddenByDefault(),
      introContentCollapsed: !this.isIntroContentCollapsedByDefault(),
      keysToggled: !this.isLegendHiddenByDefault(),
      selectedOrder: null,
      sortInitialized: false,
    };
  },
  computed: {
    ...mapState(['showSpoilers', 'highlightSeries']),
    flatCategories() {
      return this.layers.reduce((acc, layer) => [...acc, ...layer.categories], []);
    },
    hideCompletely() {
      return this.$route.query['hide-ui'] === 'true';
    },
  },
  watch: {
    selectedOrder(newOrder) {
      if (newOrder === null) {
        this.$emit('sort', false);
      } else {
        this.$emit('sort', newOrder.books);
      }
    },
    sortedBooks(newBooks) {
      if (!this.sortInitialized) {
        this.selectedOrder = newBooks.find(s => s.id === this.$route.query.order) || null;
        this.sortInitialized = true;
      }
    },
    $route(to, from) {
      if (to.query.order !== from.query.order) {
        this.selectedOrder = this.sortedBooks.find(s => s.id === to.query.order) || null;
      }
    },
  },
  mounted() {
    document.addEventListener('keyup', this.toggleUi);
  },
  destroyed() {
    document.removeEventListener('keyup', this.toggleUi);
  },
  methods: {
    handleIntroOutsideClick() {
      if (this.introToggled && this.areClosablesHiddenByDefault()) {
        localStorage.setItem('introToggled', false);
        this.introToggled = false;
      }
    },
    handleKeysOutsideClick() {
      if (this.keysToggled && this.areClosablesHiddenByDefault()) {
        localStorage.setItem('keysToggled', false);
        this.keysToggled = false;
      }
    },
    isLegendHiddenByDefault() {
      let keysToggled = localStorage.getItem('keysToggled');
      if (keysToggled === undefined) {
        keysToggled = this.areClosablesHiddenByDefault();
      }
      return !(keysToggled === 'true');
    },
    isInfoHiddenByDefault() {
      let introToggled = localStorage.getItem('introToggled');
      if (introToggled === undefined) {
        introToggled = this.areClosablesHiddenByDefault();
      }
      return !(introToggled === 'true');
    },
    isIntroContentCollapsedByDefault() {
      const introContentCollapsed = localStorage.getItem('introContentCollapsed');
      return introContentCollapsed === undefined ? true : introContentCollapsed;
    },
    areClosablesHiddenByDefault() {
      return window.matchMedia('(max-width: 1000px)').matches;
    },
    introToggleHandler() {
      localStorage.setItem('introToggled', !this.introToggled);
      this.introToggled = !this.introToggled;
    },
    legendKeysToggleHandler() {
      localStorage.setItem('keysToggled', !this.keysToggled);
      this.keysToggled = !this.keysToggled;
    },
    introMoreInfoToggleHandler() {
      localStorage.setItem('introContentCollapsed', !this.introContentCollapsed);
      this.introContentCollapsed = !this.introContentCollapsed;
    },
    toggleUi(event) {
      if (event.code !== 'KeyH') {
        return;
      }

      if (this.hideCompletely) {
        this.$router.replace({ query: { ...this.$route.query, 'hide-ui': undefined } });
      } else {
        this.$router.replace({ query: { ...this.$route.query, 'hide-ui': 'true' } });
      }
    },
    changeSort(event) {
      if (event.target.value === '') {
        this.$router.replace({ query: { ...this.$route.query, order: undefined } });
      } else {
        this.$router.replace({ query: { ...this.$route.query, order: event.target.value } });
      }
    },
    updateRoute(category, elements, trigger) {
      const allActive = elements.every(e => e.active);
      const noneActive = !elements.some(e => e.active);
      const oldQuery = this.$route.query;

      if (allActive || noneActive) {
        Object.keys(oldQuery).forEach((k) => {
          if (k.startsWith(`${category}.`)) {
            delete oldQuery[k];
          }
        });
        this.$router.replace({ query: { ...oldQuery, [category]: allActive ? 'all' : 'none' } });
        return;
      }

      this.$router.replace({
        query: {
          ...oldQuery,
          [category]: undefined,
          [`${category}.${trigger.id}`]: trigger.active.toString(),
        },
      });
    },
  },
};
</script>

<style lang="scss">
.legend {
  font-family: sans-serif;
  z-index: 10;

  h1 {
    margin: 0 0 0.5rem;
  }

  h2 {
    margin: 0.5rem 0;
    font-size: 1.2rem;
  }

  &__intro, &__keys {
    max-height: 100%;
    background: rgba(0, 0, 0, 0.5);
    display: flex;
    flex-direction: column;
    transition: transform 0.2s ease-in-out;

    &--open {
      transform: translateX(0) !important;
    }

    &-content {
      overflow-y: auto;
      padding: 1rem;
    }

    &-toggle, &-close {
      background: rgba(0, 0, 0, 0.5);
      position: absolute;
      padding: 0.5rem;
      z-index: 11;
      cursor: pointer;
    }

    &-close {
      display: none;
    }
  }

  &__intro {
    position: fixed;
    left: 0;
    top: 0;
    width: 450px;
    box-sizing: border-box;
    z-index: 12;
    transform: translateX(-100%);

    &-toggle {
      left: 100%;
    }

    &-content-toggle {
      color: #b4b4b4;
      text-decoration: none;
      border-bottom: 1px dotted #b4b4b4;
      cursor: pointer;

      &:hover, &:focus, &:active {
        color: #919191;
        border: none;
      }
    }

    &-content-collapsed .legend__intro-content-more {
      display: none;
    }

    &-close {
      right: 0;
      padding-top: 0.25rem;
      padding-right: 1rem;
      font-size: 1.5rem;
    }
  }

  &__intro p {
    padding: 0.5rem 0;
    margin: 0;

    &:first-of-type {
      padding-top: 0;
    }

    &:last-child {
      padding-bottom: 0;
    }
  }

  &__keys {
    position: fixed;
    box-sizing: border-box;
    right: 0;
    top: 0;
    z-index: 10;
    transform: translateX(100%);

    &-toggle {
      right: 100%;
    }
  }

  &-options, &-connections, &-categories {
    display: flex;
    flex-direction: column;
    align-items: flex-start;
  }

  &-categories, &-connections, &-appearances, &-feedback {
    padding-left: 0.5rem;

    h2 {
      margin-left: -0.5rem;
    }

    ul {
      margin-block: 0;
      padding-inline-start: 27px;
    }
  }

  &-categories {
    align-items: stretch;
  }

  &-options-item {
    display: flex;
    margin-bottom: 0.25rem;

    &:last-child {
      margin-bottom: 0;
    }

    label {
      margin: 0.2rem;
    }

    select {
      margin-left: 0.125rem;
    }

    input[type="checkbox"] {
      margin: 0.1rem 0.2rem;

      & + label {
        margin: 0;
      }
    }
  }

  @media (max-width: 1000px) {
    &__intro {
      height: 100%;

      &-content {
        &-toggle {
          display: none;
        }

        &-collapsed .legend__intro-content-more {
          display: block;
        }
      }
    }
  }

  @media (max-width: 640px) {
    &__intro {
      width: 100%;

      &-close {
        display: block;
      }

      &--open + .legend__keys .legend__keys-toggle {
        display: none;
      }
    }
  }
}
</style>
