<template>
  <div class="btn-group" name="HeaderSettings">
    <button class="btn btn-default dropdown-toggle" ref="dropdownBtn" type="button">
      <i class="fa" :class="[usingBak && 'text-info', processingCls || 'fa-cog']"></i>
      <span class="caret"></span>
    </button>
    <div class="dropdown-menu clearfix" :style="drpMenuStyle">
      <div class="-col-group-container">
        <column-group v-for="(columns, groupName) in colGroups"
          ref="colGroups" :key="groupName"
          :group-name="groupName" :columns="columns">
        </column-group>
      </div>
      <div class="clearfix" style="margin: 10px 0">
        <div class="btn-group btn-group-sm pull-right">
          <button class="btn btn-default" type="button" @click="apply()">
            {{ $i18nForDatatable('Apply') }}
          </button>
          <template v-if="supportBackup">
            <div class="dropdown">
                <button class="btn btn-default dropdown-toggle" data-toggle="dropdown" type="button" style="box-shadow: none">
                  <span class="caret"></span>
                </button>
                <ul class="dropdown-menu dropdown-menu-right">
                  <li class="dropdown-item" @click="apply(true)">
                    <a href="#" @click.prevent>
                      <i class="fas fa-save"></i>&nbsp;
                      {{ $i18nForDatatable('Apply & Save') }}
                    </a>
                  </li>
                  <li  class="dropdown-item" v-if="usingBak" @click="rmBackup()">
                    <a href="#" @click.prevent>
                      <i class="fa fa-trash"></i>&nbsp;
                      {{ $i18nForDatatable('Clear settings') }}
                    </a>
                  </li>
                </ul>
            </div>
          </template>
        </div>
      </div>
      <small v-if="usingBak" class="pull-left text-muted" style="margin-top: -8px">
        ( {{ $i18nForDatatable('Using local settings') }} )
      </small>
    </div>
  </div>
</template>
<script>
import ColumnGroup from './ColumnGroup.vue'
import groupBy from 'lodash/groupBy'
import keyGen from '../_utils/keyGen'
import replaceWith from '../_utils/replaceWith'
import { parseStr, stringify, saveToLS, rmFromLS, getFromLS } from '../_utils/localstorage'

export default {
  name: 'HeaderSettings',
  components: { ColumnGroup },
  props: {
    columns: { type: Array, required: true },
    supportBackup: { type: Boolean, required: true }
  },
  data () {
    const origSettings = stringify(this.columns)
    return {
      origSettings,
      usingBak: false, // is using backup
      processingCls: '',
      storageKey: 'remirox'//this.supportBackup && keyGen(origSettings)
    }
  },
  created () {
    if (!this.supportBackup) return
    console.log('*** Load backup');
    const backup = getFromLS(this.storageKey)
      console.log(backup);
    if (!backup) {
        console.log("** no backup found")
        return
    } // no backup found

    replaceWith(this.columns, backup)
    this.usingBak = true
  },
  mounted () {
    // control dropdown manually (refers to http://jsfiddle.net/rj3k550m/3)
    const $el = $(this.$el)
    $(this.$refs.dropdownBtn).on('click', this.toggle)
    $(document).on('click', e => {
      $(e.target).closest($el).length || $el.removeClass('open')
    })
  },
  computed: {
    colGroups () {
      return groupBy(
        this.columns.filter(col => col.label || col.title),
        'group'
      )
    },
    drpMenuStyle () {
      const w = Object.keys(this.colGroups).length * 150
      return {
        padding: '10px 10px 0',
        width: `${w + 25}px`,
        left: `-${w - 25}px`
      }
    }
  },
  methods: {
    apply (alsoBackup) {
      // TODO SAVE USER PREF
      this.toggle()
      this.$refs.colGroups.forEach(colGroup => { colGroup.apply() })
      alsoBackup && this.$nextTick(this.backup)
    },
    backup () {
        console.log('***backup');
        console.log(this.columns);

      saveToLS(this.storageKey, this.columns)
      this.showProcessing()
      this.usingBak = true
    },
    rmBackup () {
      rmFromLS(this.storageKey)
      this.showProcessing()
      this.usingBak = false
      
      replaceWith(this.columns, parseStr(this.origSettings)) // restore
    },
    toggle () {
      $(this.$el).toggleClass('open')
    },
    showProcessing () {
      ['fa-spinner fa-pulse', 'fa-check', ''].forEach((cls, idx) => {
        setTimeout(() => {
          this.processingCls = cls
        }, idx * 1000)
      })
    }
  }
}
</script>
<style>
.-col-group-container {
  border-bottom: 1px solid #ddd;
}
</style>
