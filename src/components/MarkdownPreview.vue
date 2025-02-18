<script lang="ts" setup>
import { defineComponent, ref, watchEffect } from "vue"
import remarkEmoji from "remark-emoji"
import { unified } from "unified"
import remarkParse from "remark-parse"
import remarkGfm from "remark-gfm"
import remarkSqueezeParagraphs from "remark-squeeze-paragraphs"
import remarkRehype from "remark-rehype"
import rehypeHighlight from "rehype-highlight"
import rehypeStringify from "rehype-stringify"
import rehypeSanitize from "rehype-sanitize"

const processor = unified()
  .use(remarkParse)
  .use(remarkEmoji)
  .use(remarkRehype)
  .use(remarkSqueezeParagraphs)
  .use(remarkGfm)
  .use(rehypeHighlight)
  .use(rehypeSanitize)
  .use(rehypeStringify)

const props = defineProps({
  md: {
    type: String,
    required: true,
  },
})

const parsedMarkdown = ref(props.md)

const throttleParse = useThrottleFn(
  (s) => {
    const a = processor.processSync(s)
    parsedMarkdown.value = String(a)
  },
  80,
  true,
  true
)

watchEffect(async () => {
  try {
    await throttleParse(props.md || "...")
  } catch (err) {
    parsedMarkdown.value = ""
  }
})
</script>
<template>
  <ClientOnly>
    <div class="markdown-body" v-html="parsedMarkdown"></div>
  </ClientOnly>
</template>
