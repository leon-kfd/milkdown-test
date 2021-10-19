<template>
  <div 
    v-if="updateFlag" 
    ref="editorRef" 
    class="milkdown-wrapper" 
    :style="{
      width: '100%',
      height: '100%',
      fontSize: fontSize + 'px'
    }"
  />
</template>

<script setup lang="ts">
import { nextTick, onMounted, ref } from "vue";
import { Editor, rootCtx, defaultValueCtx, themeFactory } from "@milkdown/core";
import { font, size, color, mixin, override, view } from "@milkdown/theme-nord";
import { commonmark } from "@milkdown/preset-commonmark";
import { listener, listenerCtx } from '@milkdown/plugin-listener';
// import { tooltip } from '@milkdown/plugin-tooltip'
// import { slash } from '@milkdown/plugin-slash';
// import { gfm } from '@milkdown/preset-gfm';
import { injectGlobal } from '@emotion/css';
import '../assets/milk-icon/index.css'

const iconMapping = {
    h1: 'h1',
    h2: 'h2',
    h3: 'h3',
    loading: 'loading',
    quote: 'list',
    code: 'code',
    table: 'table',
    divider: 'line',
    image: 'image',
    brokenImage: 'image',
    bulletList: 'unorder-list',
    orderedList: 'order-list',
    taskList: 'checklist',
    bold: 'bold',
    italic: 'italic',
    inlineCode: 'code',
    strikeThrough: 'text-delete',
    link: 'link',
    leftArrow: 'arrow-left',
    rightArrow: 'arrow-right',
    upArrow: 'arrow-up',
    downArrow: 'arrow-down',
    alignLeft: 'align-left',
    alignRight: 'align-right',
    alignCenter: 'align-center',
    delete: 'delete',
    select: 'select-all',
    unchecked: 'unchecked',
    checked: 'checked',
};

const props = defineProps({
  enableTooltip: {
    type: Boolean,
    default: false
  },
  enableSlash: {
    type: Boolean,
    default: false
  },
  enableGfm: {
    type: Boolean,
    default: false
  },
  enablePrism: {
    type: Boolean,
    default: false
  },
  markdown: {
    type: String,
    default: ''
  },
  textColor: {
    type: String,
    default: '#494e59'
  },
  fontSize: {
    type: Number,
    default: 16
  }
})

const emit = defineEmits(['change'])

const nord = themeFactory({
  font, 
  size, 
  color, 
  mixin,
  slots: () => ({
    icon: (id) => {
      const span = document.createElement('span');
      span.className = `icon ed-icon icon-${iconMapping[id]}`;
      return span;
    },
  }),
  global: (themeTool) => {
    const css = injectGlobal;
    css`
        ${view};
        ${override(themeTool)}
    `;
  }
})

const editorRef = ref()
const updateFlag = ref(true)

onMounted(() => {
  setEditorCtx()
})

const setEditorCtx = async () => {
  const e = Editor.make().config((ctx) => {
    ctx.set(rootCtx, editorRef.value);
    ctx.set(defaultValueCtx, props.markdown);
    ctx.set(listenerCtx, {
      markdown: [
        (getMarkdown) => {
          const markdown = getMarkdown()
          emit('change', markdown)
        }
      ]
    })
  }).use(nord).use(commonmark).use(listener)
  if (props.enableTooltip) {
    const { tooltip } = await import('@milkdown/plugin-tooltip')
    e.use(tooltip)
  }
  if (props.enableSlash) {
    const { slash } = await import('@milkdown/plugin-slash')
    e.use(slash)
  }
  if (props.enableGfm) {
    const { gfm } = await import('@milkdown/preset-gfm')
    e.use(gfm)
  }
  if (props.enablePrism) {
    const { prism } = await import('@milkdown/plugin-prism')
    getStyleLink()
    e.use(prism)
  }
  e.create()
}

const update = async () => {
  updateFlag.value = false
  await nextTick()
  updateFlag.value = true
  await nextTick()
  setEditorCtx()
}

const getStyleLink = () => {
  const target = document.querySelector('#prism-theme');
  if (target) {
      return target;
  }
  const link = document.createElement('link');
  link.id = 'prism-theme';
  link.setAttribute('rel', 'stylesheet');
  link.setAttribute('href', 'https://unpkg.com/prism-themes/themes/prism-material-light.css');
  document.head.appendChild(link);
  return link;
};

defineExpose({
  update
})
</script>
<style lang="scss">
.milkdown-wrapper {
  .milkdown {
    max-width: inherit;
    padding: .8em 1.2em;
    overflow-y: auto;
    background: transparent;
    color: v-bind(textColor);
    box-shadow: none;
    .h1, .h2, .h3 {
      margin: 1.5em 0 !important;
    }
    .h1 {
      font-size: 3em;
      line-height: 3em;
    }
    .h2 {
      font-size: 2.125em;
      line-height: 2.125em;
    }
    .h3 {
      font-size: 1.5em;
      line-height: 1.5em;
    }
    .paragraph {
      font-size: 1em;
      line-height: 1.5em;
    }
    .code-fence {
      font-size: 0.85em;
      .code-fence_value,
      .code-fence_select {
        width: 10.25em;
      }
    }

    .editor > * {
      margin: 1em 0;
    }

    // table
    .tableWrapper table * {
      font-size: 1em;
    }
    .tableWrapper table th, 
    .tableWrapper table td {
      height: 3em;
      padding: 1em;
    }
  }
}
</style>
