<script setup lang="ts">
import { computed, onMounted, ref } from 'vue'
import * as monaco from 'monaco-editor/esm/vs/editor/editor.api'
import 'monaco-editor/esm/vs/basic-languages/javascript/javascript.contribution'
import 'monaco-editor/esm/vs/basic-languages/yaml/yaml.contribution'

export interface Props {
  language?: string
  readOnly?: boolean
  value?: string
  options?: monaco.editor.IStandaloneEditorConstructionOptions
}

const props = withDefaults(defineProps<Props>(), {
  options: () => ({
    automaticLayout: true,
    bracketPairColorization: {
      enabled: true,
      independentColorPoolPerBracketType: true,
    },
    copyWithSyntaxHighlighting: true,
    cursorSmoothCaretAnimation: 'on',
    emptySelectionClipboard: false,
    find: {
      addExtraSpaceOnTop: true,
    },
    foldingImportsByDefault: false,
    matchOnWordStartOnly: true,
    mouseWheelZoom: true,
    smoothScrolling: true,
    tabSize: 2,
    wrappingIndent: 'indent',
  }),
})

const emit = defineEmits<{
  (e: 'onChangeText', newVal: string, event: monaco.editor.IModelContentChangedEvent): void
  (e: 'onFocusText'): void
  (e: 'onBlurText'): void
}>()

const options = computed(() => {
  const options = props.options ?? {}

  if (!isNil(props.language))
    options.language = props.language
  if (!isNil(props.readOnly))
    options.readOnly = props.readOnly
  if (!isNil(props.value))
    options.value = props.value

  return options
})

const manocaEditorRef = ref()
let ins: monaco.editor.IStandaloneCodeEditor
onMounted(() => {
  ins = monaco.editor.create(manocaEditorRef.value, options.value)

  ins.onDidChangeModelContent(useDebounceFn((e: monaco.editor.IModelContentChangedEvent) => {
    const newVal = ins.getValue()
    emit('onChangeText', newVal ?? '', e)
  }, 1000))
  ins.onDidFocusEditorText(() => {
    emit('onFocusText')
  })
  ins.onDidBlurEditorText(() => {
    emit('onBlurText')
  })
})

function setContent(val: string) {
  ins?.setValue(val)
}

function setTheme(theme: string) {
  monaco.editor.setTheme(theme)
}

defineExpose({
  monaco,
  setContent,
  setTheme,
})
</script>

<template>
  <div ref="manocaEditorRef" class="vue-monaco-editor" />
</template>

<style>
.vue-monaco-editor {
  width:100%;
  height:100%;
  border: 1px solid rgba(0, 0, 0, 0.1);
  box-shadow: 0 1px 2px 0 rgb(0 0 0 / 0.05);
}
</style>
