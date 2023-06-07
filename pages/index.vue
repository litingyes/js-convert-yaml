<script setup lang="ts">
import { parse, stringify } from 'yaml'

const JsEditorRef = ref()
const YamlEditorRef = ref()

const focusPos = ref<'js' | 'yaml'>()

function handleJsFocus() {
  focusPos.value = 'js'
}
function handleYamlFocus() {
  focusPos.value = 'yaml'
}

function handleJsChange(val: string) {
  if (focusPos.value !== 'js')
    return

  let newVal: any

  try {
    newVal = eval(`(${val})`)
  }
  catch (e) {
    newVal = val
  }
  const yamlVal = stringify(newVal)
  YamlEditorRef.value?.setContent(yamlVal)
}
function handleYamlChange(val: string) {
  if (focusPos.value !== 'yaml')
    return
  const newVal = JSON.stringify(parse(val), null, 2)
  JsEditorRef.value?.setContent(newVal)
}

const colorMode = useColorMode()
watch(() => colorMode.value, (val) => {
  if (val === 'dark')
    JsEditorRef.value?.setTheme('vs-dark')
  else
    JsEditorRef.value?.setTheme('vs')
})
</script>

<template>
  <div class="h-full flex justify-center">
    <div class="h-full w-full flex justify-between">
      <div class="h-full w-3/7">
        <h2 class="my-4 text-center font-semibold font-smiley">
          JavaScript
        </h2>
        <ClientOnly>
          <MonacoEditor ref="JsEditorRef" language="javascript" @on-change-text="handleJsChange" @on-focus-text="handleJsFocus" />
        </ClientOnly>
      </div>
      <div class="h-full w-3/7">
        <h2 class="my-4 text-center font-semibold font-smiley">
          Yaml
        </h2>
        <ClientOnly>
          <MonacoEditor ref="YamlEditorRef" language="yaml" @on-change-text="handleYamlChange" @on-focus-text="handleYamlFocus" />
        </ClientOnly>
      </div>
    </div>
  </div>
</template>
