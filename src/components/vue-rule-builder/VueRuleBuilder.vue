<template lang="pug">
  .vue-rule-builder.rules-builder(:class="[isBasic ? 'vrb-basic' : 'vrb-advanced']")
    slot(v-bind="vrbProps")
      rule-builder-group(v-bind="vrbProps" :query.sync="query")
    slot(name="toolbar" v-bind="vrbProps")

</template>

<script>

import RuleBuilderGroup from './layouts/Default/DefaultGroup.vue'
import { deepMerge, deepClone } from './utilities.js'
import { operators, ruleTypes } from './defaults.js'

var defaultLabels = {
  matchType: 'Match Type',
  matchTypes: [
    { id: 'all', label: 'All' },
    { id: 'any', label: 'Any' }
  ],
  addRule: 'Add Rule',
  removeRule: '',
  addGroup: 'Add Group',
  removeGroup: 'Remove Group',
  textInputPlaceholder: 'value'
}

export default {
  name: 'VueRuleBuilder',

  components: {
    RuleBuilderGroup
  },

  props: {
    rules: {
      Array,
      required: true
    },
    labels: {
      type: Object,
      default() {
        return defaultLabels
      }
    },
    operators: {
      type: Object,
      default() {
        return operators
      }
    },
    maxDepth: {
      type: Number,
      default: 3,
      validator: function (value) {
        return value >= 1
      }
    },
    value: Object,
    isBasic: Boolean
  },

  data() {
    return {
      query: {
        logicalOperator: this.labels.matchTypes[0].id,
        children: []
      },
      ruleTypes
    }
  },

  computed: {
    mergedLabels() {
      return Object.assign({}, defaultLabels, this.labels)
    },
    mergedOperators() {
      return Object.assign({}, operators, this.operators)
    },

    mergedRules() {
      var mergedRules = []
      this.rules.forEach((rule) => {
        if (typeof this.ruleTypes[rule.type] !== 'undefined') {
          const merged = deepMerge(this.ruleTypes[rule.type], rule)
          mergedRules.push(merged)
        } else {
          mergedRules.push(rule)
        }
      })

      return mergedRules
    },
    vrbProps() {
      return {
        index: 0,
        depth: 1,
        maxDepth: this.maxDepth,
        ruleTypes: this.ruleTypes,
        rules: this.mergedRules,
        labels: this.mergedLabels,
        operators: this.mergedOperators,
        isBasic: this.isBasic
      }
    }
  },
  mounted() {
    this.validate()
    this.$watch(
      'query',
      (newQuery, oldQuery) => {
        if (JSON.stringify(newQuery) !== JSON.stringify(this.value)) {
          this.$emit('input', deepClone(newQuery))
        }
      }, { deep: true }
    )

    this.$watch(
      'value',
      (newValue, oldValue) => {
        if (JSON.stringify(newValue) !== JSON.stringify(this.query)) {
          this.query = deepClone(newValue)
        }
      }, { deep: true }
    )

    if (typeof this.$options.propsData.value !== 'undefined') {
      this.query = Object.assign(this.query, this.$options.propsData.value)
    }
  },
  methods: {
    validate() {
      this.mergedRules.forEach(r => {
        r.operators.forEach(o => {
          if (!this.mergedOperators[o]) {
            console.error(`Invalid operator '${o}' found in rule`, r)
          }
        })
      })
    }
  }
}
</script>
<style lang="sass">
@import url("~vue-multiselect/dist/vue-multiselect.min.css")
.vue-rule-builder
  .multiselect__tag
    // font-size: 12px
    margin: 0 3px 3px 0 !important
  .multiselect__placeholder
    font-size: 12px
  .multiselect__select
    height: 32px
    padding: 0
  .multiselect__tags
    //min-height: 34px
  .multiselect__tag
    // font-size: 12px
    margin: 0 3px 3px 0 !important
  .multiselect__placeholder
    font-size: 12px
  .multiselect, .multiselect__input, .multiselect__single
    font-size: .9rem
  .multiselect
    min-height: 34px
  .multiselect__option
    padding: 5px
    min-height: 26px
    line-height: .8rem
    font-size: .8rem

  .multiselect--active
    .multiselect__input
      width: 100px !important
</style>
