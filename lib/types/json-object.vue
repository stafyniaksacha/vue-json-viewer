<script>
import JsonBox from '../json-box'

function naturalSort(a, b) {
  if (!a || !b) return 0
  if (typeof a !== "string" || typeof a !== "string" ) return 0
  b.localeCompare(a, 'fr', {ignorePunctuation: true})
}

export default {
  name: 'JsonObject',
  props: {
    jsonValue: {
      type: Object,
      required: true
    },
    keyName: {
      type: String,
      default: ''
    },
    depth: {
      type: Number,
      default: 0
    },
    expand: Boolean,
    sort: Boolean,
    primitiveFirst: Boolean,
  },
  data () {
    return {
      ordered: [],
    }
  },
  mounted () {
    // if (this.primitiveFirst) {
    let primitiveKeys = []
    let complexKeys = []
    let extraKeys = []

    for (const key in this.jsonValue) {
      if (this.jsonValue.hasOwnProperty(key)) {
        const value = this.jsonValue[key]
        if (
          this.primitiveFirst 
          && (
            value === null
            || [
              'string', 
              'number', 
              'boolean', 
              'bigint', 
              'undefined'
            ].includes(typeof value)
          )
        ) {
          primitiveKeys.push(key)
        } else if (
          this.primitiveFirst 
          && (
            ['object'].includes(typeof value)
            || Array.isArray(value)
          )
        ) {
          complexKeys.push(key)
        } else {
          extraKeys.push(key)
        }
      }
    }

    if (this.sort) {
      primitiveKeys.sort()
      complexKeys.sort()
      extraKeys.sort()
    }

    const keys = [].concat(primitiveKeys, complexKeys, extraKeys)

    this.ordered = {}
    for (const key of keys) {
      if (this.jsonValue.hasOwnProperty(key)) {
        this.ordered[key] = this.jsonValue[key]
      }
    }
  },
  methods: {
    toggle() {
      this.$emit('update:expand', !this.expand)

      try {
        this.$el.dispatchEvent(new Event('resized'))
      } catch (e) {
        // handle IE not supporting Event constructor
        var evt = document.createEvent('Event')
        evt.initEvent('resized', true, false)
        this.$el.dispatchEvent(evt)
      }
    }
  },
  render (h) {
    let elements = []

    if (!this.keyName) {
      elements.push(h('span', {
        class: {
          'jv-toggle': true,
          'open': !!this.expand,
        },
        on: {
          click: this.toggle
        }
      }))
    }

    elements.push(h('span', {
      class: {
        'jv-item': true,
        'jv-object': true,
      },
      domProps: {
        innerHTML: '{'
      }
    }))

    for (const key in this.ordered) {
      if (this.ordered.hasOwnProperty(key)) {
        const value = this.ordered[key]

        elements.push(h(JsonBox, {
          key,
          style: {
            display: !this.expand ? 'none' : undefined
          },
          props: {
            sort: this.sort,
            primitiveFirst: this.primitiveFirst,
            keyName: key,
            depth: this.depth + 1,
            value,
          }
        }))
      }
    }

    if (!this.expand && Object.keys(this.jsonValue).length) {
      elements.push(h('span', {
        style: {
          display: this.expand ? 'none' : undefined
        },
        class: {
          'jv-ellipsis': true,
        },
        on: {
          click: this.toggle
        },
        attrs: {
          title: `click to reveal object content (keys: ${Object.keys(this.ordered).join(', ')})`
        },
        domProps: {
          innerHTML: '...'
        }
      }))
    }

    elements.push(h('span', {
      class: {
        'jv-item': true,
        'jv-object': true,
      },
      domProps: {
        innerHTML: '}'
      }
    }))

    return h('span', elements)
  }
}
</script>
