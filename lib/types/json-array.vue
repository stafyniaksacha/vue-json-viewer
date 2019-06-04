<script>
import JsonBox from '../json-box'

function naturalSort(a, b) {
  if (!a || !b) return 0
  if (typeof a !== "string" || typeof a !== "string" ) return 0
  b.localeCompare(a, 'fr', {ignorePunctuation: true})
}

export default {
  name: 'JsonArray',
  props: {
    jsonValue: {
      type: Array,
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
    sort: Boolean,
    expand: Boolean
  },
  data () {
    return {
      ordered: [],
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
  mounted () {
    this.ordered = this.jsonValue

    if (this.sort) {
      this.ordered.sort(naturalSort)
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
        'jv-array': true,
      },
      domProps: {
        innerHTML: '['
      }
    }))
    
    for (const key in this.ordered) {
      if (this.ordered[key] !== undefined) {
        const value = this.ordered[key]
        
        elements.push(h(JsonBox, {
          key,
          style: {
            display: this.expand ? undefined : 'none'
          },
          props: {
            sort: this.sort,
            // keyName: key,
            depth: this.depth + 1,
            value,
          }
        }))
      }
    }

    if (!this.expand && this.jsonValue.length) {
      elements.push(h('span', {
        style: {
          display: undefined
        },
        class: {
          'jv-ellipsis': true,
        },
        on: {
          click: this.toggle
        },
        attrs: {
          title: `click to reveal ${this.jsonValue.length} hidden items`
        },
        domProps: {
          innerHTML: '...'
        }
      }))
    }

    elements.push(h('span', {
      class: {
        'jv-item': true,
        'jv-array': true,
      },
      domProps: {
        innerHTML: ']'
      }
    }))

    return h('span', elements)
  }
}
</script>
