<template lang='pug'>
  v-dialog(
    v-model='isShown'
    persistent
    width='1100'
    :fullscreen='$vuetify.breakpoint.smAndDown'
    )
    .dialog-header
      v-icon(color='white') mdi-tag-text-outline
      .subtitle-1.white--text.ml-3 {{$t('editor:props.pageProperties')}}
      v-spacer
      v-btn.mx-0(
        outlined
        dark
        @click.native='close'
        )
        v-icon(left) mdi-check
        span {{ $t('common:actions.ok') }}
    v-card.wiki-form(tile)
      v-card-text
        .overline.pb-5 {{$t('editor:props.pageInfo')}}
        v-text-field(
          ref='iptTitle'
          outlined
          :label='$t(`editor:props.title`)'
          counter='255'
          v-model='title'
          )
        v-text-field(
          outlined
          :label='$t(`editor:props.shortDescription`)'
          counter='255'
          v-model='description'
          persistent-hint
          :hint='$t(`editor:props.shortDescriptionHint`)'
          )
      v-divider
      v-card-text.grey(:class='darkMode ? `darken-3-d3` : `lighten-5`')
        .overline.pb-5 {{$t('editor:props.pathCategorization')}}
        v-container.pa-0(fluid, grid-list-lg)
          v-layout(row, wrap)
            v-flex(xs12, md2)
              v-select(
                outlined
                :label='$t(`editor:props.locale`)'
                suffix='/'
                :items='namespaces'
                v-model='locale'
                hide-details
                :disabled='mode !== "create"'
              )
            v-flex(xs12, md10)
              v-text-field(
                outlined
                :label='$t(`editor:props.path`)'
                append-icon='mdi-folder-search'
                v-model='path'
                :hint='$t(`editor:props.pathHint`)'
                persistent-hint
                @click:append='showPathSelector'
                :disabled='mode !== "create"'
                )
        v-combobox(
          chips
          deletable-chips
          :label='$t(`editor:props.tags`)'
          outlined
          multiple
          v-model='tags'
          single-line
          :hint='`COMING SOON - ` + $t(`editor:props.tagsHint`)'
          persistent-hint
          disabled
          )
      v-divider
      v-card-text.pb-5.grey(:class='darkMode ? `darken-3-d5` : `lighten-4`')
        .overline.pb-5 {{$t('editor:props.publishState')}} #[v-chip.ml-3(label, color='grey', small, outlined).white--text coming soon]
        v-container.pa-0(fluid, grid-list-lg)
          v-layout(row, wrap)
            v-flex(xs12, md4)
              v-switch(
                :label='$t(`editor:props.publishToggle`)'
                v-model='isPublished'
                color='primary'
                :hint='$t(`editor:props.publishToggleHint`)'
                persistent-hint
                disabled
                )
            v-flex(xs12, md4)
              v-dialog(
                ref='menuPublishStart'
                :close-on-content-click='false'
                v-model='isPublishStartShown'
                :return-value.sync='publishStartDate'
                full-width
                width='460px'
                :disabled='!isPublished || true'
                )
                template(v-slot:activator='{ on }')
                  v-text-field(
                    v-on='on'
                    :label='$t(`editor:props.publishStart`)'
                    v-model='publishStartDate'
                    prepend-icon='mdi-calendar-check'
                    readonly
                    outlined
                    clearable
                    :hint='$t(`editor:props.publishStartHint`)'
                    persistent-hint
                    :disabled='!isPublished || true'
                    )
                v-date-picker(
                  v-model='publishStartDate'
                  :min='(new Date()).toISOString().substring(0, 10)'
                  color='primary'
                  reactive
                  scrollable
                  landscape
                  )
                  v-spacer
                  v-btn(
                    flat=''
                    color='primary'
                    @click='isPublishStartShown = false'
                    ) {{$t('common:actions.cancel')}}
                  v-btn(
                    flat=''
                    color='primary'
                    @click='$refs.menuPublishStart.save(publishStartDate)'
                    ) {{$t('common:actions.ok')}}
            v-flex(xs12, md4)
              v-dialog(
                ref='menuPublishEnd'
                :close-on-content-click='false'
                v-model='isPublishEndShown'
                :return-value.sync='publishEndDate'
                full-width
                width='460px'
                :disabled='!isPublished || true'
                )
                template(v-slot:activator='{ on }')
                  v-text-field(
                    v-on='on'
                    :label='$t(`editor:props.publishEnd`)'
                    v-model='publishEndDate'
                    prepend-icon='mdi-calendar-remove'
                    readonly
                    outlined
                    clearable
                    :hint='$t(`editor:props.publishEndHint`)'
                    persistent-hint
                    :disabled='!isPublished || true'
                    )
                v-date-picker(
                  v-model='publishEndDate'
                  :min='(new Date()).toISOString().substring(0, 10)'
                  color='primary'
                  reactive
                  scrollable
                  landscape
                  )
                  v-spacer
                  v-btn(
                    flat=''
                    color='primary'
                    @click='isPublishEndShown = false'
                    ) {{$t('common:actions.cancel')}}
                  v-btn(
                    flat=''
                    color='primary'
                    @click='$refs.menuPublishEnd.save(publishEndDate)'
                    ) {{$t('common:actions.ok')}}

    page-selector(mode='create', v-model='pageSelectorShown', :path='path', :locale='locale', :open-handler='setPath')
    v-tour(name='editorPropertiesTour', :steps='tourSteps')
</template>

<script>
import _ from 'lodash'
import { sync, get } from 'vuex-pathify'

/* global siteLangs, siteConfig */

export default {
  props: {
    value: {
      type: Boolean,
      default: false
    }
  },
  data() {
    return {
      isPublishStartShown: false,
      isPublishEndShown: false,
      pageSelectorShown: false,
      namespaces: siteLangs.length ? siteLangs.map(ns => ns.code) : [siteConfig.lang],
      tourSteps: [
        {
          target: '.dialog-header',
          content: `First-time tour help here. <strong>TODO</strong>!`
        }
      ]
    }
  },
  computed: {
    isShown: {
      get() { return this.value },
      set(val) { this.$emit('input', val) }
    },
    darkMode: get('site/dark'),
    mode: get('editor/mode'),
    title: sync('page/title'),
    description: sync('page/description'),
    locale: sync('page/locale'),
    tags: sync('page/tags'),
    path: sync('page/path'),
    isPublished: sync('page/isPublished'),
    publishStartDate: sync('page/publishStartDate'),
    publishEndDate: sync('page/publishEndDate')
  },
  watch: {
    value(newValue, oldValue) {
      if (newValue) {
        _.delay(() => {
          this.$refs.iptTitle.focus()
          // this.$tours['editorPropertiesTour'].start()
        }, 500)
      }
    }
  },
  methods: {
    close() {
      this.isShown = false
    },
    showPathSelector() {
      this.pageSelectorShown = true
    },
    setPath({ path, locale }) {
      this.locale = locale
      this.path = path
    }
  }
}
</script>

<style lang='scss'>

</style>
