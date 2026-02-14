<script setup lang="ts">
import MarkdownIt from "markdown-it";

import { data as changelogs } from "../data/changelogs.data";

const md = new MarkdownIt();

function renderMarkdown(string: null | string | undefined) {
  const body = string ?? "No changelog provided.";
  const flavoredString = body
    .split(/---\r\n\r\n### Checksums|---\r\n\r\nMD5/)[0]
    .replace(/(?<=\(|(, ))@(.*?)(?=\)|(, ))/g, "[@$2](https://github.com/$2)")
    .replace(/#(\d+)/g, "[#$1](https://github.com/YakaTeam/issues/issues/$1)")
    .replace(/^Check out the .*past release notes.* if you're.*$/m, "")
    .replace(/https:\/\/github.com\/YakaTeam\/artifacts\/releases\/tag\/(.*?)/g, "#$1")
    .replace(/## [ \t]*([^\n\r]*)/g, "### $1")
    .trim();

  return md.render(flavoredString);
}

const dateFormatter = new Intl.DateTimeFormat("en", {
  dateStyle: "medium",
});
</script>

<template>
  <div v-for="(release, index) of changelogs" :key="release.id" class="release">
    <h2 :id="index === 0 ? 'latest' : release.tag_name" class="release__title">
      <a :href="release.html_url" target="_blank" class="no-underline">
        {{
          (() => {
            const raw = release.tag_name.replace("nightly-", "")
            const year = raw.slice(0, 4)
            const month = raw.slice(4, 6)
            const day = raw.slice(6, 8)

            const date = new Date(`${year}-${month}-${day}`)
            return date.toLocaleDateString("en-US", {
              month: "long",
              day: "2-digit",
              year: "numeric"
            })
          })()
        }}
      </a>
      <Badge v-if="index === 0" type="tip" text="Latest" />
      <Badge v-if="release.tag_name.includes('a')" type="danger" text="Unstable" />
      <Badge v-if="release.tag_name.includes('b')" type="warning" text="Unstable" />
      <Badge v-if="release.tag_name.includes('rc')" type="warning" text="Unstable" />
      <a class="header-anchor" :href="index === 0 ? '#latest' : `#${release.tag_name}`" :aria-label="`Permalink to &quot;${release.tag_name}&quot;`" />
    </h2>
    <div v-html="renderMarkdown(
      release.body
        .replace(/Automated Yukimi artifact, generated on \d{4}\/\d{2}\/\d{2}/g, '')
        .replace(/@([a-zA-Z0-9-]+)/g, '[@$1](https://github.com/$1)')
        .replace(/What's Changed/g, '')
    )"/>
  </div>
</template>

<style scoped>
.release {
  .release__title {
    margin-bottom: 0;
    display: flex;
    align-items: center;
    gap: 0.5rem;
  }

  .release__date {
    font-size: 0.875rem;
    color: var(--vp-c-text-2);
  }

  .no-underline {
    text-decoration: none;
  }
}
</style>
