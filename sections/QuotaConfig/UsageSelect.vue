<template>
  <a-select
    :value="value"
    @change="handleChange"
    :filterOption="filterOption"
    showSearch
    optionLabelProp="value">
    <a-select-option v-for="item of usages" :value="item.key" :key="item.key">
      <div style="font-size: 14px;">{{ item.key }}</div>
      <div class="text-color-help">{{ item.label }}</div>
    </a-select-option>
  </a-select>
</template>

<script>
import _ from 'lodash'

export default {
  name: 'DashboardUsageSelect',
  props: {
    usages: {
      type: Array,
      required: true,
    },
    value: {
      type: String,
      required: true,
    },
  },
  methods: {
    filterOption (input, option) {
      const desc = _.get(option, 'componentOptions.children[1].children[0].text', '')
      return option.key.toLowerCase().indexOf(input.toLowerCase()) >= 0 || desc.includes(input)
    },
    handleChange (val) {
      this.$emit('change', val)
      this.$emit('input', val)
    },
  },
}
</script>
