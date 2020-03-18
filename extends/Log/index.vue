<template>
  <div class="h-100 position-relative">
    <div class="dashboard-card-wrap">
      <div class="dashboard-card-header">
        <div class="dashboard-card-header-left">{{ form.fd.name || '磁贴名称' }}<a-icon class="ml-2" type="loading" v-if="loading" /></div>
        <div class="dashboard-card-header-right">
          <slot name="actions" :handle-edit="handleEdit" />
        </div>
      </div>
      <div class="dashboard-card-body flex-column justify-content-center">
        <template v-if="data && data.length">
          <div class="flex-fill position-relative">
            <div class="dashboard-fco-wrap">
              <template v-for="item of data">
                <div :key="item.id" class="pt-2 pb-2 d-flex align-items-center" style="border-bottom: 1px solid #eee;">
                  <div class="flex-fill mini-text">
                    <div>{{ $te(`dictionary.${item.obj_type}`) ? $t(`dictionary.${item.obj_type}`) : item.obj_type }} - {{ item.action }}</div>
                    <div class="text-color-help mt-1">{{ item.user }} · {{ $moment(item.ops_time).fromNow() }}</div>
                  </div>
                  <a-tag :color="item.success ? '#52c41a' : '#f5222d'"> {{ item.success ? '成功' : '失败' }} </a-tag>
                </div>
              </template>
            </div>
          </div>
        </template>
      </div>
    </div>
    <base-drawer :visible.sync="visible" title="配置磁贴" @ok="handleSubmit">
      <a-form
        hideRequiredMark
        :form="form.fc"
        v-bind="formItemLayout">
        <a-form-item label="磁贴名称">
          <a-input v-decorator="decorators.name" />
        </a-form-item>
        <a-form-item label="行数">
          <a-input-number v-decorator="decorators.limit" />
        </a-form-item>
      </a-form>
    </base-drawer>
  </div>
</template>

<script>
import { mapGetters } from 'vuex'
import BaseDrawer from '@Dashboard/components/BaseDrawer'
import { getRequestT } from '@/utils/utils'

export const options = {
  label: '审计日志',
  desc: '显示最近的系统操作日志',
  thumb: require('./assets/thumb.svg'),
  h: 5,
  w: 4,
  sort: 5,
}

export default {
  name: 'Log',
  components: {
    BaseDrawer,
  },
  props: {
    options: {
      type: Object,
      required: true,
    },
    params: Object,
  },
  data () {
    const initialNameValue = (this.params && this.params.name) || '审计日志'
    const initialLimitValue = (this.params && this.params.limit) || 4
    return {
      data: [],
      visible: false,
      loading: false,
      form: {
        fc: this.$form.createForm(this),
        fd: {
          name: initialNameValue,
          limit: initialLimitValue,
        },
      },
      decorators: {
        name: [
          'name',
          {
            initialValue: initialNameValue,
            rules: [
              { required: true, message: '请输入磁贴名称' },
            ],
          },
        ],
        limit: [
          'limit',
          {
            initialValue: initialLimitValue,
            rules: [
              { required: true, message: '请选择行数' },
            ],
          },
        ],
      },
      formItemLayout: {
        wrapperCol: {
          span: 18,
        },
        labelCol: {
          span: 6,
        },
      },
    }
  },
  computed: {
    ...mapGetters(['scope']),
  },
  watch: {
    'form.fd' (val) {
      this.fetchLogs()
      for (let key in this.decorators) {
        let config = this.decorators[key][1] || {}
        config = {
          ...config,
          initialValue: val[key],
        }
        this.decorators[key][1] = config
      }
    },
  },
  destroyed () {
    this.manager = null
  },
  created () {
    this.manager = new this.$Manager('actions', 'v1')
    this.fetchLogs()
    this.$emit('update', this.options.i, {
      ...this.form.fd,
    })
  },
  methods: {
    async fetchLogs () {
      this.loading = true
      try {
        const response = await this.manager.list({
          params: {
            scope: this.scope,
            limit: this.form.fd.limit,
            $t: getRequestT(),
          },
        })
        this.data = response.data.data || []
      } finally {
        this.loading = false
      }
    },
    handleEdit () {
      this.visible = true
    },
    async handleSubmit () {
      try {
        const values = await this.form.fc.validateFields()
        this.form.fd = values
        this.$emit('update', this.options.i, values)
        this.visible = false
      } catch (error) {
        throw error
      }
    },
  },
}
</script>