<template>
  <div :class="getClass">
    <PageHeader :ghost="ghost" v-bind="$attrs" ref="headerRef">
      <template #default>
        <template v-if="content">
          {{ content }}
        </template>
        <slot name="headerContent" v-else />
      </template>
      <template #[item]="data" v-for="item in getHeaderSlots">
        <slot :name="item" v-bind="data" />
      </template>
    </PageHeader>
    <div :class="[`${prefixCls}-content`, $attrs.contentClass]" :style="getContentStyle">
      <slot />
    </div>
    <PageFooter v-if="getShowFooter" ref="footerRef">
      <template #left>
        <slot name="leftFooter" />
      </template>
      <template #right>
        <slot name="rightFooter" />
      </template>
    </PageFooter>
  </div>
</template>
<script lang="ts">
  import type { CSSProperties, PropType } from 'vue';

  import { defineComponent, computed, watch, nextTick, ref, unref } from 'vue';
  import PageFooter from './PageFooter.vue';
  import { usePageContext } from '/@/hooks/component/usePageContext';

  import { useDesign } from '/@/hooks/web/useDesign';
  import { propTypes } from '/@/utils/propTypes';
  import { omit } from 'lodash-es';
  import { PageHeader } from 'ant-design-vue';
  export default defineComponent({
    name: 'PageWrapper',
    components: { PageFooter, PageHeader },
    props: {
      dense: propTypes.bool,
      ghost: propTypes.bool,
      content: propTypes.string,
      contentStyle: {
        type: Object as PropType<CSSProperties>,
      },
      contentBackground: propTypes.bool,
      contentFullHeight: propTypes.bool,
    },
    setup(props, { slots }) {
      const headerRef = ref<ComponentRef>(null);
      const footerRef = ref<ComponentRef>(null);
      const footerHeight = ref(0);
      const { prefixCls } = useDesign('page-wrapper');
      const { contentHeight, setPageHeight, pageHeight } = usePageContext();

      const getClass = computed(() => {
        return [
          prefixCls,
          {
            [`${prefixCls}--dense`]: props.dense,
          },
        ];
      });

      const getShowFooter = computed(() => slots?.leftFooter || slots?.rightFooter);

      const getHeaderSlots = computed(() => {
        return Object.keys(omit(slots, 'default', 'leftFooter', 'rightFooter', 'headerContent'));
      });

      const getContentStyle = computed(
        (): CSSProperties => {
          const { contentBackground, contentFullHeight, contentStyle } = props;
          const bg = contentBackground ? { backgroundColor: '#fff' } : {};
          if (!contentFullHeight) {
            return { ...bg, ...contentStyle };
          }
          return {
            ...bg,
            ...contentStyle,
            minHeight: `${unref(pageHeight)}px`,
            paddingBottom: `${unref(footerHeight)}px`,
          };
        }
      );

      watch(
        () => [contentHeight?.value, getShowFooter.value],
        () => {
          if (!props.contentFullHeight) {
            return;
          }
          nextTick(() => {
            const footer = unref(footerRef);
            const header = unref(headerRef);
            footerHeight.value = 0;
            const footerEl = footer?.$el;

            if (footerEl) {
              footerHeight.value += footerEl?.offsetHeight ?? 0;
            }
            let headerHeight = 0;
            const headerEl = header?.$el;
            if (headerEl) {
              headerHeight += headerEl?.offsetHeight ?? 0;
            }

            setPageHeight?.(unref(contentHeight) - unref(footerHeight) - headerHeight);
          });
        },
        {
          immediate: true,
        }
      );

      return {
        getContentStyle,
        footerRef,
        headerRef,
        getClass,
        getHeaderSlots,
        prefixCls,
        getShowFooter,
        pageHeight,
        omit,
      };
    },
  });
</script>
<style lang="less">
  @prefix-cls: ~'@{namespace}-page-wrapper';

  .@{prefix-cls} {
    position: relative;

    .ant-page-header {
      // padding: 12px 16px;

      &:empty {
        padding: 0;
      }
    }

    &-content {
      // padding: 12px;
      margin: 16px;
    }

    &--dense {
      .@{prefix-cls}-content {
        margin: 0;
      }
    }
  }
</style>
