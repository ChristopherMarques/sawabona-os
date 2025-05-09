
<script setup lang="ts">
import type { Page } from '~/types';
import { readItems } from '@directus/sdk';
import useDirectus from '~/modules/directus/runtime/composables/useDirectus';

const { path } = useRoute();
const url = useRequestURL();
const { fileUrl } = useFiles();
const { globals } = useAppConfig();

const pageFilter = computed(() => {
  let finalPath = path;

  if (finalPath === '/' || finalPath === '') {
    return {
      _or: [
        { permalink: { _eq: '/' } },
        { permalink: { _eq: '' } },
        { permalink: { _null: true } }
      ]
    };
  }

  if (finalPath.endsWith('/')) {
    finalPath = finalPath.slice(0, -1);
  }

  return { permalink: { _eq: finalPath } };
});

const { data: page } = await useAsyncData<Page | null>(
  path,
  async () => {
    const result = await useDirectus(
      readItems('pages', {
        filter: unref(pageFilter),
        fields: [
          '*',
          {
            seo: ['*'],
            blocks: [
              'id',
              'collection',
              'hide_block',
              {
                item: {
                  block_hero: [
                    'id', 'title', 'headline', 'content', 'image', 'image_position',
                    { button_group: ['*', { buttons: ['*', { page: ['permalink'], post: ['slug'] }] }] }
                  ],
                  block_faqs: ['id', 'title', 'faqs', 'headline', 'alignment'],
                  block_richtext: ['id', 'title', 'headline', 'content', 'alignment'],
                  block_testimonials: ['id', 'title', 'headline', {
                    testimonials: [{ testimonials_id: ['id', 'title', 'subtitle', 'content', 'company', 'company_logo', { image: ['id', 'title', 'description'] }] }]
                  }],
                  block_quote: ['id', 'title', 'subtitle', 'content'],
                  block_cta: ['id', 'title', 'headline', 'content', {
                    button_group: ['*', { buttons: ['*', { page: ['permalink'], post: ['slug'] }] }]
                  }],
                  block_form: ['id', 'title', 'headline', { form: ['*'] }],
                  block_logocloud: ['id', 'title', 'headline', {
                    logos: ['id', { directus_files_id: ['id', 'title', 'description'] }]
                  }],
                  block_gallery: ['id', 'title', 'headline', {
                    gallery_items: [{ directus_files_id: ['id', 'title', 'description'] }]
                  }],
                  block_steps: ['id', 'title', 'headline', 'show_step_numbers', 'alternate_image_position', {
                    steps: ['id', 'title', 'content', 'image', {
                      button_group: ['*', { buttons: ['*', { page: ['permalink'], post: ['slug'] }] }]
                    }]
                  }],
                  block_columns: ['id', 'title', 'headline', {
                    rows: ['title', 'headline', 'content', 'image_position', { image: ['id', 'title', 'description'] }, {
                      button_group: ['*', { buttons: ['*', { page: ['permalink'], post: ['slug'] }] }]
                    }]
                  }],
                  block_divider: ['id', 'title'],
                  block_team: ['*'],
                  block_html: ['*'],
                  block_video: ['*'],
                  block_cardgroup: ['*']
                }
              }
            ]
          }
        ],
        limit: 1
      })
    );

    return result?.[0] ?? null;
  }
);

if (!unref(page)) {
  throw createError({ statusCode: 404, statusMessage: 'Page Not Found' });
}

const metadata = computed(() => {
  const pageData = unref(page);
  return {
    title: pageData?.seo?.title ?? pageData?.title,
    description: pageData?.seo?.meta_description ?? pageData?.summary,
    image: globals?.og_image ? fileUrl(globals.og_image) : undefined,
    canonical: pageData?.seo?.canonical_url ?? url
  };
});

defineOgImageComponent('OgImageTemplate', {
  title: unref(metadata)?.title,
  summary: unref(metadata)?.description,
  imageUrl: unref(metadata)?.image
});

useSchemaOrg([
  defineWebPage({
    name: unref(metadata)?.title,
    description: unref(metadata)?.description
  })
]);

useHead({
  title: () => unref(metadata)?.title,
  link: [{ rel: 'canonical', href: () => unref(metadata)?.canonical }]
});

useServerSeoMeta({
  title: () => unref(metadata)?.title,
  description: () => unref(metadata)?.description,
  ogTitle: () => unref(metadata)?.title,
  ogDescription: () => unref(metadata)?.description
});
</script>

<template>
  <NuxtErrorBoundary>
    <PageBuilder v-if="page" :page="page as Page" />
    <template #error="{ error }">
      <BlockContainer>
        <VAlert type="error">{{ error }}</VAlert>
      </BlockContainer>
    </template>
  </NuxtErrorBoundary>
</template>
