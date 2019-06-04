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
    sort: Boolean
  },
  data () {
    return {
      ordered: [],
    }
  },
  mounted () {
    let keys = Object.keys(this.jsonValue)

    if (this.sort) {
      keys.sort()
    }

    console.log(keys)

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
