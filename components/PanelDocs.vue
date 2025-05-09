<script setup lang="ts">
import type { ContentNavigationItem } from '@nuxt/content'

const runtime = useRuntimeConfig()
const route = useRoute()
const { data: page } = useAsyncData(route.path, () => {
  return queryCollection('tutorials').path(route.path).first()
})
const { data: navigation } = useAsyncData(`navigation`, () => {
  return queryCollectionNavigation('tutorials')
})
const { data: surroundings } = useAsyncData(`${route.path}-surroundings`, () => {
  return queryCollectionItemSurroundings('tutorials', route.path, {
    fields: ['title', 'description'],
  })
})

const prev = computed(() => surroundings.value?.[0])
const next = computed(() => surroundings.value?.[1])
const currentPage = ref<"content" | "agent">("content")
const setPage = (ind:"content"|"agent")=>currentPage.value = ind
interface BreadcrumbItem {
  title: string
  path?: string
}

function findNavItemFromPath(
  path: string,
  items: ContentNavigationItem[] | null = navigation.value,
): ContentNavigationItem | undefined {
  const item = items?.find(i => i.path === path)
  if (item)
    return item

  const parts = path.split('/').filter(Boolean)
  for (let i = parts.length - 1; i > 0; i--) {
    const parentPath = `/${parts.slice(0, i).join('/')}`
    const parent = items?.find(i => i.path === parentPath)
    if (parent)
      return findNavItemFromPath(path, parent.children || [])
  }
}

const contentPath = computed(() => page.value?.path as string | undefined)
const breadcrumbs = computed(() => {
  const parts = contentPath.value?.split('/').filter(Boolean) || []
  const breadcrumbs = parts
    .map((part, idx): BreadcrumbItem => {
      const path = `/${parts.slice(0, idx + 1).join('/')}`
      const item = findNavItemFromPath(path)
      return {
        title: item?.title || 'Not found',
        path: item ? path : undefined,
      }
    })

  if (!breadcrumbs.find(i => i.path === '/')) {
    breadcrumbs.unshift({
      title: 'Guide',
      path: '/',
    })
  }
  return breadcrumbs
})

const ui = useUiState()

const sourceUrl = computed(() =>
  page.value?.id
    ? `${runtime.public.repoUrl}/edit/main/content/${page.value.id}`
    : undefined,
)

const docsEl = ref<HTMLElement | null>(null)
const router = useRouter()
router.beforeEach(() => {
  docsEl.value?.scrollTo({
    top: 0,
  })
})
</script>

<template>
  <div grid="~ rows-[min-content_1fr]" relative h-full>
    <div flex="~ gap-2 flex flex-row items-center" border="b base dashed" bg-faded px4 py2>
      <div i-ph-book-duotone @click="setPage('content')"  cursor-pointer  />
      <template v-for="bc, idx of breadcrumbs" :key="bc.path">
        <div v-if="idx !== 0" i-ph-caret-right mx--1 text-sm op50 />
        <NuxtLink :to="bc.path" text-sm hover="text-primary">
          {{ bc.title }}
        </NuxtLink>
         <div i-ph-chat @click="setPage('agent')" cursor-pointer />
        <div text-sm hover="text-primary cursor-pointer " @click="setPage('agent')">Agent</div>
      </template>
      <button
        h-full flex-auto
        @click="ui.isContentDropdownShown = !ui.isContentDropdownShown"
      />
      <button
        i-ph-caret-down-duotone text-sm op50 transition duration-400
        :class="ui.isContentDropdownShown ? 'rotate-180' : ''"
        @click="ui.isContentDropdownShown = !ui.isContentDropdownShown"
      />
    </div>
    <div relative h-full of-hidden>
      <article ref="docsEl" class="max-w-none prose" h-full of-auto p6>
         <ChatBot v-if="currentPage==='agent'"/>
        <ContentRenderer v-if="page && currentPage==='content'" :key="page.id" :value="page" />
        <div mt8 py2 grid="~ cols-[1fr_1fr] gap-4"  v-if="sourceUrl && currentPage==='content'">
          <div>
            <ContentNavCard
              v-if="prev"
              :to="prev.path"
              :title="prev.title"
              :description="prev.description as string"
              subheader="Previous section"
              icon="i-ph-arrow-left"
            />
          </div>
          <div>
            <ContentNavCard
              v-if="next"
              :to="next.path"
              :title="next.title"
              :description="next.description as string"
              subheader="Next section"
              icon="i-ph-arrow-right"
              items-end text-right
            />
          </div>
        </div>
       
        <div border="t base dashed" mt-8 p3  v-if="sourceUrl && currentPage==='content'">
          <NuxtLink
           
            :to="sourceUrl" target="_blank"
            flex="~ items-center gap-2"
            text-inherit op75
            hover="text-primary op100"
          >
            <div i-ph-note-pencil-duotone />
            Edit this page
          </NuxtLink>
        </div>
      </article>
      <!-- Navigration Dropdown -->
      <Transition name="nav-dropdown">
        <div
          v-if="ui.isContentDropdownShown"
          flex="~ col"
          border="b base"

          absolute left-0 right-0 top-0 max-h-60vh overflow-y-auto py2 backdrop-blur-10 bg-base important-bg-opacity-80
        >
          <ContentNavItem v-for="item of navigation" :key="item.path" :item="item"  />
        </div>
      </Transition>
    </div>
  </div>
</template>

<style>
.nav-dropdown-enter-active {
  transition: all 0.1s ease-out;
}

.nav-dropdown-leave-active {
  transition: all 0.1s cubic-bezier(1, 0.5, 0.8, 1);
}

.nav-dropdown-enter-from,
.nav-dropdown-leave-to {
  transform: translateY(-100%);
  opacity: 0;
}
</style>
