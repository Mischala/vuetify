<template>
  <div>
    <v-group v-model="selected">
      default, no explicit value
      <v-item>hello</v-item>
      <v-item>world</v-item>
      <v-item>foo</v-item>
      <v-item>bar</v-item>
      {{ selected }}
    </v-group>

    <v-group multiple v-model="multiple">
      multiple, no explicit value
      <v-item>hello</v-item>
      <v-item>world</v-item>
      <v-item>foo</v-item>
      <v-item>bar</v-item>
      {{ multiple }}
    </v-group>

    <v-group mandatory v-model="mandatory">
      mandatory, no explicit value
      <v-item>hello</v-item>
      <v-item>world</v-item>
      <v-item>foo</v-item>
      <v-item>bar</v-item>
      {{ mandatory }}
    </v-group>

    <v-group mandatory multiple v-model="mandatoryMultiple">
      mandatory and multiple, no explicit value
      <v-item>hello</v-item>
      <v-item>world</v-item>
      <v-item>foo</v-item>
      <v-item>bar</v-item>
      {{ mandatoryMultiple }}
    </v-group>

    <v-group v-model="selectedExplicit">
      default, explicit value
      <v-item value="hello">hello</v-item>
      <v-item value="world">world</v-item>
      <v-item value="foo">foo</v-item>
      <v-item value="bar">bar</v-item>
      <v-item>no value</v-item>
      {{ selectedExplicit }}
    </v-group>

    <v-group v-model="multipleExplicit" multiple>
      multiple, explicit value
      <v-item value="hello">hello</v-item>
      <v-item value="world">world</v-item>
      <v-item value="foo">foo</v-item>
      <v-item value="bar">bar</v-item>
      {{ multipleExplicit }}
    </v-group>

    default, no explicit value
    <v-carousel>
      <v-carousel-item>one</v-carousel-item>
      <v-carousel-item>two</v-carousel-item>
      <v-carousel-item>three</v-carousel-item>
    </v-carousel>

    default, explicit value, no change handler
    <v-carousel :model-value="staticValue">
      <v-carousel-item value="one">one</v-carousel-item>
      <v-carousel-item value="two">two</v-carousel-item>
      <v-carousel-item value="three">three</v-carousel-item>
    </v-carousel>

    <v-group v-model="dynamic" multiple>
      dynamic, no explicit value
      <v-item v-for="item in items" :key="item">{{ item }}</v-item>
      {{ dynamic }}
      <button @click="insert">insert</button>
      <button @click="remove">remove</button>
    </v-group>

    <v-group multiple :max="2">
      multiple, max limit 2
      <v-item value="hello">hello</v-item>
      <v-item value="world">world</v-item>
      <v-item value="foo">foo</v-item>
      <v-item value="bar">bar</v-item>
    </v-group>

    <v-checkbox-group v-model="checkboxes">
      <v-checkbox value="one" />
      <v-checkbox value="two" />
      {{ checkboxes }}
    </v-checkbox-group>
  </div>
</template>

<script lang="ts">
  import { useGroup, useGroupItem } from '../src/composables/group'
  import { h, defineComponent, ref, watch, reactive, computed, toRefs } from 'vue'

  const VGroup = defineComponent({
    name: 'VGroup',
    props: {
      modelValue: {
        type: [Array, String, Number],
        default: () => ([]),
      },
      multiple: Boolean,
      mandatory: Boolean,
      max: {
        type: Number,
      },
    },
    setup (props, context) {
      useGroup(props, context, Symbol.for('foo'))

      return () => h('div', {
        class: 'v-group',
      }, [context.slots.default?.()])
    },
  })

  const VItem = defineComponent({
    name: 'VItem',
    props: {
      value: {
        required: false,
      },
    },
    setup (props, context) {
      const item = useGroupItem(props, Symbol.for('foo'))

      return () => h('div', {
        class: {
          selected: item.isSelected.value,
        },
        onClick: () => item.toggle(),
      }, [context.slots.default?.()])
    },
  })

  const VCarousel = defineComponent({
    name: 'VCarousel',
    props: {
      modelValue: {
        required: false,
      },
    },
    setup (props, context) {
      const group = useGroup(props, context, Symbol.for('bar'))

      return () => h('div', {
        class: 'v-carousel',
      }, [
        h('div', { onClick: group.prev }, ['prev']),
        context.slots.default?.(),
        h('div', { onClick: group.next }, ['next']),
      ])
    },
  })

  const VCarouselItem = defineComponent({
    name: 'VCarouselItem',
    props: {
      value: String,
    },
    setup (props, context) {
      const item = useGroupItem(props, Symbol.for('bar'))
      return () => h('div', {
        class: {
          selected: item.isSelected.value,
        },
      }, [context.slots.default?.()])
    },
  })

  const VCheckboxGroup = defineComponent({
    name: 'VCheckboxGroup',
    props: {
      modelValue: Array,
    },
    setup (props, context) {
      useGroup(reactive({ ...toRefs(props), multiple: true }), context, Symbol.for('checkbox-group'))

      return () => h('div', {
      }, [context.slots.default?.()])
    },
  })

  const VCheckbox = defineComponent({
    name: 'VCheckbox',
    props: {
      value: [String, Number],
    },
    setup (props, context) {
      const item = useGroupItem(props, Symbol.for('checkbox-group'))

      return () => h('input', {
        type: 'checkbox',
        checked: item.isSelected.value,
        onChange: () => item.toggle(),
      })
    },
  })

  export default defineComponent({
    components: {
      VGroup,
      VItem,
      VCarousel,
      VCarouselItem,
      VCheckboxGroup,
      VCheckbox,
    },
    setup () {
      const selected = ref(null)
      const multiple = ref([])
      const mandatory = ref(null)
      const mandatoryMultiple = ref([])
      const selectedExplicit = ref(null)
      const multipleExplicit = ref([])
      const staticValue = ref('two')
      const items = ref([1, 2, 3])
      const dynamic = ref(null)
      const checkboxes = ref(['one'])

      return {
        selected,
        multiple,
        mandatory,
        mandatoryMultiple,
        selectedExplicit,
        multipleExplicit,
        staticValue,
        items,
        dynamic,
        checkboxes,
        insert: () => items.value.splice(2, 0, 10),
        remove: () => items.value.splice(2, 1),
        update: (v: any) => {
          console.log('update', v)
          checkboxes.value = v
        },
      }
    },
  })
</script>

<style>
  .v-carousel {
    display: flex;
    justify-content: space-between;
  }
  .v-group {
    border: 1px solid grey;
    margin: 10px;
  }
  .selected {
    background: red;
  }
</style>
